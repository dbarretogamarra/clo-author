# Strategy Memo
## Impacto del Gasto Ejecutado en Limpieza Pública sobre la Gestión de Residuos Sólidos Municipales en el Perú, 2011–2020

**Date:** 2026-04-12  
**Author:** Strategist Agent  
**To be reviewed by:** Strategist-Critic  
**Institution:** Universidad Nacional Agraria de la Selva (UNAS) — Maestría en Ciencias Económicas, mención en Proyectos de Inversión

---

## Pre-Strategy Report

**Research spec:** Not found as a standalone file — inferred from project document supplied in the task prompt.  
**Literature review:** Not found as a standalone file — no librarian output located in `quality_reports/`.  
**Data assessment:** Not found as a standalone file — no explorer output located in `quality_reports/`.  
**Domain profile:** Loaded from `.claude/references/domain-profile.md` — file exists but is an unfilled template. Domain-specific conventions (Public Finance / Environmental Economics, FONCOMUN instrument, clustering at province level) are taken from the project document provided in the task prompt and treated as authoritative.

**[ASSUMED: 2011–2020 per project document]** The task prompt states 2011–2020. The CLAUDE.md file refers to 2015–2019. The more specific project document takes precedence. This discrepancy should be confirmed with the user before finalizing the analysis sample.

**Research question (one sentence):** Does executed public spending on street cleaning causally affect municipal solid waste management outcomes (collection efficiency, planning, and final disposal) across Peruvian municipalities between 2011 and 2020?

**Key findings from literature (inferred from domain and context — no formal librarian output available):**
- The environmental economics literature on public waste management in developing countries consistently finds that local government spending is positively associated with collection coverage and formal disposal, but most studies are cross-sectional and cannot rule out reverse causality (municipalities with better waste problems spend more) or omitted variable bias (wealthier municipalities both spend more and have better outcomes).
- Peruvian-specific work on RENAMU and SIAF panels has used pooled OLS and random-effects models; few studies have exploited within-municipality variation via fixed effects, and none known to the strategist have used FONCOMUN transfer formula variation as an instrument for spending.
- The ordered-outcome nature of collection frequency and coverage (ordinal scales 1–5 and 1–4) is typically handled with Ordered Probit/Logit in the health and public economics literatures; LPM is used as a robustness check for interpretability.
- Fractional/bounded proportion outcomes (relleno sanitario share, botadero share, recycled share) are handled with Papke-Wooldridge (1996) Fractional Logit or Beta regression in the environmental economics literature; OLS with logit-transformed outcome is a common robustness check.

**Available data:**
- RENAMU (INEI), annual, 2011–2020: all district and provincial municipalities (~1,800/year). Contains all dependent variables across the three dimensions.
- SIAF/MEF: executed spending by functional classifier "Limpieza pública," decomposed by sub-function (recojo domiciliario, mantenimiento espacios públicos, otros servicios).
- CPV 2017 (INEI): population, urbanization rate, poverty, density — used as time-invariant controls or interpolated.
- MEF FONCOMUN: transfer amounts per municipality per year — available for instrument construction.
- Potential gap: FONCOMUN formula involves population and fiscal capacity weights; the exact formula weights need to be reconstructed from MEF administrative records to build a valid instrument.

**Candidate designs from domain profile (as supplied in task prompt):**
- OLS with municipality and year fixed effects (TWFE)
- Instrumental Variables: FONCOMUN formula-predicted transfers as instrument for spending
- Logit/Probit with average marginal effects (AME) for binary outcomes
- Ordered Logit/Probit for ordinal outcomes
- Fractional Logit for proportion outcomes bounded in [0,1]
- LPM as robustness check alongside probit for binary outcomes

**Proceeding to strategy design.**

---

## Paper Type Classification

**Primary type: Reduced-form**  
The paper estimates the causal effect of a continuous treatment (executed public spending on street cleaning) on multiple outcomes using observational panel data. The ideal experiment would randomly assign spending budgets to municipalities; the design approximates this via panel fixed effects and an IV strategy exploiting exogenous variation in FONCOMUN formula-determined transfers.

**Secondary components:** Descriptive/measurement — the paper also documents the cross-sectional and temporal distribution of waste management outcomes across Peruvian municipalities, which is itself a contribution given that RENAMU-based municipal panel evidence is sparse.

There is no structural or theory-plus-empirics component required by the research questions as stated.

---

## 1. Identification Landscape

### The Ideal Experiment

Randomly assign executed spending budgets on street cleaning to municipalities in a given year, holding all else equal. After one period, measure collection frequency, planning documents adopted, and share of waste directed to sanitary landfills. The treatment effect is the difference in outcomes between randomly high-spending and low-spending municipalities.

### Distance from the Ideal

The observational data departs from the ideal in three ways:

1. **Selection on levels:** Municipalities that spend more on waste management are systematically different from those that spend less — they tend to be larger, wealthier, more urbanized, and located in provinces with stronger fiscal capacity. A naive cross-sectional regression of outcomes on spending conflates the causal effect of spending with these baseline differences.

2. **Reverse causality:** Municipalities facing worse waste management problems (lower collection coverage, reliance on open dumpsites) may be induced to spend more, either by citizen pressure or by national MINAM/MINAM-OEFA compliance requirements. This creates a downward bias in naive OLS: spending appears negatively associated with outcomes because high-problem municipalities spend more but still have bad outcomes.

3. **Omitted time-varying confounders:** National MINAM policy cycles, pre-election spending spikes, and correlated shocks (dengue outbreaks forcing temporary cleanup campaigns) affect both spending and outcomes within a municipality over time, confounding even within-municipality estimates.

### Source of Exogenous Variation

The primary source of exogenous variation exploited is the **FONCOMUN formula**. FONCOMUN (Fondo de Compensación Municipal) is Peru's main intergovernmental fiscal transfer. Its distribution formula is set by the Ley de Tributación Municipal and Ministerial resolutions and is based on:
- Population weights (from INEI census projections, updated administratively)
- Fiscal capacity deficits (proxied by poverty indices, rurality indices, and altitude indicators)
- Municipal classification (district vs. provincial)

Critically, the formula weights are set centrally by MEF and are not chosen by individual municipalities. Year-to-year variation in a municipality's FONCOMUN receipt therefore reflects changes in the centrally-determined formula coefficients and in nationally-aggregated FONCOMUN pool size — both of which are plausibly exogenous to any individual municipality's waste management decisions.

FONCOMUN is unrestricted: municipalities can allocate it to any expenditure function. However, because FONCOMUN constitutes the dominant share of revenue for most Peruvian district municipalities (particularly small, rural, and poor municipalities that are least able to raise own-source revenue), variation in FONCOMUN receipts translates into variation in total and function-specific spending, including limpieza pública.

---

## 2. Estimation Strategy

### 2.1 Overall Identification Framework

The baseline strategy uses **Two-Way Fixed Effects (TWFE)** — municipality fixed effects absorb all time-invariant municipality characteristics (geography, historical institutions, initial infrastructure); year fixed effects absorb all national shocks common to all municipalities (MINAM policy cycles, aggregate economic conditions, changes in national transfer totals).

The IV strategy — **FONCOMUN formula-predicted transfers as an instrument for executed spending** — addresses residual endogeneity from time-varying confounders and reverse causality within municipalities.

Because RENAMU coverage is universal (all municipalities) and treatment (spending) is continuous and varies every year, the design is **staggered in intensity** rather than staggered binary treatment. The standard TWFE estimator for a continuous treatment does not face the heterogeneous treatment timing problem of Callaway-Sant'Anna / Sun-Abraham that afflicts binary staggered DiD. TWFE for continuous treatment is the appropriate estimator here, with the usual caveat that it recovers a variance-weighted average of unit-level slopes.

### 2.2 Estimating Equations

Let $i$ denote municipality, $t$ denote year, and define:

$$G_{it} = \text{log per-capita executed spending on limpieza pública (soles)}$$

The log-per-capita transformation follows field convention for public spending variables and reduces the influence of municipality size. All monetary values deflated to constant 2019 soles using the GDP deflator from INEI.

**Controls vector (minimum):** $\mathbf{X}_{it}$ includes log population, urbanization rate (interpolated from CPV 2007 and CPV 2017), provincial poverty headcount rate (interpolated from ENAHO provincial estimates), log own-source revenue per capita (to separate FONCOMUN-driven from own-revenue-driven spending effects in the IV), and a dummy for whether the municipality is classified as provincial (vs. district).

**Controls vector (extended):** adds population density, altitude quintile dummies (interacted with year), share of population without access to basic sanitation (CPV 2017 interpolated), and log total municipal budget per capita.

**Municipality FE:** $\alpha_i$  
**Year FE:** $\lambda_t$  
**Province $\times$ Year FE (alternative):** $\gamma_{p(i)t}$ — absorbs province-level time-varying shocks (GORE decisions, dengue campaigns, regional elections). Used in robustness.

---

### Dimension 1 — Eficiencia en la Recolección

#### Outcome 1a: Collection Frequency ($FR_{it}$) — Ordinal, 1–5

**Estimand:** The average marginal effect of a unit increase in log per-capita spending on the probability of falling in each ordered category of collection frequency, and on the latent index, across all Peruvian municipalities in the 2011–2020 panel.

**Design choice:** Correlated Random Effects Ordered Probit (Mundlak-Chamberlain device) as the primary specification, because Ordered Probit is the natural model for ordinal outcomes, and standard fixed-effects Ordered Probit is inconsistent (incidental parameters problem). The Mundlak-Chamberlain correction introduces municipality-level means of all time-varying regressors as additional controls, which approximates within-municipality variation while maintaining consistency of the probit.

As robustness: LPM with municipality and year FE for the binary versions "daily or interdiaria" vs. less frequent (FR $\leq 2$ vs. FR $> 2$).

**Estimating equation (latent index):**

$$FR_{it}^* = \beta_1 G_{it} + \mathbf{X}_{it}'\boldsymbol{\beta}_2 + \bar{\mathbf{X}}_i'\boldsymbol{\psi} + \lambda_t + \varepsilon_{it}$$

$$FR_{it} = j \iff \kappa_{j-1} < FR_{it}^* \leq \kappa_j, \quad j \in \{1,2,3,4,5\}$$

where $\bar{\mathbf{X}}_i$ is the vector of municipality-level time means of $\mathbf{X}_{it}$ (the Mundlak device), $\kappa_j$ are estimated thresholds, and $\varepsilon_{it} \sim N(0,1)$.

**Marginal effects reported:** Average marginal effect (AME) of $G_{it}$ on $P(FR_{it} = 1)$ (daily) and $P(FR_{it} \leq 2)$ (daily or interdiaria) — the policy-relevant categories.

**Comparison group:** A municipality with higher log per-capita spending in a given year, compared to the same municipality in a year with lower spending (within-unit variation), after removing common year shocks.

**Key assumption:** Conditional on municipality-level time means, year fixed effects, and observed time-varying controls, the within-municipality variation in $G_{it}$ is uncorrelated with $\varepsilon_{it}$. Threatened by time-varying unobserved heterogeneity such as a new municipal administration committed to both higher spending and better service organization simultaneously.

---

#### Outcome 1b: Daily Collection Quantity ($QPRS_{it}$) — Continuous (kg/day)

**Estimand:** The average treatment effect of a one-percent increase in per-capita spending on daily waste collected (kg), interpreted as an intensive-margin efficiency gain.

**Design choice:** Log-log OLS with municipality and year FE (TWFE). Log transformation of $QPRS_{it}$ stabilizes variance and permits a direct elasticity interpretation. Municipalities reporting zero collection are assigned a small positive value (0.1 kg) and a zero-collection indicator is included as an additional control, following standard practice.

**Estimating equation:**

$$\ln(QPRS_{it}) = \beta_1 G_{it} + \mathbf{X}_{it}'\boldsymbol{\beta}_2 + \alpha_i + \lambda_t + \varepsilon_{it}$$

**Comparison group:** Same municipality, different years. Year FE remove common trends.

**Key assumption:** Parallel trends in log collection quantity across municipalities with different spending trajectories, conditional on observables. Testable via pre-trend event study around spending changes.

---

#### Outcome 1c: Service Coverage ($CSRS_{it}$) — Ordinal, 1–4

**Estimand:** AME of log per-capita spending on probability of reaching the top coverage category ($>75\%$ coverage).

**Design choice:** Correlated Random Effects Ordered Probit (same Mundlak device as FR). The four ordered categories have natural monotonicity consistent with an underlying latent coverage index.

**Estimating equation:** Identical structure to FR:

$$CSRS_{it}^* = \beta_1 G_{it} + \mathbf{X}_{it}'\boldsymbol{\beta}_2 + \bar{\mathbf{X}}_i'\boldsymbol{\psi} + \lambda_t + \varepsilon_{it}$$

$$CSRS_{it} = j \iff \kappa_{j-1} < CSRS_{it}^* \leq \kappa_j, \quad j \in \{1,2,3,4\}$$

**Key assumption:** Same as for FR. The Mundlak device is the critical identification assumption — that municipality-level means of observables capture the relevant sources of time-invariant and mean-reverting unobserved heterogeneity.

---

### Dimension 2 — Planeamiento

The five planning outcomes (PIGARS, PMRS, SRRS, PTRS, PSFRSRS) are all binary (0/1). The primary specification is a **Correlated Random Effects Probit** (Mundlak-Chamberlain device) with AME reported. LPM with TWFE is reported alongside for direct comparability and to assess the sensitivity of the binary nonlinear model to functional form.

**Estimand:** The average treatment effect on the treated (ATT) of a unit increase in log per-capita spending on the probability that a municipality has adopted each planning instrument, averaging over all municipalities and years in the panel.

**Estimating equation (for each binary planning outcome $Y_{it} \in \{PIGARS, PMRS, SRRS, PTRS, PSFRSRS\}$):**

**CRE Probit (primary):**

$$P(Y_{it} = 1 \mid G_{it}, \mathbf{X}_{it}, \bar{\mathbf{X}}_i, \lambda_t) = \Phi\!\left(\beta_1 G_{it} + \mathbf{X}_{it}'\boldsymbol{\beta}_2 + \bar{\mathbf{X}}_i'\boldsymbol{\psi} + \lambda_t\right)$$

**LPM (robustness):**

$$Y_{it} = \beta_1 G_{it} + \mathbf{X}_{it}'\boldsymbol{\beta}_2 + \alpha_i + \lambda_t + \varepsilon_{it}$$

**Comparison group:** The same municipality in different years, after removing year-specific shocks common to all municipalities.

**Key assumption:** Conditional on the Mundlak correction, year FE, and time-varying controls, within-municipality spending variation is uncorrelated with unobserved planning adoption determinants. A particular threat: a new mayoral administration may both increase spending and simultaneously adopt planning documents as part of a broader governance agenda — this is an omitted variable (mayoral ideology/capacity) that is not captured by the observables. The IV strategy using FONCOMUN is the primary defense against this threat.

**Multiple testing:** Because five binary planning outcomes are tested simultaneously, Bonferroni-corrected p-values are reported alongside conventional p-values. The main inference is on the index of all five planning outcomes (a count variable, 0–5) analyzed as a single summary outcome via OLS-TWFE and CRE Poisson to reduce the multiple testing concern.

**Summary outcome:** A planning index $PI_{it} = \sum_{k=1}^{5} Y_{k,it}$ (count 0–5) is the primary dependent variable for this dimension. Individual binary components are secondary.

---

### Dimension 3 — Disposición Final

#### Outcome 3a: Presence of Sanitary Landfill ($RS_{it}$) and Open Dumpsite ($Botadero_{it}$) — Binary

**Design choice:** CRE Probit (primary) + LPM-TWFE (robustness). Same structure as Dimension 2.

**Estimating equation:**

$$P(RS_{it} = 1 \mid \cdot) = \Phi\!\left(\beta_1 G_{it} + \mathbf{X}_{it}'\boldsymbol{\beta}_2 + \bar{\mathbf{X}}_i'\boldsymbol{\psi} + \lambda_t\right)$$

$$P(Botadero_{it} = 1 \mid \cdot) = \Phi\!\left(\beta_1 G_{it} + \mathbf{X}_{it}'\boldsymbol{\beta}_2 + \bar{\mathbf{X}}_i'\boldsymbol{\psi} + \lambda_t\right)$$

**Expected sign:** $\hat{\beta}_1 > 0$ for RS; $\hat{\beta}_1 < 0$ for Botadero (spending reduces reliance on open dumpsites).

---

#### Outcome 3b: Proportion Directed to Sanitary Landfill ($PRS_{it}$) and to Dumpsite ($PBotadero_{it}$) — Proportions in $[0,1]$

**Design choice:** **Correlated Random Effects Fractional Logit** (Papke-Wooldridge 1996, extended to panel by Wooldridge 2010) as primary specification, because proportions bounded in $[0,1]$ cannot be handled by standard linear models without risking predictions outside the unit interval and heteroskedasticity of a specific form.

**Estimating equation (CRE Fractional Logit):**

$$E(PRS_{it} \mid G_{it}, \mathbf{X}_{it}, \bar{\mathbf{X}}_i, \lambda_t) = \Lambda\!\left(\beta_1 G_{it} + \mathbf{X}_{it}'\boldsymbol{\beta}_2 + \bar{\mathbf{X}}_i'\boldsymbol{\psi} + \lambda_t\right)$$

where $\Lambda(\cdot)$ is the logistic CDF. Estimated via quasi-MLE (QMLE) with robust standard errors. Partial effects (AME) are reported in the probability scale.

**OLS-TWFE robustness:** OLS on the logit transformation $\ln(PRS_{it}/(1-PRS_{it}))$ (observations with $PRS_{it} \in \{0,1\}$ replaced with small adjustments $\epsilon = 0.001$ following Smithson and Verkuilen 2006). This tests sensitivity to functional form.

**Recycled and Burned proportions:** Same Fractional Logit approach. Note that $PRS + PBotadero + PReciclados + PRSRO + \text{other} = 1$ by construction (compositional data), so a **Dirichlet regression** or **seemingly unrelated regression** approach is explored in the robustness section to account for cross-equation correlation.

**Key assumption:** Same Mundlak-based identification as other dimensions. Additional concern: some municipalities have missing data on proportions (they report presence/absence of a disposal facility but not the exact share). Missing data is handled via a two-stage approach: (1) model the extensive margin (presence/absence) with CRE Probit; (2) model the intensive margin (share conditional on using the facility) with CRE Fractional Logit on the selected sample, with a Heckman selection correction using the Mundlak device as an exclusion restriction in the selection equation.

---

### 2.3 Instrumental Variables Strategy

**Instrument:** $\hat{F}_{it}$ — the **FONCOMUN formula-predicted transfer** for municipality $i$ in year $t$, constructed as:

$$\hat{F}_{it} = \text{(national FONCOMUN pool)}_t \times w_{it}^{\text{formula}}$$

where $w_{it}^{\text{formula}}$ is municipality $i$'s formula weight in year $t$, derived from:
- Population weight: $w_{pop,it}$ from INEI projections updated annually and used in the official MEF allocation
- Fiscal capacity / poverty weight: $w_{fis,it}$ from poverty and rurality indices
- Municipal classification weight: fixed categorical adjustment

$\hat{F}_{it}$ is constructed using **only the formula weights and the national pool size** — not the actual transfer received, which may reflect side payments, arrears, or administrative delays. This "Bartik-style" leave-one-out construction ensures the instrument reflects nationally-determined variation, not municipality-specific negotiation.

**First stage:**

$$G_{it} = \pi_1 \hat{F}_{it} + \mathbf{X}_{it}'\boldsymbol{\pi}_2 + \alpha_i + \lambda_t + u_{it}$$

**Second stage:** Replace $G_{it}$ with $\hat{G}_{it}$ from the first stage in all outcome equations.

**Exclusion restriction:** $\hat{F}_{it}$ affects waste management outcomes only through its effect on municipal spending. The restriction would be violated if: (a) FONCOMUN formula weights reflect poverty or rurality in ways that directly affect waste outcomes independently of spending (poverty effect); (b) the national pool size co-moves with aggregate factors (e.g., VAT revenue cycles) that also affect municipal outcomes through non-spending channels.

**Defense of exclusion restriction:**
- (a) is addressed by including provincial poverty rates and urbanization directly in $\mathbf{X}_{it}$, and by demonstrating that formula weights conditional on these controls are orthogonal to lagged outcomes.
- (b) is addressed by including year fixed effects, which absorb all national time variation in the pool.
- A placebo test using outcomes for which spending should have no effect (e.g., cemetery maintenance outcomes if available, or the number of parks — outcomes unlikely to be affected by limpieza pública spending) is used to check for aggregate confounds.

**Relevance:** Municipalities in the lowest quintile of own-source revenue depend on FONCOMUN for 80–95% of their total revenue. Formula-predicted transfers are strongly correlated with actual transfers and hence with total spending including limpieza pública. The first-stage F-statistic (Montiel Olea-Pflueger effective F) is reported; the rule-of-thumb threshold of 10 (conventional) and 23.11 (for 5% worst-case size distortion, tF critical value) is checked.

**LATE interpretation:** Because FONCOMUN-induced spending variation is largest in small, poor, rural municipalities with low own-source revenue, the IV estimates identify the LATE for compliers — municipalities whose limpieza pública spending responds to FONCOMUN variation. This sub-population is precisely the policy-relevant margin: these are the municipalities where central government transfers are the binding constraint on local spending. The LATE is explicitly characterized in the paper.

**Implementation note:** The IV is implemented via 2SLS for the continuous outcomes (QPRS, proportions treated as linear in robustness). For the nonlinear models (CRE Probit, CRE Ordered Probit, CRE Fractional Logit), a **control function approach** (Wooldridge 2015) is used: the first-stage residual $\hat{u}_{it}$ is included as an additional regressor in the second-stage nonlinear model, and standard errors are bootstrapped to account for the two-step estimation.

---

### 2.4 Spending Decomposition

Beyond the aggregate spending measure, the sub-components of spending are analyzed separately:

$$G_{it}^{(k)}, \quad k \in \{\text{recojo}, \text{transporte}, \text{destino final}, \text{mantenimiento espacios}, \text{otros}\}$$

This decomposition allows mapping specific spending types to theoretically-motivated outcomes:
- $G^{recojo}$ and $G^{transporte}$ should primarily affect Dimension 1 (collection efficiency)
- $G^{destino final}$ should primarily affect Dimension 3 (disposal)
- $G^{mantenimiento}$ should affect collection coverage and public space cleanliness

The decomposition tests are confirmatory: a finding that $G^{destino final}$ has a large effect on Dimension 1 outcomes (collection frequency) would be anomalous and raise concerns about reverse causality or mismeasurement.

---

### 2.5 Clustering Strategy

**Primary clustering:** Province level ($p(i)$). Peru has ~196 provinces. Province-level clustering accounts for spatial correlation in outcomes and spending (neighboring municipalities within a province share infrastructure, regional government decisions, and geographic shocks) while maintaining enough clusters for asymptotic validity of cluster-robust inference.

**Rationale:** Municipality-level clustering ($\sim$1,800 clusters) would be over-precise and ignore the spatial correlation structure. Department-level clustering ($\sim$25 clusters) would be too few for reliable cluster-robust inference. Province-level is the natural administrative unit mediating between municipalities and the regional (GORE) level.

**Robustness:** Wild cluster bootstrap (Roodman et al. 2019) at the province level for the main TWFE results, given that the cluster count of ~196 may make the asymptotic approximation borderline in some subsamples. Two-way clustering (municipality + year) as an additional robustness check.

---

### 2.6 Lag Structure

**Baseline:** $G_{it}$ enters contemporaneously. This recovers the within-year effect of spending on reported outcomes (since both are measured as of the RENAMU survey date, typically end of year).

**Distributed lag:** Because capital investments in disposal infrastructure (constructing a sanitary landfill) have multi-year payoffs, a distributed lag model with $G_{it}$, $G_{it-1}$, and $G_{it-2}$ is estimated for the disposal outcomes. The sum of lag coefficients is the cumulative effect.

**Pre-treatment lagged outcome:** $Y_{it-1}$ is included as a control in a dynamic panel specification (Arellano-Bond GMM) as a robustness check on the Mundlak-based CRE models.

---

## 3. Summary Table of Outcome-Model Pairings

| Outcome | Dimension | Scale | Primary Model | Robustness |
|---------|-----------|-------|---------------|------------|
| FR (collection freq.) | 1 | Ordinal 1–5 | CRE Ordered Probit (AME) | LPM-TWFE on binary dichotomization |
| QPRS (kg/day) | 1 | Continuous | Log-log OLS TWFE | IV-2SLS, Poisson QMLE (count-like) |
| CSRS (coverage) | 1 | Ordinal 1–4 | CRE Ordered Probit (AME) | LPM-TWFE on binary dichotomization |
| PI (planning index 0–5) | 2 | Count | OLS TWFE + CRE Poisson | Individual LPM-TWFE per binary component |
| PIGARS, PMRS, SRRS, PTRS, PSFRSRS | 2 | Binary | CRE Probit (AME) | LPM-TWFE |
| RS (sanitary landfill) | 3 | Binary | CRE Probit (AME) | LPM-TWFE |
| Botadero | 3 | Binary | CRE Probit (AME) | LPM-TWFE |
| PRS (share to landfill) | 3 | Proportion [0,1] | CRE Fractional Logit (AME) | OLS on logit transform |
| PBotadero (share to dumpsite) | 3 | Proportion [0,1] | CRE Fractional Logit (AME) | OLS on logit transform |
| PReciclados | 3 | Proportion [0,1] | CRE Fractional Logit (AME) | OLS on logit transform |
| Reciclados, Quemados | 3 | Binary | CRE Probit (AME) | LPM-TWFE |

---

## 4. Referee Objections and Responses

**Objection 1: "Spending is endogenous — municipalities with worse outcomes spend more (reverse causality)."**

Response: This is the central identification threat, and it motivates both the TWFE design and the IV strategy. The TWFE removes time-invariant municipality characteristics that drive both spending and outcomes. The IV using formula-predicted FONCOMUN transfers isolates spending variation that is externally determined by MEF's formula — a municipality's FONCOMUN share does not respond to its own waste management outcomes. The first stage is reported with the Montiel Olea-Pflueger effective F statistic. The IV estimates are the headline identification results; TWFE estimates are presented as a baseline comparison.

**Objection 2: "The FONCOMUN exclusion restriction is violated — poorer municipalities get more transfers AND have worse waste management for reasons unrelated to spending."**

Response: This threat is directly addressed. Provincial poverty rates and urbanization rates are included as controls in $\mathbf{X}_{it}$, so the instrument's effect on spending is identified from within-municipality variation in formula-predicted transfers over time, not from the cross-sectional correlation between poverty and transfers. A falsification test uses outcomes unrelated to limpieza pública spending (e.g., spending on sports infrastructure or municipal cemetery management) to check that the instrument does not have a residual effect on outcomes through non-spending channels. A Conley et al. (2012) sensitivity analysis formally bounds the coefficient estimates under varying degrees of exclusion restriction violation.

**Objection 3: "The Mundlak-Chamberlain device does not fully control for municipality fixed effects in nonlinear models — why not use a linear probability model throughout?"**

Response: The LPM is reported as a robustness check for every binary and ordinal outcome. The CRE nonlinear models are preferred because: (a) for ordinal outcomes (FR, CSRS), the LPM has no natural analog; (b) for proportions close to 0 or 1, OLS predictions can be outside the unit interval; (c) the Mundlak device has been shown by Wooldridge (2010) to be consistent for panel probit under standard regularity conditions. The concordance between LPM and CRE Probit AME across all binary outcomes is shown in a summary comparison table, and robustness to the Mundlak specification (using Chamberlain's projection vs. full set of period dummies) is demonstrated.

**Objection 4: "Your results are driven by large, urban municipalities with more capacity — the effects may not apply to small, rural municipalities where waste management is most deficient."**

Response: Heterogeneity analysis is conducted by: (a) municipality population quartile; (b) urban vs. rural classification; (c) coastal vs. highlands vs. jungle macroregion; (d) poverty level (below vs. above median provincial poverty rate). The LATE interpretation of the IV results already focuses on the most policy-relevant margin: small, poor municipalities with low fiscal capacity that depend on FONCOMUN. Results are presented separately for this sub-population.

**Objection 5: "With 10–15 outcome variables tested, results may reflect multiple testing rather than true effects."**

Response: The three research questions are organized around three aggregate summary outcomes (collection efficiency index, planning index, disposal quality index), each constructed as an equally-weighted standardized index following Kling, Liebman, and Katz (2007). These three indices are the primary outcomes on which inference is based. Individual component outcomes are secondary. Bonferroni-Holm corrected p-values are reported for all individual-component tests. The pre-specified hierarchy of outcomes (summary index primary, components secondary) is declared in the thesis proposal.

---

## 5. Data Notes and Construction Decisions

1. **Sample:** All district and provincial municipalities with non-missing spending data in SIAF and at least one RENAMU response. Municipalities that never appear in RENAMU are dropped; municipalities that appear in some but not all years generate an unbalanced panel — the analysis uses the unbalanced panel and reports robustness on the balanced subsample.

2. **Spending deflation:** All spending values deflated to constant 2019 soles using the national GDP deflator from INEI. Alternatively, the provincial CPI (available for selected cities) is used in robustness checks for urban municipalities.

3. **Per-capita transformation:** All spending variables are expressed per inhabitant using INEI's annual population projections by municipality. This controls for municipality size mechanically — larger municipalities spend more in absolute terms.

4. **RENAMU response quality:** RENAMU is a self-reported census of municipalities. For outcomes that are self-reported binary indicators (e.g., PIGARS = yes/no), there is a risk of reporting bias — municipalities may over-report having planning instruments to signal compliance with MINAM. This is flagged as a limitation and partially addressed by cross-validating PIGARS adoption against MINAM's official registry of approved plans (where available).

5. **Spending-outcome timing:** SIAF records executed (devengado) spending at the end of each fiscal year. RENAMU is conducted in the same fiscal year. The contemporaneous specification is appropriate if spending and outcomes are both reported as of year-end. Lagged specifications (spending in $t-1$ affecting outcomes in $t$) are tested as robustness.

6. **[ASSUMED: 2011–2020 per project document]** — confirm with user before locking the sample.
