# Enterprise A - Unified Namespace Documentation

> **Document Version:** 1.0
> **Last Updated:** January 3, 2026
> **Scope:** Dallas Site (Glass Container Manufacturing)

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
| Site Details | Asset-specific lookups | [Read](#dallas-site-detailed-documentation) |
| Quick Reference Cards | Cheat sheets for daily work | [Read](#quick-reference-cards) |
| Complete Asset Registry | All asset paths | [Read](#complete-asset-registry) |
| Glossary | Term definitions | [Read](#glossary) |

---

## Executive Summary

### What is the Unified Namespace?

The Unified Namespace (UNS) is a single, organized structure that contains all operational data from Enterprise A's glass container manufacturing operations. Think of it as a filing system where every piece of equipment, every sensor reading, and every production metric has a specific, predictable address.

Instead of hunting through multiple databases, HMIs, and control systems to find data, the UNS gives you one consistent way to access everything.

**Before UNS:**
- "Where do I find the furnace temperature? Is it in the DCS? The historian? That SCADA screen?"

**With UNS:**
- `Enterprise A/Dallas/Line 1/HotEnd/Furnace/Status/Temperature`

Every data point has a clear, logical path that follows your plant's physical layout and process flow.

### Who Should Use This Documentation?

| Role | What You'll Use This For |
|------|--------------------------|
| **Plant Operators** | Check furnace temperatures, monitor silo levels, verify machine speed |
| **Process Engineers** | Analyze OEE trends, investigate quality defects, optimize batch recipes |
| **Maintenance Technicians** | Monitor equipment states, check sensor readings, track edge device data |
| **Quality Engineers** | Review inspection data, track defect types (CHK, SED, DIM), verify ISO 7459 compliance |
| **IT/Integration Teams** | Build dashboards, connect to BigQuery, set up automated alerts |
| **Data Analysts** | Query historical data, build reports, identify improvement opportunities |

### What Data is Available?

The UNS provides access to four categories of data across the Dallas production line:

| Data Category | What It Contains | Example Questions It Answers |
|---------------|------------------|------------------------------|
| **Status** | Equipment states, temperatures, levels, speeds | "What's the furnace temperature?" "How full is Silo 2?" |
| **Production** | Counts, defects, cycle data | "How many containers formed today?" "What's the reject rate?" |
| **Edge** | Raw sensor data, encoder values, transmitter signals | "What's the raw thermocouple reading?" |
| **OEE/BigQuery** | Aggregated metrics, shift data | "What's Line 1's current OEE?" |

### Enterprise at a Glance

```
Enterprise A
    |
    +-- Dallas (Glass Container Manufacturing)
            |
            +-- Line 1: 8-Section IS Machine Line
                    Capacity: 7,200 containers/hour
                    Product: Food & Beverage Jars
                    Current OEE: ~97.9%
                    |
                    +-- BatchHouse (Raw Material Handling)
                    |       4 Silos, Mixer, Charger
                    |
                    +-- HotEnd (Melting & Forming)
                    |       Furnace, Forehearth, IS Machine
                    |
                    +-- ColdEnd (Annealing & Inspection)
                            Lehr, Inspector, Palletizer
```

**Total:** 1 Site | 3 Areas | 13 Equipment Units | 100+ Data Points

---

## Semantic Foundations

### ISA-95 Hierarchy Alignment

This UNS follows the ISA-95 (IEC 62264) **Role-Based Equipment Hierarchy** model adapted for continuous glass manufacturing processes. The hierarchy defines containment relationships from enterprise to equipment level.

| UNS Level | ISA-95 Role | Term | Definition |
|-----------|-------------|------|------------|
| 1 | Enterprise | **Enterprise** | The top-level organizational entity (Enterprise A) |
| 2 | Site | **Site** | A geographically distinct manufacturing facility (Dallas) |
| 3 | Area | **Line** | A complete production line from batch to pack-out (Line 1) |
| 4 | Work Center | **Area** | A functional process zone within the line (BatchHouse, HotEnd, ColdEnd) |
| 5 | Work Unit | **Equipment** | Individual machines and process units (Furnace, ISMachine, etc.) |

> **Note:** This hierarchy follows ISA-95 Part 1 Section 5 equipment role definitions, not the functional control hierarchy (Levels 0-4). The role-based model describes containment; the functional hierarchy describes control system layers.

### Hierarchy Relationships

The UNS uses a process-flow containment model reflecting how glass manufacturing works:

```
Enterprise ──contains──> Site ──contains──> Line ──contains──> Area ──contains──> Equipment
```

| Relationship | How It's Expressed | Example |
|--------------|-------------------|---------|
| Physical Location | Site path segment | Dallas |
| Process Zone | Area path segment | HotEnd (melting/forming zone) |
| Equipment Unit | Equipment path segment | Furnace, ISMachine |
| Data Category | Category path segment | Status, Production, edge |

### Glass Manufacturing Process Flow

Understanding the process flow helps you navigate the UNS:

```
Raw Materials → Batch → Melt → Condition → Form → Anneal → Inspect → Pack
    │             │        │        │         │        │         │        │
  Silos      BatchMixer  Furnace  Forehearth ISMachine  Lehr   Inspector Palletizer
    │             │        │        │         │        │         │        │
 BatchHouse  BatchHouse  HotEnd   HotEnd    HotEnd   ColdEnd  ColdEnd  ColdEnd
```

---

## Equipment Type Taxonomy

### Classification by Process Type

Enterprise A equipment falls into distinct categories based on how they handle materials:

| Process Type | Characteristics | Equipment |
|--------------|-----------------|-----------|
| **Bulk Storage** | Stores raw materials, tracks fill levels | Silo01, Silo02, Silo03, Silo04 |
| **Batch Processing** | Weighs and mixes discrete batches | BatchMixer |
| **Material Conveying** | Moves material between process steps | BatchCharger |
| **High-Temp Continuous** | Continuous thermal processing at extreme temperatures | Furnace, Forehearth |
| **Discrete Forming** | Forms individual containers from molten glass | ISMachine |
| **Continuous Conveying** | Controlled cooling/annealing process | Lehr |
| **Quality Inspection** | Vision-based defect detection | Inspector |
| **Material Handling** | Robotic palletizing of finished goods | Palletizer |

### Equipment by Functional Area

| Area | Equipment | Key Characteristics |
|------|-----------|---------------------|
| **BatchHouse** | Silo01-04, BatchMixer, BatchCharger | Material levels, weights, feed rates |
| **HotEnd** | Furnace, Forehearth, ISMachine | High temperatures (2000-2700°F), forming counts |
| **ColdEnd** | Lehr, Inspector, Palletizer | Zone temperatures, defect counts, pallet counts |

### Equipment Capabilities Matrix

| Equipment | Has State | Has Status | Has Production | Has Edge | Key Metrics |
|-----------|-----------|------------|----------------|----------|-------------|
| Silo01-04 | ✗ | ✓ | ✗ | ✓ | Level (%), Material |
| BatchMixer | ✓ | ✓ | ✗ | ✓ | BatchWeight (lbs) |
| BatchCharger | ✓ | ✓ | ✗ | ✓ | FeedRate (lbs/min) |
| Furnace | ✓ | ✓ | ✗ | ✓ | Temperature (°F), GlassLevel (%) |
| Forehearth | ✓ | ✓ | ✗ | ✓ | Temperature (°F), GobTemp (°F) |
| ISMachine | ✓ | ✓ | ✓ | ✓ | MachineSpeed (cpm), ContainersFormed |
| Lehr | ✓ | ✓ | ✗ | ✓ | BeltSpeed (fpm), ZoneTemp1-3 (°F) |
| Inspector | ✓ | ✗ | ✓ | ✓ | PassCount, RejectCount, Defects |
| Palletizer | ✓ | ✗ | ✓ | ✓ | PalletsCompleted, ContainersPerPallet |

### Raw Material Silos

| Silo | Material | Batch % | Purpose |
|------|----------|---------|---------|
| Silo01 | Silica Sand | ~70% | Primary glass former (SiO₂) |
| Silo02 | Soda Ash | ~15% | Flux to reduce melting temperature |
| Silo03 | Limestone | ~10% | Stabilizer for durability |
| Silo04 | Cullet | ~5%+ | Recycled glass for energy efficiency |

---

## Data Type Specifications

### Temperature Values

| Data Type | Description | Typical Range | Unit |
|-----------|-------------|---------------|------|
| **Furnace Temperature** | Melting zone temperature | 2650-2750 | °F |
| **Forehearth Temperature** | Conditioning temperature | 2000-2100 | °F |
| **Gob Temperature** | Glass gob temperature at shears | 2000-2080 | °F |
| **Lehr Zone Temperature** | Annealing zone temperatures | 400-1100 | °F |

### Level and Weight Values

| Data Type | Description | Range | Unit |
|-----------|-------------|-------|------|
| **Silo Level** | Raw material fill level | 0-100 | % |
| **Glass Level** | Furnace glass level | 0-100 | % |
| **Batch Weight** | Current batch weight in mixer | 0-3000 | lbs |
| **Feed Rate** | Batch charger feed rate | 0-200 | lbs/min |

### Speed and Rate Values

| Data Type | Description | Typical Value | Unit |
|-----------|-------------|---------------|------|
| **Machine Speed** | IS Machine cycles per minute | 100-140 | cpm |
| **Belt Speed** | Lehr conveyor speed | 20-30 | fpm |
| **Production Rate** | Containers per hour | 6000-8000 | containers/hr |

### Count Values

| Data Type | Description | Data Type |
|-----------|-------------|-----------|
| **ContainersFormed** | Total containers formed by IS Machine | Integer |
| **CycleCount** | IS Machine cycle count | Integer |
| **TotalInspected** | Containers inspected | Integer |
| **PassCount** | Containers passing inspection | Integer |
| **RejectCount** | Containers rejected | Integer |
| **PalletsCompleted** | Pallets completed | Integer |

### Defect Types (Glass-Specific)

| Defect Code | Name | Description |
|-------------|------|-------------|
| **DefectCHK** | Check | Cracks or stress fractures in glass |
| **DefectSED** | Seed | Bubbles or inclusions in glass |
| **DefectDIM** | Dimensional | Out-of-tolerance dimensions |

### State Information

| Field | Data Type | Description |
|-------|-----------|-------------|
| `StateCurrent` | Integer | Numeric state code (e.g., 3 = Running) |
| `StateReason` | String | State reason code (e.g., "RUN-NRM") |

### Edge (Raw Sensor) Data Types

| Sensor Type | Description | Typical Range | Unit |
|-------------|-------------|---------------|------|
| **Thermocouple** | Raw thermocouple voltage | 0-32767 | mV (scaled) |
| **Level Transmitter** | 4-20mA level signal | 4.0-20.0 | mA |
| **Encoder** | Position/speed encoder count | 0-65535 | counts |
| **Motor Status** | Motor run status | 0 or 1 | boolean |

---

## Quick Start Guide

### Your First Query in 3 Steps

**Goal:** Find the current furnace temperature

**Step 1: Start with the Enterprise**
```
Enterprise A/
```

**Step 2: Navigate the hierarchy (Site > Line > Area > Equipment)**
```
Enterprise A/Dallas/Line 1/HotEnd/Furnace/
```

**Step 3: Add the data category and metric**
```
Enterprise A/Dallas/Line 1/HotEnd/Furnace/Status/Temperature
```

That's it. This path gives you the real-time furnace temperature (currently ~2705°F).

### Understanding the Path Structure

Every UNS path follows this pattern:

```
Enterprise A / {Site} / {Line} / {Area} / {Equipment} / {Category} / {Metric}
     |           |        |        |          |            |           |
     |           |        |        |          |            |           +-- The specific value
     |           |        |        |          |            +-- Status, Production, State, edge
     |           |        |        |          +-- Furnace, ISMachine, etc.
     |           |        |        +-- BatchHouse, HotEnd, ColdEnd
     |           |        +-- Line 1
     |           +-- Dallas
     +-- Always starts here
```

> **Tip:** Path segments preserve their original case (e.g., "Line 1", "HotEnd", "ISMachine").

---

## Common Use Cases

### Scenario 1: Operator Checking Furnace Conditions

**Situation:** You need to verify the furnace is operating within normal parameters.

**Paths to query:**

| Metric | Path |
|--------|------|
| Furnace Temperature | `Enterprise A/Dallas/Line 1/HotEnd/Furnace/Status/Temperature` |
| Glass Level | `Enterprise A/Dallas/Line 1/HotEnd/Furnace/Status/GlassLevel` |
| Furnace State | `Enterprise A/Dallas/Line 1/HotEnd/Furnace/State/StateCurrent` |
| State Reason | `Enterprise A/Dallas/Line 1/HotEnd/Furnace/State/StateReason` |

**Normal operating values:**
- Temperature: 2650-2750°F (target: ~2705°F)
- Glass Level: 80-90% (target: ~85%)
- StateCurrent: 3 (Running)
- StateReason: "RUN-NRM" (Running Normal)

---

### Scenario 2: Process Engineer Investigating Quality Issues

**Situation:** You're seeing higher than normal reject rates and need to identify the defect types.

**Step 1: Check inspection totals**

| Metric | Path |
|--------|------|
| Total Inspected | `Enterprise A/Dallas/Line 1/ColdEnd/Inspector/Production/TotalInspected` |
| Pass Count | `Enterprise A/Dallas/Line 1/ColdEnd/Inspector/Production/PassCount` |
| Reject Count | `Enterprise A/Dallas/Line 1/ColdEnd/Inspector/Production/RejectCount` |

**Step 2: Break down by defect type**

| Defect Type | Path | What It Means |
|-------------|------|---------------|
| Check Defects | `.../Inspector/Production/DefectCHK` | Cracks - may indicate thermal stress |
| Seed Defects | `.../Inspector/Production/DefectSED` | Bubbles - may indicate batch or melt issues |
| Dimensional Defects | `.../Inspector/Production/DefectDIM` | Size issues - may indicate IS Machine timing |

**Step 3: Correlate with process conditions**

If seeing high **Seed defects**, check:
- `Enterprise A/Dallas/Line 1/HotEnd/Furnace/Status/Temperature` (too cold = poor refining)
- `Enterprise A/Dallas/Line 1/BatchHouse/BatchMixer/Status/BatchWeight` (batch consistency)

If seeing high **Check defects**, check:
- `Enterprise A/Dallas/Line 1/ColdEnd/Lehr/Status/ZoneTemp1` (annealing temperature)
- `Enterprise A/Dallas/Line 1/ColdEnd/Lehr/Status/BeltSpeed` (cooling rate)

If seeing high **Dimensional defects**, check:
- `Enterprise A/Dallas/Line 1/HotEnd/ISMachine/Status/MachineSpeed` (timing)
- `Enterprise A/Dallas/Line 1/HotEnd/Forehearth/Status/GobTemp` (glass viscosity)

---

### Scenario 3: Building a Furnace Temperature Dashboard

**Situation:** Operations wants a dashboard showing all temperatures across the hot end process.

**Temperature data points:**

| Location | Path | Typical Value |
|----------|------|---------------|
| Furnace | `Enterprise A/Dallas/Line 1/HotEnd/Furnace/Status/Temperature` | 2705°F |
| Forehearth | `Enterprise A/Dallas/Line 1/HotEnd/Forehearth/Status/Temperature` | 2050°F |
| Gob Temperature | `Enterprise A/Dallas/Line 1/HotEnd/Forehearth/Status/GobTemp` | 2040°F |
| Lehr Zone 1 | `Enterprise A/Dallas/Line 1/ColdEnd/Lehr/Status/ZoneTemp1` | 1050°F |
| Lehr Zone 2 | `Enterprise A/Dallas/Line 1/ColdEnd/Lehr/Status/ZoneTemp2` | 750°F |
| Lehr Zone 3 | `Enterprise A/Dallas/Line 1/ColdEnd/Lehr/Status/ZoneTemp3` | 400°F |

**For raw sensor data (engineering units):**

| Sensor | Path |
|--------|------|
| Furnace Zone 1 TC | `Enterprise A/Dallas/Line 1/HotEnd/Furnace/edge/thermocouple_zone1_mv` |
| Furnace Zone 2 TC | `Enterprise A/Dallas/Line 1/HotEnd/Furnace/edge/thermocouple_zone2_mv` |
| Lehr Zone 1 TC | `Enterprise A/Dallas/Line 1/ColdEnd/Lehr/edge/zone1_thermocouple_mv` |

---

### Scenario 4: Checking Raw Material Levels

**Situation:** You need to verify silo levels before a shift change.

**Silo level paths:**

| Silo | Material | Level Path |
|------|----------|------------|
| Silo01 | Silica Sand | `Enterprise A/Dallas/Line 1/BatchHouse/Silo01/Status/Level` |
| Silo02 | Soda Ash | `Enterprise A/Dallas/Line 1/BatchHouse/Silo02/Status/Level` |
| Silo03 | Limestone | `Enterprise A/Dallas/Line 1/BatchHouse/Silo03/Status/Level` |
| Silo04 | Cullet | `Enterprise A/Dallas/Line 1/BatchHouse/Silo04/Status/Level` |

**Also available from BigQuery aggregated data:**

| Metric | Path |
|--------|------|
| Silica Sand Level | `Enterprise A/Dallas/Line 1/BigQuery/material_usage.silica_sand_level_percent` |
| Soda Ash Level | `Enterprise A/Dallas/Line 1/BigQuery/material_usage.soda_ash_level_percent` |
| Limestone Level | `Enterprise A/Dallas/Line 1/BigQuery/material_usage.limestone_level_percent` |
| Cullet Level | `Enterprise A/Dallas/Line 1/BigQuery/material_usage.cullet_level_percent` |

---

### Scenario 5: Reviewing Line OEE Performance

**Situation:** Management wants to know current line performance.

**OEE data from BigQuery:**

| Metric | Path | Current Value |
|--------|------|---------------|
| OEE | `Enterprise A/Dallas/Line 1/BigQuery/oee_metrics.oee_percent` | 97.9% |
| Availability | `Enterprise A/Dallas/Line 1/BigQuery/oee_metrics.availability_percent` | 98.5% |
| Performance | `Enterprise A/Dallas/Line 1/BigQuery/oee_metrics.performance_percent` | 101.7% |
| Quality | `Enterprise A/Dallas/Line 1/BigQuery/oee_metrics.quality_percent` | 97.9% |
| Yield Efficiency | `Enterprise A/Dallas/Line 1/BigQuery/oee_metrics.yield_efficiency_percent` | 96.7% |

**Production metrics:**

| Metric | Path |
|--------|------|
| Infeed Count | `Enterprise A/Dallas/Line 1/BigQuery/production_metrics.infeed_count` |
| Outfeed Count | `Enterprise A/Dallas/Line 1/BigQuery/production_metrics.outfeed_count` |
| Good Count | `Enterprise A/Dallas/Line 1/BigQuery/production_metrics.good_count` |
| Bad Count | `Enterprise A/Dallas/Line 1/BigQuery/production_metrics.bad_count` |
| Production Rate | `Enterprise A/Dallas/Line 1/BigQuery/production_metrics.production_rate_actual` |

---

### Scenario 6: Verifying ISO 7459 Container Compliance

**Situation:** Quality needs to verify containers meet ISO 7459 dimensional and capacity tolerances.

**ISO 7459 compliance data paths:**

| Metric Category | Path | What It Shows |
|-----------------|------|---------------|
| Capacity Status | `Enterprise A/Dallas/Line 1/ISO7459/capacity_tolerance.status` | PASS/FAIL |
| Nominal Capacity | `Enterprise A/Dallas/Line 1/ISO7459/capacity_tolerance.nominal_ml` | Target capacity (mL) |
| Measured Capacity | `Enterprise A/Dallas/Line 1/ISO7459/capacity_tolerance.measured_actual_ml` | Actual capacity |
| Tolerance % | `Enterprise A/Dallas/Line 1/ISO7459/capacity_tolerance.tolerance_percent` | Allowed deviation |

**Dimensional tolerances:**

| Dimension | Path |
|-----------|------|
| Body Diameter (actual) | `Enterprise A/Dallas/Line 1/ISO7459/dimensional_tolerances.body_diameter.measured_actual_mm` |
| Body Diameter (lower limit) | `Enterprise A/Dallas/Line 1/ISO7459/dimensional_tolerances.body_diameter.lower_limit_mm` |
| Height (actual) | `Enterprise A/Dallas/Line 1/ISO7459/dimensional_tolerances.height.measured_actual_mm` |
| Finish Diameter | `Enterprise A/Dallas/Line 1/ISO7459/dimensional_tolerances.finish_diameter.measured_actual_mm` |

**Weight tolerances:**

| Metric | Path |
|--------|------|
| Nominal Weight | `Enterprise A/Dallas/Line 1/ISO7459/weight_tolerance.nominal_g` |
| Measured Weight | `Enterprise A/Dallas/Line 1/ISO7459/weight_tolerance.measured_actual_g` |
| Weight Status | `Enterprise A/Dallas/Line 1/ISO7459/weight_tolerance.status` |

**Use this data to:**
- Generate compliance certificates for customers
- Identify trending dimensional issues before they cause rejects
- Correlate dimensional failures with IS Machine or mold conditions

---

## Quick Reference Cards

### Card 1: Hot End Process Quick Reference

```
+------------------------------------------------------------------+
|  HOT END PROCESS                                                  |
+------------------------------------------------------------------+
|                                                                  |
|  FURNACE - Path: .../HotEnd/Furnace/                             |
|  -------------------------------------------------------         |
|  Status/Temperature        Melt temperature (~2705°F)            |
|  Status/GlassLevel         Glass level (target: 85%)             |
|  State/StateCurrent        Equipment state code                  |
|  State/StateReason         State reason (e.g., RUN-NRM)          |
|                                                                  |
|  FOREHEARTH - Path: .../HotEnd/Forehearth/                       |
|  -------------------------------------------------------         |
|  Status/Temperature        Conditioning temp (~2050°F)           |
|  Status/GobTemp            Gob temperature (~2040°F)             |
|                                                                  |
|  IS MACHINE - Path: .../HotEnd/ISMachine/                        |
|  -------------------------------------------------------         |
|  Status/MachineSpeed       Cycles per minute (120 cpm)           |
|  Status/SectionsActive     Active sections (8)                   |
|  Production/ContainersFormed  Total containers formed            |
|  Production/CycleCount     Machine cycle count                   |
|                                                                  |
+------------------------------------------------------------------+
```

### Card 2: Quality Inspection Quick Reference

```
+------------------------------------------------------------------+
|  QUALITY INSPECTION - Path: .../ColdEnd/Inspector/               |
+------------------------------------------------------------------+
|                                                                  |
|  PRODUCTION COUNTS                                               |
|  -------------------------------------------------------         |
|  Production/TotalInspected    All containers inspected           |
|  Production/PassCount         Containers passed                  |
|  Production/RejectCount       Containers rejected                |
|                                                                  |
|  DEFECT BREAKDOWN                                                |
|  -------------------------------------------------------         |
|  Production/DefectCHK         Check defects (cracks)             |
|  Production/DefectSED         Seed defects (bubbles)             |
|  Production/DefectDIM         Dimensional defects                |
|                                                                  |
|  EDGE SENSORS                                                    |
|  -------------------------------------------------------         |
|  edge/camera_trigger_count    Camera triggers                    |
|  edge/vision_system_ready     Vision system status               |
|  edge/reject_gate_status      Reject gate position               |
|  edge/photoelectric_sensor    Container presence                 |
|                                                                  |
+------------------------------------------------------------------+
```

### Card 3: Raw Material Quick Reference

```
+------------------------------------------------------------------+
|  BATCH HOUSE - Raw Material Storage and Handling                 |
+------------------------------------------------------------------+
|                                                                  |
|  SILOS - Path: .../BatchHouse/Silo0X/                            |
|  -------------------------------------------------------         |
|  Silo01 = Silica Sand (70%)    Status/Level, Status/Material    |
|  Silo02 = Soda Ash (15%)       Status/Level, Status/Material    |
|  Silo03 = Limestone (10%)      Status/Level, Status/Material    |
|  Silo04 = Cullet (5%+)         Status/Level, Status/Material    |
|                                                                  |
|  BATCH MIXER - Path: .../BatchHouse/BatchMixer/                  |
|  -------------------------------------------------------         |
|  Status/BatchWeight            Current batch weight (lbs)        |
|  State/StateCurrent            Equipment state                   |
|  edge/load_cell_raw            Raw load cell reading             |
|                                                                  |
|  BATCH CHARGER - Path: .../BatchHouse/BatchCharger/              |
|  -------------------------------------------------------         |
|  Status/FeedRate               Feed rate (lbs/min)               |
|  State/StateCurrent            Equipment state                   |
|  edge/belt_speed_encoder       Belt encoder count                |
|                                                                  |
+------------------------------------------------------------------+
```

### Card 4: BigQuery Aggregated Data Quick Reference

```
+------------------------------------------------------------------+
|  BIGQUERY AGGREGATED DATA - Path: .../Line 1/BigQuery/           |
+------------------------------------------------------------------+
|                                                                  |
|  OEE METRICS                                                     |
|  -------------------------------------------------------         |
|  oee_metrics.oee_percent           Overall OEE                   |
|  oee_metrics.availability_percent  Availability                  |
|  oee_metrics.performance_percent   Performance                   |
|  oee_metrics.quality_percent       Quality                       |
|  oee_metrics.yield_efficiency_percent  Yield efficiency          |
|                                                                  |
|  PRODUCTION METRICS                                              |
|  -------------------------------------------------------         |
|  production_metrics.infeed_count   Containers in                 |
|  production_metrics.outfeed_count  Containers out                |
|  production_metrics.good_count     Good containers               |
|  production_metrics.bad_count      Rejected containers           |
|  production_metrics.production_rate_actual  Current rate         |
|                                                                  |
|  CONTEXT                                                         |
|  -------------------------------------------------------         |
|  shift                  Current shift name                       |
|  work_order             Active work order                        |
|  product_id             Product being manufactured               |
|                                                                  |
+------------------------------------------------------------------+
```

---

## Hierarchy Structure

The following diagram shows the complete organizational structure of Enterprise A.

```
ENTERPRISE A
|
+-- DALLAS (Site)
    |
    +-- LINE 1: Glass Container Manufacturing
        |
        +-- BatchHouse (Raw Material Handling)
        |   |
        |   +-- Silo01 (Silica Sand)
        |   |   +-- Status/Level, Status/Material
        |   |   +-- edge/level_transmitter_ma, edge/raw_level_sensor
        |   |
        |   +-- Silo02 (Soda Ash)
        |   |   +-- Status/Level, Status/Material
        |   |   +-- edge/level_transmitter_ma, edge/raw_level_sensor
        |   |
        |   +-- Silo03 (Limestone)
        |   |   +-- Status/Level, Status/Material
        |   |   +-- edge/level_transmitter_ma, edge/raw_level_sensor
        |   |
        |   +-- Silo04 (Cullet)
        |   |   +-- Status/Level, Status/Material
        |   |   +-- edge/level_transmitter_ma, edge/raw_level_sensor
        |   |
        |   +-- BatchMixer
        |   |   +-- Description
        |   |   +-- State/StateCurrent, State/StateReason
        |   |   +-- Status/BatchWeight
        |   |   +-- edge/load_cell_raw, edge/raw_motor_current, edge/raw_motor_status
        |   |
        |   +-- BatchCharger
        |       +-- Description
        |       +-- State/StateCurrent, State/StateReason
        |       +-- Status/FeedRate
        |       +-- edge/belt_speed_encoder, edge/feeder_vfd_output, edge/raw_motor_status
        |
        +-- HotEnd (Melting & Forming)
        |   |
        |   +-- Furnace
        |   |   +-- Description
        |   |   +-- State/StateCurrent, State/StateReason
        |   |   +-- Status/Temperature, Status/GlassLevel
        |   |   +-- edge/thermocouple_zone1_mv, edge/thermocouple_zone2_mv
        |   |   +-- edge/glass_level_radar_mm, edge/oxygen_sensor_ma
        |   |
        |   +-- Forehearth
        |   |   +-- Description
        |   |   +-- State/StateCurrent, State/StateReason
        |   |   +-- Status/Temperature, Status/GobTemp
        |   |   +-- edge/gob_pyrometer_raw, edge/thermocouple_inlet_mv, edge/thermocouple_outlet_mv
        |   |
        |   +-- ISMachine
        |       +-- Description
        |       +-- State/StateCurrent, State/StateReason
        |       +-- Status/MachineSpeed, Status/SectionsActive
        |       +-- Production/ContainersFormed, Production/CycleCount
        |       +-- edge/main_drive_encoder, edge/hydraulic_pressure_raw
        |       +-- edge/section_status_bits, edge/timing_sync_pulse
        |
        +-- ColdEnd (Annealing & Inspection)
        |   |
        |   +-- Lehr
        |   |   +-- Description
        |   |   +-- State/StateCurrent, State/StateReason
        |   |   +-- Status/BeltSpeed, Status/ZoneTemp1, Status/ZoneTemp2, Status/ZoneTemp3
        |   |   +-- edge/belt_drive_encoder, edge/belt_tension_raw
        |   |   +-- edge/zone1_thermocouple_mv, edge/zone2_thermocouple_mv, edge/zone3_thermocouple_mv
        |   |
        |   +-- Inspector
        |   |   +-- Description
        |   |   +-- State/StateCurrent, State/StateReason
        |   |   +-- Production/TotalInspected, Production/PassCount, Production/RejectCount
        |   |   +-- Production/DefectCHK, Production/DefectSED, Production/DefectDIM
        |   |   +-- edge/camera_trigger_count, edge/photoelectric_sensor
        |   |   +-- edge/reject_gate_status, edge/vision_system_ready
        |   |
        |   +-- Palletizer
        |       +-- Description
        |       +-- State/StateCurrent, State/StateReason
        |       +-- Production/PalletsCompleted, Production/ContainersPerPallet
        |       +-- edge/gripper_vacuum_psi, edge/pallet_present_sensor
        |       +-- edge/robot_position_x, edge/robot_position_y
        |
        +-- BigQuery (Aggregated Line Data)
        |   +-- oee_metrics.* (availability, performance, quality, oee, yield)
        |   +-- production_metrics.* (counts, rates, runtime)
        |   +-- quality_metrics.* (inspected, pass, reject, defects)
        |   +-- equipment_status.* (temperatures, speeds)
        |   +-- material_usage.* (batch weight, silo levels)
        |   +-- shift, work_order, product_id, timestamp
        |
        +-- OEE (Line-Level OEE Data)
        |   +-- Availability, Performance, Quality
        |   +-- GoodCount, BadCount, Waste
        |   +-- Infeed, Outfeed, RunTime
        |   +-- StandardRate, State, WorkOrder
        |
        +-- ISO7459 (Glass Container Standards)
        |   +-- capacity_tolerance.* (nominal_ml, limits, measured, status)
        |   +-- dimensional_tolerances.* (body_diameter, finish_height, overall_height, wall_thickness)
        |   +-- inspection_data.* (cpk_value, sample_size, samples_passed/failed, compliance)
        |   +-- mechanical_properties.* (internal_pressure, thermal_shock)
        |   +-- product_specification.* (product_id, material, finish_type)
        |
        +-- Site (Dallas Site Information)
            +-- CurrentShift, DateTime, LocalTime
            +-- Description, OperatingSchedule, ProcessFlow
```

---

## Topic Structure Reference

### How Topics Are Organized

Every equipment unit has data organized into categories:

```
{equipment path}/
    |
    +-- Description              <- What this equipment does
    |
    +-- State/                   <- Current operating state
    |       StateCurrent             Numeric state code
    |       StateReason              State reason text
    |
    +-- Status/                  <- Real-time process values
    |       {metric}                 Equipment-specific metrics
    |
    +-- Production/              <- Production counts (where applicable)
    |       {counter}                Count values
    |
    +-- edge/                    <- Raw sensor data
            {sensor}                 Raw sensor values
```

### Data Categories by Equipment Type

**Storage Equipment (Silos):**
- `Status/Level` - Fill level percentage
- `Status/Material` - Material name
- `edge/level_transmitter_ma` - Raw 4-20mA signal
- `edge/raw_level_sensor` - Raw sensor counts

**Thermal Processing (Furnace, Forehearth, Lehr):**
- `Status/Temperature` - Process temperature
- `Status/ZoneTemp1-3` - Zone temperatures (Lehr)
- `State/StateCurrent` - Equipment state
- `edge/thermocouple_*_mv` - Raw thermocouple voltages

**Forming Equipment (ISMachine):**
- `Status/MachineSpeed` - Cycles per minute
- `Status/SectionsActive` - Active sections
- `Production/ContainersFormed` - Total formed
- `Production/CycleCount` - Cycle count

**Inspection Equipment (Inspector):**
- `Production/TotalInspected` - Total inspected
- `Production/PassCount` - Passed inspection
- `Production/RejectCount` - Failed inspection
- `Production/DefectCHK` - Check defects
- `Production/DefectSED` - Seed defects
- `Production/DefectDIM` - Dimensional defects

**Material Handling (Palletizer):**
- `Production/PalletsCompleted` - Pallets finished
- `Production/ContainersPerPallet` - Containers per pallet
- `edge/robot_position_x` - Robot X position
- `edge/robot_position_y` - Robot Y position

---

## Dallas Site Detailed Documentation

### Site Overview

| Property | Value |
|----------|-------|
| **Site Name** | Dallas |
| **Industry** | Glass Container Manufacturing |
| **Product Type** | Food & Beverage Jars |
| **Current Product** | PROD-JAR-500ML-CLEAR |
| **UNS Path** | `Enterprise A/Dallas` |

**Scale:** 1 Line | 3 Areas | 13 Equipment Units | 100+ Data Points

### Line 1 Overview

| Property | Value |
|----------|-------|
| **Line Type** | Continuous Glass Manufacturing |
| **IS Machine** | 8-Section Individual Section Machine |
| **Rated Capacity** | 7,200 containers/hour |
| **Machine Speed** | 120 cycles/minute |
| **Current OEE** | 97.9% |
| **UNS Path** | `Enterprise A/Dallas/Line 1` |

### Area: BatchHouse

**Path:** `Enterprise A/Dallas/Line 1/BatchHouse`

Raw material storage, weighing, mixing, and charging to furnace.

| Equipment | Description | Key Metrics |
|-----------|-------------|-------------|
| Silo01 | Silica Sand storage (~70% of batch) | Level: 75% |
| Silo02 | Soda Ash storage (~15% of batch) | Level: 68.5% |
| Silo03 | Limestone storage (~10% of batch) | Level: 82% |
| Silo04 | Cullet storage (~5%+ of batch) | Level: 45% |
| BatchMixer | Blends raw materials into batch | BatchWeight: 2500 lbs |
| BatchCharger | Conveys batch to furnace | FeedRate: 150 lbs/min |

### Area: HotEnd

**Path:** `Enterprise A/Dallas/Line 1/HotEnd`

High-temperature melting, conditioning, and container forming.

| Equipment | Description | Key Metrics |
|-----------|-------------|-------------|
| Furnace | Regenerative furnace, melts batch at ~2700°F | Temperature: 2705°F, GlassLevel: 85% |
| Forehearth | Conditions glass to forming temperature | Temperature: 2050°F, GobTemp: 2040°F |
| ISMachine | 8-section IS machine, forms containers | MachineSpeed: 120 cpm, SectionsActive: 8 |

### Area: ColdEnd

**Path:** `Enterprise A/Dallas/Line 1/ColdEnd`

Annealing, quality inspection, and palletizing.

| Equipment | Description | Key Metrics |
|-----------|-------------|-------------|
| Lehr | Annealing oven with 3 temperature zones | BeltSpeed: 24 fpm, Zones: 1050/750/400°F |
| Inspector | Vision inspection system | PassCount: 116,000, RejectCount: 2,500 |
| Palletizer | Robotic palletizer | PalletsCompleted: 48, ContainersPerPallet: 2,400 |

---

## Complete Asset Registry

### All Equipment Paths

| Area | Equipment | Full UNS Path |
|------|-----------|---------------|
| BatchHouse | Silo01 | `Enterprise A/Dallas/Line 1/BatchHouse/Silo01` |
| BatchHouse | Silo02 | `Enterprise A/Dallas/Line 1/BatchHouse/Silo02` |
| BatchHouse | Silo03 | `Enterprise A/Dallas/Line 1/BatchHouse/Silo03` |
| BatchHouse | Silo04 | `Enterprise A/Dallas/Line 1/BatchHouse/Silo04` |
| BatchHouse | BatchMixer | `Enterprise A/Dallas/Line 1/BatchHouse/BatchMixer` |
| BatchHouse | BatchCharger | `Enterprise A/Dallas/Line 1/BatchHouse/BatchCharger` |
| HotEnd | Furnace | `Enterprise A/Dallas/Line 1/HotEnd/Furnace` |
| HotEnd | Forehearth | `Enterprise A/Dallas/Line 1/HotEnd/Forehearth` |
| HotEnd | ISMachine | `Enterprise A/Dallas/Line 1/HotEnd/ISMachine` |
| ColdEnd | Lehr | `Enterprise A/Dallas/Line 1/ColdEnd/Lehr` |
| ColdEnd | Inspector | `Enterprise A/Dallas/Line 1/ColdEnd/Inspector` |
| ColdEnd | Palletizer | `Enterprise A/Dallas/Line 1/ColdEnd/Palletizer` |
| - | BigQuery | `Enterprise A/Dallas/Line 1/BigQuery` |
| - | OEE | `Enterprise A/Dallas/Line 1/OEE` |
| - | ISO7459 | `Enterprise A/Dallas/Line 1/ISO7459` |
| - | Site | `Enterprise A/Dallas/Site` |

---

## Glossary

| Term | Definition |
|------|------------|
| **Annealing** | Controlled cooling process to relieve thermal stress in formed glass containers |
| **Batch** | The mixture of raw materials (silica, soda ash, limestone, cullet) charged to the furnace |
| **BatchHouse** | Area where raw materials are stored, weighed, mixed, and charged to the furnace |
| **Blank Mold** | First-stage mold in blow-and-blow process that forms the parison |
| **Blow-and-Blow** | IS Machine forming process for narrow-neck containers (bottles) |
| **Check (CHK)** | Glass defect: cracks or stress fractures in the container wall |
| **ColdEnd** | The area of the plant after forming, including annealing, inspection, and packaging |
| **Crown** | Top of the furnace melting chamber |
| **Cullet** | Recycled broken glass added to the batch to improve melt efficiency |
| **Dimensional (DIM)** | Glass defect: container dimensions outside specified tolerances |
| **Finish** | The top portion of the container including the thread/rim area |
| **Forehearth** | Channel that conditions molten glass from furnace temperature to forming temperature |
| **Furnace** | High-temperature vessel that melts raw batch into molten glass (~2700°F) |
| **Glass Level** | Height of molten glass in the furnace, measured as percentage |
| **Gob** | A precise amount of molten glass cut from the forehearth and dropped into the IS Machine |
| **HotEnd** | The high-temperature area of the plant including furnace, forehearth, and forming |
| **IS Machine** | Individual Section machine that forms glass containers using the blow-and-blow or press-and-blow process |
| **ISO 7459** | International standard specifying dimensional tolerances for glass containers |
| **Job Change** | Process of switching the IS Machine to a different container design |
| **Lehr** | Annealing oven that slowly cools containers through multiple temperature zones |
| **Mold** | Cavity that shapes molten glass in the IS Machine |
| **OEE** | Overall Equipment Effectiveness: Availability × Performance × Quality |
| **Parison** | Intermediate form of glass after first blow, before final shaping |
| **Press-and-Blow** | IS Machine forming process for wide-mouth containers (jars) |
| **Pull Rate** | Tonnage of glass pulled from the furnace per day |
| **Refining** | Process of removing bubbles from molten glass in the furnace |
| **Regenerator** | Heat recovery system in regenerative furnace that preheats combustion air |
| **Section** | One of the multiple parallel forming units in an IS Machine |
| **Seed (SED)** | Glass defect: bubbles or gaseous inclusions in the glass |
| **Swabbing** | Applying lubricant to IS Machine molds between cycles |
| **Thermocouple** | Temperature sensor used to measure furnace, forehearth, and lehr temperatures |

---

## Troubleshooting Common Issues

### "I can't find data for my equipment"

1. **Check the path structure:** Enterprise A uses spaces and mixed case
   - Correct: `Enterprise A/Dallas/Line 1/HotEnd/Furnace`
   - Wrong: `EnterpriseA/dallas/line1/hotend/furnace`

2. **Verify the area name:**
   - BatchHouse (not "BatchArea" or "RawMaterials")
   - HotEnd (not "Melting" or "Forming")
   - ColdEnd (not "Inspection" or "Packaging")

3. **Check the data category:**
   - Status/ for process values (Temperature, Level, Speed)
   - Production/ for counts (ContainersFormed, RejectCount)
   - State/ for equipment state (StateCurrent, StateReason)
   - edge/ for raw sensor data

### "The temperature value seems wrong"

- All temperatures are in °F (Fahrenheit), not Celsius
- Furnace: ~2700°F (normal)
- Forehearth: ~2050°F (normal)
- Lehr Zone 1: ~1050°F (normal)

### "Where do I find OEE data?"

OEE and aggregated metrics are in the BigQuery path:
```
Enterprise A/Dallas/Line 1/BigQuery/oee_metrics.oee_percent
```

### "What do the state codes mean?"

| StateCurrent | StateReason | Meaning |
|--------------|-------------|---------|
| 3 | RUN-NRM | Running Normal |
| 2 | IDLE | Equipment Idle |
| 1 | DOWN | Equipment Down |

---

## Appendix: Complete Topic Path Templates

### BatchHouse Equipment Topics

```
Enterprise A/Dallas/Line 1/BatchHouse/Silo01/Description
Enterprise A/Dallas/Line 1/BatchHouse/Silo01/Status/Level
Enterprise A/Dallas/Line 1/BatchHouse/Silo01/Status/Material
Enterprise A/Dallas/Line 1/BatchHouse/Silo01/edge/level_transmitter_ma
Enterprise A/Dallas/Line 1/BatchHouse/Silo01/edge/raw_level_sensor

Enterprise A/Dallas/Line 1/BatchHouse/Silo02/Description
Enterprise A/Dallas/Line 1/BatchHouse/Silo02/Status/Level
Enterprise A/Dallas/Line 1/BatchHouse/Silo02/Status/Material
Enterprise A/Dallas/Line 1/BatchHouse/Silo02/edge/level_transmitter_ma
Enterprise A/Dallas/Line 1/BatchHouse/Silo02/edge/raw_level_sensor

Enterprise A/Dallas/Line 1/BatchHouse/Silo03/Description
Enterprise A/Dallas/Line 1/BatchHouse/Silo03/Status/Level
Enterprise A/Dallas/Line 1/BatchHouse/Silo03/Status/Material
Enterprise A/Dallas/Line 1/BatchHouse/Silo03/edge/level_transmitter_ma
Enterprise A/Dallas/Line 1/BatchHouse/Silo03/edge/raw_level_sensor

Enterprise A/Dallas/Line 1/BatchHouse/Silo04/Description
Enterprise A/Dallas/Line 1/BatchHouse/Silo04/Status/Level
Enterprise A/Dallas/Line 1/BatchHouse/Silo04/Status/Material
Enterprise A/Dallas/Line 1/BatchHouse/Silo04/edge/level_transmitter_ma
Enterprise A/Dallas/Line 1/BatchHouse/Silo04/edge/raw_level_sensor

Enterprise A/Dallas/Line 1/BatchHouse/BatchMixer/Description
Enterprise A/Dallas/Line 1/BatchHouse/BatchMixer/State/StateCurrent
Enterprise A/Dallas/Line 1/BatchHouse/BatchMixer/State/StateReason
Enterprise A/Dallas/Line 1/BatchHouse/BatchMixer/Status/BatchWeight
Enterprise A/Dallas/Line 1/BatchHouse/BatchMixer/edge/load_cell_raw
Enterprise A/Dallas/Line 1/BatchHouse/BatchMixer/edge/raw_motor_current
Enterprise A/Dallas/Line 1/BatchHouse/BatchMixer/edge/raw_motor_status

Enterprise A/Dallas/Line 1/BatchHouse/BatchCharger/Description
Enterprise A/Dallas/Line 1/BatchHouse/BatchCharger/State/StateCurrent
Enterprise A/Dallas/Line 1/BatchHouse/BatchCharger/State/StateReason
Enterprise A/Dallas/Line 1/BatchHouse/BatchCharger/Status/FeedRate
Enterprise A/Dallas/Line 1/BatchHouse/BatchCharger/edge/belt_speed_encoder
Enterprise A/Dallas/Line 1/BatchHouse/BatchCharger/edge/feeder_vfd_output
Enterprise A/Dallas/Line 1/BatchHouse/BatchCharger/edge/raw_motor_status
```

### HotEnd Equipment Topics

```
Enterprise A/Dallas/Line 1/HotEnd/Furnace/Description
Enterprise A/Dallas/Line 1/HotEnd/Furnace/State/StateCurrent
Enterprise A/Dallas/Line 1/HotEnd/Furnace/State/StateReason
Enterprise A/Dallas/Line 1/HotEnd/Furnace/Status/Temperature
Enterprise A/Dallas/Line 1/HotEnd/Furnace/Status/GlassLevel
Enterprise A/Dallas/Line 1/HotEnd/Furnace/edge/thermocouple_zone1_mv
Enterprise A/Dallas/Line 1/HotEnd/Furnace/edge/thermocouple_zone2_mv
Enterprise A/Dallas/Line 1/HotEnd/Furnace/edge/glass_level_radar_mm
Enterprise A/Dallas/Line 1/HotEnd/Furnace/edge/oxygen_sensor_ma

Enterprise A/Dallas/Line 1/HotEnd/Forehearth/Description
Enterprise A/Dallas/Line 1/HotEnd/Forehearth/State/StateCurrent
Enterprise A/Dallas/Line 1/HotEnd/Forehearth/State/StateReason
Enterprise A/Dallas/Line 1/HotEnd/Forehearth/Status/Temperature
Enterprise A/Dallas/Line 1/HotEnd/Forehearth/Status/GobTemp
Enterprise A/Dallas/Line 1/HotEnd/Forehearth/edge/gob_pyrometer_raw
Enterprise A/Dallas/Line 1/HotEnd/Forehearth/edge/thermocouple_inlet_mv
Enterprise A/Dallas/Line 1/HotEnd/Forehearth/edge/thermocouple_outlet_mv

Enterprise A/Dallas/Line 1/HotEnd/ISMachine/Description
Enterprise A/Dallas/Line 1/HotEnd/ISMachine/State/StateCurrent
Enterprise A/Dallas/Line 1/HotEnd/ISMachine/State/StateReason
Enterprise A/Dallas/Line 1/HotEnd/ISMachine/Status/MachineSpeed
Enterprise A/Dallas/Line 1/HotEnd/ISMachine/Status/SectionsActive
Enterprise A/Dallas/Line 1/HotEnd/ISMachine/Production/ContainersFormed
Enterprise A/Dallas/Line 1/HotEnd/ISMachine/Production/CycleCount
Enterprise A/Dallas/Line 1/HotEnd/ISMachine/edge/main_drive_encoder
Enterprise A/Dallas/Line 1/HotEnd/ISMachine/edge/hydraulic_pressure_raw
Enterprise A/Dallas/Line 1/HotEnd/ISMachine/edge/section_status_bits
Enterprise A/Dallas/Line 1/HotEnd/ISMachine/edge/timing_sync_pulse
```

### ColdEnd Equipment Topics

```
Enterprise A/Dallas/Line 1/ColdEnd/Lehr/Description
Enterprise A/Dallas/Line 1/ColdEnd/Lehr/State/StateCurrent
Enterprise A/Dallas/Line 1/ColdEnd/Lehr/State/StateReason
Enterprise A/Dallas/Line 1/ColdEnd/Lehr/Status/BeltSpeed
Enterprise A/Dallas/Line 1/ColdEnd/Lehr/Status/ZoneTemp1
Enterprise A/Dallas/Line 1/ColdEnd/Lehr/Status/ZoneTemp2
Enterprise A/Dallas/Line 1/ColdEnd/Lehr/Status/ZoneTemp3
Enterprise A/Dallas/Line 1/ColdEnd/Lehr/edge/belt_drive_encoder
Enterprise A/Dallas/Line 1/ColdEnd/Lehr/edge/belt_tension_raw
Enterprise A/Dallas/Line 1/ColdEnd/Lehr/edge/zone1_thermocouple_mv
Enterprise A/Dallas/Line 1/ColdEnd/Lehr/edge/zone2_thermocouple_mv
Enterprise A/Dallas/Line 1/ColdEnd/Lehr/edge/zone3_thermocouple_mv

Enterprise A/Dallas/Line 1/ColdEnd/Inspector/Description
Enterprise A/Dallas/Line 1/ColdEnd/Inspector/State/StateCurrent
Enterprise A/Dallas/Line 1/ColdEnd/Inspector/State/StateReason
Enterprise A/Dallas/Line 1/ColdEnd/Inspector/Production/TotalInspected
Enterprise A/Dallas/Line 1/ColdEnd/Inspector/Production/PassCount
Enterprise A/Dallas/Line 1/ColdEnd/Inspector/Production/RejectCount
Enterprise A/Dallas/Line 1/ColdEnd/Inspector/Production/DefectCHK
Enterprise A/Dallas/Line 1/ColdEnd/Inspector/Production/DefectSED
Enterprise A/Dallas/Line 1/ColdEnd/Inspector/Production/DefectDIM
Enterprise A/Dallas/Line 1/ColdEnd/Inspector/edge/camera_trigger_count
Enterprise A/Dallas/Line 1/ColdEnd/Inspector/edge/photoelectric_sensor
Enterprise A/Dallas/Line 1/ColdEnd/Inspector/edge/reject_gate_status
Enterprise A/Dallas/Line 1/ColdEnd/Inspector/edge/vision_system_ready

Enterprise A/Dallas/Line 1/ColdEnd/Palletizer/Description
Enterprise A/Dallas/Line 1/ColdEnd/Palletizer/State/StateCurrent
Enterprise A/Dallas/Line 1/ColdEnd/Palletizer/State/StateReason
Enterprise A/Dallas/Line 1/ColdEnd/Palletizer/Production/PalletsCompleted
Enterprise A/Dallas/Line 1/ColdEnd/Palletizer/Production/ContainersPerPallet
Enterprise A/Dallas/Line 1/ColdEnd/Palletizer/edge/gripper_vacuum_psi
Enterprise A/Dallas/Line 1/ColdEnd/Palletizer/edge/pallet_present_sensor
Enterprise A/Dallas/Line 1/ColdEnd/Palletizer/edge/robot_position_x
Enterprise A/Dallas/Line 1/ColdEnd/Palletizer/edge/robot_position_y
```

### BigQuery Aggregated Topics

```
Enterprise A/Dallas/Line 1/BigQuery/oee_metrics.oee_percent
Enterprise A/Dallas/Line 1/BigQuery/oee_metrics.availability_percent
Enterprise A/Dallas/Line 1/BigQuery/oee_metrics.performance_percent
Enterprise A/Dallas/Line 1/BigQuery/oee_metrics.quality_percent
Enterprise A/Dallas/Line 1/BigQuery/oee_metrics.yield_efficiency_percent

Enterprise A/Dallas/Line 1/BigQuery/production_metrics.infeed_count
Enterprise A/Dallas/Line 1/BigQuery/production_metrics.outfeed_count
Enterprise A/Dallas/Line 1/BigQuery/production_metrics.good_count
Enterprise A/Dallas/Line 1/BigQuery/production_metrics.bad_count
Enterprise A/Dallas/Line 1/BigQuery/production_metrics.waste_count
Enterprise A/Dallas/Line 1/BigQuery/production_metrics.production_rate_actual
Enterprise A/Dallas/Line 1/BigQuery/production_metrics.production_rate_standard
Enterprise A/Dallas/Line 1/BigQuery/production_metrics.runtime_seconds

Enterprise A/Dallas/Line 1/BigQuery/quality_metrics.total_inspected
Enterprise A/Dallas/Line 1/BigQuery/quality_metrics.pass_count
Enterprise A/Dallas/Line 1/BigQuery/quality_metrics.reject_count
Enterprise A/Dallas/Line 1/BigQuery/quality_metrics.defect_check
Enterprise A/Dallas/Line 1/BigQuery/quality_metrics.defect_seed
Enterprise A/Dallas/Line 1/BigQuery/quality_metrics.defect_dimensional

Enterprise A/Dallas/Line 1/BigQuery/equipment_status.furnace_temp_f
Enterprise A/Dallas/Line 1/BigQuery/equipment_status.forehearth_temp_f
Enterprise A/Dallas/Line 1/BigQuery/equipment_status.machine_speed_cpm
Enterprise A/Dallas/Line 1/BigQuery/equipment_status.lehr_belt_speed_fpm
Enterprise A/Dallas/Line 1/BigQuery/equipment_status.sections_active

Enterprise A/Dallas/Line 1/BigQuery/material_usage.batch_weight_lbs
Enterprise A/Dallas/Line 1/BigQuery/material_usage.feed_rate_lbs_min
Enterprise A/Dallas/Line 1/BigQuery/material_usage.silica_sand_level_percent
Enterprise A/Dallas/Line 1/BigQuery/material_usage.soda_ash_level_percent
Enterprise A/Dallas/Line 1/BigQuery/material_usage.limestone_level_percent
Enterprise A/Dallas/Line 1/BigQuery/material_usage.cullet_level_percent

Enterprise A/Dallas/Line 1/BigQuery/shift
Enterprise A/Dallas/Line 1/BigQuery/work_order
Enterprise A/Dallas/Line 1/BigQuery/product_id
Enterprise A/Dallas/Line 1/BigQuery/timestamp
Enterprise A/Dallas/Line 1/BigQuery/enterprise
Enterprise A/Dallas/Line 1/BigQuery/site
Enterprise A/Dallas/Line 1/BigQuery/line
Enterprise A/Dallas/Line 1/BigQuery/data_source
Enterprise A/Dallas/Line 1/BigQuery/stream_version
```

### OEE Topics

```
Enterprise A/Dallas/Line 1/OEE/Availability
Enterprise A/Dallas/Line 1/OEE/Performance
Enterprise A/Dallas/Line 1/OEE/Quality
Enterprise A/Dallas/Line 1/OEE/GoodCount
Enterprise A/Dallas/Line 1/OEE/BadCount
Enterprise A/Dallas/Line 1/OEE/Waste
Enterprise A/Dallas/Line 1/OEE/Infeed
Enterprise A/Dallas/Line 1/OEE/Outfeed
Enterprise A/Dallas/Line 1/OEE/RunTime
Enterprise A/Dallas/Line 1/OEE/StandardRate
Enterprise A/Dallas/Line 1/OEE/StartTime
Enterprise A/Dallas/Line 1/OEE/State
Enterprise A/Dallas/Line 1/OEE/ProductionID
Enterprise A/Dallas/Line 1/OEE/WorkOrder
Enterprise A/Dallas/Line 1/OEE/YieldEfficiency
```

### ISO 7459 Compliance Topics

```
Enterprise A/Dallas/Line 1/ISO7459/standard
Enterprise A/Dallas/Line 1/ISO7459/standard_name
Enterprise A/Dallas/Line 1/ISO7459/compliance_notes
Enterprise A/Dallas/Line 1/ISO7459/next_inspection_due

Enterprise A/Dallas/Line 1/ISO7459/capacity_tolerance.nominal_ml
Enterprise A/Dallas/Line 1/ISO7459/capacity_tolerance.tolerance_percent
Enterprise A/Dallas/Line 1/ISO7459/capacity_tolerance.lower_limit_ml
Enterprise A/Dallas/Line 1/ISO7459/capacity_tolerance.upper_limit_ml
Enterprise A/Dallas/Line 1/ISO7459/capacity_tolerance.measured_actual_ml
Enterprise A/Dallas/Line 1/ISO7459/capacity_tolerance.status

Enterprise A/Dallas/Line 1/ISO7459/dimensional_tolerances.body_diameter.nominal_mm
Enterprise A/Dallas/Line 1/ISO7459/dimensional_tolerances.body_diameter.tolerance_plus_mm
Enterprise A/Dallas/Line 1/ISO7459/dimensional_tolerances.body_diameter.tolerance_minus_mm
Enterprise A/Dallas/Line 1/ISO7459/dimensional_tolerances.body_diameter.lower_limit_mm
Enterprise A/Dallas/Line 1/ISO7459/dimensional_tolerances.body_diameter.upper_limit_mm
Enterprise A/Dallas/Line 1/ISO7459/dimensional_tolerances.body_diameter.measured_actual_mm
Enterprise A/Dallas/Line 1/ISO7459/dimensional_tolerances.body_diameter.status

Enterprise A/Dallas/Line 1/ISO7459/dimensional_tolerances.overall_height.nominal_mm
Enterprise A/Dallas/Line 1/ISO7459/dimensional_tolerances.overall_height.measured_actual_mm
Enterprise A/Dallas/Line 1/ISO7459/dimensional_tolerances.overall_height.status

Enterprise A/Dallas/Line 1/ISO7459/dimensional_tolerances.finish_height.nominal_mm
Enterprise A/Dallas/Line 1/ISO7459/dimensional_tolerances.finish_height.measured_actual_mm
Enterprise A/Dallas/Line 1/ISO7459/dimensional_tolerances.finish_height.status

Enterprise A/Dallas/Line 1/ISO7459/dimensional_tolerances.finish_outer_diameter.nominal_mm
Enterprise A/Dallas/Line 1/ISO7459/dimensional_tolerances.finish_outer_diameter.measured_actual_mm
Enterprise A/Dallas/Line 1/ISO7459/dimensional_tolerances.finish_outer_diameter.status

Enterprise A/Dallas/Line 1/ISO7459/dimensional_tolerances.wall_thickness_minimum.nominal_mm
Enterprise A/Dallas/Line 1/ISO7459/dimensional_tolerances.wall_thickness_minimum.measured_actual_mm
Enterprise A/Dallas/Line 1/ISO7459/dimensional_tolerances.wall_thickness_minimum.status

Enterprise A/Dallas/Line 1/ISO7459/inspection_data.sample_size
Enterprise A/Dallas/Line 1/ISO7459/inspection_data.samples_passed
Enterprise A/Dallas/Line 1/ISO7459/inspection_data.samples_failed
Enterprise A/Dallas/Line 1/ISO7459/inspection_data.cpk_value
Enterprise A/Dallas/Line 1/ISO7459/inspection_data.overall_compliance
Enterprise A/Dallas/Line 1/ISO7459/inspection_data.inspection_timestamp
Enterprise A/Dallas/Line 1/ISO7459/inspection_data.inspector_id

Enterprise A/Dallas/Line 1/ISO7459/mechanical_properties.internal_pressure_resistance_bar.minimum_required
Enterprise A/Dallas/Line 1/ISO7459/mechanical_properties.internal_pressure_resistance_bar.measured
Enterprise A/Dallas/Line 1/ISO7459/mechanical_properties.internal_pressure_resistance_bar.status
Enterprise A/Dallas/Line 1/ISO7459/mechanical_properties.thermal_shock_resistance_c.minimum_required
Enterprise A/Dallas/Line 1/ISO7459/mechanical_properties.thermal_shock_resistance_c.measured
Enterprise A/Dallas/Line 1/ISO7459/mechanical_properties.thermal_shock_resistance_c.status

Enterprise A/Dallas/Line 1/ISO7459/product_specification.product_id
Enterprise A/Dallas/Line 1/ISO7459/product_specification.product_name
Enterprise A/Dallas/Line 1/ISO7459/product_specification.nominal_capacity_ml
Enterprise A/Dallas/Line 1/ISO7459/product_specification.material
Enterprise A/Dallas/Line 1/ISO7459/product_specification.finish_type
Enterprise A/Dallas/Line 1/ISO7459/product_specification.production_line
```

### Site Topics

```
Enterprise A/Dallas/Site/CurrentShift
Enterprise A/Dallas/Site/DateTime
Enterprise A/Dallas/Site/LocalTime
Enterprise A/Dallas/Site/Description
Enterprise A/Dallas/Site/ProcessFlow
Enterprise A/Dallas/Site/OperatingSchedule
Enterprise A/Dallas/Site/OperatingSchedule/facility
Enterprise A/Dallas/Site/OperatingSchedule/timezone
Enterprise A/Dallas/Site/OperatingSchedule/operating_days
Enterprise A/Dallas/Site/OperatingSchedule/notes
Enterprise A/Dallas/Site/OperatingSchedule/shifts[0].shift_id
Enterprise A/Dallas/Site/OperatingSchedule/shifts[0].shift_name
Enterprise A/Dallas/Site/OperatingSchedule/shifts[0].start_time
Enterprise A/Dallas/Site/OperatingSchedule/shifts[0].end_time
Enterprise A/Dallas/Site/OperatingSchedule/shifts[0].duration_hours
Enterprise A/Dallas/Site/OperatingSchedule/shifts[1].shift_id
Enterprise A/Dallas/Site/OperatingSchedule/shifts[1].shift_name
Enterprise A/Dallas/Site/OperatingSchedule/shifts[1].start_time
Enterprise A/Dallas/Site/OperatingSchedule/shifts[1].end_time
Enterprise A/Dallas/Site/OperatingSchedule/shifts[1].duration_hours
Enterprise A/Dallas/Site/OperatingSchedule/shifts[2].shift_id
Enterprise A/Dallas/Site/OperatingSchedule/shifts[2].shift_name
Enterprise A/Dallas/Site/OperatingSchedule/shifts[2].start_time
Enterprise A/Dallas/Site/OperatingSchedule/shifts[2].end_time
Enterprise A/Dallas/Site/OperatingSchedule/shifts[2].duration_hours
```

---

*Document generated from Neo4j Graph Database. Data structure reflects the Enterprise A Unified Namespace configuration.*

*For questions or updates, contact your UNS administrator.*
