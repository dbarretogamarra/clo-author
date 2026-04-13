# Falsification and Placebo Tests
## Gasto en Limpieza Pública y Gestión de Residuos Sólidos, Perú 2011–2020

**Date:** 2026-04-12  
**Logic:** A valid causal estimate should produce effects where theory predicts them and no effects where theory predicts zero. Each test below specifies: what is being tested, what result confirms validity, and what result would invalidate the strategy.

---

## F1. Pre-Trend / Placebo-in-Time Test

**Test:** Event study around large discrete jumps in municipality-level spending. Classify municipalities that experienced a spending increase above the 75th percentile of the spending change distribution between consecutive years. Estimate an event-study regression with indicators for $\tau = -3, -2, -1, 0, +1, +2, +3$ years relative to the event, using the period $\tau = -1$ as the reference category.

**Expected result if identification is valid:** Coefficients for $\tau \leq -1$ are jointly statistically indistinguishable from zero (flat pre-trend). Coefficients for $\tau \geq 0$ diverge from zero in the expected direction.

**Result that would invalidate:** Statistically significant pre-trends ($\tau = -2$ or $\tau = -3$) of the same sign and comparable magnitude to the post-event effects. This would indicate that outcome trajectories were already diverging before the spending increase — a parallel trends violation.

**Reference period:** $\tau = -1$ (one year before the spending jump). Endpoint bins: $\tau \leq -3$ and $\tau \geq +3$ to avoid sparse cell problems.

---

## F2. Placebo Outcome: Spending on Functions Unrelated to Waste

**Test:** Regress waste management outcomes (QPRS, CSRS, FR) on log per-capita executed spending in a completely unrelated budget function — specifically, spending on "Cultura y deporte" (culture and sports, function 21 in the SIAF functional classifier) or "Cementerios" (if available). Use the same TWFE specification.

**Expected result if identification is valid:** The coefficient on unrelated spending is zero (or negligibly small) and statistically insignificant for all waste management outcomes.

**Result that would invalidate:** Statistically significant positive effects of culture/sports spending on waste collection outcomes. This would indicate that what the model is picking up is a general fiscal capacity or political will effect — "wealthier or better-governed municipalities spend more on everything and have better outcomes on everything" — rather than a specific causal effect of limpieza spending.

---

## F3. Placebo Treatment: Spending on Limpieza Predicting Outcomes in Unrelated Domains

**Test:** Regress outcomes in domains theoretically unrelated to street cleaning on log per-capita limpieza spending. Candidate outcomes from RENAMU: number of municipal parks, number of municipal markets, presence of a municipal library, number of municipal employees in culture (if available in RENAMU).

**Expected result if identification is valid:** Limpieza spending has zero or negligible effects on these unrelated outcomes.

**Result that would invalidate:** Limpieza spending predicts the number of parks or municipal employees in culture — outcomes where no causal channel exists. This would reveal that the spending variable is proxying general municipal capacity rather than functioning as a targeted treatment.

---

## F4. Instrument Validity Check: FONCOMUN Formula Weights and Lagged Outcomes

**Test:** Regress the formula-predicted FONCOMUN transfer ($\hat{F}_{it}$) on lagged waste management outcomes ($Y_{it-1}, Y_{it-2}$), conditional on the full set of controls and year fixed effects. This tests whether future formula-predicted transfers are correlated with past outcomes — which would occur if the formula responds to past performance (violating exogeneity) or if there is a reverse-causality channel from outcomes to formula weights.

**Expected result if identification is valid:** $\hat{F}_{it}$ is uncorrelated with $Y_{it-1}$ and $Y_{it-2}$ conditional on controls and year FE (F-statistic on joint significance of lagged outcomes is small, p > 0.10).

**Result that would invalidate:** Lagged waste outcomes significantly predict formula-predicted transfers, even after controlling for poverty and population weights. This would suggest the formula reflects outcome performance, violating the exogeneity of the instrument.

---

## F5. Placebo Sample: Municipalities with Zero FONCOMUN Dependence

**Test:** Restrict the IV analysis to the top quintile of own-source revenue municipalities — those for whom FONCOMUN constitutes less than 10% of total revenue (large cities, primarily Lima districts). In this subsample, FONCOMUN variation should not affect limpieza spending because these municipalities have ample own revenues and do not allocate marginal FONCOMUN receipts to specific spending categories. Estimate the IV first stage on this sub-sample.

**Expected result if identification is valid:** The first-stage coefficient on $\hat{F}_{it}$ is near zero and statistically insignificant for high-own-revenue municipalities. This confirms that the instrument's relevance comes from its effect on spending in FONCOMUN-dependent municipalities, not from a spurious correlation.

**Result that would invalidate:** A strong first stage even in FONCOMUN-independent municipalities. This would suggest the instrument is correlated with spending through channels other than fiscal transfers — e.g., because formula-predicted transfers correlate with political cycles or regional economic shocks that affect all municipalities including wealthy ones.

---

## F6. Within-Component Falsification: Spending on "Destino Final" Should Not Predict Collection Frequency

**Test:** Regress collection frequency ($FR_{it}$) and collection coverage ($CSRS_{it}$) specifically on spending in the "destino final" (final disposal) sub-component of the limpieza budget, controlling for all other sub-components.

**Expected result if identification is valid:** $G^{destino}$ has zero or negligible effect on collection frequency and coverage. The "destino final" budget funds landfill construction and operation — it should affect disposal outcomes (Dimension 3) but not the frequency or coverage of household collection (Dimension 1).

**Result that would invalidate:** Spending on final disposal strongly predicts collection frequency. This would suggest severe collinearity or mis-classification of spending categories in SIAF, making the sub-component decomposition uninformative.

---

## F7. Temporal Falsification: Post-Policy Break (Pre/Post D.Leg. 1278, 2017)

**Test:** Interact all specifications with an indicator for post-2016 (after the Decreto Legislativo 1278 "Ley de Gestión Integral de Residuos Sólidos" came into force). Test whether the effect of spending on planning outcomes (Dimension 2) is significantly larger after 2016, when compliance with PIGARS and PMRS became more formally enforced.

**Expected result:** A positive and significant interaction — the spending-to-planning effect grew after 2016 because the new law made MINAM compliance requirements binding, increasing the marginal return to planning spending. This is a directional theoretical prediction, not a falsification per se, but a falsification would be: no change in the spending-planning relationship despite a major regulatory shift.

**Result that would invalidate main results:** If the interaction shows that spending had *no* effect on outcomes before 2016 (when there was less regulation) and only shows effects after 2016, this would suggest that the measured effect reflects compliance behavior (adopting plans to satisfy regulators regardless of actual spending) rather than real improvement in waste management capacity.

---

## F8. Randomization Inference on FONCOMUN Instrument

**Test:** Randomly permute the formula-predicted FONCOMUN transfers across municipalities within each year (holding the marginal distribution fixed), re-estimate the IV model on each of 1,000 permuted datasets, and compare the actual first-stage F-statistic and second-stage IV estimate to the permutation distribution.

**Expected result if identification is valid:** The actual IV estimate lies in the tail of the permutation distribution (p < 0.05). The actual first-stage F-statistic is much larger than permutation-based F-statistics.

**Result that would invalidate:** The actual IV estimate is not unusual relative to the permutation distribution — meaning the result is consistent with random noise or a spurious correlation between formula weights and outcomes unrelated to the spending mechanism.
