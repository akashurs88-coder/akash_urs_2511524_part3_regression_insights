# Model Comparison

This document compares all regression models built on this dataset. Full computed values (with
live formulas) are in `analysis/regression_workbook.xlsx` → `model_comparison`,
`simple_regression`, and `multiple_regression` sheets. A presentation-ready copy is in
`outputs/regression_summary.xlsx`. Screenshot evidence:
`screenshots/model_comparison_preview.png`.

## Model Name, Variables Used, and R-squared

| Model | Dependent Variable | Independent Variable(s) | R-squared | N |
|---|---|---|---|---|
| **Model A** — Simple | monthly_sales | marketing_spend | 0.1672 | 320 |
| **Model B** — Simple | monthly_sales | footfall | 0.7363 | 320 |
| **Model C** — Simple | monthly_sales | inventory_availability_pct | 0.0131 | 320 |
| **Model D** — Multiple (final model) | monthly_sales | footfall, marketing_spend, staff_count, inventory_availability_pct, avg_discount_pct, holiday_flag, store_type (3 dummies), region (3 dummies) | **0.8337** (Adj. R² = 0.8272) | 320 |

Model B (footfall alone) already explains nearly 74% of the variance in monthly sales — by far
the strongest single driver. Model D, the full multiple regression, explains 83.4% of the
variance, an improvement over the best single-variable model, because it captures additional
independent drivers (marketing spend, staffing, inventory, holidays, store type, and region) that
footfall alone does not.

## Significant Variables (Useful P-values)

At the 5% significance level (p < 0.05), the following variables are statistically significant
in Model D:

- footfall (p = 5.34e-25)
- marketing_spend (p = 5.16e-18)
- staff_count (p = 0.0157)
- inventory_availability_pct (p = 1.45e-10)
- holiday_flag (p = 0.0144)
- store_type_Residential (p = 0.0029)
- store_type_Airport (p = 0.0228)
- region_South (p = 0.0060)
- region_West (p = 0.0017)
- Intercept (p = 0.0127)

**Not significant at 5%:**

- avg_discount_pct (p = 0.1717)
- store_type_Mall (p = 0.0672 — borderline, just outside the conventional 5% threshold)
- region_North (p = 0.2694)

In the simple regressions, marketing_spend, footfall, and inventory_availability_pct are all
individually significant (p < 0.05) — but that significance does not tell us how much each
matters once the others are accounted for, which is exactly what Model D adds.

## Business Usefulness

**Model D (multiple regression) is the most business-useful model.** It is the only model that:

- Lets leadership compare the *relative* size of different levers (e.g., is hiring more staff or
  spending more on marketing the better-leveraged action?) under a "holding everything else
  equal" assumption.
- Distinguishes store_type and region effects from the operational levers (spend, staffing,
  inventory) that leadership can actually act on.
- Flags which variables are *not* reliable enough to act on (avg_discount_pct, region_North,
  borderline store_type_Mall), preventing leadership from over-interpreting noise as signal.

The simple regressions (Models A–C) are useful as a sanity check and as an easy way to communicate
the two strongest individual drivers (footfall, marketing spend) in isolation, but they cannot
separate the effect of one variable from the effect of correlated variables — for instance, Model
A alone would overstate marketing spend's apparent importance, since it doesn't control for
footfall, which is also higher in months when marketing spend is higher.

## Limitations — What the Models Do Not Capture

- **Causation vs. association.** All models are observational regressions on monthly data; none
  of them prove that increasing marketing spend or staff count *causes* sales to rise by the
  stated amount. Confounding factors (e.g., stores that are simply busier for reasons not in the
  dataset) could explain part of the relationship. See `outputs/final_recommendation.md`, Section 6.
- **Multicollinearity between footfall and staff_count.** These two variables are highly
  correlated (r ≈ 0.92) in the raw data — busier stores tend to be staffed more heavily. Both
  remain individually significant in Model D, but their estimated coefficients should be read with
  some caution, since the model has to split a similar signal between two correlated variables (VIF
  ≈ 6.5 for both, well below the danger zone of ~10, but worth flagging).
- **avg_discount_pct's true effect is unclear from this model.** It is not significant here, which
  could mean discounting genuinely has little net effect on monthly sales in this data range, or
  could mean the relationship is non-linear (e.g., small discounts help, large discounts hurt) and
  a linear model is the wrong functional form to detect it.
- **Time is not modeled as a trend.** The dataset only spans 4 months (Jan–Apr 2025) for 80 stores;
  any seasonal or trend effects beyond the `holiday_flag` indicator are not captured.
- **No store-level fixed effects.** Some stores may have persistent, store-specific advantages
  (e.g., a great location with much more daily traffic patterns) not explained by any predictor in
  the dataset; the model does not separate out a "store" effect distinct from its measured
  characteristics.
- **R² of 0.83, not 1.00.** About 17% of the variation in monthly sales is still unexplained by
  this model — see `analysis/residual_analysis.md` for what the largest unexplained cases
  (residuals) look like.
