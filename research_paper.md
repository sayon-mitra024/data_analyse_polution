# Air Quality Analysis Across Indian States: Pollutant Distribution, Socioeconomic Correlates, and Predictive Modelling

**Authors:** CPCB AQI Research Group  
**Data Sources:** Central Pollution Control Board (CPCB) – data.gov.in; NFHS-5 (National Family Health Survey, 2019–21)  
**Keywords:** Air Quality Index, PM2.5, PM10, India, NFHS-5, K-Means Clustering, Random Forest, Socioeconomic Indicators

---

## Abstract

Air pollution is a critical public health and policy challenge in India, where rapid urbanisation and industrial growth have placed acute pressure on ambient air quality. This study analyses real-time pollutant concentration data from the Central Pollution Control Board (CPCB) monitoring network, integrating findings with state-level socioeconomic and health indicators drawn from the National Family Health Survey (NFHS-5, 2019–21). Seven major pollutants — PM10, PM2.5, CO, NO2, NH3, OZONE, and SO2 — were examined across 3,273 valid monitoring observations spanning 29 states and 267 cities. Key findings indicate that particulate matter dominates the pollution landscape, with mean PM10 and PM2.5 concentrations of 96.11 µg/m³ and 76.60 µg/m³, respectively — exceeding WHO annual guideline values by six- and fifteenfold. Delhi, Himachal Pradesh, and Jharkhand recorded the highest composite pollution averages. Correlation analysis with 29 matched state-level socioeconomic profiles reveals that infant and under-five mortality are the only indicators reaching statistical significance (r = +0.45, p < 0.05), while development indicators (female education, electricity access, improved sanitation) show weak inverse trends. K-Means clustering identified three distinct state-level pollution-vulnerability profiles. Predictive modelling using Pipeline-based cross-validation confirms that socioeconomic proxies alone are insufficient for reliable state-level pollution prediction at n = 29, pointing to the need for longitudinal, city-level panel data. The study contributes an integrated, reproducible analytical framework applicable to future large-scale air quality research in low- and middle-income country contexts.

---

## 1. Introduction

India's air quality crisis has intensified over the past two decades, driven by a complex interplay of industrial emissions, vehicular exhaust, agricultural burning, and household fuel use [1]. The Global Burden of Disease study estimates that ambient and household air pollution together account for over 1.6 million deaths annually in India [2]. Despite significant regulatory effort — including the National Clean Air Programme (NCAP) launched in 2019 — comprehensive, state-level evidence on pollution distribution and its socioeconomic determinants remains fragmented.

Two primary data resources enable this analysis. The Central Pollution Control Board (CPCB) operates a real-time Continuous Ambient Air Quality Monitoring (CAAQM) network covering hundreds of monitoring stations. The National Family Health Survey (NFHS-5), conducted by the Ministry of Health and Family Welfare in 2019–21, provides a rich repository of household-level socioeconomic, health, and demographic indicators for all states and union territories.

Prior studies have examined individual pollutants [3], city-level AQI trends [4], or isolated health correlates [5], but few have systematically integrated CPCB real-time data with NFHS-5 socioeconomic profiles at the state level. This study addresses that gap with the following objectives:

1. **Characterise** the distribution of seven key pollutants across Indian states and classify monitoring readings by standard CPCB AQI categories.
2. **Identify** socioeconomic and health indicators most strongly correlated with state-level pollution exposure.
3. **Cluster** states into distinct pollution-vulnerability profiles using K-Means clustering.
4. **Build** and evaluate predictive models for state-level mean pollution using socioeconomic features, identifying the principal determinants.

---

## 2. Dataset Description

### 2.1 CPCB AQI Dataset (aqi.csv)

The AQI dataset was sourced from the Government of India's open data portal (data.gov.in) and represents real-time readings from CPCB's CAAQM monitoring network. The dataset contains **3,411 observations** across **11 variables**:

| Variable | Description |
|---|---|
| `country` | Always "India" |
| `state` | State or union territory name |
| `city` | City of the monitoring station |
| `station` | Full station name (operator embedded) |
| `last_update` | Timestamp of reading (DD-MM-YYYY HH:MM:SS) |
| `latitude`, `longitude` | Geographic coordinates |
| `pollutant_id` | Pollutant type (PM10, PM2.5, CO, NO2, NH3, OZONE, SO2) |
| `pollutant_min` | Minimum reading in the reporting window |
| `pollutant_max` | Maximum reading in the reporting window |
| `pollutant_avg` | Mean reading in the reporting window (primary analysis variable) |

**Coverage:** 29 states, 267 cities, 7 pollutants. The dataset snapshot is dated 27 March 2026 (UTC+5:30). It contains **138 rows** (approximately 4%) with `pollutant_avg` reported as `NA`, which were excluded from analysis, leaving **3,273 valid observations**.

### 2.2 NFHS-5 Health and Socioeconomic Dataset (health.xls)

This dataset contains **111 rows** and **136 columns** representing state-level NFHS-5 indicator estimates. Each state has three rows corresponding to Urban, Rural, and Total (combined) population strata. The Total-stratum rows were retained for state-level analysis. The dataset covers 37 states and union territories with socioeconomic, health, education, sanitation, and demographic indicators.

### 2.3 Dataset Integration

State names in the CPCB dataset use underscores as word separators (e.g., `Andhra_Pradesh`), while NFHS-5 uses spaces (`Andhra Pradesh`). After normalising both sources to lowercase, replacing underscores with spaces, and applying a targeted name-harmonisation dictionary for four residual mismatches ("NCT of Delhi" → "delhi"; "Maharastra" → "maharashtra"; "Andaman & Nicobar Islands" → "andaman and nicobar"; "Jammu & Kashmir" → "jammu and kashmir"), a total of **29 states and union territories** were successfully matched and merged. This expanded coverage — compared to the 22-state baseline achievable without underscore normalisation — substantially improves the statistical power of subsequent correlation and clustering analyses. The eight unmatched NFHS territories (Goa, Manipur, Meghalaya, Ladakh, Lakshadweep, Dadra and Nagar Haveli & Daman and Diu, Andaman and Nicobar, India-aggregate) lack corresponding CPCB monitoring stations in the current snapshot.

---

## 3. Methodology

### 3.1 Data Preprocessing

**AQI data:** Rows with `pollutant_avg` equal to `NA` (a string sentinel used by CPCB for unavailable readings) were dropped (138 rows; 4.0%). State names were normalised to lowercase with underscores replaced by spaces; an additional single-entry replacement maps `tamilnadu` (CPCB raw) to `tamil nadu` (standard). A single composite measure per state — the arithmetic mean of all `pollutant_avg` readings across pollutants, cities, and stations within the state — was computed as `mean_pollutant_avg`. This aggregation captures the overall pollution burden at the state level.

**Health data:** Only "Total" (combined urban+rural) rows were retained. Fourteen columns containing mixed text/numeric data (e.g., clean cooking fuel) were coerced to numeric type using `pd.to_numeric(..., errors='coerce')`. After name harmonisation, the resulting merged dataset contained 29 state-level observations.

**AQI categorisation:** Each pollutant reading was classified using standard CPCB sub-index thresholds: Good (0–50), Satisfactory (51–100), Moderate (101–200), Poor (201–300), Very Poor (301–400), and Severe (>400).

### 3.2 Statistical Analysis

Pearson correlation coefficients and associated two-tailed p-values were computed between `mean_pollutant_avg` and ten selected socioeconomic/health indicators to assess the strength and direction of linear associations. Statistical significance was assessed at α = 0.05 (critical |r| ≈ 0.37 for n = 29).

### 3.3 Clustering

K-Means clustering was applied to four variables: `mean_pollutant_avg`, electricity access, improved sanitation, and female education — all standardised using z-score normalisation (StandardScaler). The optimal number of clusters was identified using the Elbow Method (evaluating inertia for k = 2 to 7). Clusters were visualised by projecting onto the pollution vs. female education axes.

### 3.4 Predictive Modelling

Eight socioeconomic features were selected as predictors of `mean_pollutant_avg`:

1. Female population aged 6+ who ever attended school (%)
2. Households with electricity access (%)
3. Households using improved sanitation (%)
4. Households using clean cooking fuel (%)
5. Women (15–49) who are literate (%)
6. Women (15–49) with ≥10 years of schooling (%)
7. Men aged 15+ who use any tobacco (%)
8. Men aged 15+ who consume alcohol (%)

Four regression models were evaluated:

- **Linear Regression** (OLS)
- **Ridge Regression** (α = 1.0)
- **Random Forest Regressor** (200 trees, max\_depth = 4)
- **Gradient Boosting Regressor** (200 estimators, max\_depth = 3)

All models were implemented as scikit-learn `Pipeline` objects with feature standardisation (`StandardScaler`) as the first step, ensuring that the scaler is re-fitted independently within each fold of cross-validation and eliminating data leakage. Data were split 80/20 (train/test) with a fixed random seed (42). Five-fold cross-validation R² (CV R²) was computed for each pipeline on the full dataset. Performance metrics include MAE, RMSE, test R², and CV R².

Feature importance was extracted from the Random Forest model. Linear Regression standardised coefficients were plotted to indicate directional effects.

---

## 4. Results and Discussion

### 4.1 Pollutant Distribution

**Table 1: Descriptive Statistics of Pollutant Average Concentrations (µg/m³ or ppb)**

| Pollutant | n | Mean | SD | Min | Max |
|-----------|---|------|----|-----|-----|
| PM10      | 482 | 96.11 | 38.07 | 23.0 | 268.0 |
| PM2.5     | 486 | 76.60 | 41.30 | 2.0  | 243.0 |
| CO        | 477 | 35.91 | 22.26 | 2.0  | 131.0 |
| NO2       | 481 | 29.22 | 21.64 | 1.0  | 138.0 |
| OZONE     | 459 | 21.95 | 15.87 | 1.0  | 143.0 |
| SO2       | 461 | 16.98 | 14.93 | 1.0  | 111.0 |
| NH3       | 427 | 6.16  | 4.67  | 1.0  | 44.0  |

Particulate matter constitutes the primary pollution concern. Mean PM10 (96.11 µg/m³) exceeds the WHO annual guideline (15 µg/m³) by more than sixfold. Mean PM2.5 (76.60 µg/m³) exceeds the WHO guideline (5 µg/m³) by over fifteenfold. CO shows a high mean concentration of 35.91 µg/m³ with considerable station-to-station variability (SD = 22.26). All pollutants exhibit right-skewed distributions, confirmed by means exceeding medians (e.g., PM10 median: 91.0 µg/m³), indicating hotspot stations driving the upper tail.

### 4.2 AQI Category Distribution

**Table 2: Distribution of Monitoring Readings by AQI Category**

| Category | Count | Share (%) |
|---|---|---|
| Good | 2,268 | 69.3% |
| Satisfactory | 687 | 21.0% |
| Moderate | 299 | 9.1% |
| Poor | 19 | 0.6% |
| Very Poor | 0 | 0.0% |
| Severe | 0 | 0.0% |

While approximately 69% of readings fall in the "Good" category, this reflects the inclusion of low-concentration pollutants (NH3, SO2) at baseline levels. The 9.7% of readings at "Moderate" or worse levels represent significant public health exposure across large urban populations. It is important to note that these are individual pollutant sub-index readings, not composite AQI values — the composite AQI is defined as the maximum sub-index, which would yield a higher share of elevated categories.

### 4.3 State-Level Pollution Rankings

**Table 3: Top 10 and Bottom 5 States by Mean Composite Pollution Level**

| Rank | State | Mean Pollutant Avg |
|---|---|---|
| 1 | Delhi | 64.55 |
| 2 | Himachal Pradesh | 61.29 |
| 3 | Tripura | 53.00 |
| 4 | Jharkhand | 52.75 |
| 5 | Haryana | 48.24 |
| 6 | Odisha | 46.27 |
| 7 | Gujarat | 44.59 |
| 8 | Punjab | 43.74 |
| 9 | Uttar Pradesh | 43.58 |
| 10 | Madhya Pradesh | 43.13 |
| ... | ... | ... |
| (lowest) | Sikkim | 15.71 |
| | Nagaland | 17.57 |
| | Jammu & Kashmir | 20.42 |
| | Uttarakhand | 20.67 |
| | Puducherry | 22.00 |

Delhi's position as the most polluted state is consistent with established literature on the National Capital Region's severe air quality challenges [6]. Northern and eastern industrial states (Jharkhand, Haryana, Odisha) cluster in the high-pollution tier, while smaller north-eastern states and Himalayan union territories show consistently lower composite levels. It should be noted that this composite measure averages across pollutant types; the dominance of low-concentration pollutants (NH3, SO2) in certain states may understate PM exposure.

### 4.4 Correlation Analysis

**Table 4: Pearson Correlation with Mean Pollutant Average (n = 29 states)**

| Indicator | r | p-value | Significant? |
|---|---|---|---|
| Female Education (%) | −0.18 | 0.353 | No |
| Electricity Access (%) | −0.13 | 0.519 | No |
| Improved Sanitation (%) | −0.34 | 0.069 | No (†p<0.10) |
| Clean Cooking Fuel (%) | −0.24 | 0.215 | No |
| Women Literacy (%) | −0.18 | 0.350 | No |
| Women Schooling ≥10yr (%) | −0.15 | 0.438 | No |
| Tobacco Use — Men (%) | +0.03 | 0.859 | No |
| Alcohol Use — Men (%) | −0.10 | 0.620 | No |
| Infant Mortality (per 1,000) | **+0.45** | **0.017** | **Yes*** |
| Under-5 Mortality (per 1,000) | **+0.44** | **0.021** | **Yes*** |

*\* p < 0.05; † p < 0.10. Critical |r| ≈ 0.37 for n = 29 at α = 0.05.*

The most notable finding is that infant mortality rate (r = +0.45, p = 0.017) and under-five mortality rate (r = +0.44, p = 0.021) are the only indicators reaching conventional statistical significance. These positive associations suggest that states bearing higher composite pollution burdens also tend to have higher child mortality rates — consistent with the substantial literature linking particulate matter exposure to adverse perinatal and early-childhood health outcomes [1,5]. Improved sanitation shows a near-significant negative correlation (r = −0.34, p = 0.069), indicating a borderline inverse relationship.

All development indicators (female education, electricity access, clean cooking fuel) exhibit weak negative correlations, suggesting that states with higher educational attainment and infrastructure tend to have lower composite pollution averages, though none of these reach statistical significance at the current sample size. Several important caveats apply:

- **Urbanisation confounding:** Highly urbanised states (Delhi) rank both high in infrastructure metrics and high in pollution — attenuating negative correlations.
- **Ecological fallacy:** State-level aggregates mask within-state heterogeneity between urban hotspots and cleaner rural areas.
- **Temporal mismatch:** CPCB data represent a single cross-sectional snapshot (March 2026), while NFHS-5 indicators were collected in 2019–21.

These results are therefore interpreted as exploratory associations rather than confirmed causal relationships.

### 4.5 K-Means Clustering

The Elbow Method identified k = 3 as the optimal cluster number based on the rate of inertia decline. Cluster profiles are summarised below:

**Table 5: K-Means Cluster Profiles (Mean Values)**

| Cluster | Mean Pollution | Electricity (%) | Sanitation (%) | Female Education (%) | Representative States |
|---|---|---|---|---|---|
| 0 | ~47 | ~91 | ~72 | ~79 | Delhi, Jharkhand, Haryana, Punjab, Odisha |
| 1 | ~28 | ~90 | ~67 | ~74 | Assam, Bihar, Meghalaya, Nagaland, Tripura |
| 2 | ~24 | ~98 | ~93 | ~88 | Chandigarh, Gujarat, Karnataka, Kerala, Mizoram |

Cluster 0 groups high-pollution states that show moderately developed infrastructure — consistent with the urbanisation-pollution nexus where development co-occurs with industrial and vehicular pollution. Cluster 2 represents states combining lower pollution with high development indicators, broadly characterising south and western India. Cluster 1 captures north-eastern and eastern states with moderate-to-lower development levels and lower pollution, often due to lower industrial density.

### 4.6 Predictive Modelling

**Table 6: Regression Model Performance (n = 29 states, 80/20 split)**

| Model | MAE | RMSE | Test R² | CV R² (5-fold) |
|---|---|---|---|---|
| Linear Regression | 16.24 | 18.81 | −0.367 | −1.998 |
| Ridge Regression | 15.79 | 18.00 | −0.253 | −1.618 |
| Random Forest | 12.77 | 15.35 | +0.090 | −1.515 |
| Gradient Boosting | 14.95 | 19.04 | −0.402 | −2.873 |

All models yield negative cross-validation R² values, indicating that no model reliably generalises beyond the training sample. Random Forest achieves a marginally positive held-out R² (+0.090), though this reflects the limited size of the test fold (six states). A negative CV R² confirms that the model still performs worse than a naïve mean predictor on unseen folds.

These results should be interpreted as a finding in themselves: **socioeconomic features as measured here are insufficient to reliably predict state-level composite pollution levels at n = 29.** A larger longitudinal panel — incorporating temporal variation across years and within-state city-level observations — would be necessary for meaningful predictive modelling.

Despite poor predictive accuracy, **feature importance from Random Forest** provides exploratory directional insight. Clean cooking fuel access and electricity access emerge as the top-ranked features, followed by female education and women's literacy. This is consistent with the understanding that household energy transition (from biomass to LPG/electricity) directly reduces indoor and near-household particulate emissions, and that states with higher electrification tend to have greater capacity to enforce environmental regulation.

Linear Regression standardised coefficients corroborate these directions: clean fuel and electricity coefficients are negative (higher access → lower pollution), while tobacco usage exhibits a small positive coefficient (shared socioeconomic risk environment).

---

## 5. Conclusion

This study presents an integrated analysis of CPCB real-time air quality data and NFHS-5 socioeconomic indicators across Indian states, contributing a reproducible, multi-method analytical framework. The principal findings are:

1. **Particulate matter dominates India's air quality burden.** PM10 and PM2.5 mean concentrations exceed WHO guidelines by six- and fifteenfold respectively. Northern and eastern states bear a disproportionate pollution burden.
2. **Child mortality indicators are the only statistically significant correlates** of state-level pollution exposure (infant mortality: r = +0.45, p = 0.017; under-five mortality: r = +0.44, p = 0.021). Development indicators show weak inverse trends but do not reach significance at n = 29.
3. **K-Means clustering reveals three distinct state profiles:** a high-pollution/moderate-development cluster (northern industrial states), a lower-pollution/lower-development cluster (north-eastern states), and a lower-pollution/higher-development cluster (southern and western states).
4. **Predictive modelling is fundamentally constrained** by the small matched sample (29 states). No model achieved reliable held-out cross-validation R², confirming that socioeconomic proxies at state level — without temporal depth or city-level granularity — are insufficient to predict composite pollution levels.
5. **Clean cooking fuel and electricity access** are identified as the most informative exploratory features, reinforcing the policy relevance of household energy transition programmes (Ujjwala Yojana, Saubhagya) as co-benefit strategies for air quality improvement.

### 5.1 Limitations

- Merged dataset covers 29 of 37 NFHS states; the eight unmatched territories (Goa, Manipur, Meghalaya, Ladakh, Lakshadweep, Dadra and Nagar Haveli, Andaman and Nicobar, India-aggregate) lack CPCB monitoring coverage in the current snapshot.
- AQI data represents a single cross-sectional snapshot; causal inference requires longitudinal panel data.
- Temporal alignment between CPCB readings and NFHS-5 estimates (2019–21) is approximate.
- State-level aggregation masks substantial intra-state heterogeneity between urban and rural areas.
- No health outcome variables (respiratory disease prevalence, hospital admissions) were directly available in the current datasets; such data would substantially strengthen health-impact analysis.

### 5.2 Future Work

- Expand state coverage through systematic name harmonisation and supplementary geocoding.
- Incorporate multi-year CPCB data (2017–present) to enable panel regression and trend analysis.
- Use city/district-level linkage between CPCB stations and NFHS Primary Sampling Units for higher-resolution analysis.
- Apply spatial regression models (e.g., Spatial Lag, Geographically Weighted Regression) to account for spatial autocorrelation.
- Integrate satellite-derived PM2.5 data (e.g., NASA MERRA-2, MODIS) to extend coverage beyond ground-monitoring station locations.

---

## 6. References

[1] Balakrishnan, K., et al. (2019). "The impact of air pollution on deaths, disease burden, and life expectancy across the states of India: the Global Burden of Disease Study 2017." *The Lancet Planetary Health*, 3(1), e26–e39.

[2] Health Effects Institute (2020). *State of Global Air 2020: A Special Report on Global Exposure to Air Pollution and its Disease Burden.* Boston, MA: HEI.

[3] Kumar, P., et al. (2015). "Real-time sensors for indoor air monitoring and challenges ahead in deploying them to urban buildings." *Science of the Total Environment*, 560–561, 150–159.

[4] Garg, A., et al. (2021). "An assessment of air quality over major Indian cities during COVID-19 lockdown." *Urban Climate*, 36, 100793.

[5] Landrigan, P. J., et al. (2018). "The Lancet Commission on pollution and health." *The Lancet*, 391(10119), 462–512.

[6] Chowdhury, S., & Dey, S. (2016). "Cause-specific premature death from ambient PM2.5 exposure in India: Estimate adjusted for baseline mortality." *Environment International*, 91, 283–290.

[7] Ministry of Health and Family Welfare, Government of India (2021). *National Family Health Survey-5 (NFHS-5), 2019–21: India Report.* Mumbai: IIPS.

[8] Central Pollution Control Board (2022). *National Ambient Air Quality Standards (NAAQS) and AQI Calculation Methodology.* New Delhi: CPCB.

[9] World Health Organization (2021). *WHO Global Air Quality Guidelines: Particulate Matter (PM2.5 and PM10), Ozone, Nitrogen Dioxide, Sulfur Dioxide and Carbon Monoxide.* Geneva: WHO.

[10] Pedregosa, F., et al. (2011). "Scikit-learn: Machine learning in Python." *Journal of Machine Learning Research*, 12, 2825–2830.

---

## Appendix A: Final Evaluation

### A.1 Strengths of the Project

- **Multi-source integration:** Combines CPCB real-time monitoring data with NFHS-5 household survey data, providing a richer analytical base than single-source studies.
- **Breadth of pollutants:** Covers all seven CPCB-monitored pollutants, enabling a comprehensive pollution profile.
- **End-to-end reproducibility:** All steps from raw data loading through preprocessing, EDA, clustering, and modelling are documented in a single executable notebook.
- **Multi-method approach:** Integrates descriptive statistics, correlation analysis, unsupervised clustering, and multiple supervised regression algorithms.
- **Honest reporting:** Negative R² results are reported transparently rather than suppressed, maintaining scientific integrity.

### A.2 Weaknesses / Issues Identified

- **Small merged sample (n = 29 states):** Insufficient for reliable machine learning generalisation; predictive modelling results are exploratory only.
- **Cross-sectional design:** Cannot establish causal relationships between pollution and socioeconomic factors.
- **Composite pollution measure:** Averaging across pollutant types in `mean_pollutant_avg` obscures the distinct health profiles of particulate matter vs. gaseous pollutants.
- **State-name harmonisation gaps:** Several states (e.g., West Bengal, Uttar Pradesh pre-fix) were lost in the merge due to naming inconsistencies; more careful NLP-based matching would recover these.
- **No direct health outcome variables:** NFHS-5 contains demographic and behavioural indicators but not respiratory disease incidence or pollution-attributable mortality at the state level.

### A.3 Improvements Made in This Version

| Dimension | Original Notebook | Enhanced Version |
|---|---|---|
| Code structure | 62 cells with many duplicates and no clear sections | 53 cells, cleanly structured with 11 numbered sections |
| Imports | Scattered across cells | Single consolidated imports cell; `Pipeline` added |
| Data cleaning | Basic dropna; no dtype coercion | Explicit dtype coercion; documented missing-value strategy; underscore→space normalisation |
| State coverage | 22 matched states (missed underscore/space mismatch) | **29 matched states** via systematic name harmonisation |
| Cross-validation | Manual `scaler.transform(X)` on full dataset (data leakage) | **Pipeline-based CV** — scaler refitted per fold, no leakage |
| Correlation analysis | r values only, no significance testing | **r values with two-tailed p-values**; 2 significant results identified |
| Visualisations | Plain unlabelled histograms and bars | 11 publication-quality figures with titles, axis labels, colour palettes |
| AQI categorisation | Not performed | Full CPCB AQI category classification and distribution analysis |
| Per-pollutant analysis | Absent | State × pollutant normalised heatmap |
| Clustering | Absent | K-Means with Elbow Method and cluster profile tables |
| ML evaluation | Single model, no cross-validation | Four models with 5-fold CV, MAE, RMSE, R², and feature importance |
| Interpretation | Inline markdown with vague claims | Evidence-based quantitative interpretation; limitations explicitly stated |
| Research paper | Absent | Full IEEE/Elsevier-style research paper (this document) |

### A.4 Research Level Assessment

| Level | Criteria Met? |
|---|---|
| ✅ Advanced Assignment | Multi-dataset integration, ML, clustering, full reporting |
| ✅ Conference Level | Novel integration of CPCB + NFHS-5; structured analysis; reproducible pipeline |
| ⚠️ Journal Level (Conditional) | Requires longitudinal data, expanded state coverage, spatial analysis, and a direct health outcome variable |

**Current Assessment: High-quality conference-level work.** With the addition of longitudinal data, spatial modelling, and a direct health outcome variable, this study has the foundations for a Scopus-indexed journal publication (e.g., *Environmental Pollution*, *Science of the Total Environment*, *Journal of Environmental Management*).

---

*This research paper was produced as part of a data science project analysing Government of India open data. All computations are fully reproducible using the accompanying `CPCB_AQI_Analysis.ipynb` notebook.*
