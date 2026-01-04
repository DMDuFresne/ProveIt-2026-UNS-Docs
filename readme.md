# From MQTT Broker to Complete Documentation in Hours

> **Prepared for:** [ProveIt! Conference 2026](https://www.proveitconference.com/)
> **Prepared by:** [Abelara](https://abelara.com)
> **Date:** January 2026

---

## 🎟️ Attending ProveIt! 2026?

**Visit us at Booth 19** to see the UNS Visualizer in action and discuss how Unified Namespace can transform your operations.

**See Enterprise B Solutions Live:**
- **Come see the solutions we built for Enterprise B** with our friends, the experts at **[Tupinix](https://tupinix.com/)** - Live demonstrations at Booth 19

**Save 25% on Conference Registration:**
- **Promo Code:** `ABELARA`
- **Valid through:** February 16, 2026
- **[Register with discount →](https://www.proveitconference.com/offers/GVr8MysP?coupon_code=ABELARA)**

---

## The Premise

What if the Unified Namespace, when implemented correctly, could document itself?

We set out to prove a bold claim: **A well-structured UNS contains everything you need to understand an operation**. The hierarchy, the equipment, the metrics, the relationships between assets, the context of production orders, even the language operators use on the floor.

To test this, we asked multiple manufacturing enterprises for one thing: access to their MQTT brokers.

That is all we got.

---

## Important Clarification

**This documentation represents our understanding and interpretation of the ProveIt! 2026 Enterprise MQTT Brokers (Unified Namespaces).** 

We are not making claims about the enterprises themselves, their operations, or the accuracy of the data flowing through their brokers. Everything documented here was derived solely from observing MQTT message traffic. This is our analysis of what the namespace structure and message patterns revealed to us—our understanding of how these Unified Namespaces are structured and what they appear to represent.

---

## What We Did Not Have

Let us be clear about what was NOT provided:

- No PLC programs or ladder logic
- No SCADA screen exports
- No historian database access
- No equipment manuals or P&IDs
- No existing documentation of any kind
- No site tours or operator interviews
- No database credentials or API keys

**We had only the live MQTT message stream** flowing through each enterprise's broker.

---

## What We Produced

Using only broker access and AI-powered analysis, we generated complete operational documentation for three distinct manufacturing enterprises:

| Enterprise | Industry | Sites | Equipment | Documentation |
|------------|----------|-------|-----------|---------------|
| **[Enterprise A](./Enterprise%20A/readme.md)** | Glass Container Manufacturing | 1 (Dallas) | Furnaces, IS Machines, Inspection | 1,226 lines |
| **[Enterprise B](./Enterprise%20B/readme.md)** | Beverage Bottling & Packaging | 3 sites | Fillers, Tanks, Labelers, Palletizers | 1,569 lines |
| **[Enterprise C](./Enterprise%20C/readme.md)** | Bioprocessing/Pharmaceutical | 1 facility | Bioreactor, Chromatography, TFF | 1,378 lines |

Each document includes:

- Complete ISA-95 aligned hierarchy maps
- Equipment taxonomy with capabilities matrices
- Data type specifications with real value ranges
- Quick start guides for first-time users
- Practical use case scenarios
- Quick reference cards for daily operations
- Complete topic path templates
- Domain-specific glossaries (32+ terms each)

---

## The Point We Are Proving

The Unified Namespace is more than a data integration pattern. When done right, it becomes a **semantic layer** that describes your operation.

Consider what we discovered just from message traffic:

**From Enterprise A (Glass Manufacturing):**
- A furnace running at 2,705 degrees Fahrenheit
- An 8-section IS Machine forming containers at 120 cycles per minute
- Three distinct process areas: BatchHouse, HotEnd, ColdEnd
- ISO 7459 compliance data for dimensional tolerances
- Raw thermocouple voltages alongside engineering units
- Operators tracking defects by type: CHK (cracks), SED (bubbles), DIM (dimensional)

**From Enterprise B (Beverage Bottling):**
- Three geographically distinct sites with consistent naming conventions
- 120 assets across four functional areas per site
- Vats with friendly names (Jeff, Raymond, Billy, Bob) that operators actually use
- Work orders flowing down to equipment level
- OEE calculations with all contributing inputs exposed

None of this was told to us. **This is what we understood from the namespace.**

---

## How It Works

We built a pipeline that treats MQTT messages as the source of truth:

```
                    MQTT Brokers
                         |
                         v
            +---------------------------+
            |   UNS Visualizer Tool     |
            |   (Capture & Analysis)    |
            +---------------------------+
                    |         |
                    v         v
            +----------+ +----------+
            |  Neo4j   | | QuestDB  |
            | (Graph)  | | (Series) |
            +----------+ +----------+
                    |         |
                    v         v
            +---------------------------+
            |   MCP Tool Servers        |
            |   (Expose to Claude)      |
            +---------------------------+
                         |
                         v
            +---------------------------+
            |      Claude Code          |
            |   with Expert Agents      |
            +---------------------------+
                         |
                         v
                  Documentation
```

### The Components

**[Abelara UNS Visualizer](https://abelara.com/proveit2026)** captures and analyzes MQTT traffic, building a model of the namespace structure as messages flow through the broker. [View the ProveIt! 2026 visualization →](https://abelara.com/proveit2026)

**Neo4j** stores the hierarchical relationships between assets. Enterprise contains Sites. Sites contain Areas. Areas contain Lines. Lines contain WorkCenters. The graph makes these relationships queryable.

**QuestDB** stores the time-series history. What values has this topic seen? What is the typical range? How frequently does it update?

**MCP Tool Servers** expose both data stores to Claude Code using the Model Context Protocol. The AI can query the graph for structure and the time-series for actual values.

**Expert Agents** validate the output:
- A Data Ontology Expert verifies ISA-95 alignment
- A Technical Writer ensures documentation is accessible
- Domain Experts check industry-specific terminology

---

## What Makes a UNS Self-Documenting

Not every MQTT implementation produces documentation-ready data. The enterprises we analyzed shared characteristics that made their namespaces interpretable:

### Consistent Hierarchy
The enterprises followed predictable patterns:
- Enterprise A & B: `Enterprise / Site / Area / Line / Equipment / Category / Metric`
- Enterprise C: `Enterprise / Device / Tags` (flat structure for batch processes)

This is not accidental. It reflects ISA-95/ISA-88 principles applied to topic design.

### Semantic Naming
Equipment names carry meaning:
- `Furnace`, `Forehearth`, `ISMachine` tell you about glass forming
- `Washer`, `Filler`, `CapLoader` tell you about bottling lines
- `BatchHouse`, `HotEnd`, `ColdEnd` describe process zones

### Complete Context
Work orders, lot numbers, and product identifiers travel with operational data. Quality issues can be traced to specific batches. Equipment states include reason codes.

### Exposed Relationships
Parent-child relationships between assets are explicit. Enterprise A uses path-based identity. Enterprise B includes explicit `parentassetid` fields. Either approach works as long as it is consistent.

---

## Patterns We Observed

### State Management
Enterprise A and B use a common pattern:
- `StateCurrent` or `state/code`: Numeric state identifier
- `StateReason` or `state/name`: Human-readable state description

State codes appear standardized: 1=DOWN, 2=IDLE, 3=RUNNING

Enterprise C uses a `STATE` tag with string values (Running, Idle, Complete, Holding) aligned with ISA-88 batch control.

### OEE Organization
Enterprise A aggregates OEE at the line level with a dedicated `/OEE/` path plus BigQuery integration.

Enterprise B calculates OEE at multiple levels, with metrics nested under each equipment unit.

Both approaches are valid. The documentation captures each pattern accurately.

### Naming Conventions

**Path Naming:**
- Enterprise A uses mixed case with spaces: `Enterprise A/Dallas/Line 1/HotEnd/Furnace`
- Enterprise B uses lowercase without spaces: `Enterprise B/Site1/fillerproduction/fillingline01/filler`
- Enterprise C uses Sparkplug B format: `spBv1.0/Enterprise C/NDATA/`

**Tag Naming - A Best Practice:**

Enterprise C demonstrates an especially sophisticated approach using **ISA-5.1 instrument codes** (ANSI/ISA-5.1 standard). This pattern embeds meaning directly into tag names:

| Code | Meaning | Example |
|------|---------|---------|
| **AIC** | Analytical Indicator Controller | `AIC-250-001_PV_percent` (Dissolved Oxygen) |
| **TIC** | Temperature Indicator Controller | `TIC-250-001_PV_Celsius` |
| **FIC** | Flow Indicator Controller | `FIC-250-001_PV_SLPM` |
| **PIC** | Pressure Indicator Controller | `PIC-250-001_PV_psi` |
| **SIC** | Speed Indicator Controller | `SIC-250-008_PV_RPM` |

**Why this matters:** Tag names like `AIC-250-001_DO_PV` immediately tell you:
- **What it measures** (Analytical - DO/pH/Conductivity)
- **Where it is** (Equipment 250, Loop 001)
- **What type of value** (PV = Process Variable, SP = Setpoint)
- **Units** (embedded in suffix: `_percent`, `_Celsius`, `_psi`)

This is **self-documenting at the tag level**—any engineer familiar with ISA-5.1 can understand the tag's purpose without consulting documentation. Enterprise C's approach shows how industry standards can be leveraged to create truly semantic namespaces.

These conventions emerged directly from the message traffic. We documented what exists, not what should exist.

---

## The Implications

If a UNS can produce its own documentation, several things follow:

**Documentation stays current.** Regenerate it whenever the namespace changes. No more stale documentation that drifts from reality.

**New team members onboard faster.** Query the namespace to understand the operation. The answers are in the data.

**Integration becomes discoverable.** What data is available? Where does it live? What values does it contain? Subscribe to the broker and find out.

**Auditing improves.** Compliance documentation can be generated from the same source of truth that runs production.

---

### Explore the Tools

**Abelara UNS Visualizer** - The tool used to capture and analyze the MQTT brokers in this demonstration:
- **[View the ProveIt! 2026 Broker Visualization](https://UnifiedNamespace.Info)** - Interactive visualization of the three enterprise namespaces analyzed in this documentation

**Abelara** - Coaching, Consulting, and Integration solutions:
- **[Visit Abelara.com](https://abelara.com)** - Learn more about our UNS Visualizer and industrial data integration solutions
- **Visit us at ProveIt! 2026 - Booth 19** - See the UNS Visualizer live and discuss your Unified Namespace implementation
- **Enterprise B Solutions** - Come see the solutions we built for Enterprise B with our friends, the experts at **[Tupinix](https://tupinix.com/)** - Live demonstrations at Booth 19

### Read the Generated Documentation
- [Enterprise A: Glass Container Manufacturing](./Enterprise%20A/readme.md) (1,226 lines)
- [Enterprise B: Beverage Bottling & Packaging](./Enterprise%20B/readme.md) (1,569 lines)
- [Enterprise C: Bioprocessing/Pharmaceutical](./Enterprise%20C/readme.md) (1,378 lines)

### How We Did It
- **[How We Did It: Building a Self-Documenting Unified Namespace](./Explanation/readme.md)** - Technical deep dive into the methodology, architecture, and implementation behind this documentation generation

---

## The Bottom Line

In a matter of hours, with only MQTT broker access, we produced comprehensive documentation for three manufacturing operations spanning:

- 5 sites/facilities
- 16+ areas
- 45+ lines/process units
- 89+ equipment units
- 600+ data points
- 4,173 lines of validated documentation

No interviews. No access to source systems. No existing documentation.

**From our analysis, the namespace appeared to contain everything we needed to understand these operations.**

This is the promise of the Unified Namespace: not just that data flows through a common infrastructure, but that the infrastructure itself can become self-documenting. When you structure your UNS thoughtfully, you create a system that can explain itself to anyone who analyzes it.

---

## Documentation Index

### Technical Documentation

| Document | Description |
|----------|-------------|
| [Enterprise A Documentation](./Enterprise%20A/readme.md) | Glass Container Manufacturing (Dallas) - 1,226 lines |
| [Enterprise B Documentation](./Enterprise%20B/readme.md) | Beverage Bottling & Packaging (3 Sites) - 1,569 lines |
| [Enterprise C Documentation](./Enterprise%20C/readme.md) | Bioprocessing/Pharmaceutical (rBMN-42 Production) - 1,378 lines |
| [How We Did It: Technical Deep Dive](./Explanation/readme.md) | Methodology, architecture, and implementation details behind the documentation generation |

### Executive Summaries

| Author | Scope | Perspective |
|--------|-------|-------------|
| [VP of Operations](./Enterprise%20B/Executive%20Summary/readme.md) | Enterprise B - All Sites | "Three islands operating under one P&L" |
| [Dallas Site Leader](./Enterprise%20A/Executive%20Summary/readme.md) | Enterprise A - Glass Manufacturing | "Data everywhere, visibility nowhere" |
| [Site Director](./Enterprise%20C/Executive%20Summary/readme.md) | Enterprise C - Bioprocessing | GMP Compliance & Data Integrity |
| [Marcus Chen](./Enterprise%20B/Executive%20Summary/Site%201/readme.md) | Site 1 - The Cap Shack | The Pioneer |
| [Diane Kowalski](./Enterprise%20B/Executive%20Summary/Site%202/readme.md) | Site 2 - Filler Central | Skeptic to Evangelist |
| [Kevin Okonkwo](./Enterprise%20B/Executive%20Summary/Site%203/readme.md) | Site 3 | The Optimizer |

---

<p align="center">
  <strong>Prepared by <a href="https://abelara.com">Abelara</a> for <a href="https://www.proveitconference.com/">ProveIt! 2026</a></strong><br>
  <em>Your data already tells its own story. You just need the right tools to listen.</em><br><br>
  <strong>Visit us at Booth 19</strong> | <strong>Save 25% with code: <a href="https://www.proveitconference.com/offers/GVr8MysP?coupon_code=ABELARA">ABELARA</a></strong> (Valid through Feb 16, 2026)
</p>
