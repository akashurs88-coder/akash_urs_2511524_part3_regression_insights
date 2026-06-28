# Residual Analysis

Final model used: the multiple regression model described in `outputs/model_equations.md` and
computed in `analysis/regression_workbook.xlsx` → `multiple_regression` and
`predictions_residuals` sheets. Screenshot evidence: `screenshots/residuals_preview.png`.

## 1. Predicted Sales

Predicted monthly sales were calculated for all 320 store-month records by applying the final
model's intercept and coefficients to each record's actual values for footfall, marketing_spend,
staff_count, inventory_availability_pct, avg_discount_pct, holiday_flag, and the store_type/region
dummy values. Live formulas are in `analysis/regression_workbook.xlsx` →
`predictions_residuals`, column F.

## 2. Residuals

Residual = actual `monthly_sales` − predicted `monthly_sales`, computed for every record
(`predictions_residuals`, column G).

- Mean residual ≈ 0 (as expected for OLS — by construction, the average prediction error across
  all 320 records is essentially zero, ~ -5.5e-11, i.e. zero within floating-point precision).
- Residual standard deviation (≈ RMSE): **$42,312**. This is the typical size of the model's
  prediction error in dollar terms — roughly 6% of the average monthly_sales value (~$680,629).

## 3. Top 5 Largest Positive Residuals (model under-predicts — actual sales beat the model)

| Rank | Store ID | Month | Store Type | Region | Actual | Predicted | Residual |
|---|---|---|---|---|---|---|---|
| 1 | STR-1028 | 2025-04 | Mall | East | $713,611 | $605,689 | **+$107,922** |
| 2 | STR-1026 | 2025-04 | Mall | East | $625,514 | $520,809 | **+$104,705** |
| 3 | STR-1051 | 2025-02 | Airport | East | $787,716 | $684,085 | **+$103,631** |
| 4 | STR-1030 | 2025-02 | Residential | West | $820,519 | $716,894 | **+$103,625** |
| 5 | STR-1073 | 2025-03 | Residential | East | $813,317 | $718,009 | **+$95,308** |

## 4. Top 5 Largest Negative Residuals (model over-predicts — actual sales fall short of the model)

| Rank | Store ID | Month | Store Type | Region | Actual | Predicted | Residual |
|---|---|---|---|---|---|---|---|
| 1 | STR-1017 | 2025-03 | High Street | West | $685,379 | $860,407 | **−$175,028** |
| 2 | STR-1012 | 2025-01 | Residential | West | $595,468 | $729,837 | **−$134,369** |
| 3 | STR-1023 | 2025-02 | Mall | South | $627,172 | $753,250 | **−$126,078** |
| 4 | STR-1007 | 2025-02 | Mall | West | $800,452 | $909,274 | **−$108,822** |
| 5 | STR-1077 | 2025-02 | Residential | South | $538,376 | $634,782 | **−$96,406** |

## 5. What These Residuals May Mean in Business Terms

**A notable pattern in the negative-residual group**: 3 of the 5 largest negative residuals
(STR-1012, STR-1023, and STR-1007) are exactly the store-months that had unusually high
`marketing_spend` outliers identified during data cleaning (see `README.md`, data quality table —
all 3 were among the 6 marketing_spend values above the IQR outlier threshold). In other words,
these are cases where a store spent heavily on marketing in a given month but **did not** see
sales rise to match what the model — built on the average relationship across all stores — would
predict given that spend level and their other characteristics. In business terms, this looks like
**inefficient or under-performing marketing spend** in those specific store-months, worth a
follow-up review (was the campaign poorly targeted? was there a stockout or local disruption that
offset the marketing push?).

**The positive-residual group** is more mixed: it includes two Mall/East stores in the same month
(STR-1028 and STR-1026, both April 2025), an Airport store, and two Residential stores. These are
store-months that **outperformed** what their footfall, spend, staffing, and other measured
characteristics would predict — possible explanations include a local event, a particularly
effective in-store promotion not captured by `avg_discount_pct`, or simply favorable
month-to-month variation that a 4-month dataset can't fully average out.

## 6. Is the Model Systematically Under- or Over-Predicting Certain Store Types?

Because `store_type` and `region` dummies are included as predictors, the **average** residual
within every store_type and every region is essentially exactly zero by construction (OLS forces
the average error to be zero for any group represented by a dummy variable in the model). So the
model is not systematically biased *on average* for any store type or region — that bias has
already been absorbed into the store_type/region coefficients themselves.

However, looking at where the **largest individual** residuals (top 10 in each direction) fall:

- Positive residuals (under-predicted) skew slightly toward **Residential** stores (6 of the top
  10) and the **East/West** regions.
- Negative residuals (over-predicted) are more spread across **Residential, High Street, and
  Mall** stores, with a concentration in the **West** region (6 of the top 10).

This suggests the model's *average* fit by category is fine, but its *variance* (how far any
single store-month can swing from its predicted value) may be somewhat higher for Residential
stores and for the West region specifically — both directions of error appear there more than for
Airport stores, for example, where no large negative residuals appear in the top 5. This is a
useful flag for leadership: the model's predictions for Residential stores and West-region stores
should be treated as a bit less precise than for other categories, even though there's no overall
directional bias.
