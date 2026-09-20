# Production Loss Assessment & Downtime Optimization in the Volve Field

A production engineering operations screening project built on Equinor's open Volve field
production dataset. This is **not** a machine learning, reservoir simulation or forecasting
project, it is an operational analysis of well uptime, downtime and production loss
opportunity, built to answer the question a production engineer is actually asked:

> **Where is production being lost and which wells should be prioritized for production
> recovery efforts?**

## Project Overview

Volve was a North Sea oil field operated by Equinor (and licence partners including
ExxonMobil Exploration & Production Norway AS) from 2008 to 2016 and released as an open
dataset in 2018. This project uses the field's daily production records to build a
reproducible operations screening workflow: clean the data, quantify uptime and downtime by
well, estimate the production opportunity lost to that downtime, evaluate rate efficiency
while online and combine those signals into a defensible well prioritization framework.

## Business Problem

Production engineers are responsible for maximizing output from wells that already exist
which starts with knowing which wells are underperforming, by how much and why. Two wells
can both show "high downtime" and still represent very different problems: one because it's
genuinely unreliable, another because its sheer production rate means even routine downtime
costs the field a lot of oil. This project builds the screening layer that turns raw
production data into a ranked, engineering-defensible shortlist for further investigation.

## Dataset Description

- **Source:** [Volve field data set](https://www.equinor.com/energy/volve-data-sharing),
  released by Equinor and the Volve licence partners under the Equinor Open Data Licence
  (non-commercial use; not for resale — see the source page for full terms).
- **File used:** `Daily Production Data` sheet, `Volve production data.xlsx`
- **Scope:** 6 oil-producer wells, 9,143 well-days after cleaning, 2008-02-12 to 2016-09-17
- **Key fields:** `DATEPRD`, `WELL_BORE_CODE`, `FLOW_KIND`, `WELL_TYPE`, `ON_STREAM_HRS`,
  `BORE_OIL_VOL`, `BORE_GAS_VOL`, `BORE_WAT_VOL`

## Methodology

1. **Data cleaning** — filtered to oil producers only; capped 13 well-days with
   `ON_STREAM_HRS` slightly above 24 (all fall on the daylight-saving "fall back" date, a
   legitimate 25-hour calendar day, not a data error); flagged (without deleting) one row
   with zero recorded hours but a positive oil volume; confirmed no duplicate well-days.
2. **Uptime analysis** — `Uptime % = ON_STREAM_HRS / 24 × 100`, summarized by well.
3. **Downtime analysis** — `Downtime Hours = 24 − ON_STREAM_HRS`, both as a cumulative total
   and a daily average per well, plus a field-wide yearly trend.
4. **Production loss estimation** — estimated lost oil using each day's own implied rate
   (`BORE_OIL_VOL / ON_STREAM_HRS`), **cross-checked against a well-level benchmark-rate
   method** (75th percentile of each well's near-full-uptime daily rate) to correct for the
   same-day method's tendency to understate loss on exactly its worst days.
5. **Production efficiency** — the project brief's literal `Actual / Potential` formula was
   tested and shown to be mathematically identical to uptime percentage when "potential" uses
   the same-day rate; efficiency here is instead computed against each well's benchmark rate,
   so it measures something uptime doesn't.
6. **Opportunity matrix** — a four-quadrant, median-split prioritization framework
   (production level × downtime level), reconciled against the absolute lost-oil ranking
   where the two disagree.

## Key Metrics

| Metric | Formula |
|---|---|
| Uptime % | `ON_STREAM_HRS / 24 × 100` |
| Downtime Hours | `24 − ON_STREAM_HRS` |
| Potential Production Rate | `BORE_OIL_VOL / ON_STREAM_HRS` |
| Estimated Lost Oil (same-day method) | `Potential Rate × Downtime Hours` |
| Benchmark Rate | 75th percentile of a well's daily rate on `ON_STREAM_HRS ≥ 20` days |
| Estimated Lost Oil (benchmark method) | `Benchmark Rate × Downtime Hours` |
| Production Efficiency % | `BORE_OIL_VOL / (Benchmark Rate × ON_STREAM_HRS) × 100` |

## Key Findings

- Field-wide average uptime was **84.1%**, with **~34,800 total downtime hours** (~1,450
  well-days-equivalent) across all producers.
- **`F-1 C`** is the field's clear reliability outlier — 55.8% average uptime, fully shut in
  on 41.3% of its producing days, highest average daily downtime (10.6 hrs/day) in the field.
- **`F-14 H`** and **`F-12 H`** together hold **over 85% of the field's total estimated
  lost-oil opportunity** (~1.04M bbl and ~0.98M bbl respectively; ~2.37M bbl field-wide, ~24%
  of actual field production) — driven by their scale and by below-benchmark rate efficiency
  (63.8% and 58.3% average efficiency, the two lowest in the field), not by poor uptime (both
  are above the field average).
- A simple relative-downtime quadrant framework classifies both of those wells as lower-urgency
  **"Monitor"**, disagreeing with the absolute lost-oil ranking — a limitation this project
  states explicitly and resolves in favor of the volume-based ranking.
- Field-wide downtime **increased in the later years** of production (2014-2016 vs.
  2010-2013), a trend independent of any single well.

## Engineering Insights

- Uptime, downtime, and rate-efficiency each surface a **different** operational pattern —
  treating every underperforming well as a "downtime problem" would misdirect engineering
  attention on the wells that carry the most barrels.
- A same-day-rate loss estimate systematically **understates** loss on a well's worst days,
  because the rate itself is depressed on exactly those days; a benchmark-rate cross-check is
  necessary to catch this.
- A literal reading of a textbook efficiency formula can silently reduce to a KPI you already
  computed — worth verifying algebraically before reporting it as a new signal.

## Results

All quantitative results, ranking tables, and charts are produced in
[`notebooks/production_loss_and_downtime_optimization.ipynb`](notebooks/production_loss_and_downtime_optimization.ipynb),
with supporting figures saved to [`figures/`](figures/).

## Limitations

- Estimated lost-oil figures are **estimates**, not measured losses, and depend on an assumed
  constant rate; the two methods used here materially disagree in magnitude (though not in
  ranking).
- The dataset does not distinguish planned from unplanned downtime, so all downtime hours are
  treated as equally "recoverable," which likely overstates true recoverable opportunity.
- With only 6 wells, the opportunity-matrix quadrant framework is illustrative, not
  statistically robust, and should be read alongside the absolute lost-oil ranking.
- Pressure, choke, and temperature fields were intentionally excluded from this
  screening-level analysis; they are the natural next step for root-cause diagnosis.

## Future Improvements

- Incorporate downhole pressure, choke size, and wellhead pressure/temperature to diagnose
  *why* the wells flagged here underperform.
- If planned-maintenance logs become available, split downtime into planned vs. unplanned to
  produce a more defensible "recoverable loss" figure.
- Extend the opportunity-matrix framework to a larger well count where quartile- or
  percentile-based cuts would be more statistically meaningful than a 6-well median split.

## Technologies Used

- Python, Pandas, NumPy
- Matplotlib
- Jupyter Notebook
- openpyxl (Excel I/O)

## Repository Structure

```
Production-Loss-Assessment-Downtime-Optimization/
│
├── data/                 # Volve daily production data (Equinor Open Data Licence)
├── notebooks/            # Full analysis notebook (executed, with outputs)
├── figures/              # Exported chart images
├── README.md
├── requirements.txt
└── presentation/         # Interview prep notes
```

## Data License & Attribution

This project uses the Volve field production dataset, released by **Equinor** and the Volve
licence partners (including **ExxonMobil Exploration & Production Norway AS**) under the
[Equinor Open Data Licence](https://www.equinor.com/energy/volve-data-sharing), for research,
study, and development purposes. This repository is a derivative, non-commercial analysis and
attributes the original data to Equinor and the Volve licence partners.
