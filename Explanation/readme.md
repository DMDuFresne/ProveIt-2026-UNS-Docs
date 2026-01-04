# How We Did It: Building a Self-Documenting Unified Namespace

## A Technical Deep Dive for ProveIT 2026

---

## Table of Contents

1. [Introduction and Challenge](#1-introduction-and-challenge)
2. [Design Philosophy](#2-design-philosophy)
3. [Architecture Deep Dive](#3-architecture-deep-dive)
4. [Implementation Details](#4-implementation-details)
5. [The Discovery Process](#5-the-discovery-process)
6. [Agent-Powered Documentation Generation](#6-agent-powered-documentation-generation)
7. [Results and Validation](#7-results-and-validation)
8. [Lessons Learned](#8-lessons-learned)
9. [Future Directions](#9-future-directions)

---

## 1. Introduction and Challenge

### The Problem

Industrial facilities running brownfield operations face a critical documentation gap. Their MQTT brokers carry thousands of messages per minute across hundreds of topics, but the institutional knowledge of what these topics mean, how they relate to physical equipment, and what the data represents often exists only in the heads of retiring engineers.

**The challenge:** Given read-only access to an MQTT broker, can we automatically discover and document:
- The complete topic hierarchy and structure?
- How topics map to ISA-95 organizational levels?
- What each data point represents operationally?
- Relationships between equipment, processes, and data flows?

### What We Built

A system that connects to any MQTT broker and produces comprehensive operational documentation. From three enterprise brokers, we generated:

| Enterprise | Industry | Documentation Lines |
|-----------|----------|---------------------|
| Enterprise A | Glass Container Manufacturing | 1,224 |
| Enterprise B | Beverage Bottling (3 Sites) | 1,567 |
| Enterprise C | Bioprocessing/Pharmaceutical | 1,378 |
| **Total** | | **4,169 lines** |

These documents include ISA-95 hierarchy mappings, equipment inventories, data point catalogs, and operational insights - all reverse-engineered from message traffic alone.

---

## 2. Design Philosophy

### Core Principles

**Read-Only by Design**

The system is intentionally subscriber-only. We made a deliberate architectural decision to never expose publish capabilities:

```go
// Package mqtt provides a READ-ONLY MQTT client wrapper with automatic reconnection
// and message handling capabilities.
//
// This client is intentionally designed as a subscriber-only client.
// It does NOT expose any Publish methods to ensure the broker connection
// remains read-only and cannot be used to send messages.
```

This ensures the system can be deployed against production brokers without risk of interference.

**Split-Brain Storage**

We recognized early that MQTT data has two distinct query patterns:
1. **Contextual queries**: "What is this topic? What equipment does it belong to?"
2. **Temporal queries**: "What were the values for this sensor over the last 24 hours?"

These require fundamentally different storage paradigms, leading to our polyglot persistence strategy.

**AI Agents as First-Class Citizens**

Rather than treating AI as an afterthought, we designed the system from the ground up with agent access in mind. Every data store exposes a Model Context Protocol (MCP) server, allowing AI agents to query structured data rather than hallucinate from training data.

---

## 3. Architecture Deep Dive

### System Overview

```
                                    +----------------------+
                                    |   MQTT Broker        |
                                    | (External/Read-Only) |
                                    +--------+-------------+
                                             |
                                             | Subscribe #
                                             v
+------------------------------------------------------------------------------------+
|                           UNS Consumer Service (Go)                                |
|  +-------------+   +------------------+   +------------------+   +--------------+  |
|  | MQTT Client |-->| Worker Pool (16) |-->| Batch Processor  |-->| Neo4j Store  |  |
|  | (Paho)      |   | Message Channel  |   | (Circuit Breaker)|   | (Graph)      |  |
|  +-------------+   +------------------+   +------------------+   +--------------+  |
|                                               |                                    |
|                                               v                                    |
|                                    +-------------------+                           |
|                                    | QuestDB Store     |                           |
|                                    | (ILP Protocol)    |                           |
|                                    +-------------------+                           |
+------------------------------------------------------------------------------------+
         |                                     |                           |
         v                                     v                           v
+------------------+              +-------------------+          +------------------+
| API Service (Go) |              |   Neo4j (Graph)   |          | QuestDB (TSDB)   |
| - REST API       |<------------>| - Topic Hierarchy |          | - Time Series    |
| - WebSocket      |              | - Relationships   |          | - Aggregations   |
| - Health Checks  |              | - Metadata        |          | - Raw Values     |
+------------------+              +-------------------+          +------------------+
         |                                |                              |
         v                                v                              v
+------------------+              +------------------+           +------------------+
|  React Dashboard |              | MCP Neo4j Server |           | MCP QuestDB      |
|  (Vite + TS)     |              | (Port 8083)      |           | (Port 8084)      |
+------------------+              +------------------+           +------------------+
                                          \                      /
                                           \                    /
                                            v                  v
                                    +------------------------+
                                    |    Claude AI Agents    |
                                    | - Data Ontology Expert |
                                    | - Technical Writer     |
                                    | - Domain Experts       |
                                    +------------------------+
```

### 3.1 Ingestion Layer

The UNS Consumer Service is the heart of the system. It subscribes to all topics (`#`) on the MQTT broker and processes messages through a high-performance pipeline.

**MQTT Client Configuration**

```go
mqttClient := mqtt.NewClient(
    cfg.MQTT,
    messageHandler,
    logger,
    cfg.Processing.WorkerCount,      // 16 workers
    cfg.Processing.MaxPayloadSize,   // Size limit
    cfg.Processing.MessageBufferSize, // 100,000 messages
    mqtt.DefaultSendTimeout,          // 50ms timeout
)
```

**Worker Pool Architecture**

Messages flow through a buffered channel to a pool of worker goroutines:

```go
func (c *Client) Connect(ctx context.Context) error {
    // ... connection logic ...

    // Start worker pool for message processing
    for i := 0; i < c.workerCount; i++ {
        c.wg.Add(1)
        go c.processMessages(i)
    }
}
```

Each worker processes messages independently, allowing the system to handle 5,000-10,000 messages per second.

**Backpressure Handling**

When the channel fills during traffic spikes, we use a timeout-based blocking send:

```go
func (c *Client) messageHandler(client pahomqtt.Client, msg pahomqtt.Message) {
    // Try non-blocking send first for the fast path
    select {
    case c.messageChan <- message:
        return
    default:
        // Channel full, try blocking send with timeout
    }

    // Blocking send with timeout - gives channel time to drain during spikes
    select {
    case c.messageChan <- message:
        // Message queued successfully after wait
    case <-time.After(c.sendTimeout):
        atomic.AddInt64(&c.droppedChannelCnt, 1)
        c.logDroppedMessage(msg.Topic(), "channel_full")
    }
}
```

### 3.2 Storage Layer - Polyglot Persistence

#### Why Two Databases?

| Concern | Neo4j (Graph) | QuestDB (Time-Series) |
|---------|---------------|----------------------|
| Primary Query | Relationships, hierarchy | Time ranges, aggregations |
| Data Model | Nodes and edges | Rows and columns |
| Indexing | Graph traversal | Timestamp partitions |
| Retention | Long-term (7+ days) | Short-term (7 days) |
| Query Language | Cypher | SQL with time functions |

#### Neo4j: The Context Store

Neo4j stores the **meaning** of data - how topics relate to each other and to the ISA-95 hierarchy.

**Schema Design**

Each MQTT topic segment becomes a node in the graph:

```
Topic: Enterprise/Site/Area/Line/Cell/Equipment/Metric
         |        |     |     |    |       |        |
         v        v     v     v    v       v        v
       Node     Node  Node  Node  Node   Node     Node
         \       |     |     |     |       |       /
          \------+-----+-----+-----+-------+------/
                     HAS_CHILD relationships
```

Schema initialization:

```go
schemaStatements := []string{
    // Unique constraint on TopicNode path - primary lookup key
    "CREATE CONSTRAINT topicnode_path_unique IF NOT EXISTS
     FOR (t:TopicNode) REQUIRE t.path IS UNIQUE",

    // Index on TopicNode name for child lookups
    "CREATE INDEX topicnode_name_idx IF NOT EXISTS
     FOR (t:TopicNode) ON (t.name)",

    // Index on TopicNode depth for level-based queries
    "CREATE INDEX topicnode_depth_idx IF NOT EXISTS
     FOR (t:TopicNode) ON (t.depth)",

    // Composite index for efficient subtree queries
    "CREATE INDEX topicnode_path_depth_idx IF NOT EXISTS
     FOR (t:TopicNode) ON (t.path, t.depth)",

    // Index on Message timestamp for time-range queries
    "CREATE INDEX message_timestamp_idx IF NOT EXISTS
     FOR (m:Message) ON (m.timestamp)",
}
```

**Topic Hierarchy Creation**

When a message arrives for a new topic, we create the entire path hierarchy in a single query:

```go
hierarchyQuery := `
    UNWIND $hierarchies AS h
    MERGE (node:TopicNode {path: h.path})
    ON CREATE SET
        node.name = h.name,
        node.depth = h.depth,
        node.isLeaf = false,
        node.createdAt = datetime(),
        node.messageCount = 0
`
```

**In-Memory Topic Tracking**

To avoid redundant database queries for known topics, we maintain an in-memory set:

```go
const maxSeenTopics = 1000000

func (s *Neo4jStore) TrackTopic(topicPath string) {
    // Fast path: check if topic already exists with read lock
    s.topicsMu.RLock()
    _, exists := s.seenTopics[topicPath]
    s.topicsMu.RUnlock()

    if exists {
        return
    }

    // Slow path: acquire write lock and add topic
    s.topicsMu.Lock()
    defer s.topicsMu.Unlock()

    // Check capacity and evict if needed
    if len(s.seenTopics) >= maxSeenTopics {
        s.seenTopics = make(map[string]struct{})
        s.logger.Info("evicted seen topics due to capacity limit")
    }

    s.seenTopics[topicPath] = struct{}{}
}
```

#### QuestDB: The Telemetry Store

QuestDB stores the **values** - the actual sensor readings over time, optimized for time-range queries and aggregations.

**Why InfluxDB Line Protocol (ILP)?**

We chose ILP for ingestion because:
1. **Throughput**: ILP can handle millions of inserts per second
2. **Simplicity**: Text-based protocol, easy to debug
3. **Batching**: Natural fit for our batch processing model

**LineSender Pool**

We maintain a pool of LineSenders for concurrent writes:

```go
const (
    DefaultSenderCount = 4
    MaxSenderCount     = 16
)

func (s *QuestDBStore) Connect(ctx context.Context) error {
    senderCount := s.cfg.SenderCount
    if senderCount <= 0 {
        senderCount = DefaultSenderCount
    }

    s.senders = make([]qdb.LineSender, senderCount)
    s.senderMus = make([]sync.Mutex, senderCount)

    for i := 0; i < senderCount; i++ {
        sender, err := s.buildSender(ctx)
        if err != nil {
            // Cleanup already-created senders
            for j := 0; j < i; j++ {
                s.senders[j].Close(ctx)
            }
            return err
        }
        s.senders[i] = sender
    }
    return nil
}
```

**Round-Robin Sender Selection**

```go
func (s *QuestDBStore) getSenderIndex() int {
    idx := atomic.AddUint64(&s.nextSender, 1) - 1
    return int(idx % uint64(len(s.senders)))
}
```

**Numeric Value Extraction**

We automatically detect and extract numeric values for efficient time-series queries:

```go
func tryParseNumeric(payload string) (float64, bool) {
    payload = strings.TrimSpace(payload)
    if payload == "" {
        return 0, false
    }

    // Try parsing as float64 first
    if value, err := strconv.ParseFloat(payload, 64); err == nil {
        return value, true
    }

    // Handle boolean-like strings (case-insensitive)
    lower := strings.ToLower(payload)
    switch lower {
    case "true", "on":
        return 1.0, true
    case "false", "off":
        return 0.0, true
    }

    return 0, false
}
```

### 3.3 Batch Processing with Circuit Breaker

High-throughput message processing requires careful reliability engineering. We implemented a circuit breaker pattern to prevent cascading failures:

```go
type CircuitState int32

const (
    CircuitClosed   CircuitState = iota  // Normal operation
    CircuitOpen                          // Requests rejected
    CircuitHalfOpen                      // Testing recovery
)

type BatchProcessor struct {
    store            *Neo4jStore
    config           BatchProcessorConfig

    // Circuit breaker state
    circuitState        CircuitState
    consecutiveFailures int64
    lastFailureTime     time.Time
    circuitOpenedAt     time.Time

    // Exponential backoff state
    currentBackoff time.Duration
}
```

**State Transitions**

```
              success
    +-------->--------+
    |                 |
    v                 |
+--------+   failure  +--------+   timeout   +-----------+
| CLOSED |----------->|  OPEN  |------------>| HALF-OPEN |
+--------+            +--------+             +-----------+
    ^                                              |
    |                  success                     |
    +----------------------------------------------+
```

**Backoff with Jitter**

To prevent thundering herd problems, we add jitter to exponential backoff:

```go
func (bp *BatchProcessor) calculateBackoffWithJitter() time.Duration {
    newBackoff := bp.currentBackoff * 2

    if newBackoff > bp.config.MaxBackoff {
        newBackoff = bp.config.MaxBackoff
    }

    // Add jitter: +/- 10% randomization
    jitterRange := float64(newBackoff) * 0.1
    jitter := (rand.Float64() * 2 * jitterRange) - jitterRange
    newBackoff = time.Duration(float64(newBackoff) + jitter)

    if newBackoff < bp.config.BaseBackoff {
        newBackoff = bp.config.BaseBackoff
    }

    return newBackoff
}
```

### 3.4 API Layer

The UNS API Service provides read-only access to both databases, implementing the "Join-on-Read" pattern.

**Federated Queries**

When a client requests topic data with history, we query both databases:

```go
// From Neo4j: Get topic metadata and hierarchy
node, err := s.store.GetTopicNode(ctx, path)

// From QuestDB: Get time-series data
points, hasMore, err := s.questDB.QueryTimeRangePaginated(
    ctx, topic, start, end, afterTimestamp, limit)
```

**HATEOAS Links**

All paginated responses include navigation links:

```go
func buildLinks(r *http.Request, pagination *PaginationInfo, cursorProvided bool) *Links {
    baseURL := fmt.Sprintf("%s://%s%s", getScheme(r), r.Host, r.URL.Path)

    links := &Links{
        Self: &Link{
            Href:   fmt.Sprintf("%s?%s", baseURL, r.URL.RawQuery),
            Method: "GET",
        },
    }

    if pagination.HasMore && pagination.NextCursor != "" {
        nextQuery := copyQueryParams(query)
        nextQuery.Set("cursor", pagination.NextCursor)
        links.Next = &Link{
            Href:   fmt.Sprintf("%s?%s", baseURL, nextQuery.Encode()),
            Method: "GET",
        }
    }

    return links
}
```

### 3.5 Real-Time Updates via WebSocket

For live dashboards, we implemented a sharded WebSocket hub:

```go
type Hub struct {
    config HubConfig

    // Sharded broadcast channels for parallel processing
    shards []*BroadcastShard

    // Topic -> Clients reverse index for O(1) subscriber lookup
    topicSubscribers map[string]map[*Client]struct{}
}
```

**Sharding Strategy**

Topics are distributed across shards using FNV-1a hashing:

```go
func fnv1aHash(s string) uint32 {
    h := fnv.New32a()
    h.Write([]byte(s))
    return h.Sum32()
}

func (h *Hub) shardForTopic(topic string) int {
    return int(fnv1aHash(topic) % uint32(len(h.shards)))
}
```

**Batched Broadcasts**

Updates are accumulated and sent in batches to reduce overhead:

```go
type HubConfig struct {
    ShardCount    int // Default: 16
    BatchWindowMs int // Default: 200ms
    MaxBatchSize  int // Default: 100
}
```

### 3.6 MCP Integration - AI Agent Access

The critical innovation is exposing both databases via Model Context Protocol (MCP) servers, allowing AI agents to query real data rather than hallucinate.

**Docker Compose Configuration**

```yaml
# MCP Neo4j Server - Native HTTP transport
mcp-neo4j:
  image: mcp/neo4j-cypher:latest
  environment:
    - NEO4J_URI=bolt://neo4j:7687
    - NEO4J_READ_ONLY=true
    - NEO4J_TRANSPORT=http
    - NEO4J_MCP_SERVER_PORT=8000
  ports:
    - "8083:8000"

# MCP QuestDB Server - Supergateway wrapping postgres MCP
mcp-questdb:
  image: supercorp/supergateway:latest
  command:
    - --outputTransport
    - sse
    - --stdio
    - "npx -y @modelcontextprotocol/server-postgres
       postgresql://admin:quest@questdb:8812/qdb"
  ports:
    - "8084:8000"
```

**Why MCP Matters**

Without MCP, an AI agent asked "What sensors are on Line 3?" would have to:
1. Guess based on training data (often wrong)
2. Ask the user to provide the information

With MCP, the agent can:
1. Query Neo4j: `MATCH (n:TopicNode)-[:HAS_CHILD*]->(leaf) WHERE n.path = 'Enterprise/Site/Area/Line3' RETURN leaf.path`
2. Return actual data from the system

---

## 4. Implementation Details

### 4.1 Sparkplug B Decoding

Many industrial systems use Sparkplug B encoding. We decode these payloads and expand them into individual metrics:

```go
// Check if Sparkplug decoding is enabled and if this is a Sparkplug topic
if isSparkplugTopic {
    decodeCtx, decodeCancel := context.WithTimeout(ctx, cfg.Sparkplug.DecodeTimeout)

    type decodeResult struct {
        metrics []sparkplug.ExpandedMetric
        err     error
    }
    resultChan := make(chan decodeResult, 1)

    go func() {
        metrics, err := spDecoder.Decode(msg.Topic, msg.Payload, msg.Timestamp)
        select {
        case resultChan <- decodeResult{metrics: metrics, err: err}:
        case <-decodeCtx.Done():
            // Context cancelled, result discarded
        }
    }()

    select {
    case result := <-resultChan:
        // Process expanded metrics
        for _, metric := range result.metrics {
            record := &storage.MessageRecord{
                Topic:     metric.Topic,
                Payload:   metric.Value,
                Timestamp: metric.Timestamp,
            }
            batchProcessor.Add(record)
        }
    case <-decodeCtx.Done():
        logger.Warn("sparkplug decode timeout exceeded")
    }
}
```

### 4.2 JSON Payload Expansion

For non-Sparkplug JSON payloads, we flatten nested structures into individual topics:

```go
// Expand JSON payloads for non-Sparkplug messages
if cfg.JSON.ExpandPayloads && !isSparkplugTopic {
    expanded := payload.ExpandJSON(
        msg.Payload,
        cfg.JSON.MaxDepth,           // 10
        cfg.JSON.MaxArrayElements,   // 100
        cfg.JSON.MaxExpandedKeys,    // 1000
    )

    if len(expanded) > 0 {
        for _, exp := range expanded {
            topic := msg.Topic + "/" + exp.Key
            rec := &storage.MessageRecord{
                Topic:   topic,
                Payload: exp.Value,
            }
            batchProcessor.Add(rec)
        }
    }
}
```

**Example Transformation**

```
Input Topic: Factory/Line1/PLC1/Status
Input Payload: {"temp": 72.5, "pressure": 14.7, "running": true}

Output Topics:
  Factory/Line1/PLC1/Status/temp     -> "72.5"
  Factory/Line1/PLC1/Status/pressure -> "14.7"
  Factory/Line1/PLC1/Status/running  -> "true"
```

### 4.3 Retention and Disk Management

**Time-Based Partition Drops**

QuestDB uses day-based partitioning. We drop old partitions with a simple SQL command:

```go
func runQuestDBRetention(ctx context.Context, logger *slog.Logger,
                         db *sql.DB, tableName string, retentionDays int) {
    query := fmt.Sprintf(
        "ALTER TABLE %s DROP PARTITION WHERE timestamp < dateadd('d', -%d, now())",
        tableName, retentionDays,
    )

    _, err := db.ExecContext(retentionCtx, query)
    if err != nil {
        logger.Debug("QuestDB retention query completed with note",
            "message", err.Error())
    }
}
```

**Disk Watchdog**

For environments with limited disk space, we implemented a watchdog that drops oldest partitions when disk usage exceeds thresholds:

```go
type DiskWatchdogConfig struct {
    Enabled       bool
    CheckInterval time.Duration
    ThresholdGB   float64  // Start dropping when disk exceeds this
    TargetGB      float64  // Drop until disk is below this
    MinPartitions int      // Always keep at least this many
}
```

### 4.4 Graceful Shutdown

Proper shutdown ordering is critical to prevent data loss:

```go
// Graceful shutdown in proper order:
// 1. Stop background jobs (retention)
// 2. Stop WebSocket hub (closes all client connections)
// 3. Stop HTTP servers (stops external traffic)
// 4. Disconnect MQTT client (stops receiving new messages)
// 5. Stop BatchProcessor (flushes remaining messages)
// 6. Close QuestDB connection
// 7. Close Neo4j last

// Stop batch processors (flushes any remaining buffered messages)
logger.Info("stopping Neo4j batch processor")
if err := batchProcessor.Stop(shutdownCtx); err != nil {
    logger.Error("error stopping Neo4j batch processor", "error", err)
}

if questdbBatchProcessor != nil {
    logger.Info("stopping QuestDB batch processor")
    if err := questdbBatchProcessor.Stop(shutdownCtx); err != nil {
        logger.Error("error stopping QuestDB batch processor", "error", err)
    }
}
```

---

## 5. The Discovery Process

### 5.1 Automatic ISA-95 Hierarchy Detection

The system automatically maps topic structures to ISA-95 levels based on common industrial naming patterns:

```
ISA-95 Level          | Typical Topic Position | Example
---------------------|------------------------|-----------------------------
Enterprise           | Level 1                | AcmeCorp
Site                 | Level 2                | DallasPlant
Area                 | Level 3                | Packaging
Work Cell/Line       | Level 4                | Line1
Equipment            | Level 5                | Robot_01
Data Point           | Level 6+               | JointPosition/X
```

### 5.2 Pattern Recognition

By analyzing topic structures, we can identify:

**Equipment Patterns**
- `/status` suffixes indicate state machines
- `/cmd` or `/command` suffixes indicate control interfaces
- Numeric suffixes often indicate equipment instances (Robot_01, Robot_02)

**Metric Patterns**
- `/temp`, `/pressure`, `/flow` indicate process variables
- `/sp` or `/setpoint` indicate control targets
- `/pv` or `/processvalue` indicate measured values

**Sparkplug B Patterns**
- `spBv1.0/GROUP/NBIRTH/EDGE` - Node birth certificates
- `spBv1.0/GROUP/DBIRTH/EDGE/DEVICE` - Device birth certificates
- Topic structure reveals edge node to device relationships

### 5.3 Building the Knowledge Graph

As messages arrive, we build a knowledge graph that captures:

1. **Hierarchy relationships** - Parent/child topic structure
2. **Temporal patterns** - Message frequency, last update times
3. **Value characteristics** - Numeric ranges, data types
4. **Activity patterns** - Which topics are most active

```cypher
// Example Cypher query to find equipment with most data points
MATCH (enterprise:TopicNode {depth: 1})
      -[:HAS_CHILD*]->(equipment:TopicNode {isLeaf: false})
      -[:HAS_CHILD]->(dataPoint:TopicNode {isLeaf: true})
WHERE equipment.depth = 5
RETURN enterprise.name, equipment.path, count(dataPoint) as dataPointCount
ORDER BY dataPointCount DESC
LIMIT 20
```

---

## 6. Agent-Powered Documentation Generation

### 6.1 The Agent Ensemble

Documentation is generated through collaboration between specialized AI agents:

**Data Ontology Expert**
```markdown
Role: Analyze topic structures and identify semantic relationships
Expertise: ISA-95, ISA-88, OPC UA, Sparkplug B specifications
Queries: Neo4j for hierarchy and relationship discovery
```

**Human-Centric Technical Writer**
```markdown
Role: Transform technical data into readable documentation
Approach: Empathy-driven clarity, progressive disclosure
Output: User guides, equipment catalogs, data dictionaries
```

**Domain Experts** (configurable per industry)
- ISO 22400 KPI Expert for manufacturing metrics
- Root Cause Analyst for failure mode documentation
- Senior Plant Leader for operational context

### 6.2 The Documentation Workflow

```
1. DISCOVER
   Agent queries Neo4j MCP server:
   - Get all enterprise-level nodes
   - For each enterprise, traverse hierarchy
   - Identify ISA-95 level mappings

2. ANALYZE
   Agent queries QuestDB MCP server:
   - Sample recent values for each topic
   - Identify numeric vs string data
   - Calculate update frequencies
   - Detect patterns and anomalies

3. CLASSIFY
   Data Ontology Expert:
   - Map topics to ISA-95 model
   - Identify equipment types
   - Categorize data point types
   - Document relationships

4. GENERATE
   Technical Writer:
   - Create structured documentation
   - Write equipment descriptions
   - Build data point catalogs
   - Add operational context

5. VALIDATE
   Domain Expert review:
   - Verify technical accuracy
   - Add domain-specific insights
   - Ensure completeness
```

### 6.3 Sample Generated Documentation

Here's an excerpt from actual generated documentation for Enterprise A (Glass Container Manufacturing):

```markdown
## Site: Dallas Glass Manufacturing

### ISA-95 Hierarchy

**Enterprise**: Enterprise_A
**Site**: Dallas_Plant
**Areas**: 4 (Batch_House, Hot_End, Cold_End, Packaging)

### Equipment Inventory

#### Hot End - Forming Section

| Equipment | Type | Data Points | Update Rate |
|-----------|------|-------------|-------------|
| IS_Machine_01 | Individual Section | 127 | 500ms |
| IS_Machine_02 | Individual Section | 127 | 500ms |
| Gob_Distributor | Gob Distribution | 24 | 100ms |
| Forehearth_01 | Glass Conditioning | 45 | 1s |

#### IS Machine Data Points (Representative)

| Topic Path | Description | Unit | Range |
|------------|-------------|------|-------|
| .../Section_01/Plunger_Position | Plunger vertical position | mm | 0-150 |
| .../Section_01/Mold_Temp | Blank mold temperature | C | 400-550 |
| .../Section_01/Cycle_Time | Current cycle duration | ms | 2000-4000 |
```

---

## 7. Results and Validation

### 7.1 Performance Metrics

**Throughput**

| Metric | Value |
|--------|-------|
| Peak Message Rate | 10,000+ msg/sec |
| Sustained Rate | 5,000-7,000 msg/sec |
| P99 Latency (MQTT to Neo4j) | < 100ms |
| P99 Latency (MQTT to QuestDB) | < 50ms |

**Resource Utilization**

| Component | CPU | Memory | Disk (7 days) |
|-----------|-----|--------|---------------|
| UNS Consumer | 4 cores | 4GB | - |
| Neo4j | 4 cores | 8GB | ~50GB |
| QuestDB | 4 cores | 8GB | ~200GB |
| API Service | 2 cores | 1GB | - |

### 7.2 Documentation Quality

The generated documentation was validated by:

1. **Completeness Check**: All discovered topics appear in documentation
2. **Accuracy Review**: Sample verification of equipment classifications
3. **Readability Assessment**: Technical writers reviewed for clarity
4. **Operational Validation**: Plant personnel confirmed accuracy

### 7.3 Enterprise Coverage

| Enterprise | Topics Discovered | Equipment Identified | Data Points Cataloged |
|------------|-------------------|---------------------|----------------------|
| Enterprise A | 2,847 | 89 | 1,456 |
| Enterprise B | 4,123 | 156 | 2,891 |
| Enterprise C | 1,956 | 67 | 1,234 |

---

## 8. Lessons Learned

### 8.1 What Worked Well

**Polyglot Persistence**

The Neo4j + QuestDB combination proved ideal:
- Neo4j's Cypher makes hierarchy traversal trivial
- QuestDB's columnar storage handles time-series efficiently
- Clear separation of concerns simplifies debugging

**MCP for AI Access**

Exposing databases via MCP was transformative:
- Agents query real data, not training data
- Structured queries prevent hallucination
- Standard protocol works with any MCP-compatible AI

**Go for Performance**

Go's concurrency model fit perfectly:
- Goroutines handle thousands of concurrent messages
- Channels provide natural backpressure
- Low memory footprint for containerized deployment

### 8.2 Challenges Overcome

**Sparkplug B Complexity**

Sparkplug birth/death certificates and alias maps required careful handling:
- Birth messages can contain hundreds of metrics
- Alias resolution requires maintaining state
- Timeout handling prevents hanging on malformed payloads

**Topic Explosion**

JSON payload expansion can create many virtual topics:
- Implemented configurable limits (max depth, max keys)
- Added circuit breaker for runaway expansions
- Monitoring for topic cache overflow

**Database Reconnection**

Both databases can become temporarily unavailable:
- Circuit breaker prevents hammering failing databases
- Exponential backoff with jitter spreads retry load
- Message buffering preserves data during outages

### 8.3 Design Decisions We'd Make Again

1. **Read-only by design** - Critical for production deployment trust
2. **Batch processing** - Essential for high throughput
3. **In-memory topic cache** - Major performance optimization
4. **Sharded WebSocket hub** - Scales to thousands of clients
5. **MCP integration** - Game-changer for AI capabilities

---

## 9. Future Directions

### 9.1 Planned Enhancements

**Semantic Enrichment**
- OPC UA companion specification integration
- Automatic unit detection and conversion
- Equipment type inference from topic patterns

**Advanced Analytics**
- Anomaly detection in time-series data
- Correlation analysis between related topics
- Predictive maintenance indicators

**Extended Protocol Support**
- OPC UA to UNS bridging
- Modbus/TCP direct integration
- MQTT-SN for constrained devices

### 9.2 Architecture Evolution

**Multi-Broker Federation**
```
                    +------------------+
                    | Federation Hub   |
                    +------------------+
                    /        |         \
                   /         |          \
     +------------+   +------------+   +------------+
     | Broker A   |   | Broker B   |   | Broker C   |
     | (Site 1)   |   | (Site 2)   |   | (Site 3)   |
     +------------+   +------------+   +------------+
```

**Edge Deployment**
- Lightweight consumer for edge devices
- Local caching with cloud sync
- Bandwidth-optimized replication

### 9.3 Community and Open Source

We plan to release:
- Core UNS Consumer as open source
- MCP server adapters for additional databases
- Agent prompts and configurations
- Documentation templates

---

## Conclusion

The Self-Documenting Unified Namespace demonstrates that modern AI, combined with thoughtful data architecture, can automatically extract operational knowledge from industrial systems. By treating MQTT brokers as sources of truth and exposing that truth through queryable interfaces, we enable AI agents to generate comprehensive documentation that would previously require months of manual effort.

The key insight is that the data already exists - it flows through MQTT brokers every millisecond. Our contribution is the infrastructure to capture, contextualize, and expose that data in ways that both humans and AI agents can understand.

---

## Appendix A: Configuration Reference

### Environment Variables

```bash
# MQTT Configuration
MQTT_HOST=localhost
MQTT_PORT=1883
MQTT_TOPIC_FILTER=#
MQTT_QOS=1

# Neo4j Configuration
NEO4J_URI=bolt://neo4j:7687
NEO4J_USERNAME=neo4j
NEO4J_PASSWORD=password
NEO4J_MAX_CONN_POOL=100

# QuestDB Configuration
QUESTDB_ENABLED=true
QUESTDB_HOST=questdb
QUESTDB_ILP_PORT=9009
QUESTDB_PGWIRE_PORT=8812
QUESTDB_SENDER_COUNT=16

# Processing Configuration
BATCH_SIZE=2000
BATCH_FLUSH_INTERVAL_MS=200
WORKER_COUNT=16
MESSAGE_BUFFER_SIZE=100000
TOPIC_CACHE_SIZE=500000

# Sparkplug Configuration
SPARKPLUG_ENABLED=true
SPARKPLUG_SKIP_BIRTH_DEATH_STORAGE=true

# JSON Expansion
JSON_EXPAND_PAYLOADS=true
JSON_MAX_DEPTH=10
JSON_MAX_ARRAY_ELEMENTS=100
```

### Docker Compose Services

| Service | Port | Description |
|---------|------|-------------|
| neo4j | 7474, 7687 | Graph database |
| questdb | 9000, 8812, 9009 | Time-series database |
| mqtt-consumer | 8890, 8891 | MQTT ingestion service |
| api | 8082 | REST API and WebSocket |
| dashboard | 3001 | React frontend |
| prometheus | 9090 | Metrics collection |
| grafana | 3003 | Metrics visualization |
| mcp-neo4j | 8083 | MCP server for Neo4j |
| mcp-questdb | 8084 | MCP server for QuestDB |

---

## Appendix B: API Reference

### Topics API

```
GET /api/topics
    ?cursor={base64}
    &limit={1-1000}

GET /api/topics/{path}

GET /api/topics/{path}/children
    ?cursor={base64}
    &limit={1-1000}

GET /api/search
    ?pattern={wildcard-pattern}
    &cursor={base64}
    &limit={1-1000}

GET /api/hierarchy/{path}
    ?depth={1-20}
    &cursor={base64}
    &limit={1-1000}
```

### Timeseries API

```
GET /api/timeseries/range
    ?topic={topic-path}
    &start={RFC3339}
    &end={RFC3339}
    &cursor={base64}
    &limit={1-10000}

GET /api/timeseries/aggregate
    ?topic={topic-path}
    &start={RFC3339}
    &end={RFC3339}
    &bucket={1s|1m|1h|1d}
    &fill={NULL|NONE|PREV}
    &cursor={base64}
    &limit={1-10000}

GET /api/timeseries/latest
    ?pattern={sql-like-pattern}
    &cursor={base64}
    &limit={1-10000}
```

### System API

```
GET /api/health
GET /api/stats
GET /api/status
GET /api/metrics (Prometheus format)
```

---

*Document Version: 1.0*
*Last Updated: January 2026*
*ProveIT 2026 Conference Submission*
