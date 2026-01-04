# ENTERPRISE B
## VESSEL ENGINEERING SPECIFICATION

**Document Number:** EB-VES-SPEC-001  
**Revision:** A  
**Date:** January 4, 2026  
**Classification:** Internal Use  

---

## 1. SCOPE

### 1.1 Purpose
This specification defines the engineering requirements for all process vessels (vats and tanks) installed across Enterprise B manufacturing facilities. It establishes standardized vessel types, dimensions, materials, and construction requirements to ensure operational consistency, maintainability, and regulatory compliance.

### 1.2 Applicability
This specification applies to:
- All mixing vats in Mix Room operations
- All storage tanks in Tank Storage operations
- Sites 1, 2, and 3 of Enterprise B

### 1.3 Reference Documents

| Document | Title |
|----------|-------|
| ASME BPE-2022 | Bioprocessing Equipment |
| ASME BPVC Section VIII, Div. 1 | Pressure Vessels |
| API 650 | Welded Tanks for Oil Storage |
| ASME B16.5 | Pipe Flanges and Flanged Fittings |
| 3-A Sanitary Standards | Food Equipment |
| FDA 21 CFR Part 211 | Current Good Manufacturing Practice |

---

## 2. VESSEL CLASSIFICATION

### 2.1 Vessel Types

Enterprise B utilizes four (4) standardized vessel types:

| Type Code | Description | Function | Quantity |
|-----------|-------------|----------|----------|
| VAT-20 | 20,000 L Mixing Vat | Batch mixing and blending | 7 |
| TANK-25 | 25,000 L Storage Tank | Intermediate/product storage | 4 |
| TANK-35 | 35,000 L Storage Tank | Intermediate/product storage | 6 |
| TANK-45 | 45,000 L Storage Tank | Bulk product storage | 1 |

---

## 3. VAT-20 MIXING VAT SPECIFICATION

### 3.1 General Arrangement

```
                    ┌─── Vent/Pressure Relief
                    │    (DN50 PN16)
                    ▼
              ═══════════
            ╱             ╲        ← Torispherical Head
           ╱               ╲         (10% dish radius)
          │                 │
          │    ┌───────┐    │      ← Top Manway (500mm)
          │    └───────┘    │
          │                 │
    CIP ──┼──●              │      ← CIP Spray Ball
          │                 │
          │                 │
          │   ╭─────────╮   │      ← Agitator Shaft
          │   │    ●    │   │        (top-entry, center-mounted)
          │   │    │    │   │
          │   │  ──┼──  │   │      ← Impeller (45° pitched blade)
          │   │    │    │   │
          │   │  ──┼──  │   │      ← Impeller (Rushton)
          │   │    │    │   │
          │   ╰────┴────╯   │
          │                 │
           ╲               ╱
            ╲    ┌───┐    ╱        ← Torispherical Bottom
             ════╧═══╧════          (10% dish radius)
                  │
                  ▼               ← Bottom Outlet (DN80)
                 ═══

          ◄─── 2,600 mm ───►
```

### 3.2 Dimensional Data

| Parameter | Value | Tolerance |
|-----------|-------|-----------|
| Nominal Capacity | 20,000 L | — |
| Working Capacity | 18,000 L | — |
| Inside Diameter | 2,600 mm | ±5 mm |
| Shell Height (tan-to-tan) | 3,200 mm | ±5 mm |
| Top Head Height | 550 mm | ±3 mm |
| Bottom Head Height | 550 mm | ±3 mm |
| Overall Height | 4,300 mm | ±10 mm |
| Shell Thickness | 6 mm | +1/-0 mm |
| Head Thickness | 8 mm | +1/-0 mm |
| Head Type (top) | Torispherical (10:1) | — |
| Head Type (bottom) | Torispherical (10:1) | — |

### 3.3 Design Conditions

| Parameter | Value |
|-----------|-------|
| Design Pressure | 0.5 barg (internal) |
| Design Vacuum | -0.5 barg |
| Design Temperature | -10°C to +95°C |
| Operating Pressure | Atmospheric |
| Operating Temperature | 5°C to 82°C |
| Hydrostatic Test Pressure | 0.75 barg |
| Corrosion Allowance | 0 mm |

### 3.4 Materials of Construction

| Component | Material | Specification |
|-----------|----------|---------------|
| Shell | 316L Stainless Steel | ASTM A240 |
| Heads | 316L Stainless Steel | ASTM A240 |
| Nozzles | 316L Stainless Steel | ASTM A312 TP316L |
| Flanges | 316L Stainless Steel | ASTM A182 F316L |
| Gaskets | PTFE-Encapsulated Silicone | FDA Compliant |
| Agitator Shaft | 316L Stainless Steel | ASTM A479 |
| Impellers | 316L Stainless Steel | Cast, ASTM A351 CF3M |

### 3.5 Surface Finish

| Location | Finish | Ra Value |
|----------|--------|----------|
| Interior wetted surfaces | Electropolished | ≤ 0.5 µm |
| Interior welds | Ground and polished | ≤ 0.8 µm |
| Exterior surfaces | 2B mill finish | ≤ 1.6 µm |

### 3.6 Nozzle Schedule

| Nozzle | Size | Type | Quantity | Purpose |
|--------|------|------|----------|---------|
| N1 | DN80 | Flanged (ASME B16.5) | 1 | Bottom outlet |
| N2 | DN50 | Tri-clamp | 2 | Inlet |
| N3 | DN50 | Tri-clamp | 1 | CIP supply |
| N4 | DN50 | Flanged | 1 | Vent/PRV |
| N5 | DN25 | Tri-clamp | 2 | Sample ports |
| N6 | DN25 | Tri-clamp | 1 | Temperature probe |
| N7 | DN25 | Tri-clamp | 1 | Level transmitter |
| N8 | DN25 | Threaded | 3 | Load cells |
| M1 | 500 mm | Hinged, davit arm | 1 | Top manway |
| SG1 | DN50 | — | 1 | Sight glass |

### 3.7 Agitation System

| Parameter | Value |
|-----------|-------|
| Agitator Type | Top-entry, center-mounted |
| Drive | 7.5 kW, 1450 RPM motor with VFD |
| Shaft Speed | 30–150 RPM (variable) |
| Shaft Diameter | 80 mm |
| Seal Type | Double mechanical seal, water-flushed |
| Impeller 1 (lower) | 4-blade Rushton turbine, Ø 900 mm |
| Impeller 2 (upper) | 4-blade 45° pitched, Ø 1,000 mm |
| Baffles | 4 × vertical, 200 mm wide |

### 3.8 Instrumentation

| Tag | Description | Range | Output |
|-----|-------------|-------|--------|
| TT-xxx | Temperature transmitter (RTD) | -20 to +120°C | 4-20 mA |
| LT-xxx | Load cells (×3) | 0–25,000 kg | 4-20 mA |
| FT-xxx | Flowrate (inlet) | 0–2,500 L/min | 4-20 mA |
| PT-xxx | Pressure transmitter | -1 to +2 barg | 4-20 mA |

---

## 4. TANK-25 STORAGE TANK SPECIFICATION

### 4.1 General Arrangement

```
              ═══════════
            ╱             ╲        ← Torispherical Head
           ╱    ┌─────┐    ╲        (Manway 500mm)
          │     └─────┘     │
          │                 │
    CIP ──┼──●              │      ← CIP Spray Ball
          │                 │
          │                 │
          │                 │
          │                 │
          │                 │
          │                 │
          │                 │
          │                 │
          │                 │
          │                 │
          │                 │
          │                 │
          │_________________│      ← Flat Bottom
          ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓        (on ring foundation)
                  │
                  ▼               ← Side-bottom Outlet (DN100)
                 ═══

          ◄─── 2,400 mm ───►
```

### 4.2 Dimensional Data

| Parameter | Value | Tolerance |
|-----------|-------|-----------|
| Nominal Capacity | 25,000 L | — |
| Working Capacity | 22,500 L | — |
| Inside Diameter | 2,400 mm | ±5 mm |
| Shell Height | 5,500 mm | ±10 mm |
| Top Head Height | 500 mm | ±3 mm |
| Overall Height | 6,000 mm | ±15 mm |
| Shell Thickness | 5 mm | +1/-0 mm |
| Bottom Plate Thickness | 6 mm | +1/-0 mm |
| Head Thickness | 6 mm | +1/-0 mm |
| Bottom Type | Flat, sloped to outlet (1:100) | — |

### 4.3 Design Conditions

| Parameter | Value |
|-----------|-------|
| Design Pressure | Atmospheric + hydrostatic head |
| Design Temperature | -10°C to +60°C |
| Operating Temperature | 4°C to 50°C |
| Specific Gravity (design) | 1.2 max |
| Corrosion Allowance | 1.5 mm |

### 4.4 Materials of Construction

| Component | Material | Specification |
|-----------|----------|---------------|
| Shell | 304 Stainless Steel | ASTM A240 |
| Top Head | 304 Stainless Steel | ASTM A240 |
| Bottom Plate | 304 Stainless Steel | ASTM A240 |
| Nozzles | 304 Stainless Steel | ASTM A312 TP304 |

### 4.5 Surface Finish

| Location | Finish | Ra Value |
|----------|--------|----------|
| Interior wetted surfaces | Pickled and passivated | ≤ 0.8 µm |
| Interior welds | Ground flush | ≤ 1.2 µm |
| Exterior surfaces | 2B mill finish | — |

### 4.6 Nozzle Schedule

| Nozzle | Size | Type | Quantity | Purpose |
|--------|------|------|----------|---------|
| N1 | DN100 | Flanged | 1 | Bottom outlet |
| N2 | DN80 | Flanged | 1 | Inlet |
| N3 | DN50 | Tri-clamp | 1 | CIP supply |
| N4 | DN100 | Flanged | 1 | Vent |
| N5 | DN25 | Tri-clamp | 1 | Temperature probe |
| N6 | DN25 | Threaded | 3 | Load cells |
| M1 | 500 mm | Bolted | 1 | Top manway |
| SG1 | — | Tubular | 1 | Level gauge |

---

## 5. TANK-35 STORAGE TANK SPECIFICATION

### 5.1 Dimensional Data

| Parameter | Value | Tolerance |
|-----------|-------|-----------|
| Nominal Capacity | 35,000 L | — |
| Working Capacity | 31,500 L | — |
| Inside Diameter | 2,600 mm | ±5 mm |
| Shell Height | 6,600 mm | ±10 mm |
| Top Head Height | 500 mm | ±3 mm |
| Overall Height | 7,100 mm | ±15 mm |
| Shell Thickness | 5 mm | +1/-0 mm |
| Bottom Plate Thickness | 6 mm | +1/-0 mm |
| Head Thickness | 6 mm | +1/-0 mm |
| Bottom Type | Flat, sloped to outlet (1:100) | — |

### 5.2 Design Conditions

| Parameter | Value |
|-----------|-------|
| Design Pressure | Atmospheric + hydrostatic head |
| Design Temperature | -10°C to +60°C |
| Operating Temperature | 4°C to 50°C |
| Specific Gravity (design) | 1.2 max |
| Corrosion Allowance | 1.5 mm |

### 5.3 Materials of Construction

Same as TANK-25 (Section 4.4).

### 5.4 Nozzle Schedule

| Nozzle | Size | Type | Quantity | Purpose |
|--------|------|------|----------|---------|
| N1 | DN100 | Flanged | 1 | Bottom outlet |
| N2 | DN80 | Flanged | 1 | Inlet |
| N3 | DN50 | Tri-clamp | 1 | CIP supply |
| N4 | DN150 | Flanged | 1 | Vent |
| N5 | DN25 | Tri-clamp | 1 | Temperature probe |
| N6 | DN25 | Threaded | 3 | Load cells |
| M1 | 500 mm | Bolted | 1 | Top manway |
| SG1 | — | Tubular | 1 | Level gauge |

---

## 6. TANK-45 STORAGE TANK SPECIFICATION

### 6.1 Dimensional Data

| Parameter | Value | Tolerance |
|-----------|-------|-----------|
| Nominal Capacity | 45,000 L | — |
| Working Capacity | 40,500 L | — |
| Inside Diameter | 2,800 mm | ±5 mm |
| Shell Height | 7,300 mm | ±10 mm |
| Top Head Height | 500 mm | ±3 mm |
| Overall Height | 7,800 mm | ±15 mm |
| Shell Thickness | 6 mm | +1/-0 mm |
| Bottom Plate Thickness | 8 mm | +1/-0 mm |
| Head Thickness | 6 mm | +1/-0 mm |
| Bottom Type | Flat, sloped to outlet (1:100) | — |

### 6.2 Design Conditions

| Parameter | Value |
|-----------|-------|
| Design Pressure | Atmospheric + hydrostatic head |
| Design Temperature | -10°C to +60°C |
| Operating Temperature | 4°C to 50°C |
| Specific Gravity (design) | 1.2 max |
| Corrosion Allowance | 1.5 mm |

### 6.3 Materials of Construction

Same as TANK-25 (Section 4.4).

### 6.4 Nozzle Schedule

| Nozzle | Size | Type | Quantity | Purpose |
|--------|------|------|----------|---------|
| N1 | DN150 | Flanged | 1 | Bottom outlet |
| N2 | DN100 | Flanged | 1 | Inlet |
| N3 | DN50 | Tri-clamp | 1 | CIP supply |
| N4 | DN150 | Flanged | 1 | Vent |
| N5 | DN25 | Tri-clamp | 1 | Temperature probe |
| N6 | DN25 | Threaded | 3 | Load cells |
| M1 | 600 mm | Bolted | 1 | Top manway |
| SG1 | — | Tubular | 1 | Level gauge |

---

## 7. VESSEL INSTALLATION REGISTRY

### 7.1 Site 1 — The Cap Shack

| Asset ID | Tag | Location | Vessel Type | UNS Path |
|----------|-----|----------|-------------|----------|
| 31 | MR01-VAT-001 | Mix Room 01 | VAT-20 | `Enterprise B/Site1/liquidprocessing/mixroom01/vat01` |
| 32 | MR01-VAT-002 | Mix Room 01 | VAT-20 | `Enterprise B/Site1/liquidprocessing/mixroom01/vat02` |
| 33 | MR01-VAT-003 | Mix Room 01 | VAT-20 | `Enterprise B/Site1/liquidprocessing/mixroom01/vat03` |
| 34 | MR01-VAT-004 | Mix Room 01 | VAT-20 | `Enterprise B/Site1/liquidprocessing/mixroom01/vat04` |
| 35 | TS01-TNK-001 | Tank Storage 01 | TANK-35 | `Enterprise B/Site1/liquidprocessing/tankstorage01/tank01` |
| 36 | TS01-TNK-002 | Tank Storage 01 | TANK-35 | `Enterprise B/Site1/liquidprocessing/tankstorage01/tank02` |
| 37 | TS01-TNK-003 | Tank Storage 01 | TANK-25 | `Enterprise B/Site1/liquidprocessing/tankstorage01/tank03` |
| 38 | TS01-TNK-004 | Tank Storage 01 | TANK-35 | `Enterprise B/Site1/liquidprocessing/tankstorage01/tank04` |
| 39 | TS01-TNK-005 | Tank Storage 01 | TANK-45 | `Enterprise B/Site1/liquidprocessing/tankstorage01/tank05` |
| 40 | TS01-TNK-006 | Tank Storage 01 | TANK-25 | `Enterprise B/Site1/liquidprocessing/tankstorage01/tank06` |

### 7.2 Site 2 — Filler Central

| Asset ID | Tag | Location | Vessel Type | UNS Path |
|----------|-----|----------|-------------|----------|
| 86 | MR01-VAT-001 | Mix Room 01 | VAT-20 | `Enterprise B/Site2/liquidprocessing/mixroom01/vat01` |
| 87 | MR01-VAT-002 | Mix Room 01 | VAT-20 | `Enterprise B/Site2/liquidprocessing/mixroom01/vat02` |
| 88 | TS01-TNK-001 | Tank Storage 01 | TANK-35 | `Enterprise B/Site2/liquidprocessing/tankstorage01/tank01` |
| 89 | TS01-TNK-002 | Tank Storage 01 | TANK-35 | `Enterprise B/Site2/liquidprocessing/tankstorage01/tank02` |
| 90 | TS01-TNK-003 | Tank Storage 01 | TANK-35 | `Enterprise B/Site2/liquidprocessing/tankstorage01/tank03` |

### 7.3 Site 3 — Plant3

| Asset ID | Tag | Location | Vessel Type | UNS Path |
|----------|-----|----------|-------------|----------|
| 116 | MR01-VAT-001 | Mix Room 01 | VAT-20 | `Enterprise B/Site3/liquidprocessing/mixroom01/vat01` |
| 117 | TS01-TNK-001 | Tank Storage 01 | TANK-25 | `Enterprise B/Site3/liquidprocessing/tankstorage01/tank01` |
| 118 | TS01-TNK-002 | Tank Storage 01 | TANK-25 | `Enterprise B/Site3/liquidprocessing/tankstorage01/tank02` |

---

## 8. SUMMARY SPECIFICATIONS

### 8.1 Quick Reference Table

| Parameter | VAT-20 | TANK-25 | TANK-35 | TANK-45 |
|-----------|--------|---------|---------|---------|
| **Capacity (nominal)** | 20,000 L | 25,000 L | 35,000 L | 45,000 L |
| **Capacity (working)** | 18,000 L | 22,500 L | 31,500 L | 40,500 L |
| **Inside Diameter** | 2,600 mm | 2,400 mm | 2,600 mm | 2,800 mm |
| **Overall Height** | 4,300 mm | 6,000 mm | 7,100 mm | 7,800 mm |
| **Shell Material** | 316L SS | 304 SS | 304 SS | 304 SS |
| **Design Pressure** | 0.5 barg | Atm | Atm | Atm |
| **Design Temp** | -10/+95°C | -10/+60°C | -10/+60°C | -10/+60°C |
| **Interior Finish** | EP ≤0.5µm | P&P ≤0.8µm | P&P ≤0.8µm | P&P ≤0.8µm |
| **Bottom Type** | Dished | Flat | Flat | Flat |
| **Agitation** | Yes | No | No | No |
| **CIP Capable** | Yes | Yes | Yes | Yes |
| **Quantity** | 7 | 4 | 6 | 1 |

### 8.2 Total Enterprise Capacity

| Vessel Type | Count | Unit Capacity | Total Capacity |
|-------------|-------|---------------|----------------|
| VAT-20 | 7 | 20,000 L | 140,000 L |
| TANK-25 | 4 | 25,000 L | 100,000 L |
| TANK-35 | 6 | 35,000 L | 210,000 L |
| TANK-45 | 1 | 45,000 L | 45,000 L |
| **TOTAL** | **18** | — | **495,000 L** |

---

## 9. FABRICATION REQUIREMENTS

### 9.1 Welding

| Requirement | Specification |
|-------------|---------------|
| Welding procedure | AWS D18.1 / ASME Section IX |
| Welder qualification | ASME Section IX |
| Interior welds | Full penetration, ground flush |
| Exterior welds | Full penetration, cap acceptable |
| Weld examination | 100% visual, 10% radiographic (random) |
| Weld finish (interior) | Ground to Ra ≤ 0.8 µm minimum |

### 9.2 Inspection and Testing

| Test | Requirement |
|------|-------------|
| Dimensional inspection | Per Section 3.2, 4.2, 5.1, 6.1 |
| Visual inspection | 100% interior and exterior |
| Dye penetrant (PT) | All interior welds |
| Hydrostatic test | 1.5× design pressure, 30 min hold |
| Surface roughness | Certified Ra measurements |
| Passivation verification | Per ASTM A380 |
| Documentation | MTRs, weld maps, test reports |

### 9.3 Cleaning and Passivation

| Step | Description |
|------|-------------|
| 1 | Degrease with alkaline cleaner |
| 2 | Rinse with deionized water |
| 3 | Passivate per ASTM A380 (citric acid method) |
| 4 | Final rinse with deionized water |
| 5 | Dry with filtered air |
| 6 | Protect openings for shipment |

---

## 10. DOCUMENTATION DELIVERABLES

The following documentation shall be provided with each vessel:

- [ ] General arrangement drawing
- [ ] Fabrication drawings
- [ ] Nozzle orientation drawing
- [ ] Bill of materials
- [ ] Welding procedure specifications (WPS)
- [ ] Procedure qualification records (PQR)
- [ ] Welder qualification records (WQR)
- [ ] Material test reports (MTR)
- [ ] Dimensional inspection report
- [ ] Weld map
- [ ] NDE reports (PT, RT as applicable)
- [ ] Hydrostatic test report
- [ ] Surface finish certification
- [ ] Passivation certificate
- [ ] Nameplate rubbing / data sheet

---

## REVISION HISTORY

| Rev | Date | Description | Author |
|-----|------|-------------|--------|
| A | 2026-01-04 | Initial release | Enterprise B Engineering |

---

## APPROVAL

| Role | Name | Signature | Date |
|------|------|-----------|------|
| Prepared by | | | |
| Reviewed by | | | |
| Approved by | | | |

---

*END OF DOCUMENT*
