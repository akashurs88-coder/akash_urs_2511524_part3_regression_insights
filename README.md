# akash_urs_2511524_part3_regression_insights
# Regression-Based Business Insights & Model Interpretation

## 1. Business Problem Summary

A retail chain's leadership wants to understand what factors drive monthly sales performance
across stores, in order to decide between business actions such as increasing marketing spend,
improving inventory availability, changing discounting strategy, reallocating staff, or
prioritizing certain store types or regions. The task is to use regression analysis to identify
which factors appear most strongly associated with monthly sales and provide a business
recommendation.

## 2. Dataset Description

Source file: `data/business_regression_data.xlsx`, sheet `store_performance` (320 rows: 80 stores
× 4 months, Jan–Apr 2025), plus a `data_dictionary` reference sheet.

| Column | Description |
|---|---|
| store_id | Unique store identifier (80 unique stores) |
| month | Reporting month (Jan, Feb, Mar, Apr 2025) |
| region | Business region (East, West, North, South) |
| store_type | Mall, High Street, Residential, or Airport |
| marketing_spend | Monthly marketing spend for the store |
| footfall | Monthly visitor count |
| avg_discount_pct | Average discount percentage for the month |
| staff_count | Number of store staff |
| inventory_availability_pct | Average product availability percentage |
| competitor_distance_km | Distance to nearest competitor |
| holiday_flag | 1 if the month/store period had a meaningful holiday effect, else 0 |
| customer_rating | Average customer rating |
| monthly_sales | **Dependent variable** |
| monthly_profit | Optional secondary business metric (not used as dependent variable here) |

### Data Quality Checks (Task 1)

| Check | Finding | Action Taken |
|---|---|---|
| Missing values | `competitor_distance_km`: 6 missing; `customer_rating`: 8 missing; all other columns complete | Both variables were **excluded from the final model** (see Section 3 below) rather than imputed, since neither was strongly correlated with `monthly_sales` in exploratory checks (r ≈ −0.11 and r ≈ −0.03 respectively) and excluding them avoided introducing imputation bias or losing rows to `dropna`. |
| Duplicate rows | 0 fully duplicate rows; 0 duplicate (store_id, month) pairs | No action needed — every store has exactly one row per month, for all 4 months. |
| Group/category counts | region: East 104, West 92, North 64, South 60. store_type: High Street 116, Residential 100, Mall 76, Airport 28 | No rebalancing needed; uneven but large-enough counts in every category for dummy variable regression. |
| Invalid values | `holiday_flag` contains only {0,1}; no negative values in marketing_spend, footfall, staff_count, inventory_availability_pct, avg_discount_pct | No invalid values found. |
| Outliers | `marketing_spend` has 6 values above the IQR-based outlier threshold (~$104,898), up to $172,416 | Reviewed individually — all 6 are plausible high-spend months, not data errors. **Retained** in the model; in fact, several of these specific store-months turned out to be the largest negative residuals in the final model (see `analysis/residual_analysis.md`), which is itself a useful finding (these were months where heavy spend did not translate into proportional sales). |
| Segment distribution | Each store_type and region has enough observations (minimum 28, for Airport) to support dummy-variable regression | No action needed. |

## 3. Dependent and Independent Variables

- **Dependent variable**: `monthly_sales`
- **Numerical independent variables considered**: `marketing_spend`, `footfall`,
  `avg_discount_pct`, `staff_count`, `inventory_availability_pct`, `competitor_distance_km`,
  `customer_rating`, `holiday_flag`
- **Categorical independent variables considered**: `store_type`, `region`
- **Variables that may need cleaning/transformation**: `competitor_distance_km` and
  `customer_rating` (missing values, see above); `store_type` and `region` (require dummy-variable
  encoding before they can enter a regression).
- **Variables not used in the final model**: `store_id` and `month` are identifiers/time labels,
  not predictors, and were excluded; `monthly_profit` is a secondary outcome variable, not an
  independent variable, and was excluded to avoid mixing two dependent variables; `competitor_distance_km`
  and `customer_rating` were excluded due to missingness combined with weak correlation to sales
  (see Section 2 of `analysis/model_comparison.md` discussion and Task 1 review).

## 4. Regression Approach

1. Reviewed the dataset and data dictionary; identified the dependent variable, candidate
   predictors, and cleaning needs (this README, Section 2–3).
2. Built `analysis/regression_workbook.xlsx` with separate `original_data` and `cleaned_data`
   sheets (original left untouched, per Task 2 requirement).
3. Created dummy variables for `store_type` and `region` (`dummy_variables` sheet).
4. Ran 3 **simple regressions** (`simple_regression` sheet): monthly_sales against
   marketing_spend, footfall, and inventory_availability_pct individually.
5. Ran 1 **multiple regression** (`multiple_regression` sheet) using 6 numerical predictors and
   6 dummy variables (3 for store_type, 3 for region) simultaneously.
6. Computed predicted sales and residuals for all 320 records, and identified the top 5 largest
   positive and negative residuals (`predictions_residuals` sheet,
   `analysis/residual_analysis.md`).
7. Compared all models side by side (`model_comparison` sheet, `analysis/model_comparison.md`).
8. Documented equations and dummy-variable logic in business language
   (`outputs/model_equations.md`).
9. Wrote the final business recommendation, tying the statistical evidence to specific actions and
   explicitly addressing association vs. causation (`outputs/final_recommendation.md`).

## 5. Dummy Variable Approach

`store_type` (4 categories) and `region` (4 categories) were each converted to 3 dummy (0/1)
columns, dropping one reference category each to avoid the dummy-variable trap (perfect
multicollinearity with the intercept):

- **store_type reference category: "High Street"** (most common, 116/320 rows)
- **region reference category: "East"** (most common, 104/320 rows)

All store_type and region coefficients in the final model are interpreted as "difference vs. this
reference group, holding other variables constant." Full explanation:
`outputs/model_equations.md`, Sections 4–5.

## 6. Model Comparison Summary

| Model | Variable(s) | R-squared | Significant? |
|---|---|---|---|
| Simple — marketing_spend | marketing_spend | 0.167 | Yes (p < 0.001) |
| Simple — footfall | footfall | 0.736 | Yes (p < 0.001) |
| Simple — inventory_availability_pct | inventory_availability_pct | 0.013 | Yes but very weak (p = 0.041) |
| **Multiple regression (final model)** | 6 numeric + 6 dummy variables | **0.834** (Adj. 0.827) | Most variables yes; avg_discount_pct, region_North, and (borderline) store_type_Mall are not |

Full comparison with all required fields: `analysis/model_comparison.md` and
`outputs/regression_summary.xlsx`.

## 7. Final Model Selected

**The multiple regression model** (Model D) — see `outputs/model_equations.md` Section 6–7 for
the full equation and justification. It was selected because it has the highest R² (0.834 vs. a
maximum of 0.736 for any simple regression), and because it isolates each driver's effect while
controlling for the others, which is required to give leadership a usable, non-misleading view of
what's actually driving sales.

## 8. Business Recommendation

Footfall, marketing spend, and inventory availability are the strongest, most actionable, and
statistically reliable levers identified. Store type and region differences are useful for
prioritization but are not directly actionable levers themselves. avg_discount_pct, region_North,
and (borderline) store_type_Mall should not be used to justify decisions based on this analysis
alone. Full recommendation, risks, and the association-vs-causation discussion:
`outputs/final_recommendation.md`.

## 9. Assumptions and Limitations

- `competitor_distance_km` and `customer_rating` were excluded from the final model due to
  missingness and weak observed correlation with `monthly_sales`; a different model that imputes
  and includes them might tell a slightly different story, though their weak correlation suggests
  limited impact.
- The dataset spans only 4 months across 80 stores — too short to robustly separate seasonal
  effects beyond the provided `holiday_flag` from short-term noise.
- `footfall` and `staff_count` are highly correlated (r ≈ 0.92) in this data; their individual
  coefficients in the multiple regression should be read as a *combined* signal more than as two
  fully separable effects (VIF ≈ 6.5 for both — not dangerous, but not negligible).
- All relationships are estimated by ordinary least squares on observational (non-experimental)
  data; regression here shows association, not proof of causation (`outputs/final_recommendation.md`,
  Section 6).
- Outlier marketing_spend values were reviewed and retained as plausible real data rather than
  removed; they turned out to coincide with several of the model's largest negative residuals,
  which is treated as a finding (inefficient spend in those months) rather than a data error.

## 10. Screenshots Included

| File | Shows |
|---|---|
| `screenshots/simple_regression_output.png` | Simple regression output (monthly_sales ~ footfall) |
| `screenshots/multiple_regression_output.png` | Multiple regression output (final model, all coefficients) |
| `screenshots/residuals_preview.png` | Top 5 positive and top 5 negative residuals |
| `screenshots/model_comparison_preview.png` | Model comparison table (all 4 models) |

## 11. Repository Structure

```
part3_regression_insights/
├── data/
│   └── business_regression_data.xlsx
├── analysis/
│   ├── regression_workbook.xlsx
│   ├── model_comparison.md
│   └── residual_analysis.md
├── outputs/
│   ├── regression_summary.xlsx
│   ├── final_recommendation.md
│   └── model_equations.md
├── screenshots/
│   ├── simple_regression_output.png
│   ├── multiple_regression_output.png
│   ├── residuals_preview.png
│   └── model_comparison_preview.png
└── README.md
```
