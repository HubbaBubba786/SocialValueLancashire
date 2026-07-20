# Lancashire Social Value Analysis

An analytical framework evaluating nine public sector indicator sheets across Lancashire's 14 local authorities, featuring:
  - Trend Forecasting: Time-series tracking paired with 3-year predictive horizon
  - Clustering: Descriptive grouping that maps regional profiles
  - Predictive Modeling: A Random Forest architecture using socioeconomic indicators to predict recorded crime rates

## Requirements

```
pip install pandas numpy scipy scikit-learn matplotlib openpyxl
pip install shap   #Optional
```

## Input data

Put `SocialValueData.xlsx` in the same folder as the notebook. Sheets are found by
a loose keyword match on their name, so exact tab naming doesn't matter as long as
each keyword appears somewhere in it:

| Keyword | Expected columns |
|---|---|
| `GrossDisposableIncome` | Region name, Transaction, Year, Value |
| `ChildrenLowIncome` | Local Authority, Type of Low Income Family, Year, Percentage |
| `WasteRecycled` | Local Authority, Attribute, Year, Value |
| `CrimeRates` | Local Authority, Attribute, Value |
| `IMD` | Local Authority District name (2024), Deprivation Measures, Attribute, Value |
| `LifeSatisfaction` | Local Authority, ScoreType, Score, Year, Value |
| `SleepingRough` | Local Authority, Month, Time Range, Value |
| `Post16-18Employment` | Local Authority, Sex, Academic Year, Attribute, Value |
| `AdultsActive` | Local Authority, Date Range, Value |

## How to run

Open `social_value_analysis.ipynb` and run the cells top to bottom (or
**Restart & Run All**). Each cell builds on variables from the ones before it, so
they need to run in order the first time through.

## What each cell does

1. **Imports** : pandas/numpy/sklearn/matplotlib, optional shap
2. **Settings** : file paths, the 14 local authorities of Lancashire, forecast
   thresholds, plot styling
3. **Helper code** : name-cleaning and sheet-loading helpers
4. **1 - loading workbook** : opens the Excel file, loads all 9 sheets
5. **2 - tidy sheets** : reshapes each sheet into one common
   `(local_authority, year, indicator, value)` format and stacks them into one
   long panel
6. **3 - cross sectional sheets** : builds the crime target (single snapshot,
   not a time series) and the IMD deprivation features
7. **4 - trajectory features** : fits a trend line per local authority per
   indicator, giving a `level` (mean) and `trend` (slope) feature for each
8. **5 - forecasting** : for indicators with enough LA/year coverage, forecasts
   3 years ahead with a 95% interval
9. **6 - assemble base table** : merges everything into one row per local
   authority, reports feature completeness, imputes missing values, checks for
   highly correlated (redundant) features
10. **7 - clustering** : compares KMeans/Agglomerative/GMM across several k
    values, picks the best by silhouette score, checks cluster stability via
    bootstrap resampling, and checks whether the clusters actually track crime
    levels
11. **8 - predictive modelling** : Random Forest with Leave-One-Out CV,
    comparing raw features vs PCA-reduced features, with feature importance
    (RF and SHAP) projected back onto the original indicators
12. **9 - figures** : generates all output charts

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
  indicator per LA, history + forecast


