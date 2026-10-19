# CGMacros data-access check

Inspection completed on October 18, 2026 using the official CGMacros v1.0.0 release. These checks establish access and identify preprocessing issues. They do not establish forecasting accuracy.

## Source and integrity

- Dataset: [CGMacros on PhysioNet](https://physionet.org/content/cgmacros/1.0.0/).
- Archive: `CGMacros_dateshifted365.zip`.
- Downloaded archive size: 657,187,340 bytes.
- Expected and observed SHA-256: `05c8b0e6f1a2757050aced55ce4bf6ab2ac9b30f2fd8ca193056812d9c621d4d`.
- The checksum matches PhysioNet's published `SHA256SUMS.txt`.
- Every ZIP entry passed its CRC integrity check.
- All 45 participant CSVs and the three supplementary CSVs opened successfully.
- Original source values were preserved. Header whitespace was stripped in memory for the inspection.

The full archive contains 48 CSV files and 3,454 JPG images. Numeric tables were extracted for inspection; meal photographs remain available in the original archive. This public repository redistributes neither participant tables nor photographs.

## Observed coverage

| Item | Observed value |
| --- | --- |
| Participants with time-series files | 45 |
| Released minute-timeline rows | 687,580 |
| Nonmissing Dexcom rows | 629,825 |
| Missing Dexcom values among existing rows | 8.3997% |
| Nonmissing Libre rows | 687,360 |
| Missing Libre values among existing rows | 0.0320% |
| Logged meal records, including snacks | 1,706 |
| Meals with calories, carbohydrate, protein, fat, and fiber present | 1,705 |
| Invalid timestamps | 0 |
| Duplicate timestamps within a participant | 0 |
| Participants whose timestamps are out of order | 0 |
| Participants with gaps between minute rows | 10 |
| Longest interval between consecutive rows | 901 minutes |
| Median first-to-last timestamp span | 10.6347 days |

Missing-value percentages use the rows present in the released tables. They do not count absent timestamps as additional missing rows. First-to-last spans include gaps and should not be interpreted as uninterrupted sensor coverage.

## Issues to handle before modeling

### Interpolation and sampling

The tables use a one-minute timeline, but the source documentation gives native sampling intervals of five minutes for Dexcom and fifteen minutes for Libre. Numerous adjacent released values lie exactly on straight-line segments, consistent with interpolation.

This matters for forecasting: an interpolated value at time `t` can depend on a sensor observation after `t`. A chronological split alone does not remove this issue. Native measurement times, available observation masks, and the interpolation procedure must be checked before choosing forecast origins or making real-time performance claims. Do not assume that sampling at wall-clock multiples of five minutes recovers the original observations.

### Gaps and short histories

Some trajectories contain gaps. Targets must be joined at exactly `t + 60 minutes`, and long gaps should define separate segments. Forecast windows must not silently jump across gaps or participants.

Each participant has a short history. Personal calibration and later testing must use separate periods, and all calibration budgets must be compared on the same test period.

### Meal and activity fields

- 131 meal records have `Amount Consumed` values outside the dictionary's stated 0–100 range. The field will be omitted initially rather than corrected by guessing.
- Two participant files omit `Amount Consumed` entirely.
- Some headers have trailing spaces or additional index columns.
- Activity schemas vary: files may contain `METs`, `Intensity`, or `Steps`. These fields must not be treated as interchangeable without documentation.
- Snack labels appear in several spellings and need an explicit category mapping if used.
- Three nonempty image paths did not match an entry in the downloaded archive after trimming surrounding whitespace.
- Post-meal photos and the final fraction consumed are unavailable at meal onset. Their use would require a different forecast-origin assumption.

### Dates and supplementary metadata

Calendar dates were shifted for de-identification. Preserve elapsed time and within-person order; do not infer a shared real calendar between participants.

The demographic table has 45 rows and the microbiome table has 45 rows, each matching the released participant time-series IDs. The gut-health table has 47 rows, including IDs 24 and 25 without a released time series. Join metadata using participant ID, not row position.

## Initial assessment

The dataset supports a focused study of short-horizon glucose forecasting and limited personal calibration. The shared glucose and meal-nutrient fields provide a reasonable starting point. Sampling and interpolation need explicit resolution before the forecasting pipeline can be called leakage-resistant.

The inspection used local scripts and aggregate checks. Those scripts are not included in this initial documentation-only repository; an executable, documented data pipeline remains future work.
