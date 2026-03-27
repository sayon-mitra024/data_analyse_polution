# Air Quality Analysis Across Indian States

**Multi-source analysis of CPCB real-time pollutant data and NFHS-5 socioeconomic indicators**

> Data sources: Central Pollution Control Board (CPCB) — data.gov.in; National Family Health Survey 5 (NFHS-5, 2019–21)

---

## Overview

This project analyses real-time air pollutant concentration data from the CPCB's Continuous Ambient Air Quality Monitoring (CAAQM) network across 29 Indian states, integrates the findings with state-level socioeconomic and health indicators from NFHS-5, and applies exploratory statistics, unsupervised clustering, and supervised regression to characterise India's air quality landscape.

Key results are documented in [`research_paper.md`](research_paper.md) — a full IEEE/Elsevier-style research paper.

---

## Repository Contents

| File | Description |
|---|---|
| `CPCB_AQI_Analysis.ipynb` | Main analysis notebook (53 cells, fully executable) |
| `aqi.csv` | CPCB real-time AQI dataset (3,411 rows, 29 states, 7 pollutants) |
| `health.xls` | NFHS-5 state-level socioeconomic and health indicators |
| `research_paper.md` | Publication-quality research paper |
| `fig1_pollutant_distribution.png` | Pollutant distribution histogram and box-plot |
| `fig2_top_polluted_states.png` | Top 15 most polluted states (horizontal bar chart) |
| `fig3_aqi_categories.png` | AQI category distribution (bar + pie) |
| `fig4_state_pollutant_heatmap.png` | Normalised state × pollutant heatmap |
| `fig5_correlation_matrix.png` | Pearson correlation matrix |
| `fig6_scatter_plots.png` | Scatter plots — pollution vs socioeconomic indicators |
| `fig7_elbow.png` | K-Means elbow method |
| `fig8_kmeans_clusters.png` | K-Means cluster visualisation |
| `fig9_feature_importance.png` | Random Forest feature importance |
| `fig10_actual_vs_predicted.png` | Actual vs predicted pollution (best model) |
| `fig11_lr_coefficients.png` | Linear Regression standardised coefficients |
| `requirements.txt` | Python dependency list |

---

## Setup & Reproduction

### 1. Install dependencies

```bash
pip install -r requirements.txt
```

### 2. Run the notebook

```bash
jupyter notebook CPCB_AQI_Analysis.ipynb
```

Or execute all cells non-interactively:

```bash
jupyter nbconvert --to notebook --execute CPCB_AQI_Analysis.ipynb --output CPCB_AQI_Analysis_executed.ipynb
```

The notebook saves all 11 figures to the working directory automatically.

---

## Key Findings

| Finding | Value |
|---|---|
| PM10 mean concentration | 96.11 µg/m³ (6× WHO guideline) |
| PM2.5 mean concentration | 76.60 µg/m³ (15× WHO guideline) |
| Most polluted state | Delhi (mean: 64.55 µg/m³) |
| States matched (CPCB × NFHS-5) | 29 of 37 |
| Significant correlations with pollution | Infant mortality (r=+0.45, p=0.017); Under-5 mortality (r=+0.44, p=0.021) |
| Best model test R² | +0.09 (Random Forest) |
| K-Means clusters identified | 3 |

---

## Methodology Summary

1. **Data cleaning** — missing value removal, state name normalisation (underscore→space harmonisation)
2. **Descriptive statistics** — per-pollutant and per-state summaries
3. **AQI categorisation** — CPCB sub-index thresholds (Good / Satisfactory / Moderate / Poor / Very Poor / Severe)
4. **Correlation analysis** — Pearson r with two-tailed p-values
5. **K-Means clustering** — Elbow Method to select k=3; four features (pollution, electricity, sanitation, female education)
6. **Predictive modelling** — four regression algorithms in `sklearn.Pipeline` with 5-fold cross-validation (no data leakage)

---

## Citation

If you use this work, please cite:

> CPCB AQI Research Group (2026). *Air Quality Analysis Across Indian States: Pollutant Distribution, Socioeconomic Correlates, and Predictive Modelling.* Government of India Open Data (data.gov.in).

---

## Data Sources

- **CPCB AQI data:** [https://data.gov.in](https://data.gov.in) — Central Pollution Control Board real-time monitoring
- **NFHS-5:** Ministry of Health and Family Welfare, Government of India (2021). *National Family Health Survey-5, 2019–21.*
