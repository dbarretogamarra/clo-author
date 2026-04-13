# Robustness Plan
## Gasto en Limpieza Pública y Gestión de Residuos Sólidos, Perú 2011–2020

**Date:** 2026-04-12  
All robustness checks are pre-specified. Each is listed with its rationale and the threat it addresses.

---

## A. Identification Robustness

### A1. IV vs. TWFE Comparison
**Check:** Compare OLS-TWFE estimates to IV-2SLS estimates for all continuous outcomes (QPRS, proportions). Report both in the same table.  
**Rationale:** If TWFE and IV estimates are similar in sign and magnitude, residual endogeneity after controlling for municipality and year FE is limited. Large differences indicate that time-varying confounders or reverse causality are distorting TWFE. Direction of the difference (IV > OLS suggests downward bias from reverse causality; IV < OLS suggests upward bias from selection on gains) informs the interpretation.

### A2. Lagged Spending (One-Period and Two-Period Lag)
**Check:** Replace contemporaneous $G_{it}$ with $G_{it-1}$ and $G_{it-2}$ as alternative treatment specifications for all outcomes.  
**Rationale:** If the effect of spending on outcomes operates with a lag (construction of landfills takes time; planning documents take years to produce), the lagged specification should show stronger or more stable effects. Contemporaneous effects could reflect reporting simultaneity (RENAMU collected at year-end, spending reported at year-end). Inconsistency between contemporaneous and lagged effects raises a timing concern.

### A3. Cumulative Distributed Lag (for Disposal Outcomes)
**Check:** Include $G_{it}$, $G_{it-1}$, and $G_{it-2}$ simultaneously. Report the sum of coefficients and its standard error (Newey-West correction for serial correlation in the sum).  
**Rationale:** Disposal infrastructure investment is cumulative. The total long-run effect may be larger than the within-year effect. This test is most relevant for PRS (share to landfill) and RS (presence of sanitary landfill), where capital-intensive infrastructure accumulates over multiple years.

### A4. Province × Year Fixed Effects (Absorbing Regional Shocks)
**Check:** Replace year fixed effects with province $\times$ year fixed effects ($\gamma_{p(i)t}$) in all OLS-TWFE models.  
**Rationale:** Regional governments (GOREs) make decisions about waste infrastructure at the province or department level. A GORE initiative to build a shared sanitary landfill may simultaneously increase both spending (via co-financing) and outcomes for all municipalities within the province. Province × year FE absorb these regional-level time-varying confounders. The identifying variation in this specification is within-province, between-municipality, conditional on provincial shocks — a narrower but more credibly exogenous source of variation.

### A5. Conley et al. (2012) Sensitivity to IV Exclusion Restriction Violation
**Check:** Implement the Conley-Hansen-Rossi (2012) "plausibly exogenous" bounds. Allow the direct effect of the FONCOMUN instrument on outcomes to be non-zero, parameterized as $\delta \neq 0$, and trace out how the IV estimate changes as $\delta$ varies over a plausible range.  
**Rationale:** The exclusion restriction is not testable. This analysis formalizes how much direct effect the instrument could have before the sign or significance of the IV estimate is reversed, making the strength of the exclusion assumption transparent to the reader.

---

## B. Model Specification Robustness

### B1. LPM vs. Nonlinear Models for All Binary Outcomes
**Check:** For every binary outcome (RS, Botadero, Reciclados, Quemados, PIGARS, PMRS, SRRS, PTRS, PSFRSRS), present LPM-TWFE estimates alongside CRE Probit AME.  
**Rationale:** Addresses the incidental parameters critique of nonlinear models with fixed effects. If LPM and Probit AME give qualitatively similar results, the functional form assumption is not driving the findings. Divergences are flagged and discussed.

### B2. OLS on Logit-Transformed Proportions
**Check:** For all proportion outcomes (PRS, PBotadero, PReciclados, PRSRO), run OLS-TWFE on the logit transformation $\ln(p/(1-p))$ (using 0.001 boundary adjustment).  
**Rationale:** Tests sensitivity of the Fractional Logit model. OLS on the logit transform is a standard alternative that is computationally simpler and directly interpretable in terms of log-odds. Consistency between Fractional Logit AME and OLS-on-logit supports the robustness of results.

### B3. Compositional Consistency Check (Dirichlet / SUR)
**Check:** Estimate a Seemingly Unrelated Regression (SUR) system with PRS, PBotadero, PReciclados, and PRSRO as joint outcomes, imposing (or testing) that their sum equals 1.  
**Rationale:** The proportion outcomes are compositional — they must sum to 1. Analyzing each separately ignores the cross-equation constraint and may produce internally inconsistent predictions. SUR accounts for cross-equation error correlation and tests whether imposing the adding-up restriction changes the coefficients.

### B4. Alternative Mundlak Specification (Full Set of Period Dummies)
**Check:** Replace municipality-level time means (standard Mundlak 1978) with Chamberlain's (1982) full projection on period-specific values $\mathbf{X}_{i1}, \mathbf{X}_{i2}, \ldots, \mathbf{X}_{iT}$ in all CRE nonlinear models.  
**Rationale:** The Mundlak approximation (means) is an approximation to the Chamberlain projection. In short panels (T = 10), the full Chamberlain projection may provide a better approximation to the true conditional distribution. Comparing both specifications tests sensitivity to the Mundlak device's parametric restriction.

### B5. Dynamic Panel (Arellano-Bond GMM) for Continuous Outcomes
**Check:** Estimate a dynamic panel model for QPRS and PRS with one lag of the outcome included, using Arellano-Bond system GMM with collapsed instruments.  
**Rationale:** Outcomes may be persistent (a municipality with high collection coverage in $t-1$ tends to have high coverage in $t$ regardless of spending). Ignoring this persistence biases the static TWFE estimate. The dynamic panel estimate of $\beta_1$ is the short-run effect holding the lagged outcome constant. The long-run effect is $\hat{\beta}_1 / (1 - \hat{\rho})$ where $\hat{\rho}$ is the coefficient on the lagged outcome.

---

## C. Sample Robustness

### C1. Balanced Panel Subsample
**Check:** Re-estimate all main models on the balanced panel — municipalities observed in all 10 years (2011–2020) without gaps.  
**Rationale:** Municipalities that enter or exit the RENAMU sample may do so for reasons correlated with spending and outcomes (e.g., newly created municipalities starting with zero infrastructure; failing municipalities that stop reporting). The balanced panel removes this attrition concern at the cost of potential selection into the balanced sample.

### C2. Exclude Lima Metropolitana
**Check:** Drop the 43 municipalities of Lima Metropolitana and Callao.  
**Rationale:** Lima is an extreme outlier in size, spending, and waste management capacity. Its inclusion may drive the variance-weighted average in TWFE. Results should hold for the other 1,750+ municipalities if the relationship between spending and outcomes is real.

### C3. Exclude Provincial Capital Municipalities
**Check:** Run the analysis on district municipalities only (excluding the 196 provincial capital municipalities).  
**Rationale:** Provincial capitals manage waste for a much larger population, have different fiscal structures, and benefit from co-investment with the provincial government. Their inclusion may obscure effects for the modal municipality, which is a small district.

### C4. Sub-Period Analysis (2011–2015 vs. 2016–2020)
**Check:** Split the sample at 2015 and re-estimate the main models for each sub-period.  
**Rationale:** Several national policies changed over the sample period: Ley de Gestión Integral de Residuos Sólidos (D.Leg. 1278, 2017) and associated MINAM regulations intensified compliance pressure on municipalities. The FONCOMUN formula also changed in 2016. If the spending-outcome relationship shifted after these policy changes, the aggregate estimate masks heterogeneity in treatment effects over time.

### C5. Dropping Municipalities with Implausible Spending Values
**Check:** Re-estimate after trimming municipalities in the bottom and top 1% of log per-capita spending.  
**Rationale:** SIAF data may contain data entry errors — municipalities with implausibly low or high recorded spending per capita. Trimming extreme values tests whether results are driven by outliers.

---

## D. Clustering and Inference Robustness

### D1. Wild Cluster Bootstrap (Province Level)
**Check:** Apply the wild cluster bootstrap (Roodman et al. 2019, Mackinnon-Webb 2018) for all main TWFE specifications.  
**Rationale:** With ~196 province clusters, the asymptotic approximation for cluster-robust standard errors may be unreliable in smaller subsamples (e.g., jungle region has fewer provinces). Wild cluster bootstrap provides more reliable p-values under few clusters.

### D2. Two-Way Clustering (Municipality + Year)
**Check:** Report two-way clustered standard errors (municipality and year dimensions) for all OLS-TWFE models.  
**Rationale:** Shocks may be correlated within municipalities over time (serial correlation) and across municipalities within a year (cross-sectional dependence). Two-way clustering accounts for both dimensions simultaneously. Cameron-Gelbach-Miller (2011) formula.

### D3. Conley Spatial Standard Errors
**Check:** Compute Conley (1999) spatial standard errors for the main TWFE models, with distance cutoffs at 100 km and 200 km.  
**Rationale:** Neighboring municipalities may share waste infrastructure (regional landfills, joint collection routes), inducing spatial correlation in residuals that province-level clustering does not fully absorb.

---

## E. Spending Measure Robustness

### E1. Total Municipal Budget as Alternative Scale
**Check:** Use log per-capita total municipal budget instead of log per-capita limpieza spending as the treatment variable, then use the share of limpieza in total budget as a separate treatment variable.  
**Rationale:** The share of budget devoted to limpieza pública tests whether it is the composition of spending — not just the total level — that drives outcomes.

### E2. Sub-Component Spending Tests
**Check:** Replace the aggregate $G_{it}$ with each of the four sub-components ($G^{recojo}$, $G^{transporte}$, $G^{destino}$, $G^{mantenimiento}$) in separate regressions.  
**Rationale:** If results are driven by a specific spending category, this reveals the mechanism. Expected pattern: $G^{recojo}$ and $G^{transporte}$ should predict collection frequency and coverage (Dimension 1); $G^{destino}$ should predict disposal quality (Dimension 3). Cross-dimension effects (e.g., $G^{mantenimiento}$ predicting PIGARS adoption) would be anomalous.

### E3. Spending Deflated by Provincial CPI (Urban Municipalities)
**Check:** For the subsample of municipalities in cities covered by INEI's provincial CPI, use CPI deflation rather than national GDP deflator.  
**Rationale:** Municipal input prices (labor for waste collection, fuel for trucks) may differ from national aggregate price levels, particularly in Lima and other major cities. CPI deflation provides a more precise real spending measure for urban municipalities.
