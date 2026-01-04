# Executive Summary: Site 3

> **Author:** Kevin Okonkwo, Plant Manager
> **Site:** Site 3
> **Enterprise B - Beverage Bottling & Packaging**
> **January 2026**

---

## Continuous Improvement at Scale

Site 3 is the smallest operation in Enterprise B—one filling line, one mix room, two storage tanks, and a single manual palletizing station. Some would call us the test bed.

I prefer to think of us as the optimization laboratory.

Our 20 assets generate clean data sets that let me experiment without enterprise-wide risk, then export proven improvements to the larger sites.

## Beyond Implementation

The UNS implementation here was technically straightforward. What I have done since go-live is where the value compounds.

I built a correlation analysis between our single vat's process variables—temperature, flowrate, and weight—and the downstream defect counts at the labeler:

```
Enterprise B/Site3/liquidprocessing/mixroom01/vat/processdata/process/temperature
Enterprise B/Site3/fillerproduction/fillingline01/labeler/metric/defectcount
```

We discovered that batches mixed when vat temperature exceeded a specific threshold produced labels with adhesion failures at a rate 2.3x higher than normal. This was invisible in aggregate OEE numbers but crystal clear in the linked data.

We adjusted the mix protocol. Labeler quality jumped from 0.96 to 0.99 within three weeks.

## Predictive Operations

Our North and South tanks in storage now operate on a predictive scheduling model. By tracking weight depletion rates against work order quantities, I can predict changeover windows with 94% accuracy:

```
Enterprise B/Site3/liquidprocessing/tankstorage01/northtank/processdata/process/weight
Enterprise B/Site3/fillerproduction/fillingline01/filler/workorder/currentworkorderid
```

The single filling line no longer waits for tank availability—we sequence production to match liquid processing cycles.

## Constraint Management

The manual palletizing station was our constraint. I used the UNS state and duration data to establish baseline cycle times, then worked with the operator to identify micro-delays. The workstation's availability improved 8% without capital expenditure—purely through process refinement informed by data.

## The Optimization Laboratory

Site 3 proves that UNS value scales down as effectively as it scales up. We are small, but we are fast, and we share every optimization playbook with our colleagues at The Cap Shack and Filler Central.

The enterprise wins when data flows and learnings travel with it.

---

**Kevin Okonkwo**
*Plant Manager*
*Site 3*
