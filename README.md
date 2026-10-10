# OPERA DISP-S1 velocity viewer

Prototype interactive map of monthly LOS velocity from OPERA DISP-S1.

**Viewer:** https://gracebato.github.io/operadisp-viewer/assets/operadisp_viewer.html

- A slider steps through the monthly velocity maps (each over the last 4
  calendar years). Positive = motion toward the satellite.
- Stations are labelled with that month's velocity at their nearest pixel.
- The panel below the map shows each station's displacement time series.
- A toggle (key M) switches between two ways of handling masked pixels:
  **Proposed** and **Current (ASF)**, the rule of the ASF Displacement Portal.

## Why masked pixels need care

Each DISP-S1 product is the displacement between a **reference date** and a
**secondary date**. A pixel's time series is built by chaining them:

```
position(secondary date) = position(reference date) + product
```

Where the `recommended_mask` is 0 (unreliable pixel), the product has no value.
On an ordinary date that loses one date. On a **reference date** it loses the
position that every later product needs.

In forward mode the products are sequential: each date is referenced to the
previous acquisition (instead of a ministack of about 15 dates sharing one
reference date). Each date is therefore also the reference for the next one, so
every masked date breaks the chain.

## Proposed vs. Current (ASF)

**Time series**

| | Proposed | Current (ASF) |
|---|---|---|
| Masked date (not a reference date) | last value held, hollow dot (display only) | last value held, hollow dot |
| Masked reference date | **gap** from that date: nothing is filled in | **held** at the last valid value |
| When data return | **new segment starting at 0**, dashed line | continues from the held value |
| Assumption | none: the jump between segments is unknown and never used | the pixel did not move while masked |
| If that is wrong | no error from the masking | every later value is offset by the motion during the masked time |
| Forward mode | every masked date is a gap and a new segment | every masked date adds an assumed "no motion" step |

**Velocity**

| | Proposed | Current (ASF) |
|---|---|---|
| Pixels on the map | valid on at least 80% of the dates | valid on **every** date |
| Map velocity | one slope through all valid dates, **one offset per segment** | straight line (pixels valid on every date have one segment) |
| Chart velocity | same fit, shown only if a segment spans at least a year | straight line through the unmasked dates |
| Map and chart agree | yes: same pixel, same rule, same fit | the chart line also spans held stretches |

## How the Proposed velocity is computed

Each segment may sit higher or lower than the others by an unknown amount, so
each gets its own offset and only the motion *within* segments sets the slope:

1. In each segment, subtract the segment's average time and average
   displacement from its dates and values.
2. Fit one slope through all the centred values:
   `velocity = Σ(Δt × Δd) / Σ(Δt²)`.

With one segment this is the ordinary straight-line fit. Example: a pixel moving
4 mm/yr, in two segments of a year each (the second restarted at 0), gives
4 mm/yr this way; one plain line through all points gives 0.6 mm/yr, because it
reads the restart as motion. GNSS processing treats a known equipment change
(a step of unknown size) the same way.
