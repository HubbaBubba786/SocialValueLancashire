# Lancashire Social Value Analysis

An analytical framework evaluating social value indicators across Lancashire's 14
local authorities, featuring:
  - Trend Forecasting: Time-series tracking paired with 3-year predictive horizon
  - Clustering: Descriptive grouping that maps regional profiles
  - Predictive Modeling: A Random Forest architecture using socioeconomic indicators to predict recorded crime rates


## Requirements

```
pip install pandas numpy scipy scikit-learn matplotlib openpyxl
pip install shap   #Optional
```

## Input data

Two places for the data and the notebook expects both:

### `SocialValueData.xlsx`

Sheets under 1000 rows never hit Power BI's copy-paste cap, so these
read straight from the workbook:

| Keyword | Expected columns |
|---|---|
| `ChildrenLowIncome` | Local Authority, Type of Low Income Family, Year, Percentage |
| `WasteRecycled` | Local Authority, Attribute, Year, Value |
| `CrimeRates` | Local Authority, Attribute, Value |
| `IMD` | Attribute, Local Authority District name (2024), Deprivation Measures, Value |
| `Post16-18Employment` | Local Authority, Sex, Institution Type, Academic Year, Attribute, Value |
| `AdultsActive` | Local Authority, Date Range, Value |
| `B&BHouseholds` | Household Type, Local Authority, Attribute, Value |
| `TempAccom+DutiesOwed` | Local Authority, Attribute, Value |

### `datasets/` folder

Sheets that were exactly 1000 rows in the workbook were truncated. These now load
from full CSV exports taken directly from the Power BI model instead, matched by
exact filename:

| File | Expected columns | Replaces workbook sheet |
|---|---|---|
| `GrossDisposableIncome.csv` | Region name, Transaction, Year, Sum of Value | GrossDisposableIncome (was 3/14 LAs, now 13/14) |
| `GreenhouseEmissions.csv` | Attribute, Local Authority, Sum of Value, Year | not previously used (new) |
| `ResidualWaste.csv` | Local Authority, Attribute, Financial Year, Sum of Value | not previously used (new) |
| `Fingertips.csv` | Local Authority, Sex, Statistics, Time period, Indicator Name, Age, Sum of Value | FingertipsData (was 1 indicator, now 7) |
| `LifeSatisfaction.csv` | ScoreType, Score, Value, Year, Local Authority | LifeSatisfaction (life satisfaction specifically was 7/14 LAs, now 13/14) |
| `SleepingRough.csv` | Local Authority, Time Range, Year, Month, Sum of Value | SleepingRough (long-term count was 1/14 LAs, now 14/14) |


## How to run

Open `Pipeline.ipynb` and run the cells top to bottom (or **Restart & Run All**).
Each cell builds on variables from the ones before it, so they need to run in
order the first time through. Make sure `SocialValueData.xlsx` and the
`datasets/` folder (with all six CSVs) both sit alongside the notebook.

## What each cell does

1. **Imports** : pandas/numpy/sklearn/matplotlib, optional shap
2. **Settings** : file paths, the 14 local authorities of Lancashire, the
   Fingertips indicator/age-band lookup, forecast thresholds, plot styling
3. **Helper code** : name-cleaning, sheet-loading, CSV-loading, and
   Fingertips time-period-to-year helpers
- **1 - loading workbook** : opens the Excel file and loads the 8 sheets under
   1000 rows, then loads the 6 full CSV exports from `datasets/`
- **2 - tidy sheets** : reshapes each source into one common
   `(local_authority, year, indicator, value)` format and stacks them into one
   long panel - 20 longitudinal indicators in total (see below)
- **3 - cross sectional sheets** : builds the crime target (single snapshot,
   not a time series), the IMD deprivation features, and the B&B
   households / temporary accommodation / duty-decision features (also
   single-snapshot)
- **4 - trajectory features** : fits a trend line per local authority per
   indicator, giving a `level` (mean) and `trend` (slope) feature for each
- **5 - forecasting** : for indicators with enough LA/year coverage, forecasts
   3 years ahead with a 95% interval
- **6 - assemble base table** : merges everything into one row per local
   authority, reports feature completeness, imputes missing values, checks for
   highly correlated (redundant) features
- **7 - clustering** : compares KMeans/Agglomerative/GMM across several k
    values, picks the best by silhouette score, checks cluster stability via
    bootstrap resampling, and checks whether the clusters actually track crime
    levels
- **8 - predictive modelling** : Random Forest with Leave-One-Out CV,
    comparing raw features vs PCA-reduced features, with feature importance
    (RF and SHAP) projected back onto the original indicators
- **9 - figures** : generates all output charts

### Longitudinal indicators (level + trend each)

`gdhi`, `children_low_income_relative`, `children_low_income_absolute`,
`pct_waste_recycled`, `residual_waste_per_household_kg`, `anxiety_high_pct`,
`life_satisfaction_low_pct`, `rough_sleeping_single_night`,
`rough_sleeping_longterm`, `post16_18_positive_destination_pct`,
`pct_adults_inactive`, `household_recycling_rejects_tonnes`,
`per_capita_emissions_tCO2e`, `violent_crime_hospital_admissions`,
`mental_health_qof_prevalence`, `smoking_qof_prevalence`,
`violent_crime_sexual_offences_per_1000`,
`violent_crime_violence_offences_per_1000`, `reoffending_pct`,
`obesity_prevalence_adults`.

### Cross-sectional features (single snapshot, no trend)

`mean_imd_score` + 6 IMD sub-domains, `temp_accom_households`,
`temp_accom_dependent_children_households`,
`temp_accom_sent_elsewhere_per_1000`, `homelessness_duty_accepted_pct`.
`total_recorded_crime` is the prediction target, not a feature.

51 features in total feed the model.

## Outputs

**`results_csv/`**
- `trend_forecasts.csv` : 3-year forecasts per indicator/LA
- `correlation_matrix_full.csv` / `correlation_high_pairs.csv` : full feature correlations
- `algorithm_comparison.csv` : silhouette scores per clustering algorithm/k
- `bootstrap_ari_values.csv` : the 200 bootstrap stability scores
- `results_full_by_local_authority.csv` : every feature + cluster + crime, one row per LA
- `loo_predictions.csv` : actual vs predicted crime (raw features and PCA)
- `correlation_matrix_display.csv` : the curated subset shown in Fig 3
- `feature_importance.csv` : RF + SHAP importance side by side (only if shap is installed)

**`figures/`**
- `fig1` algorithm comparison, `fig2` bootstrap stability, `fig3` correlation
  heatmap, `fig4` cluster scatter, `fig5` feature importance, `fig6` LOO actual
  vs predicted, `fig7` crime by cluster
- `fig8_timeseries_by_la/<indicator>/<local_authority>.png` : one chart per
  indicator per LA, history + forecast - one subfolder per longitudinal
  indicator that clears the forecast coverage threshold

