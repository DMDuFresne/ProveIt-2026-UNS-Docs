# Executive Summary: Dallas Glass Container Manufacturing

> **From the Desk of the Site Leader**
> **Enterprise A - Dallas Facility**
> **January 2026**

---

When I took over this facility eight years ago, we were operating in the dark. I do not mean literally—though the HotEnd at 2700 degrees Fahrenheit provides plenty of light. I mean operationally. We had data everywhere and visibility nowhere.

## The Problem We Faced

Our furnace runs 24/7, 365 days a year. You cannot shut down a glass furnace without a multi-million dollar rebuild. Yet when my shift supervisors needed to know the furnace temperature, they would walk to the DCS terminal. When quality engineers investigated seed defects, they dug through the historian. When I needed OEE for the morning production meeting, someone pulled a spreadsheet. We had thirteen pieces of equipment across three process areas, each with its own control system, its own data silo, its own language.

The real cost was not just inefficiency. It was the decisions we could not make. When reject rates climbed on second shift, we spent hours correlating data manually. Was it the batch composition? The gob temperature? The lehr belt speed? By the time we found the root cause, we had already sent thousands of defective jars to inspection.

## What Changed

The Unified Namespace gave us a single source of truth. Every data point now has a clear, logical address:

```
Enterprise A/Dallas/Line 1/HotEnd/Furnace/Status/Temperature
```

My operators know exactly where to find it. My engineers can correlate it with anything else in the plant. My maintenance team can see raw thermocouple voltages alongside calibrated readings.

## The Business Impact

We are currently running at 97.9 percent OEE on Line 1 with our 8-section IS machine producing 7,200 containers per hour. More importantly, when we see dimensional defects trending up, we can immediately correlate that with IS machine speed and gob temperature in the same query. When check defects spike, we cross-reference lehr zone temperatures and belt speed within minutes, not hours.

Our ISO 7459 compliance data now flows directly from inspection to a single accessible path. Customer audits that used to require three days of preparation now take an afternoon. We can demonstrate dimensional tolerances, capacity measurements, and Cpk values from one system.

## Looking Forward

The visibility this system provides is not just about efficiency. It is about enabling the next generation of operators. When my most experienced furnace operator retires next year, the knowledge of what "normal" looks like will not walk out the door with him. It will be in the data, accessible to everyone who follows.

---

*This is what modern manufacturing looks like: not more data, but the right data, in the right place, at the right time.*

---

**Site Leader**
*Dallas Facility*
*Enterprise A Glass Container Manufacturing*
