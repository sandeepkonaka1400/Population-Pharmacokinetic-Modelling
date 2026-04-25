# Population Pharmacokinetic Analysis of an Oral Analgesic

![Language](https://img.shields.io/badge/Language-Julia-9558B2?style=flat-square&logo=julia&logoColor=white)
![Tool](https://img.shields.io/badge/Tool-Pumas%20v2.8.0-4C8CBF?style=flat-square)
![Report](https://img.shields.io/badge/Report-Quarto-4A90D9?style=flat-square)
![Lines](https://img.shields.io/badge/Code-1423%20lines-lightgrey?style=flat-square)
![Models](https://img.shields.io/badge/Models%20Fitted-12-blueviolet?style=flat-square)

A full end-to-end population pharmacokinetic (PopPK) analysis of an investigational oral analgesic compound, written as a reproducible Quarto report executed in Julia using Pumas v2.8.0. Covers non-compartmental analysis, structural model development, IIV evaluation, and model selection across 12 candidate models.

https://sandeep-konakagautamdas.github.io/Population-Pharmacokinetic-Modelling/
---

## Study Design

- **120 subjects** across three single-dose groups: **5 mg, 20 mg, and 80 mg** (oral)
- Plasma concentrations collected at **12 time points** per subject (0–8 hours post-dose)
- Covariates: age (18–55 years), body weight (40–110 kg)
- No placebo arm; all subjects received active drug

---

## Analysis Structure

### Section 1 — Introduction & Objectives
- Describes the clinical context and study design
- States primary objectives: develop a PopPK model, quantify IIV, assess dose proportionality, and identify covariates

### Section 2 — Data Summary & Exploratory Analysis
- Demographics table stratified by dose group using `SummaryTables`
- Concentration summary by time and dose (mean, SD, n)
- Data quality assessment: missing value audit, IQR-based outlier detection (8 outliers identified; all retained)
- **Spaghetti plot** of individual concentration–time profiles by dose
- **Mean ± SD concentration–time profiles** with Cmax annotations and terminal phase labelling, using `CairoMakie` and `AlgebraOfGraphics`

### Section 3 — Non-Compartmental Analysis (NCA)
Performed using Pumas `read_nca` / `run_nca`:

| Parameter | Finding |
|-----------|---------|
| Cmax | 0.356 mg/L (5 mg) → 5.79 mg/L (80 mg) |
| Tmax | ~0.7 h — consistent across all doses |
| AUC∞ | 1.60 → 29.5 mg·h/L |
| t½ | 3.5–4.1 h (stable across doses) |
| CL/F | 2.95–3.52 L/h |
| Vz/F | ~16.5 L |

- **Dose proportionality** evaluated via power model (`DoseLinearityPowerModel`) for Cmax and AUC∞
  - Cmax: β ≈ 1 → dose-proportional
  - AUC∞: β slightly > 1 → mild supra-proportionality at higher doses, suggesting possible saturation of elimination
- **NCA quality flags:** 45% of subjects had AUC extrapolation > 20%; 4 subjects flagged with > 40% extrapolation (IDs: 32, 94, 100, 104)
- **Multiple-dose simulation** via superposition: BID at 100 mg produces ~11% higher Cmax,ss vs QD — consistent with linear PK

### Section 4 — Population PK Model Development

Initial parameter estimates (CL, V, Ka) derived directly from NCA. IIV initialised at 30% CV; residual error at 20%.

#### One-Compartment Models (6 models)

| Model | Estimation |
|-------|-----------|
| 1-CMT, first-order elimination (ke only) | NaivePooled |
| 1-CMT, first-order absorption (Ka) | NaivePooled |
| 1-CMT + IIV on CL, V, Ka | FOCE |
| 1-CMT + IIV + additive error | FOCE |
| 1-CMT + IIV + proportional error | FOCE |
| 1-CMT + IIV + combined error | FOCE |

Combined error consistently outperformed additive and proportional structures. However, biphasic decline in individual profiles indicated a two-compartment structure was needed.

#### Two-Compartment Models (6 models)

| Model | Estimation |
|-------|-----------|
| 2-CMT + Ka (base) | NaivePooled |
| 2-CMT + full IIV (CL, V1, Q, V2, Ka) | FOCE |
| 2-CMT + full IIV + combined error | FOCE |
| 2-CMT, no IIV on V2 | FOCE |
| 2-CMT, no IIV on Q | FOCE |
| 2-CMT, no IIV on Ka | FOCE |

IIV reduction was performed stepwise based on shrinkage and parameter uncertainty. IIV on V2 was removed first — it did not meaningfully improve fit and increased parameter uncertainty.

#### Final Model

> **Two-compartment model with first-order absorption**
> IIV on: CL, V1, Q, Ka (exponential random effects)
> IIV removed from: V2
> Residual error: proportional
> Estimation: FOCE

Selected based on **lowest BIC** across all 12 candidate models. AIC and ΔOFV were comparable to the combined-error two-compartment variant, but the simpler error structure was preferred on grounds of parsimony and biological plausibility.

---

## Files

| File | Description |
|------|-------------|
| `SandeepKG_PopPK_Report.qmd` | Full analysis — EDA, NCA, model development, model comparison |
| `pk_painrelief_new.csv` | Clinical PK dataset (120 subjects, 3 oral dose groups) |

---

## Dependencies

Requires Julia with Pumas v2.8.0:

```julia
using Pumas, PumasUtilities, SummaryTables, PharmaDatasets
using CSV, DataFramesMeta, CategoricalArrays
using AlgebraOfGraphics, CairoMakie, PairPlots
using Statistics, StatsBase, Distributions
using GLM, LinearAlgebra, StatsPlots
```

To render the report:
```bash
quarto render SandeepKG_PopPK_Report.qmd
```

---

## How to Run

1. Install [Pumas v2.8.0](https://pumas.ai) and [Quarto](https://quarto.org)
2. Update the `cd(...)` path in the first code chunk to point to your local data directory
3. Run `quarto render SandeepKG_PopPK_Report.qmd` — the report renders to HTML with all figures, tables, and model outputs inline

---

## Context

Completed as the model building exercise in the third semester of the **Certificate Course in Pharmacometrics** — Society of Pharmacometrics and Health Analytics. Demonstrates a full model-informed drug development (MIDD) workflow from raw PK data through NCA, structural model selection, and population parameter estimation in Pumas.

---

*Part of my pharmacometrics portfolio — see my [GitHub profile](https://github.com/sandeep-konakagautamdas) for more.*
