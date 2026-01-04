# Enterprise B - Unified Namespace Documentation

> **Document Version:** 2.1
> **Last Updated:** January 3, 2026
> **Scope:** Site 1 (The Cap Shack), Site 2 (Filler Central), Site 3

---

## What You'll Find in This Document

| Section | Best For | Jump To |
|---------|----------|---------|
| Executive Summary | Everyone - Start here | [Read](#executive-summary) |
| Semantic Foundations | ISA-95 alignment, hierarchy relationships | [Read](#semantic-foundations) |
| Equipment Taxonomy | Equipment types and capabilities | [Read](#equipment-type-taxonomy) |
| Data Type Specifications | Understanding value formats | [Read](#data-type-specifications) |
| Quick Start Guide | Getting your first data point | [Read](#quick-start-guide) |
| Common Use Cases | Real-world examples | [Read](#common-use-cases) |
| Hierarchy Structure | Understanding the data model | [Read](#hierarchy-structure) |
| Topic Reference | Building query paths | [Read](#topic-structure-reference) |
| Site Details | Asset-specific lookups | [Read](#site-1-detailed-documentation) |
| Quick Reference Cards | Cheat sheets for daily work | [Read](#quick-reference-cards) |
| Complete Asset Registry | All asset IDs and paths | [Read](#complete-asset-registry) |
| Glossary | Term definitions | [Read](#glossary) |

---

## Executive Summary

### What is the Unified Namespace?

The Unified Namespace (UNS) is a single, organized structure that contains all operational data from Enterprise B's manufacturing operations. Think of it as a filing system where every piece of equipment, every sensor reading, and every production metric has a specific, predictable address.

Instead of hunting through multiple databases, spreadsheets, and control systems to find data, the UNS gives you one consistent way to access everything.

**Before UNS:**
- "Where do I find Tank 3's temperature? Is it in the historian? The SCADA system? That Excel file someone made?"

**With UNS:**
- `Enterprise B/Site1/liquidprocessing/tankstorage01/tank03/processdata/process/temperature`

Every data point has a clear, logical path that follows your plant's physical layout.

### Who Should Use This Documentation?

| Role | What You'll Use This For |
|------|--------------------------|
| **Plant Operators** | Find real-time equipment status, check production counts, monitor line performance |
| **Process Engineers** | Analyze OEE trends, investigate quality issues, compare line performance |
| **Maintenance Technicians** | Check equipment states, review downtime metrics, access asset information |
| **Quality Engineers** | Trace lot numbers, investigate defects, track product through the process |
| **IT/Integration Teams** | Build dashboards, connect systems, set up automated alerts |
| **Data Analysts** | Query historical data, build reports, identify improvement opportunities |

### What Data is Available?

The UNS provides access to four categories of data across 120 assets:

| Data Category | What It Contains | Example Questions It Answers |
|---------------|------------------|------------------------------|
| **Asset Identity** | Equipment names, IDs, hierarchy | "What's the ID for the East Robot?" |
| **OEE Metrics** | Availability, Performance, Quality, OEE | "What's Line A's current OEE?" |
| **Process Data** | States, counts, rates, temperatures | "Is Tank 2 running? What's the temperature?" |
| **Work Orders** | Production orders, lot numbers, items | "What product is running on Labeler B?" |

### Enterprise at a Glance

```
Enterprise B
    |
    +-- Site 1: The Cap Shack (44 work centers)
    |       Largest site, 3 filling lines, 4 labeler lines
    |
    +-- Site 2: Filler Central (22 work centers)
    |       Medium site, 2 filling lines, 2 labeler lines
    |
    +-- Site 3: Plant3 (10 work centers)
            Smallest site, 1 filling line, 1 labeler line
```

**Total:** 3 Sites | 12 Areas | 29 Lines | 76 Work Centers | 120 Assets

---

## Semantic Foundations

### ISA-95 Hierarchy Alignment

This UNS follows the ISA-95 (IEC 62264) functional hierarchy model. Understanding this alignment helps when integrating with other manufacturing systems.

| UNS Level | ISA-95 Level | Term | Definition |
|-----------|--------------|------|------------|
| 0 | Level 4 | **Enterprise** | The top-level organizational entity representing the entire business unit |
| 1 | Level 3 | **Site** | A geographically distinct manufacturing facility with independent operations |
| 2 | Level 2 | **Area** | A functional subdivision of a site organized by manufacturing process type |
| 3 | Level 1 | **Line** | A logically grouped set of equipment that works together to perform a production function |
| 4 | Level 0 | **WorkCenter** | An individual piece of equipment, machine, or work station capable of performing operations |

### Hierarchy Relationships

The UNS uses a strict parent-child containment model:

```
Enterprise ──contains──> Site ──contains──> Area ──contains──> Line ──contains──> WorkCenter
```

| Relationship | How It's Expressed | Example |
|--------------|-------------------|---------|
| Parent Reference | `parentassetid` topic | WorkCenter 23 (Filler) has parentassetid = 7 (FillingLine01) |
| Full Path | `assetpath` topic | `Enterprise B/Site1/fillerproduction/fillingline01/filler` |
| Asset Type | `assettypename` topic | Returns "Site", "Area", "Line", or "WorkCenter" |

### Asset Identification

Each asset is uniquely identified by three complementary identifiers:

| Identifier | Purpose | Example |
|------------|---------|---------|
| **AssetID** | System-assigned immutable integer for database references | `23` |
| **AssetName** | Machine-readable identifier used in UNS paths (PascalCase stored, lowercase in paths) | `Filler` → `filler` |
| **DisplayName** | Human-readable label for UI display (may contain spaces, may be null) | `"Line A Filler"` |

> **Note:** When constructing UNS paths, always use the lowercase version of AssetName, not the DisplayName.

---

## Equipment Type Taxonomy

### Classification by Process Type

Enterprise B equipment falls into two main categories based on how they handle materials:

| Process Type | Characteristics | Equipment |
|--------------|-----------------|-----------|
| **Discrete Processing** | Handles countable units, has infeed/outfeed counts | Filler, Washer, CapLoader, Labeler, Packager, Sealer, Robot |
| **Continuous Processing** | Handles bulk materials, has process variables (temp, flow, weight) | Vat, Tank |

### Equipment by Functional Area

| Area | Equipment Types | Key Characteristics |
|------|-----------------|---------------------|
| **FillerProduction** | Washer, Filler, CapLoader | Discrete counts, work orders at Line level |
| **LiquidProcessing** | Vat, Tank | Process variables, work orders at WorkCenter level |
| **Packaging** | Labeler, Packager, Sealer | Discrete counts, work orders at Line level |
| **Palletizing** | Robot, Pallet, Wrapper, Workstation | Mixed - Robot has counts, others state-only |

### Equipment Capabilities Matrix

| Equipment | Has OEE | Has Counts | Has Process Vars | Has Work Order |
|-----------|---------|------------|------------------|----------------|
| Washer | ✓ | ✓ | ✗ | At Line |
| Filler | ✓ | ✓ | ✗ | At Line |
| CapLoader | ✓ | ✓ | ✗ | At Line |
| Vat | ✓ | ✗ | ✓ (temp, flow, weight) | ✓ |
| Tank | ✓ | ✗ | ✓ (temp, flow, weight) | ✓ |
| Labeler | ✓ | ✓ | ✗ | At Line |
| Packager | ✓ | ✓ | ✗ | At Line |
| Sealer | ✓ | ✓ | ✗ | At Line |
| Robot | ✓ | ✓ | ✗ | At Line |
| Pallet | ✓ | ✗ | ✗ | At Line |
| Wrapper | ✓ | ✗ | ✗ | At Line |
| Workstation | ✓ | ✗ | ✗ | At Line |

### Display Name Conventions

Some equipment has informal names used by plant personnel:

| Asset Path | Technical Name | Display Name | Common Usage |
|------------|----------------|--------------|--------------|
| `.../mixroom01/vat01` | Vat01 | Jeff | "Check Jeff's temperature" |
| `.../mixroom01/vat02` | Vat02 | Raymond | "Raymond is mixing batch 42" |
| `.../mixroom01/vat03` | Vat03 | Billy | |
| `.../mixroom01/vat04` | Vat04 | Bob | |
| `.../palletizer01` | Palletizer01 | East Robot | Location-based naming |
| `.../palletizer02` | Palletizer02 | West Robot | Location-based naming |

---

## Data Type Specifications

### Metric Value Types

| Data Type | Description | Range | Example |
|-----------|-------------|-------|---------|
| **OEE Factor** | Decimal representing percentage | 0.0 - 1.0 | `0.85` = 85% |
| **Count** | Integer count of units | 0 - MAX_INT | `15423` |
| **Rate** | Units per time period | ≥ 0.0 | `125.5` |
| **Duration** | Time in seconds | ≥ 0 | `3600` = 1 hour |

### Process Variable Types

| Data Type | Description | Typical Range | Unit |
|-----------|-------------|---------------|------|
| **Temperature** | Process temperature | Varies by equipment | Degrees |
| **FlowRate** | Liquid flow rate | ≥ 0.0 | Volume/time |
| **Weight** | Vessel weight/level | ≥ 0.0 | Weight units |

### State Information Types

| Field | Data Type | Description |
|-------|-----------|-------------|
| `code` | Integer | Numeric state identifier |
| `name` | String | Human-readable state name |
| `type` | String | State category (e.g., "Production", "Downtime") |
| `duration` | Integer | Seconds in current state |

### Identifier Types

| Field | Data Type | Constraints |
|-------|-----------|-------------|
| `assetid` | Integer | Unique, immutable, system-assigned |
| `assetname` | String | PascalCase, no spaces |
| `displayname` | String or null | May contain spaces, optional |
| `workorderid` | Integer | Unique order identifier |
| `workordernumber` | String | Human-readable order number |
| `lotnumber` | String | Batch/lot tracking number |
| `itemid` | Integer | Product identifier |

### Null Value Representation

| Context | How Null Appears | Example |
|---------|------------------|---------|
| Missing DisplayName | Empty or null | Site3/Plant3 has no display name |
| No active work order | Empty or null | Equipment between orders |
| In documentation | `*(none)*` | Indicates not configured |

---

## Quick Start Guide

### Your First Query in 3 Steps

**Goal:** Find the current OEE for Filling Line A at Site 1

**Step 1: Start with the Enterprise**
```
Enterprise B/
```

**Step 2: Navigate the hierarchy (Site > Area > Line)**
```
Enterprise B/Site1/fillerproduction/fillingline01/
```

**Step 3: Add the data you want**
```
Enterprise B/Site1/fillerproduction/fillingline01/metric/oee
```

That's it. This path gives you the real-time OEE value for Line A.

### Understanding the Path Structure

Every UNS path follows this pattern:

```
Enterprise B / {Site} / {Area} / {Line} / {WorkCenter} / {Category} / {DataPoint}
     |           |        |        |          |             |            |
     |           |        |        |          |             |            +-- The specific value
     |           |        |        |          |             +-- metric, processdata, workorder, node
     |           |        |        |          +-- Optional: specific equipment
     |           |        |        +-- Production line or functional group
     |           |        +-- Functional area (fillerproduction, packaging, etc.)
     |           +-- Site1, Site2, or Site3
     +-- Always starts here
```

> **Tip:** All path segments are lowercase with no spaces. Use the asset names from the registry tables, not display names.

---

## Common Use Cases

### Scenario 1: Operator Checking Equipment OEE

**Situation:** You're an operator on the floor and need to check how Line B is performing this shift.

**What you need:**
- Current OEE (Overall Equipment Effectiveness)
- The three factors that make up OEE: Availability, Performance, Quality

**Paths to query:**

| Metric | Path |
|--------|------|
| OEE | `Enterprise B/Site1/fillerproduction/fillingline02/metric/oee` |
| Availability | `Enterprise B/Site1/fillerproduction/fillingline02/metric/availability` |
| Performance | `Enterprise B/Site1/fillerproduction/fillingline02/metric/performance` |
| Quality | `Enterprise B/Site1/fillerproduction/fillingline02/metric/quality` |

**Reading the values:**
- OEE metrics return values between 0 and 1
- Multiply by 100 for percentage (0.85 = 85%)
- Target is typically 0.85 (85%) or higher

**If OEE is low, dig deeper into the inputs:**

| Input Metric | Path | What It Tells You |
|--------------|------|-------------------|
| Running Time | `.../metric/input/timerunning` | Seconds the line was producing |
| Idle Time | `.../metric/input/timeidle` | Seconds waiting with no faults |
| Unplanned Down | `.../metric/input/timedownunplanned` | Seconds lost to breakdowns |
| Defect Count | `.../metric/input/countdefect` | Units rejected |

---

### Scenario 2: Engineer Tracing a Quality Issue

**Situation:** A customer complaint came in about Lot #LT-2026-0142. You need to trace where this product was made and what conditions existed during production.

**Step 1: Find which equipment processed this lot**

Query work order topics across liquid processing (where lots originate):

```
Enterprise B/Site1/liquidprocessing/mixroom01/vat01/workorder/lotnumber/lotnumber
Enterprise B/Site1/liquidprocessing/mixroom01/vat02/workorder/lotnumber/lotnumber
Enterprise B/Site1/liquidprocessing/mixroom01/vat03/workorder/lotnumber/lotnumber
Enterprise B/Site1/liquidprocessing/mixroom01/vat04/workorder/lotnumber/lotnumber
```

> **Note:** Vats have friendly names: Vat01 = "Jeff", Vat02 = "Raymond", Vat03 = "Billy", Vat04 = "Bob"

**Step 2: Once you find the vat, check process conditions**

If the lot was mixed in Vat02 (Raymond):

| Data Point | Path |
|------------|------|
| Temperature during batch | `.../vat02/processdata/process/temperature` |
| Flow rate | `.../vat02/processdata/process/flowrate` |
| Batch weight | `.../vat02/processdata/process/weight` |
| Equipment state | `.../vat02/processdata/state/name` |

**Step 3: Get product details**

```
Enterprise B/Site1/liquidprocessing/mixroom01/vat02/workorder/lotnumber/item/itemname
Enterprise B/Site1/liquidprocessing/mixroom01/vat02/workorder/lotnumber/item/itemclass
Enterprise B/Site1/liquidprocessing/mixroom01/vat02/workorder/lotnumber/item/bottlesize
```

---

### Scenario 3: IT Building a Tank Temperature Dashboard

**Situation:** Operations wants a dashboard showing all tank temperatures across the enterprise. You need to identify all relevant data points.

**Step 1: Identify all tanks**

Tanks exist in the `liquidprocessing` area under `tankstorage01` lines:

| Site | Line | Tanks | Path Pattern |
|------|------|-------|--------------|
| Site 1 | TankStorage01 | Tank01-06 | `Enterprise B/Site1/liquidprocessing/tankstorage01/tank0X/` |
| Site 2 | TankStorage01 | Tank01-03 | `Enterprise B/Site2/liquidprocessing/tankstorage01/tank0X/` |
| Site 3 | TankStorage01 | Tank01-02 | `Enterprise B/Site3/liquidprocessing/tankstorage01/tank0X/` |

**Step 2: Build temperature topic list**

```
-- Site 1 Tanks (6 tanks)
Enterprise B/Site1/liquidprocessing/tankstorage01/tank01/processdata/process/temperature
Enterprise B/Site1/liquidprocessing/tankstorage01/tank02/processdata/process/temperature
Enterprise B/Site1/liquidprocessing/tankstorage01/tank03/processdata/process/temperature
Enterprise B/Site1/liquidprocessing/tankstorage01/tank04/processdata/process/temperature
Enterprise B/Site1/liquidprocessing/tankstorage01/tank05/processdata/process/temperature
Enterprise B/Site1/liquidprocessing/tankstorage01/tank06/processdata/process/temperature

-- Site 2 Tanks (3 tanks)
Enterprise B/Site2/liquidprocessing/tankstorage01/tank01/processdata/process/temperature
Enterprise B/Site2/liquidprocessing/tankstorage01/tank02/processdata/process/temperature
Enterprise B/Site2/liquidprocessing/tankstorage01/tank03/processdata/process/temperature

-- Site 3 Tanks (2 tanks)
Enterprise B/Site3/liquidprocessing/tankstorage01/tank01/processdata/process/temperature
Enterprise B/Site3/liquidprocessing/tankstorage01/tank02/processdata/process/temperature
```

**Step 3: Add display names for the dashboard**

| Topic Path | Display Name |
|------------|--------------|
| `.../Site1/.../tank01/...` | Site 1 - Tank 1 |
| `.../Site1/.../tank02/...` | Site 1 - Tank 2 |
| ... | ... |
| `.../Site3/.../tank02/...` | Site 3 - South Tank |

> **Pro Tip:** Query the `displayname` topic to get the friendly name configured in the system:
> `Enterprise B/Site3/liquidprocessing/tankstorage01/tank02/node/assetidentifier/displayname`

---

### Scenario 4: Finding All Equipment in a Specific State

**Situation:** You want to find all work centers that are currently idle across Site 1.

**Data point needed:** `processdata/state/name`

**Build your query list:**

For filling equipment:
```
Enterprise B/Site1/fillerproduction/fillingline01/washer/processdata/state/name
Enterprise B/Site1/fillerproduction/fillingline01/filler/processdata/state/name
Enterprise B/Site1/fillerproduction/fillingline01/caploader/processdata/state/name
...repeat for fillingline02, fillingline03...
```

For packaging equipment:
```
Enterprise B/Site1/packaging/labelerline01/labeler/processdata/state/name
Enterprise B/Site1/packaging/labelerline01/packager/processdata/state/name
Enterprise B/Site1/packaging/labelerline01/sealer/processdata/state/name
...repeat for labelerline02, 03, 04...
```

> **Tip:** The state `name` field returns human-readable state names. The `code` field returns numeric codes for programmatic filtering.

---

### Scenario 5: Comparing Line Performance Across Sites

**Situation:** You want to compare filling line performance across all three sites.

**Query these paths:**

| Site | Line | OEE Path |
|------|------|----------|
| Site 1 | Line A | `Enterprise B/Site1/fillerproduction/fillingline01/metric/oee` |
| Site 1 | Line B | `Enterprise B/Site1/fillerproduction/fillingline02/metric/oee` |
| Site 1 | High Cap | `Enterprise B/Site1/fillerproduction/fillingline03/metric/oee` |
| Site 2 | Line 1 | `Enterprise B/Site2/fillerproduction/fillingline01/metric/oee` |
| Site 2 | Line 2 | `Enterprise B/Site2/fillerproduction/fillingline02/metric/oee` |
| Site 3 | Filling | `Enterprise B/Site3/fillerproduction/fillingline01/metric/oee` |

For deeper analysis, also pull:
- `metric/performance` - Are lines running at target speed?
- `metric/input/rateactual` vs `metric/input/ratestandard` - What's the actual vs. expected rate?

---

## Quick Reference Cards

### Card 1: OEE Metrics Quick Reference

```
+------------------------------------------------------------------+
|  OEE METRICS - Path: {asset}/metric/                             |
+------------------------------------------------------------------+
|                                                                  |
|  CALCULATED METRICS (0.0 to 1.0, multiply by 100 for %)          |
|  -------------------------------------------------------         |
|  oee             Overall Equipment Effectiveness                 |
|  availability    Time equipment was available                    |
|  performance     Speed vs. target rate                           |
|  quality         Good units vs. total units                      |
|                                                                  |
|  INPUT METRICS - Path: {asset}/metric/input/                     |
|  -------------------------------------------------------         |
|  countinfeed     Total units into equipment                      |
|  countoutfeed    Total units out of equipment                    |
|  countdefect     Total rejected units                            |
|  rateactual      Current production rate                         |
|  ratestandard    Target production rate                          |
|  timerunning     Seconds producing                               |
|  timeidle        Seconds waiting (no fault)                      |
|  timedownplanned    Seconds - scheduled downtime                 |
|  timedownunplanned  Seconds - unscheduled downtime               |
|                                                                  |
+------------------------------------------------------------------+
```

### Card 2: Process Data Quick Reference

```
+------------------------------------------------------------------+
|  PROCESS DATA - Path: {asset}/processdata/                       |
+------------------------------------------------------------------+
|                                                                  |
|  STATE INFO - Path: {asset}/processdata/state/                   |
|  -------------------------------------------------------         |
|  code            Numeric state code                              |
|  name            Human-readable state name                       |
|  type            State category                                  |
|  duration        Seconds in current state                        |
|                                                                  |
|  COUNTS - Path: {asset}/processdata/count/                       |
|  -------------------------------------------------------         |
|  infeed          Current infeed count                            |
|  outfeed         Current outfeed count                           |
|  defect          Current defect count                            |
|                                                                  |
|  RATES - Path: {asset}/processdata/rate/                         |
|  -------------------------------------------------------         |
|  instant         Real-time production rate                       |
|                                                                  |
|  LIQUID PROCESSING ONLY - Path: {asset}/processdata/process/     |
|  -------------------------------------------------------         |
|  temperature     Process temperature                             |
|  flowrate        Current flow rate                               |
|  weight          Current weight/level                            |
|                                                                  |
+------------------------------------------------------------------+
```

### Card 3: Work Order Quick Reference

```
+------------------------------------------------------------------+
|  WORK ORDER DATA - Path: {asset}/workorder/                      |
+------------------------------------------------------------------+
|                                                                  |
|  ORDER INFO                                                      |
|  -------------------------------------------------------         |
|  workorderid        Unique order identifier                      |
|  workordernumber    Order number string                          |
|  quantitytarget     Target production quantity                   |
|  quantityactual     Actual quantity produced                     |
|  quantitydefect     Defective quantity                           |
|  uom                Unit of measure                              |
|                                                                  |
|  LOT INFO - Path: {asset}/workorder/lotnumber/                   |
|  -------------------------------------------------------         |
|  lotnumber          Lot number string                            |
|  lotnumberid        Lot number identifier                        |
|                                                                  |
|  ITEM INFO - Path: {asset}/workorder/lotnumber/item/             |
|  -------------------------------------------------------         |
|  itemid             Item identifier                              |
|  itemname           Product name                                 |
|  itemclass          Product classification                       |
|  bottlesize         Bottle size specification                    |
|  labelvariant       Label variant code                           |
|  packcount          Units per pack                               |
|                                                                  |
+------------------------------------------------------------------+
```

### Card 4: Site Quick Lookup

```
+------------------------------------------------------------------+
|  SITE LOOKUP CARD                                                |
+------------------------------------------------------------------+

  SITE 1: The Cap Shack (ID: 2)
  Path prefix: Enterprise B/Site1/
  +-----------------+------------------+---------------------------+
  | Area            | Lines            | Key Equipment             |
  +-----------------+------------------+---------------------------+
  | fillerproduction| fillingline01    | washer, filler, caploader |
  |                 | fillingline02    | washer, filler, caploader |
  |                 | fillingline03    | washer, filler, caploader |
  +-----------------+------------------+---------------------------+
  | liquidprocessing| mixroom01        | vat01-04 (Jeff, Raymond,  |
  |                 |                  |  Billy, Bob)              |
  |                 | tankstorage01    | tank01-06                 |
  +-----------------+------------------+---------------------------+
  | packaging       | labelerline01-04 | labeler, packager, sealer |
  +-----------------+------------------+---------------------------+
  | palletizing     | palletizer01-02  | robot, pallet01-02,       |
  |                 |                  |  wrapper                  |
  |                 | palletizermanual | workstation               |
  |                 |  01-04           |                           |
  +-----------------+------------------+---------------------------+

  SITE 2: Filler Central (ID: 65)
  Path prefix: Enterprise B/Site2/
  +-----------------+------------------+---------------------------+
  | fillerproduction| fillingline01-02 | washer, filler, caploader |
  | liquidprocessing| mixroom01        | vat01-02                  |
  |                 | tankstorage01    | tank01-03                 |
  | packaging       | labelerline01-02 | labeler, packager, sealer |
  | palletizing     | palletizer01     | robot, pallet01-02,       |
  |                 |                  |  wrapper                  |
  |                 | palletizermanual | workstation               |
  |                 |  01-02           |                           |
  +-----------------+------------------+---------------------------+

  SITE 3: Plant3 (ID: 103)
  Path prefix: Enterprise B/Site3/
  +-----------------+------------------+---------------------------+
  | fillerproduction| fillingline01    | washer, filler, caploader |
  | liquidprocessing| mixroom01        | vat01                     |
  |                 | tankstorage01    | tank01-02                 |
  | packaging       | labelerline01    | labeler, packager, sealer |
  | palletizing     | palletizermanual | workstation               |
  |                 |  01              |                           |
  +-----------------+------------------+---------------------------+

+------------------------------------------------------------------+
```

---

## Hierarchy Structure

The following diagram shows the complete organizational structure of Enterprise B. Use this to understand how assets relate to each other and to construct valid UNS paths.

```
ENTERPRISE B
|
+-- SITE 1: The Cap Shack [ID: 2]
|   |
|   +-- fillerproduction (Production) [ID: 3]
|   |   |
|   |   +-- fillingline01 (Line A) [ID: 7]
|   |   |   +-- caploader (Capper)
|   |   |   +-- filler (Filler)
|   |   |   +-- washer (Washer)
|   |   |
|   |   +-- fillingline02 (Line B) [ID: 8]
|   |   |   +-- caploader, filler, washer
|   |   |
|   |   +-- fillingline03 (High Capacity Line) [ID: 9]
|   |       +-- caploader, filler, washer
|   |
|   +-- liquidprocessing (Processing) [ID: 4]
|   |   |
|   |   +-- mixroom01 (Mix Room) [ID: 10]
|   |   |   +-- vat01 (Jeff)      <- Process variables available
|   |   |   +-- vat02 (Raymond)
|   |   |   +-- vat03 (Billy)
|   |   |   +-- vat04 (Bob)
|   |   |
|   |   +-- tankstorage01 (North Tanks) [ID: 11]
|   |       +-- tank01 through tank06   <- Process variables available
|   |
|   +-- packaging (Packaging) [ID: 5]
|   |   |
|   |   +-- labelerline01 (Labeler A) [ID: 12]
|   |   |   +-- labeler, packager, sealer
|   |   |
|   |   +-- labelerline02 (Labeler B) [ID: 13]
|   |   |   +-- labeler, packager, sealer
|   |   |
|   |   +-- labelerline03 (Labeler 1) [ID: 14]
|   |   |   +-- labeler, packager, sealer
|   |   |
|   |   +-- labelerline04 (Labeler 2) [ID: 15]
|   |       +-- labeler, packager, sealer
|   |
|   +-- palletizing (Palletizing) [ID: 6]
|       |
|       +-- palletizer01 (East Robot) [ID: 16]
|       |   +-- robot, pallet01, pallet02, wrapper
|       |
|       +-- palletizer02 (West Robot) [ID: 17]
|       |   +-- robot, pallet01, pallet02, wrapper
|       |
|       +-- palletizermanual01-04 (Manual Stackers) [IDs: 18-21]
|           +-- workstation (each)
|
+-- SITE 2: Filler Central [ID: 65]
|   |
|   +-- fillerproduction [ID: 66]
|   |   +-- fillingline01 (Line 1) -> caploader, filler, washer
|   |   +-- fillingline02 (Line 2) -> caploader, filler, washer
|   |
|   +-- liquidprocessing [ID: 67]
|   |   +-- mixroom01 (Mix Room) -> vat01, vat02 (North Vat)
|   |   +-- tankstorage01 (Central Tanks) -> tank01-03
|   |
|   +-- packaging [ID: 68]
|   |   +-- labelerline01 (Labeler Left) -> labeler, packager, sealer
|   |   +-- labelerline02 (Labeler Right) -> labeler, packager, sealer
|   |
|   +-- palletizing [ID: 69]
|       +-- palletizer01 -> robot, pallet01, pallet02, wrapper
|       +-- palletizermanual01 (Left Station) -> workstation
|       +-- palletizermanual02 (Right Station) -> workstation
|
+-- SITE 3: Plant3 [ID: 103]
    |
    +-- fillerproduction (Production) [ID: 104]
    |   +-- fillingline01 (Filling) -> caploader, filler, washer
    |
    +-- liquidprocessing [ID: 105]
    |   +-- mixroom01 (Mix Room) -> vat01
    |   +-- tankstorage01 -> tank01 (North), tank02 (South)
    |
    +-- packaging (Packaging) [ID: 106]
    |   +-- labelerline01 -> labeler, packager, sealer
    |
    +-- palletizing (Palletizing) [ID: 107]
        +-- palletizermanual01 (Stacker) -> workstation
```

### Key Points About the Hierarchy

| Level | Description | Path Example |
|-------|-------------|--------------|
| **Enterprise** | Top level, always "Enterprise B" | `Enterprise B/` |
| **Site** | Physical plant location | `Enterprise B/Site1/` |
| **Area** | Functional grouping within a site | `Enterprise B/Site1/fillerproduction/` |
| **Line** | Production line or equipment group | `Enterprise B/Site1/fillerproduction/fillingline01/` |
| **WorkCenter** | Individual equipment unit | `Enterprise B/Site1/fillerproduction/fillingline01/filler/` |

> **Important:** Work orders are typically associated at the Line level, while real-time process data is available at the WorkCenter level.

---

## Topic Structure Reference

### How Topics Are Organized

Every asset that can produce data (Lines and WorkCenters) has data organized into four main categories:

```
{asset path}/
    |
    +-- node/assetidentifier/    <- WHO: Identity and metadata
    |       assetid, assetname, displayname, etc.
    |
    +-- metric/                  <- HOW WELL: Performance metrics
    |       oee, availability, performance, quality
    |       +-- input/           <- Raw metric inputs
    |
    +-- processdata/             <- WHAT NOW: Real-time operational data
    |       +-- state/           <- Current equipment state
    |       +-- count/           <- Production counts
    |       +-- rate/            <- Production rate
    |       +-- process/         <- Liquid processing only
    |
    +-- workorder/               <- WHAT FOR: Production context
            workorderid, lotnumber, item details
```

### Asset Identifier Topics

**Path:** `{asset}/node/assetidentifier/`

These topics tell you about the asset itself.

| Topic | What It Contains | Example Value |
|-------|------------------|---------------|
| `assetid` | Unique numeric identifier | `23` |
| `assetname` | Technical name (used in paths) | `Filler` |
| `displayname` | Human-friendly name | `Line A Filler` |
| `assetpath` | Full hierarchy path | `Enterprise B/Site1/fillerproduction/fillingline01/filler` |
| `assettypename` | Asset classification | `WorkCenter` |
| `parentassetid` | ID of parent asset | `7` |
| `sortorder` | Display sequence | `2` |

### OEE Metrics Topics

**Path:** `{asset}/metric/`

These topics provide performance metrics.

**Calculated Metrics** (derived from inputs):

| Topic | Description | Value Range | Good Target |
|-------|-------------|-------------|-------------|
| `oee` | Overall Equipment Effectiveness | 0.0 - 1.0 | > 0.85 |
| `availability` | Percentage of scheduled time equipment was available | 0.0 - 1.0 | > 0.90 |
| `performance` | Actual speed vs. target speed | 0.0 - 1.0 | > 0.95 |
| `quality` | Good units vs. total units | 0.0 - 1.0 | > 0.99 |

**Input Metrics** (`{asset}/metric/input/`):

| Topic | Description | Data Type |
|-------|-------------|-----------|
| `countinfeed` | Total units into equipment | Integer |
| `countoutfeed` | Total units out of equipment | Integer |
| `countdefect` | Total defective units | Integer |
| `rateactual` | Current production rate | Float |
| `ratestandard` | Target production rate | Float |
| `timerunning` | Seconds equipment was running | Integer |
| `timeidle` | Seconds equipment was idle | Integer |
| `timedownplanned` | Seconds of scheduled downtime | Integer |
| `timedownunplanned` | Seconds of unscheduled downtime | Integer |

### Process Data Topics

**Path:** `{asset}/processdata/`

Real-time operational data from the equipment.

**State Information** (`processdata/state/`):

| Topic | Description | Example |
|-------|-------------|---------|
| `code` | Numeric state code | `100` |
| `name` | Human-readable state | `Running` |
| `type` | State category | `Production` |
| `duration` | Seconds in current state | `3600` |

**Count Information** (`processdata/count/`):

| Topic | Description |
|-------|-------------|
| `infeed` | Current infeed counter |
| `outfeed` | Current outfeed counter |
| `defect` | Current defect counter |

**Rate Information** (`processdata/rate/`):

| Topic | Description |
|-------|-------------|
| `instant` | Real-time production rate |

**Process Variables** (`processdata/process/`) - *Liquid Processing Only*:

| Topic | Description | Equipment |
|-------|-------------|-----------|
| `temperature` | Process temperature | Vats, Tanks |
| `flowrate` | Current flow rate | Vats, Tanks |
| `weight` | Current weight/level | Vats, Tanks |

> **Note:** Process variables (`temperature`, `flowrate`, `weight`) are only available on liquid processing equipment (Vats and Tanks), not on discrete equipment like fillers or packagers.

### Work Order Topics

**Path:** `{asset}/workorder/`

Production order context. Available at Line level and on Liquid Processing WorkCenters.

| Topic | Description |
|-------|-------------|
| `workorderid` | Unique order identifier |
| `workordernumber` | Order number string |
| `quantitytarget` | Target quantity to produce |
| `quantityactual` | Actual quantity produced |
| `quantitydefect` | Defective quantity |
| `uom` | Unit of measure |

**Lot Information** (`workorder/lotnumber/`):

| Topic | Description |
|-------|-------------|
| `lotnumber` | Lot/batch number |
| `lotnumberid` | Lot identifier |

**Item Information** (`workorder/lotnumber/item/`):

| Topic | Description |
|-------|-------------|
| `itemid` | Product identifier |
| `itemname` | Product name |
| `itemclass` | Product classification |
| `bottlesize` | Bottle size specification |
| `labelvariant` | Label variant code |
| `packcount` | Units per pack |
| `parentitemid` | Parent item reference |

---

## Equipment Types and Their Available Data

Not all equipment types have the same topics available. Use this section to understand what data you can expect from each equipment type.

### Filling Line Equipment

**Equipment:** Washer, Filler, CapLoader

| Has | Does Not Have |
|-----|---------------|
| Asset Identifier (all) | Process variables (temperature, flowrate, weight) |
| OEE Metrics (all) | Work Order (at line level instead) |
| Process Data (state, count, rate) | |

**Example paths:**
```
Enterprise B/Site1/fillerproduction/fillingline01/filler/metric/oee
Enterprise B/Site1/fillerproduction/fillingline01/filler/processdata/state/name
Enterprise B/Site1/fillerproduction/fillingline01/filler/processdata/count/outfeed
```

### Liquid Processing Equipment

**Equipment:** Vat, Tank

| Has | Special Features |
|-----|------------------|
| Asset Identifier (all) | Process variables: temperature, flowrate, weight |
| OEE Metrics (all) | Work Order data at WorkCenter level |
| Process Data (state + process) | Lot and Item information |

**Example paths:**
```
Enterprise B/Site1/liquidprocessing/mixroom01/vat01/processdata/process/temperature
Enterprise B/Site1/liquidprocessing/tankstorage01/tank01/processdata/process/weight
Enterprise B/Site1/liquidprocessing/mixroom01/vat01/workorder/lotnumber/item/itemname
```

### Packaging Equipment

**Equipment:** Labeler, Packager, Sealer

| Has | Does Not Have |
|-----|---------------|
| Asset Identifier (all) | Process variables |
| OEE Metrics (all) | Work Order (at line level instead) |
| Process Data (state, count, rate) | |

**Example paths:**
```
Enterprise B/Site1/packaging/labelerline01/labeler/metric/oee
Enterprise B/Site1/packaging/labelerline01/packager/processdata/count/outfeed
Enterprise B/Site1/packaging/labelerline01/workorder/workordernumber
```

### Palletizing Equipment

**Equipment:** Robot, Pallet, Wrapper, Workstation

| Has | Notes |
|-----|-------|
| Asset Identifier (all) | |
| OEE Metrics (all) | |
| Process Data (state, rate) | Count data on Robot only |

**Example paths:**
```
Enterprise B/Site1/palletizing/palletizer01/robot/metric/oee
Enterprise B/Site1/palletizing/palletizer01/pallet01/processdata/state/name
Enterprise B/Site1/palletizing/palletizermanual01/workstation/metric/availability
```

---

## Site 1 Detailed Documentation

### Site Overview

| Property | Value |
|----------|-------|
| **Site ID** | 2 |
| **Asset Name** | Plant1 |
| **Display Name** | The Cap Shack |
| **UNS Path** | `Enterprise B/Site1` |

**Scale:** 4 Areas | 15 Lines | 43 WorkCenters | 63 Total Assets

This is the largest site in Enterprise B, featuring three filling lines (including a high-capacity line), four labeler lines, and both automated and manual palletizing.

### Area: FillerProduction (Production)

**Path:** `Enterprise B/Site1/fillerproduction`
**ID:** 3

| Line | Display Name | ID | Equipment |
|------|--------------|-----|-----------|
| `fillingline01` | Line A | 7 | CapLoader, Filler, Washer |
| `fillingline02` | Line B | 8 | CapLoader, Filler, Washer |
| `fillingline03` | High Capacity Line | 9 | CapLoader, Filler, Washer |

**WorkCenter Details:**

| Line | WorkCenter | Display Name | ID |
|------|------------|--------------|-----|
| fillingline01 | caploader | Capper | 22 |
| fillingline01 | filler | Filler | 23 |
| fillingline01 | washer | Washer | 24 |
| fillingline02 | caploader | Capper | 25 |
| fillingline02 | filler | Filler | 26 |
| fillingline02 | washer | Washer | 27 |
| fillingline03 | caploader | Capper | 28 |
| fillingline03 | filler | Filler | 29 |
| fillingline03 | washer | Washer | 30 |

### Area: LiquidProcessing (Processing)

**Path:** `Enterprise B/Site1/liquidprocessing`
**ID:** 4

| Line | Display Name | ID | Equipment |
|------|--------------|-----|-----------|
| `mixroom01` | Mix Room | 10 | Vat01-04 |
| `tankstorage01` | North Tanks | 11 | Tank01-06 |

**WorkCenter Details:**

| Line | WorkCenter | Display Name | ID |
|------|------------|--------------|-----|
| mixroom01 | vat01 | Jeff | 31 |
| mixroom01 | vat02 | Raymond | 32 |
| mixroom01 | vat03 | Billy | 33 |
| mixroom01 | vat04 | Bob | 34 |
| tankstorage01 | tank01 | Tank 1 | 35 |
| tankstorage01 | tank02 | Tank 2 | 36 |
| tankstorage01 | tank03 | Tank 3 | 37 |
| tankstorage01 | tank04 | Tank 4 | 38 |
| tankstorage01 | tank05 | Tank 5 | 39 |
| tankstorage01 | tank06 | Tank 6 | 40 |

> **Note:** The vats have informal names (Jeff, Raymond, Billy, Bob) that staff may use. Use the technical names (vat01, vat02, etc.) in UNS paths.

### Area: Packaging

**Path:** `Enterprise B/Site1/packaging`
**ID:** 5

| Line | Display Name | ID | Equipment |
|------|--------------|-----|-----------|
| `labelerline01` | Labeler A | 12 | Labeler, Packager, Sealer |
| `labelerline02` | Labeler B | 13 | Labeler, Packager, Sealer |
| `labelerline03` | Labeler 1 | 14 | Labeler, Packager, Sealer |
| `labelerline04` | Labeler 2 | 15 | Labeler, Packager, Sealer |

**WorkCenter IDs:**

| Line | Labeler ID | Packager ID | Sealer ID |
|------|------------|-------------|-----------|
| labelerline01 | 41 | 42 | 43 |
| labelerline02 | 44 | 45 | 46 |
| labelerline03 | 47 | 48 | 49 |
| labelerline04 | 50 | 51 | 52 |

### Area: Palletizing

**Path:** `Enterprise B/Site1/palletizing`
**ID:** 6

**Automated Palletizers:**

| Line | Display Name | ID | Equipment |
|------|--------------|-----|-----------|
| `palletizer01` | East Robot | 16 | Robot (53), Pallet01 (54), Pallet02 (55), Wrapper (56) |
| `palletizer02` | West Robot | 17 | Robot (57), Pallet01 (58), Pallet02 (59), Wrapper (60) |

**Manual Palletizers:**

| Line | Display Name | ID | Equipment |
|------|--------------|-----|-----------|
| `palletizermanual01` | East Robot 1st Stacker | 18 | Workstation (61) |
| `palletizermanual02` | East Robot 2nd Stacker | 19 | Workstation (62) |
| `palletizermanual03` | West Robot 1st Stacker | 20 | Workstation (63) |
| `palletizermanual04` | West Robot 2nd Stacker | 21 | Workstation (64) |

---

## Site 2 Detailed Documentation

### Site Overview

| Property | Value |
|----------|-------|
| **Site ID** | 65 |
| **Asset Name** | Plant2 |
| **Display Name** | Filler Central |
| **UNS Path** | `Enterprise B/Site2` |

**Scale:** 4 Areas | 9 Lines | 23 WorkCenters | 35 Total Assets

A mid-sized facility with two filling lines and comprehensive liquid processing capabilities.

### Area: FillerProduction

**Path:** `Enterprise B/Site2/fillerproduction`
**ID:** 66

| Line | Display Name | ID | Equipment (IDs) |
|------|--------------|-----|-----------------|
| `fillingline01` | Line 1 | 70 | CapLoader (80), Filler (81), Washer (82) |
| `fillingline02` | Line 2 | 71 | CapLoader (83), Filler (84), Washer (85) |

### Area: LiquidProcessing

**Path:** `Enterprise B/Site2/liquidprocessing`
**ID:** 67

| Line | Display Name | ID | Equipment (IDs) |
|------|--------------|-----|-----------------|
| `mixroom01` | Mix Room | 73 | Vat01 (86), Vat02 (87, North Vat) |
| `tankstorage01` | Central Tanks | 74 | Tank01 (88, Left Tank), Tank02 (89), Tank03 (90) |

### Area: Packaging

**Path:** `Enterprise B/Site2/packaging`
**ID:** 68

| Line | Display Name | ID | Equipment (IDs) |
|------|--------------|-----|-----------------|
| `labelerline01` | Labeler Left | 75 | Labeler (91), Packager (92), Sealer (124) |
| `labelerline02` | Labeler Right | 76 | Labeler (93), Packager (94), Sealer (96) |

### Area: Palletizing

**Path:** `Enterprise B/Site2/palletizing`
**ID:** 69

| Line | Display Name | ID | Equipment (IDs) |
|------|--------------|-----|-----------------|
| `palletizer01` | *(automated)* | 77 | Pallet01 (97), Pallet02 (98, West Pallet), Robot (99), Wrapper (100) |
| `palletizermanual01` | Left Station | 78 | Workstation (101) |
| `palletizermanual02` | Right Station | 79 | Workstation (102) |

---

## Site 3 Detailed Documentation

### Site Overview

| Property | Value |
|----------|-------|
| **Site ID** | 103 |
| **Asset Name** | Plant3 |
| **Display Name** | *(not set)* |
| **UNS Path** | `Enterprise B/Site3` |

**Scale:** 4 Areas | 5 Lines | 10 WorkCenters | 20 Total Assets

The smallest site, with single production lines in each area. Good for testing queries before scaling to larger sites.

### Area: FillerProduction (Production)

**Path:** `Enterprise B/Site3/fillerproduction`
**ID:** 104

| Line | Display Name | ID | Equipment (IDs) |
|------|--------------|-----|-----------------|
| `fillingline01` | Filling | 108 | CapLoader (113, Cap Loader), Filler (114), Washer (115) |

### Area: LiquidProcessing

**Path:** `Enterprise B/Site3/liquidprocessing`
**ID:** 105

| Line | Display Name | ID | Equipment (IDs) |
|------|--------------|-----|-----------------|
| `mixroom01` | Mix Room | 109 | Vat01 (116, Vat) |
| `tankstorage01` | *(not set)* | 110 | Tank01 (117, North Tank), Tank02 (118, South Tank) |

### Area: Packaging (Packaging)

**Path:** `Enterprise B/Site3/packaging`
**ID:** 106

| Line | Display Name | ID | Equipment (IDs) |
|------|--------------|-----|-----------------|
| `labelerline01` | *(not set)* | 111 | Labeler (119), Packager (120), Sealer (121) |

### Area: Palletizing (Palletizing)

**Path:** `Enterprise B/Site3/palletizing`
**ID:** 107

| Line | Display Name | ID | Equipment (IDs) |
|------|--------------|-----|-----------------|
| `palletizermanual01` | Stacker | 112 | Workstation (123, Stacker) |

---

## Enterprise Summary

### Asset Counts by Site

| Metric | Site 1 | Site 2 | Site 3 | Enterprise Total |
|--------|--------|--------|--------|------------------|
| Areas | 4 | 4 | 4 | **12** |
| Lines | 15 | 9 | 5 | **29** |
| WorkCenters | 43 | 23 | 10 | **76** |
| **Total Assets** | 63 | 37 | 20 | **120** |

### Equipment Counts by Type

| Equipment Type | Site 1 | Site 2 | Site 3 | Total |
|----------------|--------|--------|--------|-------|
| Filling Lines | 3 | 2 | 1 | 6 |
| Vats | 4 | 2 | 1 | 7 |
| Tanks | 6 | 3 | 2 | 11 |
| Labeler Lines | 4 | 2 | 1 | 7 |
| Automated Palletizers | 2 | 1 | 0 | 3 |
| Manual Palletizers | 4 | 2 | 1 | 7 |

---

## Complete Asset Registry

This section provides a comprehensive lookup table for every asset in Enterprise B with their IDs, parent relationships, and UNS paths.

### Site Level

| ID | Asset Name | Display Name | Type | Parent ID | UNS Path |
|----|------------|--------------|------|-----------|----------|
| 2 | Plant1 | The Cap Shack | Site | 1 | `Enterprise B/Site1` |
| 65 | Plant2 | Filler Central | Site | 1 | `Enterprise B/Site2` |
| 103 | Plant3 | *(none)* | Site | 1 | `Enterprise B/Site3` |

### Area Level

#### Site 1 Areas

| ID | Asset Name | Display Name | Type | Parent ID | UNS Path |
|----|------------|--------------|------|-----------|----------|
| 3 | FillerProduction | Production | Area | 2 | `Enterprise B/Site1/fillerproduction` |
| 4 | LiquidProcessing | Processing | Area | 2 | `Enterprise B/Site1/liquidprocessing` |
| 5 | Packaging | Packaging | Area | 2 | `Enterprise B/Site1/packaging` |
| 6 | Palletizing | Palletizing | Area | 2 | `Enterprise B/Site1/palletizing` |

#### Site 2 Areas

| ID | Asset Name | Display Name | Type | Parent ID | UNS Path |
|----|------------|--------------|------|-----------|----------|
| 66 | FillerProduction | *(none)* | Area | 65 | `Enterprise B/Site2/fillerproduction` |
| 67 | LiquidProcessing | *(none)* | Area | 65 | `Enterprise B/Site2/liquidprocessing` |
| 68 | Packaging | *(none)* | Area | 65 | `Enterprise B/Site2/packaging` |
| 69 | Palletizing | *(none)* | Area | 65 | `Enterprise B/Site2/palletizing` |

#### Site 3 Areas

| ID | Asset Name | Display Name | Type | Parent ID | UNS Path |
|----|------------|--------------|------|-----------|----------|
| 104 | FillerProduction | Production | Area | 103 | `Enterprise B/Site3/fillerproduction` |
| 105 | LiquidProcessing | *(none)* | Area | 103 | `Enterprise B/Site3/liquidprocessing` |
| 106 | Packaging | Packaging | Area | 103 | `Enterprise B/Site3/packaging` |
| 107 | Palletizing | Palletizing | Area | 103 | `Enterprise B/Site3/palletizing` |

### Line Level

#### Site 1 Lines

| ID | Asset Name | Display Name | Type | Parent ID | Area | UNS Path |
|----|------------|--------------|------|-----------|------|----------|
| 7 | FillingLine01 | Line A | Line | 3 | FillerProduction | `Enterprise B/Site1/fillerproduction/fillingline01` |
| 8 | FillingLine02 | Line B | Line | 3 | FillerProduction | `Enterprise B/Site1/fillerproduction/fillingline02` |
| 9 | FillingLine03 | High Capacity Line | Line | 3 | FillerProduction | `Enterprise B/Site1/fillerproduction/fillingline03` |
| 10 | MixRoom01 | Mix Room | Line | 4 | LiquidProcessing | `Enterprise B/Site1/liquidprocessing/mixroom01` |
| 11 | TankStorage01 | North Tanks | Line | 4 | LiquidProcessing | `Enterprise B/Site1/liquidprocessing/tankstorage01` |
| 12 | LabelerLine01 | Labeler A | Line | 5 | Packaging | `Enterprise B/Site1/packaging/labelerline01` |
| 13 | LabelerLine02 | Labeler B | Line | 5 | Packaging | `Enterprise B/Site1/packaging/labelerline02` |
| 14 | LabelerLine03 | Labeler 1 | Line | 5 | Packaging | `Enterprise B/Site1/packaging/labelerline03` |
| 15 | LabelerLine04 | Labeler 2 | Line | 5 | Packaging | `Enterprise B/Site1/packaging/labelerline04` |
| 16 | Palletizer01 | East Robot | Line | 6 | Palletizing | `Enterprise B/Site1/palletizing/palletizer01` |
| 17 | Palletizer02 | West Robot | Line | 6 | Palletizing | `Enterprise B/Site1/palletizing/palletizer02` |
| 18 | PalletizerManual01 | East Robot 1st Stacker | Line | 6 | Palletizing | `Enterprise B/Site1/palletizing/palletizermanual01` |
| 19 | PalletizerManual02 | East Robot 2nd Stacker | Line | 6 | Palletizing | `Enterprise B/Site1/palletizing/palletizermanual02` |
| 20 | PalletizerManual03 | West Robot 1st Stacker | Line | 6 | Palletizing | `Enterprise B/Site1/palletizing/palletizermanual03` |
| 21 | PalletizerManual04 | West Robot 2nd Stacker | Line | 6 | Palletizing | `Enterprise B/Site1/palletizing/palletizermanual04` |

#### Site 2 Lines

| ID | Asset Name | Display Name | Type | Parent ID | Area | UNS Path |
|----|------------|--------------|------|-----------|------|----------|
| 70 | FillingLine01 | Line 1 | Line | 66 | FillerProduction | `Enterprise B/Site2/fillerproduction/fillingline01` |
| 71 | FillingLine02 | Line 2 | Line | 66 | FillerProduction | `Enterprise B/Site2/fillerproduction/fillingline02` |
| 73 | MixRoom01 | Mix Room | Line | 67 | LiquidProcessing | `Enterprise B/Site2/liquidprocessing/mixroom01` |
| 74 | TankStorage01 | Central Tanks | Line | 67 | LiquidProcessing | `Enterprise B/Site2/liquidprocessing/tankstorage01` |
| 75 | LabelerLine01 | Labeler Left | Line | 68 | Packaging | `Enterprise B/Site2/packaging/labelerline01` |
| 76 | LabelerLine02 | Labeler Right | Line | 68 | Packaging | `Enterprise B/Site2/packaging/labelerline02` |
| 77 | Palletizer01 | *(none)* | Line | 69 | Palletizing | `Enterprise B/Site2/palletizing/palletizer01` |
| 78 | PalletizerManual01 | Left Station | Line | 69 | Palletizing | `Enterprise B/Site2/palletizing/palletizermanual01` |
| 79 | PalletizerManual02 | Right Station | Line | 69 | Palletizing | `Enterprise B/Site2/palletizing/palletizermanual02` |

#### Site 3 Lines

| ID | Asset Name | Display Name | Type | Parent ID | Area | UNS Path |
|----|------------|--------------|------|-----------|------|----------|
| 108 | FillingLine01 | Filling | Line | 104 | FillerProduction | `Enterprise B/Site3/fillerproduction/fillingline01` |
| 109 | MixRoom01 | Mix Room | Line | 105 | LiquidProcessing | `Enterprise B/Site3/liquidprocessing/mixroom01` |
| 110 | TankStorage01 | *(none)* | Line | 105 | LiquidProcessing | `Enterprise B/Site3/liquidprocessing/tankstorage01` |
| 111 | LabelerLine01 | *(none)* | Line | 106 | Packaging | `Enterprise B/Site3/packaging/labelerline01` |
| 112 | PalletizerManual01 | Stacker | Line | 107 | Palletizing | `Enterprise B/Site3/palletizing/palletizermanual01` |

### WorkCenter Level

#### Site 1 - FillerProduction WorkCenters

| ID | Asset Name | Display Name | Type | Parent ID | Parent Line | UNS Path |
|----|------------|--------------|------|-----------|-------------|----------|
| 22 | CapLoader | Capper | WorkCenter | 7 | FillingLine01 | `Enterprise B/Site1/fillerproduction/fillingline01/caploader` |
| 23 | Filler | Filler | WorkCenter | 7 | FillingLine01 | `Enterprise B/Site1/fillerproduction/fillingline01/filler` |
| 24 | Washer | Washer | WorkCenter | 7 | FillingLine01 | `Enterprise B/Site1/fillerproduction/fillingline01/washer` |
| 25 | CapLoader | Capper | WorkCenter | 8 | FillingLine02 | `Enterprise B/Site1/fillerproduction/fillingline02/caploader` |
| 26 | Filler | Filler | WorkCenter | 8 | FillingLine02 | `Enterprise B/Site1/fillerproduction/fillingline02/filler` |
| 27 | Washer | Washer | WorkCenter | 8 | FillingLine02 | `Enterprise B/Site1/fillerproduction/fillingline02/washer` |
| 28 | CapLoader | Capper | WorkCenter | 9 | FillingLine03 | `Enterprise B/Site1/fillerproduction/fillingline03/caploader` |
| 29 | Filler | Filler | WorkCenter | 9 | FillingLine03 | `Enterprise B/Site1/fillerproduction/fillingline03/filler` |
| 30 | Washer | Washer | WorkCenter | 9 | FillingLine03 | `Enterprise B/Site1/fillerproduction/fillingline03/washer` |

#### Site 1 - LiquidProcessing WorkCenters

| ID | Asset Name | Display Name | Type | Parent ID | Parent Line | UNS Path |
|----|------------|--------------|------|-----------|-------------|----------|
| 31 | Vat01 | Jeff | WorkCenter | 10 | MixRoom01 | `Enterprise B/Site1/liquidprocessing/mixroom01/vat01` |
| 32 | Vat02 | Raymond | WorkCenter | 10 | MixRoom01 | `Enterprise B/Site1/liquidprocessing/mixroom01/vat02` |
| 33 | Vat03 | Billy | WorkCenter | 10 | MixRoom01 | `Enterprise B/Site1/liquidprocessing/mixroom01/vat03` |
| 34 | Vat04 | Bob | WorkCenter | 10 | MixRoom01 | `Enterprise B/Site1/liquidprocessing/mixroom01/vat04` |
| 35 | Tank01 | Tank 1 | WorkCenter | 11 | TankStorage01 | `Enterprise B/Site1/liquidprocessing/tankstorage01/tank01` |
| 36 | Tank02 | Tank 2 | WorkCenter | 11 | TankStorage01 | `Enterprise B/Site1/liquidprocessing/tankstorage01/tank02` |
| 37 | Tank03 | Tank 3 | WorkCenter | 11 | TankStorage01 | `Enterprise B/Site1/liquidprocessing/tankstorage01/tank03` |
| 38 | Tank04 | Tank 4 | WorkCenter | 11 | TankStorage01 | `Enterprise B/Site1/liquidprocessing/tankstorage01/tank04` |
| 39 | Tank05 | Tank 5 | WorkCenter | 11 | TankStorage01 | `Enterprise B/Site1/liquidprocessing/tankstorage01/tank05` |
| 40 | Tank06 | Tank 6 | WorkCenter | 11 | TankStorage01 | `Enterprise B/Site1/liquidprocessing/tankstorage01/tank06` |

#### Site 1 - Packaging WorkCenters

| ID | Asset Name | Display Name | Type | Parent ID | Parent Line | UNS Path |
|----|------------|--------------|------|-----------|-------------|----------|
| 41 | Labeler | Labeler | WorkCenter | 12 | LabelerLine01 | `Enterprise B/Site1/packaging/labelerline01/labeler` |
| 42 | Packager | Packager | WorkCenter | 12 | LabelerLine01 | `Enterprise B/Site1/packaging/labelerline01/packager` |
| 43 | Sealer | Sealer | WorkCenter | 12 | LabelerLine01 | `Enterprise B/Site1/packaging/labelerline01/sealer` |
| 44 | Labeler | Labeler | WorkCenter | 13 | LabelerLine02 | `Enterprise B/Site1/packaging/labelerline02/labeler` |
| 45 | Packager | Packager | WorkCenter | 13 | LabelerLine02 | `Enterprise B/Site1/packaging/labelerline02/packager` |
| 46 | Sealer | Sealer | WorkCenter | 13 | LabelerLine02 | `Enterprise B/Site1/packaging/labelerline02/sealer` |
| 47 | Labeler | Labeler | WorkCenter | 14 | LabelerLine03 | `Enterprise B/Site1/packaging/labelerline03/labeler` |
| 48 | Packager | Packager | WorkCenter | 14 | LabelerLine03 | `Enterprise B/Site1/packaging/labelerline03/packager` |
| 49 | Sealer | Sealer | WorkCenter | 14 | LabelerLine03 | `Enterprise B/Site1/packaging/labelerline03/sealer` |
| 50 | Labeler | Labeler | WorkCenter | 15 | LabelerLine04 | `Enterprise B/Site1/packaging/labelerline04/labeler` |
| 51 | Packager | Packager | WorkCenter | 15 | LabelerLine04 | `Enterprise B/Site1/packaging/labelerline04/packager` |
| 52 | Sealer | Sealer | WorkCenter | 15 | LabelerLine04 | `Enterprise B/Site1/packaging/labelerline04/sealer` |

#### Site 1 - Palletizing WorkCenters

| ID | Asset Name | Display Name | Type | Parent ID | Parent Line | UNS Path |
|----|------------|--------------|------|-----------|-------------|----------|
| 53 | Robot | Robot | WorkCenter | 16 | Palletizer01 | `Enterprise B/Site1/palletizing/palletizer01/robot` |
| 54 | Pallet01 | Pallet 1 | WorkCenter | 16 | Palletizer01 | `Enterprise B/Site1/palletizing/palletizer01/pallet01` |
| 55 | Pallet02 | Pallet 2 | WorkCenter | 16 | Palletizer01 | `Enterprise B/Site1/palletizing/palletizer01/pallet02` |
| 56 | Wrapper | Wrapper | WorkCenter | 16 | Palletizer01 | `Enterprise B/Site1/palletizing/palletizer01/wrapper` |
| 57 | Robot | Robot | WorkCenter | 17 | Palletizer02 | `Enterprise B/Site1/palletizing/palletizer02/robot` |
| 58 | Pallet01 | Pallet 1 | WorkCenter | 17 | Palletizer02 | `Enterprise B/Site1/palletizing/palletizer02/pallet01` |
| 59 | Pallet02 | Pallet 2 | WorkCenter | 17 | Palletizer02 | `Enterprise B/Site1/palletizing/palletizer02/pallet02` |
| 60 | Wrapper | Wrapper | WorkCenter | 17 | Palletizer02 | `Enterprise B/Site1/palletizing/palletizer02/wrapper` |
| 61 | Workstation | Workstation | WorkCenter | 18 | PalletizerManual01 | `Enterprise B/Site1/palletizing/palletizermanual01/workstation` |
| 62 | Workstation | Workstation | WorkCenter | 19 | PalletizerManual02 | `Enterprise B/Site1/palletizing/palletizermanual02/workstation` |
| 63 | Workstation | Workstation | WorkCenter | 20 | PalletizerManual03 | `Enterprise B/Site1/palletizing/palletizermanual03/workstation` |
| 64 | Workstation | Workstation | WorkCenter | 21 | PalletizerManual04 | `Enterprise B/Site1/palletizing/palletizermanual04/workstation` |

#### Site 2 - All WorkCenters

| ID | Asset Name | Display Name | Type | Parent ID | Parent Line | UNS Path |
|----|------------|--------------|------|-----------|-------------|----------|
| 80 | CapLoader | *(none)* | WorkCenter | 70 | FillingLine01 | `Enterprise B/Site2/fillerproduction/fillingline01/caploader` |
| 81 | Filler | *(none)* | WorkCenter | 70 | FillingLine01 | `Enterprise B/Site2/fillerproduction/fillingline01/filler` |
| 82 | Washer | *(none)* | WorkCenter | 70 | FillingLine01 | `Enterprise B/Site2/fillerproduction/fillingline01/washer` |
| 83 | CapLoader | *(none)* | WorkCenter | 71 | FillingLine02 | `Enterprise B/Site2/fillerproduction/fillingline02/caploader` |
| 84 | Filler | *(none)* | WorkCenter | 71 | FillingLine02 | `Enterprise B/Site2/fillerproduction/fillingline02/filler` |
| 85 | Washer | *(none)* | WorkCenter | 71 | FillingLine02 | `Enterprise B/Site2/fillerproduction/fillingline02/washer` |
| 86 | Vat01 | *(none)* | WorkCenter | 73 | MixRoom01 | `Enterprise B/Site2/liquidprocessing/mixroom01/vat01` |
| 87 | Vat02 | North Vat | WorkCenter | 73 | MixRoom01 | `Enterprise B/Site2/liquidprocessing/mixroom01/vat02` |
| 88 | Tank01 | Left Tank | WorkCenter | 74 | TankStorage01 | `Enterprise B/Site2/liquidprocessing/tankstorage01/tank01` |
| 89 | Tank02 | *(none)* | WorkCenter | 74 | TankStorage01 | `Enterprise B/Site2/liquidprocessing/tankstorage01/tank02` |
| 90 | Tank03 | *(none)* | WorkCenter | 74 | TankStorage01 | `Enterprise B/Site2/liquidprocessing/tankstorage01/tank03` |
| 91 | Labeler | *(none)* | WorkCenter | 75 | LabelerLine01 | `Enterprise B/Site2/packaging/labelerline01/labeler` |
| 92 | Packager | *(none)* | WorkCenter | 75 | LabelerLine01 | `Enterprise B/Site2/packaging/labelerline01/packager` |
| 124 | Sealer | *(none)* | WorkCenter | 75 | LabelerLine01 | `Enterprise B/Site2/packaging/labelerline01/sealer` |
| 93 | Labeler | *(none)* | WorkCenter | 76 | LabelerLine02 | `Enterprise B/Site2/packaging/labelerline02/labeler` |
| 94 | Packager | *(none)* | WorkCenter | 76 | LabelerLine02 | `Enterprise B/Site2/packaging/labelerline02/packager` |
| 96 | Sealer | *(none)* | WorkCenter | 76 | LabelerLine02 | `Enterprise B/Site2/packaging/labelerline02/sealer` |
| 97 | Pallet01 | *(none)* | WorkCenter | 77 | Palletizer01 | `Enterprise B/Site2/palletizing/palletizer01/pallet01` |
| 98 | Pallet02 | West Pallet | WorkCenter | 77 | Palletizer01 | `Enterprise B/Site2/palletizing/palletizer01/pallet02` |
| 99 | Robot | *(none)* | WorkCenter | 77 | Palletizer01 | `Enterprise B/Site2/palletizing/palletizer01/robot` |
| 100 | Wrapper | *(none)* | WorkCenter | 77 | Palletizer01 | `Enterprise B/Site2/palletizing/palletizer01/wrapper` |
| 101 | Workstation | *(none)* | WorkCenter | 78 | PalletizerManual01 | `Enterprise B/Site2/palletizing/palletizermanual01/workstation` |
| 102 | Workstation | *(none)* | WorkCenter | 79 | PalletizerManual02 | `Enterprise B/Site2/palletizing/palletizermanual02/workstation` |

#### Site 3 - All WorkCenters

| ID | Asset Name | Display Name | Type | Parent ID | Parent Line | UNS Path |
|----|------------|--------------|------|-----------|-------------|----------|
| 113 | CapLoader | Cap Loader | WorkCenter | 108 | FillingLine01 | `Enterprise B/Site3/fillerproduction/fillingline01/caploader` |
| 114 | Filler | *(none)* | WorkCenter | 108 | FillingLine01 | `Enterprise B/Site3/fillerproduction/fillingline01/filler` |
| 115 | Washer | *(none)* | WorkCenter | 108 | FillingLine01 | `Enterprise B/Site3/fillerproduction/fillingline01/washer` |
| 116 | Vat01 | Vat | WorkCenter | 109 | MixRoom01 | `Enterprise B/Site3/liquidprocessing/mixroom01/vat01` |
| 117 | Tank01 | North Tank | WorkCenter | 110 | TankStorage01 | `Enterprise B/Site3/liquidprocessing/tankstorage01/tank01` |
| 118 | Tank02 | South Tank | WorkCenter | 110 | TankStorage01 | `Enterprise B/Site3/liquidprocessing/tankstorage01/tank02` |
| 119 | Labeler | *(none)* | WorkCenter | 111 | LabelerLine01 | `Enterprise B/Site3/packaging/labelerline01/labeler` |
| 120 | Packager | *(none)* | WorkCenter | 111 | LabelerLine01 | `Enterprise B/Site3/packaging/labelerline01/packager` |
| 121 | Sealer | *(none)* | WorkCenter | 111 | LabelerLine01 | `Enterprise B/Site3/packaging/labelerline01/sealer` |
| 123 | Workstation | Stacker | WorkCenter | 112 | PalletizerManual01 | `Enterprise B/Site3/palletizing/palletizermanual01/workstation` |

---

## Glossary

| Term | Definition |
|------|------------|
| **Area** | A functional grouping within a site (e.g., FillerProduction, Packaging). The third level of the UNS hierarchy. |
| **Asset** | Any entity in the UNS hierarchy: Enterprise, Site, Area, Line, or WorkCenter. |
| **Asset ID** | A unique numeric identifier assigned to each asset in the system. |
| **Availability** | OEE component measuring the percentage of scheduled time that equipment was available for production (not down for maintenance or breakdowns). |
| **Display Name** | The human-friendly name for an asset, which may differ from the technical asset name used in paths. |
| **Enterprise** | The top level of the UNS hierarchy, representing the entire organization. |
| **Line** | A production line or functional equipment group within an area. The fourth level of the hierarchy. |
| **Lot Number** | A tracking identifier assigned to a batch of product for traceability purposes. |
| **OEE** | Overall Equipment Effectiveness. A manufacturing metric calculated as Availability x Performance x Quality. Values range from 0 to 1 (or 0% to 100%). World-class OEE is typically considered to be 85% or higher. |
| **Performance** | OEE component measuring actual production speed versus the target or ideal speed. |
| **Process Data** | Real-time operational data from equipment, including states, counts, rates, and (for liquid processing) temperatures and flow rates. |
| **Quality** | OEE component measuring the ratio of good units to total units produced. |
| **Site** | A physical plant location within the enterprise. The second level of the hierarchy. |
| **Topic** | A specific data point within the UNS, identified by its full path (e.g., `.../metric/oee`). |
| **UNS** | Unified Namespace. A hierarchical data organization system that provides a single, consistent way to access all operational data. |
| **UNS Path** | The full hierarchical address of an asset or data point in the namespace (e.g., `Enterprise B/Site1/fillerproduction/fillingline01/metric/oee`). |
| **Work Order** | A production order specifying what product to make, in what quantity, with associated lot and item information. |
| **WorkCenter** | The lowest level of the hierarchy, representing individual equipment units (e.g., a specific filler, labeler, or robot). |

---

## Troubleshooting Common Issues

### "I can't find data for my equipment"

1. **Check the path case:** All path segments are lowercase
   - Wrong: `Enterprise B/Site1/FillerProduction/`
   - Correct: `Enterprise B/Site1/fillerproduction/`

2. **Use asset names, not display names:**
   - Wrong: `Enterprise B/Site1/fillerproduction/Line A/`
   - Correct: `Enterprise B/Site1/fillerproduction/fillingline01/`

3. **Verify the equipment exists at the expected level:**
   - Work orders are at Line level, not WorkCenter level (except for liquid processing)
   - Process variables (temperature, flowrate) only exist on Vats and Tanks

### "The OEE value seems wrong"

- OEE values are between 0 and 1, not 0 and 100
- Multiply by 100 to get a percentage
- A value of `0.85` means 85% OEE

### "I need the display name, not the technical name"

Query the `displayname` topic for any asset:
```
Enterprise B/Site1/liquidprocessing/mixroom01/vat01/node/assetidentifier/displayname
```
This returns "Jeff" for vat01 at Site 1.

### "Where do I find temperature data?"

Temperature data is only available on liquid processing equipment:
- Vats: `Enterprise B/{Site}/liquidprocessing/mixroom01/vat0X/processdata/process/temperature`
- Tanks: `Enterprise B/{Site}/liquidprocessing/tankstorage01/tank0X/processdata/process/temperature`

Discrete equipment (fillers, labelers, etc.) does not have temperature topics.

---

## Appendix: Complete Topic Path Templates

### FillerProduction Line Topics

Replace `{site}` with Site1, Site2, or Site3. Replace `{line}` with the specific line (fillingline01, etc.).

```
Enterprise B/{site}/fillerproduction/{line}/metric/availability
Enterprise B/{site}/fillerproduction/{line}/metric/oee
Enterprise B/{site}/fillerproduction/{line}/metric/performance
Enterprise B/{site}/fillerproduction/{line}/metric/quality
Enterprise B/{site}/fillerproduction/{line}/metric/input/countdefect
Enterprise B/{site}/fillerproduction/{line}/metric/input/countinfeed
Enterprise B/{site}/fillerproduction/{line}/metric/input/countoutfeed
Enterprise B/{site}/fillerproduction/{line}/metric/input/rateactual
Enterprise B/{site}/fillerproduction/{line}/metric/input/ratestandard
Enterprise B/{site}/fillerproduction/{line}/metric/input/timedownplanned
Enterprise B/{site}/fillerproduction/{line}/metric/input/timedownunplanned
Enterprise B/{site}/fillerproduction/{line}/metric/input/timeidle
Enterprise B/{site}/fillerproduction/{line}/metric/input/timerunning
Enterprise B/{site}/fillerproduction/{line}/node/assetidentifier/assetid
Enterprise B/{site}/fillerproduction/{line}/node/assetidentifier/assetname
Enterprise B/{site}/fillerproduction/{line}/node/assetidentifier/assetpath
Enterprise B/{site}/fillerproduction/{line}/node/assetidentifier/assettypename
Enterprise B/{site}/fillerproduction/{line}/node/assetidentifier/displayname
Enterprise B/{site}/fillerproduction/{line}/node/assetidentifier/parentassetid
Enterprise B/{site}/fillerproduction/{line}/node/assetidentifier/sortorder
Enterprise B/{site}/fillerproduction/{line}/workorder/assetid
Enterprise B/{site}/fillerproduction/{line}/workorder/quantityactual
Enterprise B/{site}/fillerproduction/{line}/workorder/quantitydefect
Enterprise B/{site}/fillerproduction/{line}/workorder/quantitytarget
Enterprise B/{site}/fillerproduction/{line}/workorder/uom
Enterprise B/{site}/fillerproduction/{line}/workorder/workorderid
Enterprise B/{site}/fillerproduction/{line}/workorder/workordernumber
Enterprise B/{site}/fillerproduction/{line}/workorder/lotnumber/lotnumber
Enterprise B/{site}/fillerproduction/{line}/workorder/lotnumber/lotnumberid
Enterprise B/{site}/fillerproduction/{line}/workorder/lotnumber/item/bottlesize
Enterprise B/{site}/fillerproduction/{line}/workorder/lotnumber/item/itemclass
Enterprise B/{site}/fillerproduction/{line}/workorder/lotnumber/item/itemid
Enterprise B/{site}/fillerproduction/{line}/workorder/lotnumber/item/itemname
Enterprise B/{site}/fillerproduction/{line}/workorder/lotnumber/item/labelvariant
Enterprise B/{site}/fillerproduction/{line}/workorder/lotnumber/item/packcount
Enterprise B/{site}/fillerproduction/{line}/workorder/lotnumber/item/parentitemid
```

### FillerProduction WorkCenter Topics

Replace `{workcenter}` with washer, filler, or caploader.

```
Enterprise B/{site}/fillerproduction/{line}/{workcenter}/metric/availability
Enterprise B/{site}/fillerproduction/{line}/{workcenter}/metric/oee
Enterprise B/{site}/fillerproduction/{line}/{workcenter}/metric/performance
Enterprise B/{site}/fillerproduction/{line}/{workcenter}/metric/quality
Enterprise B/{site}/fillerproduction/{line}/{workcenter}/metric/input/countdefect
Enterprise B/{site}/fillerproduction/{line}/{workcenter}/metric/input/countinfeed
Enterprise B/{site}/fillerproduction/{line}/{workcenter}/metric/input/countoutfeed
Enterprise B/{site}/fillerproduction/{line}/{workcenter}/metric/input/rateactual
Enterprise B/{site}/fillerproduction/{line}/{workcenter}/metric/input/ratestandard
Enterprise B/{site}/fillerproduction/{line}/{workcenter}/metric/input/timedownplanned
Enterprise B/{site}/fillerproduction/{line}/{workcenter}/metric/input/timedownunplanned
Enterprise B/{site}/fillerproduction/{line}/{workcenter}/metric/input/timeidle
Enterprise B/{site}/fillerproduction/{line}/{workcenter}/metric/input/timerunning
Enterprise B/{site}/fillerproduction/{line}/{workcenter}/node/assetidentifier/assetid
Enterprise B/{site}/fillerproduction/{line}/{workcenter}/node/assetidentifier/assetname
Enterprise B/{site}/fillerproduction/{line}/{workcenter}/node/assetidentifier/assetpath
Enterprise B/{site}/fillerproduction/{line}/{workcenter}/node/assetidentifier/assettypename
Enterprise B/{site}/fillerproduction/{line}/{workcenter}/node/assetidentifier/displayname
Enterprise B/{site}/fillerproduction/{line}/{workcenter}/node/assetidentifier/parentassetid
Enterprise B/{site}/fillerproduction/{line}/{workcenter}/node/assetidentifier/sortorder
Enterprise B/{site}/fillerproduction/{line}/{workcenter}/processdata/count/defect
Enterprise B/{site}/fillerproduction/{line}/{workcenter}/processdata/count/infeed
Enterprise B/{site}/fillerproduction/{line}/{workcenter}/processdata/count/outfeed
Enterprise B/{site}/fillerproduction/{line}/{workcenter}/processdata/input/infeedtooutfeed
Enterprise B/{site}/fillerproduction/{line}/{workcenter}/processdata/rate/instant
Enterprise B/{site}/fillerproduction/{line}/{workcenter}/processdata/state/code
Enterprise B/{site}/fillerproduction/{line}/{workcenter}/processdata/state/duration
Enterprise B/{site}/fillerproduction/{line}/{workcenter}/processdata/state/name
Enterprise B/{site}/fillerproduction/{line}/{workcenter}/processdata/state/type
```

### LiquidProcessing Topics (Vats and Tanks)

Replace `{line}` with mixroom01 or tankstorage01. Replace `{workcenter}` with vat01-04 or tank01-06.

```
Enterprise B/{site}/liquidprocessing/{line}/{workcenter}/metric/availability
Enterprise B/{site}/liquidprocessing/{line}/{workcenter}/metric/oee
Enterprise B/{site}/liquidprocessing/{line}/{workcenter}/metric/performance
Enterprise B/{site}/liquidprocessing/{line}/{workcenter}/metric/quality
Enterprise B/{site}/liquidprocessing/{line}/{workcenter}/metric/input/countdefect
Enterprise B/{site}/liquidprocessing/{line}/{workcenter}/metric/input/countinfeed
Enterprise B/{site}/liquidprocessing/{line}/{workcenter}/metric/input/countoutfeed
Enterprise B/{site}/liquidprocessing/{line}/{workcenter}/metric/input/rateactual
Enterprise B/{site}/liquidprocessing/{line}/{workcenter}/metric/input/ratestandard
Enterprise B/{site}/liquidprocessing/{line}/{workcenter}/metric/input/timedownplanned
Enterprise B/{site}/liquidprocessing/{line}/{workcenter}/metric/input/timedownunplanned
Enterprise B/{site}/liquidprocessing/{line}/{workcenter}/metric/input/timeidle
Enterprise B/{site}/liquidprocessing/{line}/{workcenter}/metric/input/timerunning
Enterprise B/{site}/liquidprocessing/{line}/{workcenter}/node/assetidentifier/assetid
Enterprise B/{site}/liquidprocessing/{line}/{workcenter}/node/assetidentifier/assetname
Enterprise B/{site}/liquidprocessing/{line}/{workcenter}/node/assetidentifier/assetpath
Enterprise B/{site}/liquidprocessing/{line}/{workcenter}/node/assetidentifier/assettypename
Enterprise B/{site}/liquidprocessing/{line}/{workcenter}/node/assetidentifier/displayname
Enterprise B/{site}/liquidprocessing/{line}/{workcenter}/node/assetidentifier/parentassetid
Enterprise B/{site}/liquidprocessing/{line}/{workcenter}/node/assetidentifier/sortorder
Enterprise B/{site}/liquidprocessing/{line}/{workcenter}/processdata/process/flowrate
Enterprise B/{site}/liquidprocessing/{line}/{workcenter}/processdata/process/temperature
Enterprise B/{site}/liquidprocessing/{line}/{workcenter}/processdata/process/weight
Enterprise B/{site}/liquidprocessing/{line}/{workcenter}/processdata/state/code
Enterprise B/{site}/liquidprocessing/{line}/{workcenter}/processdata/state/duration
Enterprise B/{site}/liquidprocessing/{line}/{workcenter}/processdata/state/name
Enterprise B/{site}/liquidprocessing/{line}/{workcenter}/processdata/state/type
Enterprise B/{site}/liquidprocessing/{line}/{workcenter}/workorder/assetid
Enterprise B/{site}/liquidprocessing/{line}/{workcenter}/workorder/quantityactual
Enterprise B/{site}/liquidprocessing/{line}/{workcenter}/workorder/quantitydefect
Enterprise B/{site}/liquidprocessing/{line}/{workcenter}/workorder/quantitytarget
Enterprise B/{site}/liquidprocessing/{line}/{workcenter}/workorder/uom
Enterprise B/{site}/liquidprocessing/{line}/{workcenter}/workorder/workorderid
Enterprise B/{site}/liquidprocessing/{line}/{workcenter}/workorder/workordernumber
Enterprise B/{site}/liquidprocessing/{line}/{workcenter}/workorder/lotnumber/lotnumber
Enterprise B/{site}/liquidprocessing/{line}/{workcenter}/workorder/lotnumber/lotnumberid
Enterprise B/{site}/liquidprocessing/{line}/{workcenter}/workorder/lotnumber/item/bottlesize
Enterprise B/{site}/liquidprocessing/{line}/{workcenter}/workorder/lotnumber/item/itemclass
Enterprise B/{site}/liquidprocessing/{line}/{workcenter}/workorder/lotnumber/item/itemid
Enterprise B/{site}/liquidprocessing/{line}/{workcenter}/workorder/lotnumber/item/itemname
Enterprise B/{site}/liquidprocessing/{line}/{workcenter}/workorder/lotnumber/item/labelvariant
Enterprise B/{site}/liquidprocessing/{line}/{workcenter}/workorder/lotnumber/item/packcount
Enterprise B/{site}/liquidprocessing/{line}/{workcenter}/workorder/lotnumber/item/parentitemid
```

---

*Document generated from Neo4j Graph Database. Data structure reflects the Enterprise B Unified Namespace configuration.*

*For questions or updates, contact your UNS administrator.*
