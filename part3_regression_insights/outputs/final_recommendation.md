# Final Recommendation: Monthly Sales Drivers

## 1. Which Factors Appear Most Strongly Associated With Monthly Sales?

Based on the final multiple regression model (R² = 0.834, see
`outputs/model_equations.md` and `analysis/model_comparison.md`), the factors most strongly and
reliably associated with higher monthly sales, in order of statistical strength (lowest p-value
first), are:

1. **Footfall** (p = 5.3e-25) — by far the single strongest driver; +$27.99 in sales per
   additional monthly visitor, holding other factors equal.
2. **Marketing spend** (p = 5.2e-18) — +$1.15 in sales per additional $1 spent.
3. **Inventory availability %** (p = 1.5e-10) — +$3,033 in sales per 1-point increase in product
   availability.
4. **Store type** (Residential and Airport both significant; Mall borderline) — Airport stores
   outperform the High Street baseline by ~$21,131/month; Residential stores underperform it by
   ~$17,888/month.
5. **Region** (South and West both significant) — both regions outperform the East baseline by
   ~$19,700–$19,800/month.
6. **Staff count** (p = 0.016) — +$3,026 in sales per additional staff member.
7. **Holiday flag** (p = 0.014) — holiday months bring in ~$16,167 more in sales than non-holiday
   months.

## 2. Which Variables Should Leadership Focus On?

**Footfall, marketing spend, and inventory availability** are the three variables leadership has
the clearest, most statistically reliable evidence to act on:

- **Driving footfall** (e.g., through local advertising, store visibility, partnerships, or
  location-based promotions) has the single largest estimated impact on sales of anything in the
  model.
- **Marketing spend** shows a positive, significant, and roughly dollar-for-dollar association
  with sales even after controlling for footfall and other factors — suggesting it is not simply
  riding on footfall's coattails, but contributing something incremental.
- **Inventory availability** is cheap, operational, and directly actionable (reducing stockouts),
  and it is highly significant.

**Staffing levels and holiday-period planning** are reasonable secondary levers — both
significant, with sensible business logic (more staff → more capacity to serve customers and
close sales; holidays → naturally higher demand).

**Store type and region** are useful for *prioritization*, not for direct action — leadership can
use them to decide where to focus the actionable levers above (e.g., Airport stores may have
room to convert their structural advantage into even more sales with the right marketing/staffing
mix; Residential stores may need a different playbook given their structural disadvantage).

## 3. Which Variables Should Not Be Over-Interpreted?

- **avg_discount_pct** is **not** statistically significant (p = 0.172). The model shows a
  negative point estimate (more discounting associated with lower sales), but this should not be
  treated as established — it could be true, it could be noise, or it could be a non-linear
  relationship (e.g., small discounts help, large discounts hurt or vice versa) that a linear
  model can't see. Leadership should not change discounting strategy based on this model alone.
- **region_North** is not statistically significant (p = 0.269) — there isn't reliable evidence
  that the North region differs from the East baseline once other factors are controlled for.
- **store_type_Mall** is borderline (p = 0.067) — suggestive of a positive effect but just outside
  the conventional 5% threshold; treat as a hypothesis to monitor, not a confirmed finding.
- **footfall and staff_count together**: these two are highly correlated with each other (r ≈
  0.92) in the raw data. Both remain individually significant, but because they move together so
  closely in this dataset, the model has some difficulty cleanly separating "the effect of more
  visitors" from "the effect of more staff" — leadership should treat the *combination* of higher
  footfall and adequate staffing as the actionable insight, rather than fixating on the exact
  dollar split between the two coefficients.

## 4. What Business Action Would You Recommend?

1. **Prioritize footfall-driving initiatives** (local marketing, visibility, location-based
   promotions, partnerships) as the highest-leverage lever identified in this analysis.
2. **Maintain or selectively increase marketing spend** where it is currently constrained, since
   the model shows a positive, significant, incremental return even after controlling for
   footfall — but track actual sales lift per campaign, since some high-spend months in this data
   (see `analysis/residual_analysis.md`) did *not* see proportional sales gains, suggesting spend
   efficiency varies by execution, not just by amount.
3. **Tighten inventory availability**, especially in stores currently below the ~88% average —
   it's a significant, directly controllable lever.
4. **Avoid making discounting strategy changes based on this analysis alone** — `avg_discount_pct`
   is not statistically significant here; a separate, more targeted analysis (ideally with more
   discount-percentage variation or a non-linear specification) would be needed before changing
   discount policy.
5. **Use store type and region to prioritize, not to directly intervene** — e.g., investigate what
   makes Airport stores structurally outperform High Street stores, and see if any of that
   advantage (assortment, footfall capture, staffing model) can be replicated at underperforming
   Residential stores.

## 5. What Risks or Limitations Should Leadership Keep in Mind?

- This model explains 83% of the variance in monthly sales — a strong fit, but 17% remains
  unexplained, and the typical prediction error is about **$42,000/month** (residual standard
  deviation), which is non-trivial at the individual-store level.
- The dataset covers only **4 months (Jan–Apr 2025) across 80 stores** — it is not long enough to
  separate genuine seasonal effects (beyond the holiday flag) from short-term noise, and any
  recommendation should be revisited once more months of data are available.
- **Footfall and staff_count are highly correlated**, which can make their individual coefficients
  less stable than they appear; this doesn't invalidate the model's overall conclusions but means
  the exact dollar-for-dollar split between the two should be treated as approximate.
- **avg_discount_pct, region_North, and (borderline) store_type_Mall** are not statistically
  reliable findings in this dataset and should not be used to justify business decisions on their
  own (see Section 3).
- The largest individual prediction errors (`analysis/residual_analysis.md`) cluster somewhat in
  Residential stores and the West region — predictions for those segments should be treated with
  more caution than for, say, Airport stores.

## 6. Why Does Regression Show Association But Not Automatically Prove Causation?

Regression measures **statistical association** — how a dependent variable tends to move together
with one or more independent variables, after accounting for the other variables in the model. It
does **not**, on its own, prove that changing an independent variable will *cause* the dependent
variable to change by the estimated coefficient, for several reasons:

- **Reverse causality is possible.** For example, stores that are already selling well might
  justify a *larger* marketing budget (sales → marketing spend), rather than marketing spend
  driving sales (marketing spend → sales). The regression coefficient cannot, by itself,
  distinguish between these two directions.
- **Omitted variables (confounding) are possible.** A factor not in this dataset — for example,
  local population growth, a competitor closing nearby, or store manager quality — could be
  driving both footfall and sales simultaneously, making footfall look like a stronger cause of
  sales than it actually is in isolation.
- **This is observational, not experimental data.** No variable here was randomly assigned to
  stores (unlike, for example, the A/B-tested campaign in Part 2 of this project). Without random
  assignment, we cannot rule out that the stores with higher marketing spend, more staff, or
  better inventory availability also differ in other, unmeasured ways that independently explain
  their higher sales.
- **Correlation between predictors (e.g., footfall and staff_count) muddies causal attribution.**
  Even if the relationship between the *combination* of these factors and sales is causal, the
  model cannot cleanly assign how much of that causal effect belongs to footfall versus staffing
  individually, because they move together so closely in this data.

**Practical implication for leadership:** treat this model's coefficients as strong evidence of
*which factors travel with higher sales* and use them to prioritize where to test further (e.g., a
controlled pilot that increases staffing or inventory availability at a subset of stores and
measures the actual sales change) — but do not treat the coefficients as a guaranteed dollar return
on any single action without that follow-up validation.

This recommendation is supported by the regression evidence summarized in
`analysis/model_comparison.md`, `outputs/model_equations.md`, and the residual diagnostics in
`analysis/residual_analysis.md`.
