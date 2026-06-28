# Model Equations

## 1. Simple Regression Equations

### Model A: monthly_sales ~ marketing_spend
> **monthly_sales = 560,777.35 + 2.1296 × marketing_spend**

- R² = 0.1672, p < 0.001
- For every additional $1 of monthly marketing spend, predicted monthly sales increase by
  approximately $2.13, holding nothing else constant.

### Model B: monthly_sales ~ footfall
> **monthly_sales = 446,410.58 + 35.6780 × footfall**

- R² = 0.7363, p < 0.001
- For every additional store visitor in a month, predicted monthly sales increase by
  approximately $35.68, holding nothing else constant.

### Model C: monthly_sales ~ inventory_availability_pct
> **monthly_sales = 484,814.26 + 2,217.83 × inventory_availability_pct**

- R² = 0.0131, p = 0.041
- For every 1 percentage point increase in inventory availability, predicted monthly sales
  increase by approximately $2,218 — statistically detectable but explains almost none of the
  variation in sales on its own.

## 2. Multiple Regression Equation (Final Model)

> **monthly_sales = 107,624.91**
> **+ 27.99 × footfall**
> **+ 1.15 × marketing_spend**
> **+ 3,026.30 × staff_count**
> **+ 3,032.60 × inventory_availability_pct**
> **− 49,804.12 × avg_discount_pct**
> **+ 16,166.63 × holiday_flag**
> **+ 11,856.48 × store_type_Mall**
> **− 17,888.22 × store_type_Residential**
> **+ 21,131.24 × store_type_Airport**
> **+ 7,710.78 × region_North**
> **+ 19,692.49 × region_South**
> **+ 19,831.83 × region_West**

R² = 0.8337, Adjusted R² = 0.8272, F = 128.30 (p < 0.001), N = 320

## 3. Explanation of Each Coefficient

| Variable | Coefficient | Business meaning |
|---|---|---|
| Intercept | $107,624.91 | Predicted monthly sales for a High Street store in the East region, with all numeric predictors at 0. (Not a realistic store on its own — it anchors the equation, not a real-world scenario.) |
| footfall | +$27.99 | Each additional monthly visitor is associated with ~$28 more in monthly sales, holding all else equal. The strongest, most reliable driver in the model. |
| marketing_spend | +$1.15 | Each additional $1 of marketing spend is associated with ~$1.15 more in monthly sales, holding all else equal — i.e., marketing spend is associated with roughly breaking even to slightly positive direct sales return, before accounting for profit margin or longer-term brand effects. |
| staff_count | +$3,026.30 | Each additional staff member is associated with ~$3,026 more in monthly sales, holding all else equal. |
| inventory_availability_pct | +$3,032.60 | Each 1-point increase in inventory availability (%) is associated with ~$3,033 more in monthly sales, holding all else equal. |
| avg_discount_pct | −$49,804.12 (not significant) | Direction suggests heavier discounting is associated with lower sales, but this is **not statistically significant** (p = 0.172) — see Section 6, do not treat this as a reliable effect. |
| holiday_flag | +$16,166.63 | Months with a meaningful holiday effect are associated with ~$16,167 more in monthly sales than non-holiday months, holding all else equal. |
| store_type_Mall | +$11,856.48 (borderline, p = 0.067) | Mall stores are associated with ~$11,856 more monthly sales than High Street stores (reference), holding all else equal — borderline significance, treat with some caution. |
| store_type_Residential | −$17,888.22 | Residential stores are associated with ~$17,888 less monthly sales than High Street stores (reference), holding all else equal. |
| store_type_Airport | +$21,131.24 | Airport stores are associated with ~$21,131 more monthly sales than High Street stores (reference), holding all else equal. |
| region_North | +$7,710.78 (not significant) | North region stores show no statistically significant sales difference from East (reference) once other factors are controlled for (p = 0.269). |
| region_South | +$19,692.49 | South region stores are associated with ~$19,692 more monthly sales than East (reference), holding all else equal. |
| region_West | +$19,831.83 | West region stores are associated with ~$19,832 more monthly sales than East (reference), holding all else equal. |

## 4. Explanation of Dummy Variables

Two categorical variables were converted into dummy (0/1) variables:

- **store_type** → `store_type_Mall`, `store_type_Residential`, `store_type_Airport`
- **region** → `region_North`, `region_South`, `region_West`

Each dummy equals 1 if a row belongs to that category, 0 otherwise. A store can only be one
store_type and one region at a time, so exactly one dummy per group is "on" for each row (or none,
if the row belongs to the reference category).

## 5. Reference Category Used

- **store_type reference category: "High Street"** (the most common store type, 116 of 320 rows)
- **region reference category: "East"** (the most common region, 104 of 320 rows)

These reference categories were **not** given their own dummy column. This is required to avoid
the "dummy variable trap": if a dummy were created for every category of a variable *and* the
model also has an intercept, the dummy columns would sum to 1 for every row, creating perfect
multicollinearity with the intercept and making the regression unsolvable. By dropping one
category as the reference, all other dummy coefficients are interpreted as "difference vs. this
reference group," and the intercept represents the reference group's baseline.

## 6. Final Model Selected

**Model D: the multiple regression model** (all numeric predictors + store_type and region
dummies) is the final model selected for the business recommendation.

## 7. Reason for Selecting This Model

- It has by far the highest explanatory power: **R² = 0.834** vs. the best single-variable model
  (footfall alone, R² = 0.736) and far above marketing_spend alone (R² = 0.167) or inventory
  availability alone (R² = 0.013).
- It lets us see each driver's effect **holding the others constant** — e.g., we can now say
  marketing spend has a positive, significant association with sales even after controlling for
  footfall, staff, and store characteristics, which a simple regression cannot do on its own.
- It surfaces store-type and region differences (e.g., Airport stores outperforming High Street
  stores by ~$21k/month, holding other factors equal) that the simple models completely miss.
- It correctly identifies which variables are **not** reliable (avg_discount_pct, region_North,
  and — borderline — store_type_Mall), which is itself valuable for telling leadership where not
  to over-interpret the data.

The simple regression models are still kept and reported because they are easier to communicate
individually and confirm the direction of the two strongest single drivers (footfall and
marketing spend) before the full model is introduced.
