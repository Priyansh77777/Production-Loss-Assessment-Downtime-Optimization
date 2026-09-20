# Volve Field Production Performance & Well Optimization

A Production Engineering analysis of Equinor's open **Volve field** dataset — built to demonstrate production-engineering thinking (rather than standard data science or reservoir simulation workflows).

This project answers one fundamental question: **which wells deserve engineering attention, and why?** It focuses not just on what happened to production, but why it is happening and what a Production Engineer would investigate next.

## Project Overview

Volve was a North Sea oil field produced by Equinor (then Statoil) from 2008–2016, with the full daily production dataset released publicly under an open license. This project uses that dataset to run the kind of systematic review a Production Engineer would conduct on a real field:

1. Clean and validate the daily well data (catching real physical data-quality issues, not just handling missing values).
2. Establish which wells actually carry the field's production.
3. Track water cut and decline trends well-by-well.
4. Sanity-check what is driving oil rate — with an explicit correlation-vs-causation check.
5. Build a transparent, defensible screening score to prioritize wells for engineering review.
6. Write cautious, data-supported recommendations — rather than prescriptive fixes the data cannot justify.

Every analytical step is followed by a **Production Engineering Interpretation** — explaining what was done, why it matters, and how an engineer uses the result.

## Dataset Description

* **Source:** Equinor Volve field open dataset, "Daily Production Data" sheet (`data/Volve_production_data.xlsx`)
* **License:** Equinor released the Volve dataset under CC BY-NC-SA 4.0 for education/research use
* **Scope:** 15,634 raw daily records across 7 wellbores (producers and injectors), Sep 2007 – Dec 2016
* **In scope for this project:** 6 producing wells only (`WELL_TYPE == 'OP'` and `FLOW_KIND == 'production'`) — **F-1 C, F-5 AH, F-11 H, F-12 H, F-14 H, F-15 D** — 9,143 producer rows, Feb 2008 – Sep 2016
* **Key fields:** `ON_STREAM_HRS`, `AVG_CHOKE_SIZE_P`, `AVG_WHP_P`, `AVG_DOWNHOLE_PRESSURE`, `AVG_DP_TUBING`, `BORE_OIL_VOL`, `BORE_GAS_VOL`, `BORE_WAT_VOL`

## Methodology

| Phase | Action |
| --- | --- |
| **Cleaning** | Filtered to producer/production records; clipped 4 immaterial negative water-volume rows to 0; dropped 1 internally inconsistent row (0 on-stream hours, oil > 0); verified 13 apparent ">24 hour" days as genuine EU daylight-saving 25-hour calendar days. |
| **Data Quality** | Identified and nulled 1,924 physically impossible zero downhole-pressure readings during flowing days — traced to a **permanent downhole gauge failure on F-12 H** from Oct 2010 onward (~67% of its flowing days) plus isolated blips on other wells. |
| **Field Overview** | Calculated cumulative oil/gas/water and uptime by well. |
| **Water Cut** | Calculated lifetime volume-weighted water cut, plus early-vs-late monthly trend per well. |
| **Decline Analysis** | Performed log-linear fit of monthly oil volume vs. time to establish a simple monthly decline % per well. |
| **Operating Conditions** | Calculated per-well Pearson correlation of choke/WHP/downhole pressure/tubing ΔP vs. on-stream-normalized oil rate, deliberately **not pooled across wells** to avoid confounding; included explicit discussion of a time-confounded choke-vs-rate correlation. |
| **Screening** | Built a transparent weighted score (decline 35% / water cut 30% / downtime 15% / pressure trend 20%), normalized 0–1 within the well set, and classified into High/Medium/Low priority. |

## Key Findings

* **F-12 H and F-14 H together produce ~85% of field oil** (45.6% and 39.3% respectively) — the field's plateau wells. F-11 H is a strong secondary contributor (11.4%) with the field's best uptime (92.8%).
* **Water cut rose sharply on every well with sufficient history** — from near-0% early in each well's life to 59–97% in recent producing months. This is the dominant story in the dataset, consistent with water breakthrough from the field's injection scheme.
* **Downhole pressure is flat to slightly increasing**, not declining, on wells with valid gauge data — indicating the oil decline is driven mainly by rising water cut / fractional flow rather than reservoir pressure depletion.
* **Choke opening is negatively correlated with oil rate on 4 of 6 wells** — read as a time-confounded relationship (chokes were progressively opened as the field aged and rates fell), not evidence that opening the choke reduces rate. This is an explicit correlation-vs-causation check.
* **Screening flags F-1 C, F-12 H, and F-15 D as High Priority** — for three different reasons (steep decline/low uptime, water-handling scale on the top producer, and a moderate signal on a small well that needs confirmation), each explained rather than just ranked.

## Engineering Insights

* A data-quality check based on physical reality (is a value *physically possible*) surfaced a real instrumentation issue — the F-12 H downhole gauge failure — independent of the production analysis itself.
* Reading rate-vs-pressure-vs-water-cut together changes the diagnosis: pressure alone would suggest a healthy reservoir; oil rate alone would suggest across-the-board decline. Together, they point to water breakthrough as the primary driver.
* Correlation results were checked well-by-well and against time, specifically to avoid presenting a confounded relationship as causal.

## Technologies Used

* Python, pandas, NumPy
* Matplotlib
* Jupyter Notebook

## Repository Structure

```text
Volve-Production-Performance/
│
├── data/                 # Volve daily production data and exported CSVs
├── notebooks/            # Volve_Field_Production_Performance_Well_Optimization.ipynb
├── figures/              # Exported plots and charts
└── README.md

```

## Results

All screening outputs, summary tables, and figures are fully reproducible from `notebooks/Volve_Field_Production_Performance_Well_Optimization.ipynb`, with outputs saved to the `data/` and `figures/` directories.

## Future Work

* Re-screen F-15 D and F-5 AH once more producing history accumulates (both currently have limited data behind their scores).
* Incorporate injection-allocation data, if available, to confirm which injector(s) are driving water breakthrough into which producers.
* Extend the decline analysis to a proper Arps hyperbolic/harmonic fit per well for remaining-life estimates.
* Flag the F-12 H downhole gauge failure to a surveillance/instrumentation workflow for repair evaluation.

## Data Source & License

Volve field data released by Equinor under CC BY-NC-SA 4.0. This project uses the dataset for educational and research purposes in accordance with the license terms.
