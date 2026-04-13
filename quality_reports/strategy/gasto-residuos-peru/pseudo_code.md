# Pseudo-Code: Main Estimation
## Gasto en Limpieza Pública y Gestión de Residuos Sólidos, Perú 2011–2020

**Note:** This is specification-level pseudo-code. It describes the estimation logic without running any code.  
All paths are relative to the project root via `here()`.

---

## 0. Setup and Data Assembly

```r
# ----- 0. Packages (loaded at top, never inside functions) -----
library(here)
library(tidyverse)
library(haven)
library(fixest)        # feols, feglm — fast TWFE and CRE models
library(survival)      # for Efron approximation if needed
library(lmtest)
library(sandwich)
library(clubSandwich)  # cluster-robust for nonlinear models
library(margins)       # average marginal effects
library(MASS)          # polr() for ordered logit/probit
library(geepack)       # GEE as robustness
library(fwildclusterboot) # wild cluster bootstrap

set.seed(20240101)     # once, at top

# ----- 0.1 Load RENAMU panel (cleaned by data-engineer) -----
renamu <- read_dta(here("data", "cleaned", "renamu_panel_2011_2020.dta"))

# ----- 0.2 Load SIAF spending (cleaned) -----
siaf   <- read_dta(here("data", "cleaned", "siaf_limpieza_2011_2020.dta"))

# ----- 0.3 Load FONCOMUN formula-predicted transfers (instrument) -----
foncomun <- read_dta(here("data", "cleaned", "foncomun_formula_predicted.dta"))

# ----- 0.4 Load controls -----
controls <- read_dta(here("data", "cleaned", "controls_municipal_panel.dta"))
# Contains: log_pop, urban_rate, poverty_rate, log_own_revenue_pc,
#           provincial_dummy, pop_density, altitude_quintile, sanitation_deficit

# ----- 0.5 Merge to panel -----
panel <- renamu |>
  left_join(siaf,     by = c("ubigeo", "year")) |>
  left_join(foncomun, by = c("ubigeo", "year")) |>
  left_join(controls, by = c("ubigeo", "year"))

# ----- 0.6 Construct key variables -----
panel <- panel |>
  mutate(
    # Log per-capita spending (main treatment)
    G_pc     = log(gasto_limpieza_deflactado / poblacion + 1),
    # Sub-components
    G_recojo = log(gasto_recojo_deflactado / poblacion + 1),
    G_dest   = log(gasto_destino_deflactado / poblacion + 1),
    G_mant   = log(gasto_mantenimiento_deflactado / poblacion + 1),
    # Instrument: log per-capita formula-predicted FONCOMUN
    Z_foncomun = log(foncomun_formula_pc + 1),
    # Planning index (count 0-5)
    PI = PIGARS + PMRS + SRRS + PTRS + PSFRSRS,
    # Log outcomes where appropriate
    ln_QPRS = log(QPRS + 0.1),     # small constant for zeros
    zero_coll = as.integer(QPRS == 0),
    # Proportion small-constant adjustment for OLS robustness
    PRS_adj        = pmax(pmin(PRS, 1 - 0.001), 0.001),
    PBotadero_adj  = pmax(pmin(PBotadero, 1 - 0.001), 0.001),
    PReciclados_adj = pmax(pmin(PReciclados, 1 - 0.001), 0.001),
    # Logit transforms for OLS robustness on proportions
    logit_PRS       = log(PRS_adj / (1 - PRS_adj)),
    logit_PBotadero = log(PBotadero_adj / (1 - PBotadero_adj)),
    # Mundlak time-means (within-municipality means of time-varying regressors)
    # Computed after grouping by ubigeo
  ) |>
  group_by(ubigeo) |>
  mutate(
    mean_G_pc          = mean(G_pc, na.rm = TRUE),
    mean_log_pop       = mean(log(poblacion), na.rm = TRUE),
    mean_poverty_rate  = mean(poverty_rate, na.rm = TRUE),
    mean_urban_rate    = mean(urban_rate, na.rm = TRUE),
    mean_log_own_rev   = mean(log_own_revenue_pc + 1, na.rm = TRUE)
  ) |>
  ungroup()

# Factor variables
panel <- panel |>
  mutate(
    FR_f    = factor(FR,    levels = 1:5, ordered = TRUE),
    CSRS_f  = factor(CSRS,  levels = 1:4, ordered = TRUE),
    year_f  = factor(year),
    ubigeo_f = factor(ubigeo)
  )
```

---

## 1. Dimension 1 — Eficiencia en la Recolección

### 1a. Collection Frequency (FR) — CRE Ordered Probit

```r
# Controls formula (time-varying)
controls_formula <- ~ log(poblacion) + urban_rate + poverty_rate +
                       log_own_revenue_pc + provincial_dummy +
                       zero_coll + year_f

# Mundlak terms (time-means)
mundlak_terms <- ~ mean_G_pc + mean_log_pop + mean_poverty_rate +
                    mean_urban_rate + mean_log_own_rev

# CRE Ordered Probit
m_FR_cre_oprobit <- MASS::polr(
  FR_f ~ G_pc + log(poblacion) + urban_rate + poverty_rate +
         log_own_revenue_pc + provincial_dummy + year_f +
         mean_G_pc + mean_log_pop + mean_poverty_rate +
         mean_urban_rate + mean_log_own_rev,
  data   = panel,
  method = "probit",
  Hess   = TRUE
)

# Average marginal effects on P(FR = 1) and P(FR <= 2)
# Use margins package or manual delta-method computation
ame_FR <- margins::margins(m_FR_cre_oprobit,
                           variables  = "G_pc",
                           at         = list(category = 1))

# Cluster-robust SEs at province level via clubSandwich
vcov_FR <- clubSandwich::vcovCR(m_FR_cre_oprobit,
                                cluster = panel$province_code,
                                type    = "CR1S")

# LPM robustness: dichotomize FR <= 2 vs FR > 2
panel <- panel |>
  mutate(FR_freq_high = as.integer(FR <= 2))

m_FR_lpm <- fixest::feols(
  FR_freq_high ~ G_pc + log(poblacion) + urban_rate + poverty_rate +
                 log_own_revenue_pc + provincial_dummy |
                 ubigeo + year,
  data     = panel,
  cluster  = ~province_code
)
```

### 1b. Daily Collection Quantity (QPRS) — Log-Log TWFE

```r
m_QPRS_twfe <- fixest::feols(
  ln_QPRS ~ G_pc + log(poblacion) + urban_rate + poverty_rate +
             log_own_revenue_pc + provincial_dummy + zero_coll |
             ubigeo + year,
  data    = panel,
  cluster = ~province_code
)

# IV version (2SLS)
m_QPRS_iv <- fixest::feols(
  ln_QPRS ~ log(poblacion) + urban_rate + poverty_rate +
             log_own_revenue_pc + provincial_dummy + zero_coll |
             ubigeo + year |
             G_pc ~ Z_foncomun,
  data    = panel,
  cluster = ~province_code
)

# First-stage F: fixest reports this automatically; check effective F
summary(m_QPRS_iv, stage = 1)
```

### 1c. Service Coverage (CSRS) — CRE Ordered Probit

```r
m_CSRS_cre_oprobit <- MASS::polr(
  CSRS_f ~ G_pc + log(poblacion) + urban_rate + poverty_rate +
            log_own_revenue_pc + provincial_dummy + year_f +
            mean_G_pc + mean_log_pop + mean_poverty_rate +
            mean_urban_rate + mean_log_own_rev,
  data   = panel,
  method = "probit",
  Hess   = TRUE
)

vcov_CSRS <- clubSandwich::vcovCR(m_CSRS_cre_oprobit,
                                  cluster = panel$province_code,
                                  type    = "CR1S")
```

---

## 2. Dimension 2 — Planeamiento

### 2a. Planning Index (PI, count 0–5) — OLS TWFE (primary)

```r
m_PI_twfe <- fixest::feols(
  PI ~ G_pc + log(poblacion) + urban_rate + poverty_rate +
       log_own_revenue_pc + provincial_dummy |
       ubigeo + year,
  data    = panel,
  cluster = ~province_code
)

# CRE Poisson for count robustness
m_PI_cre_poisson <- fixest::feglm(
  PI ~ G_pc + log(poblacion) + urban_rate + poverty_rate +
       log_own_revenue_pc + provincial_dummy + year_f +
       mean_G_pc + mean_log_pop + mean_poverty_rate +
       mean_urban_rate + mean_log_own_rev,
  data   = panel,
  family = poisson(),
  cluster = ~province_code
)
```

### 2b. Individual Binary Planning Outcomes — CRE Probit + LPM

```r
planning_outcomes <- c("PIGARS", "PMRS", "SRRS", "PTRS", "PSFRSRS")

# CRE Probit for each (function to avoid code repetition)
fit_cre_probit <- function(outcome, data) {
  fmla <- as.formula(
    paste0(outcome, " ~ G_pc + log(poblacion) + urban_rate + poverty_rate +",
           " log_own_revenue_pc + provincial_dummy + year_f +",
           " mean_G_pc + mean_log_pop + mean_poverty_rate +",
           " mean_urban_rate + mean_log_own_rev")
  )
  glm(fmla, data = data, family = binomial(link = "probit"))
}

# Pre-allocate results list
models_probit <- vector("list", length(planning_outcomes))
names(models_probit) <- planning_outcomes

for (k in seq_along(planning_outcomes)) {
  models_probit[[k]] <- fit_cre_probit(planning_outcomes[k], panel)
}

# LPM with TWFE for each
models_lpm <- vector("list", length(planning_outcomes))
names(models_lpm) <- planning_outcomes

for (k in seq_along(planning_outcomes)) {
  fmla_lpm <- as.formula(
    paste0(planning_outcomes[k],
           " ~ G_pc + log(poblacion) + urban_rate + poverty_rate +",
           " log_own_revenue_pc + provincial_dummy | ubigeo + year")
  )
  models_lpm[[k]] <- fixest::feols(fmla_lpm, data = panel,
                                    cluster = ~province_code)
}

# Bonferroni-Holm adjusted p-values across the five tests
# Collect p-values from LPM models, apply p.adjust(method = "holm")
p_values_lpm <- sapply(models_lpm, function(m) {
  coef(summary(m))["G_pc", "Pr(>|t|)"]
})
p_adjusted <- p.adjust(p_values_lpm, method = "holm")
```

---

## 3. Dimension 3 — Disposición Final

### 3a. Binary Outcomes (RS, Botadero, Reciclados) — CRE Probit + LPM

```r
disposal_binary <- c("RS", "Botadero", "Reciclados", "Quemados")

models_disp_probit <- vector("list", length(disposal_binary))
names(models_disp_probit) <- disposal_binary

for (k in seq_along(disposal_binary)) {
  models_disp_probit[[k]] <- fit_cre_probit(disposal_binary[k], panel)
}

# LPM versions: same loop structure as planning outcomes above
```

### 3b. Proportion Outcomes — CRE Fractional Logit

```r
prop_outcomes <- c("PRS", "PBotadero", "PReciclados")

fit_cre_fractlogit <- function(outcome, data) {
  fmla <- as.formula(
    paste0(outcome, " ~ G_pc + log(poblacion) + urban_rate + poverty_rate +",
           " log_own_revenue_pc + provincial_dummy + year_f +",
           " mean_G_pc + mean_log_pop + mean_poverty_rate +",
           " mean_urban_rate + mean_log_own_rev")
  )
  # Quasi-MLE with binomial family and logit link (Papke-Wooldridge)
  glm(fmla, data = data, family = quasibinomial(link = "logit"))
}

models_frac <- vector("list", length(prop_outcomes))
names(models_frac) <- prop_outcomes

for (k in seq_along(prop_outcomes)) {
  models_frac[[k]] <- fit_cre_fractlogit(prop_outcomes[k], panel)
}

# Province-clustered SEs for each fractional logit model
for (k in seq_along(prop_outcomes)) {
  vcov_k <- sandwich::vcovCL(models_frac[[k]],
                             cluster = ~province_code,
                             data    = panel)
  # Store vcov for inference
}

# OLS robustness on logit-transformed proportions
m_logitPRS_twfe <- fixest::feols(
  logit_PRS ~ G_pc + log(poblacion) + urban_rate + poverty_rate +
               log_own_revenue_pc + provincial_dummy |
               ubigeo + year,
  data    = panel,
  cluster = ~province_code
)
```

---

## 4. Control Function IV for Nonlinear Models

```r
# Step 1: First stage OLS to obtain residuals
first_stage <- fixest::feols(
  G_pc ~ Z_foncomun + log(poblacion) + urban_rate + poverty_rate +
         log_own_revenue_pc + provincial_dummy |
         ubigeo + year,
  data    = panel,
  cluster = ~province_code
)
panel$vhat <- residuals(first_stage)  # first-stage residuals

# Step 2: Include vhat in nonlinear models (control function approach)
# Example for RS binary
m_RS_cf_probit <- glm(
  RS ~ G_pc + vhat + log(poblacion) + urban_rate + poverty_rate +
       log_own_revenue_pc + provincial_dummy + year_f +
       mean_G_pc + mean_log_pop + mean_poverty_rate +
       mean_urban_rate + mean_log_own_rev,
  data   = panel,
  family = binomial(link = "probit")
)

# Step 3: Bootstrap SEs (B = 500) to account for two-step estimation
# Cluster bootstrap at province level
# [outer loop over bootstrap samples; inner loop fits both stages]
B <- 500
boot_coefs <- matrix(NA, nrow = B, ncol = length(coef(m_RS_cf_probit)))

for (b in 1:B) {
  # Sample province clusters with replacement
  sampled_provinces <- sample(unique(panel$province_code),
                               replace = TRUE)
  boot_data <- panel |>
    filter(province_code %in% sampled_provinces)
  # Re-fit first stage on boot_data
  # Re-compute vhat on boot_data
  # Re-fit CF probit on boot_data
  # Store coef
}
# AME and bootstrap CIs from boot_coefs
```

---

## 5. Heterogeneity Analysis

```r
# Quintile of own-source revenue (proxy for fiscal capacity)
panel <- panel |>
  group_by(year) |>
  mutate(own_rev_quintile = ntile(log_own_revenue_pc, 5)) |>
  ungroup()

# Interaction with FONCOMUN-dependence indicator (bottom 2 quintiles = high dependence)
panel <- panel |>
  mutate(high_foncomun_dep = as.integer(own_rev_quintile <= 2))

m_het_QPRS <- fixest::feols(
  ln_QPRS ~ G_pc * high_foncomun_dep + log(poblacion) + urban_rate +
             poverty_rate + log_own_revenue_pc + provincial_dummy |
             ubigeo + year,
  data    = panel,
  cluster = ~province_code
)

# Urban vs rural, region fixed effects interactions: similar structure
```

---

## 6. Export Tables

```r
# All tables exported as bare tabular environments (no \begin{table}, no \caption)
# per INV-13 — paper/main.tex wraps them

# Example: main TWFE results for Dimension 1 continuous outcome
fixest::etable(
  m_QPRS_twfe, m_QPRS_iv,
  file       = here("paper", "tables", "tab_dim1_qprs.tex"),
  tex        = TRUE,
  style.tex  = style.tex("aer"),
  signif.code = c("***"=0.01, "**"=0.05, "*"=0.10),
  se.below   = TRUE,
  keep       = "G_pc"
)
```
