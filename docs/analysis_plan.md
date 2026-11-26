# Benchmarking Analysis Plan: Photon and Heavy-Ion Cell Survival Models

> **Objective:** Define a pre-registered, leakage-free benchmarking protocol comparing candidate radiobiological survival models (**M1–M5**) across photon and heavy-ion datasets, with strict parameter parity, objective exclusion rules, and pre-specified success/failure criteria.

---

## 1. Model Formulations & Parameter Rules

### Mathematical Definitions
Write out the exact equations, required inputs, and free parameters for each candidate model:

* **M1 (Fixed LQ):** Baseline Linear-Quadratic survival model with fixed radiosensitivity parameters across conditions:
  $$S(D) = \exp\!\left(-\alpha D - \beta D^2\right)$$
  * **Inputs:** Absorbed dose $D$.
  * **Free parameters:** $\alpha$, $\beta$.

* **M2 (Empirical LET Scaling):** Linear-Quadratic formulation where radiosensitivity parameterizes empirically as a function of Linear Energy Transfer ($\text{LET}$):
  $$S(D, \text{LET}) = \exp\!\left(-\alpha(\text{LET})\,D - \beta(\text{LET})\,D^2\right)$$
  * **Inputs:** Absorbed dose $D$, $\text{LET}$.
  * **Free parameters:** Parameters governing $\alpha(\text{LET})$ and $\beta(\text{LET})$ (e.g., baseline and LET-slope/curvature terms).

* **M3 (MKM Domain Model):** Microdosimetric Kinetic Model partitioning the cell nucleus into domains with lesion formation driven by specific energy / microdosimetric quantities ($z_D$, $y_D$) and domain/nucleus geometry:
  $$S(D, z_D) = \exp\!\left(-\alpha(z_D)\,D - \beta(z_D)\,D^2\right), \qquad \alpha(z_D) = \alpha_0 + \beta_0 z_D, \qquad \beta(z_D) = \beta_0$$
  *(or full domain/saturation MKM formulation as pre-specified)*
  * **Inputs:** Absorbed dose $D$, microdosimetric quantities (e.g., dose-mean specific energy $z_D$ or dose-mean lineal energy $y_D$), and domain/nuclear geometric inputs.
  * **Free parameters:** $\alpha_0$, $\beta_0$, and domain/saturation parameters as defined in the model specification.

* **M4 (Damage Complexity Yields):** Survival modeled directly from DNA double-strand break (DSB) / clustered damage yield and complexity distributions (e.g., complexity-weighted or polynomial/mechanistic lesion yield mapping):
  $$S = f\!\left(Y_{\text{DSB}}, C_{\text{DSB}}, D\right)$$
  * **Inputs:** Absorbed dose $D$, DSB / clustered damage yields ($Y_{\text{DSB}}$) and complexity metrics ($C_{\text{DSB}}$).
  * **Free parameters:** Yield-to-lethality / complexity weights and baseline survival parameters.

* **M5 (Complexity Plus Fast/Slow Repair ODE Kinetics):** Mechanistic damage-complexity input coupled with ordinary differential equation (ODE) repair-misrepair dynamics across fast and slow repair components and lethal lesion mapping:
  $$\frac{dN_{\text{fast}}}{dt} = \eta_{\text{fast}}\dot{D}(t) - \varepsilon_{\text{fast}}N_{\text{fast}} - \varepsilon_{2,\text{fast}}N_{\text{fast}}^2 - \kappa_{\text{fast}}N_{\text{fast}}$$
  $$\frac{dN_{\text{slow}}}{dt} = \eta_{\text{slow}}\dot{D}(t) - \varepsilon_{\text{slow}}N_{\text{slow}} - \varepsilon_{2,\text{slow}}N_{\text{slow}}^2 - \kappa_{\text{slow}}N_{\text{slow}}$$
  $$\frac{dN_L}{dt} = \eta_L\dot{D}(t) + \kappa_{\text{fast}}N_{\text{fast}} + \kappa_{\text{slow}}N_{\text{slow}} + \varepsilon_{2,\text{fast}}N_{\text{fast}}^2 + \varepsilon_{2,\text{slow}}N_{\text{slow}}^2 + \varepsilon_{12}N_{\text{fast}}N_{\text{slow}}$$
  $$S = \exp\!\left(-N_L\right)$$
  * **Inputs:** Dose rate $\dot{D}(t)$ (or dose $D$ and exposure profile) and radiation/DSB complexity inputs.
  * **Free parameters:** Production ($\eta$), linear repair ($\varepsilon$), binary misrepair ($\varepsilon_2$, $\varepsilon_{12}$), and fixation ($\kappa$) rate parameters for fast and slow repair pathways.

### Parameter Parity & Uniform Fitting
* Explicitly document the exact count of **free parameters** for every model (**M1** through **M5**).
* Enforce **identical optimizers, convergence tolerances, parameter bounds, literature priors, and fitting configurations** across models so no candidate gains unstated tuning or optimization advantages.

### Availability Gate Rule
* If required public inputs for **M3** or **M4** cannot be obtained for a dataset or split, **formally drop** the affected model for that evaluation rather than substituting unverified guesses or ad-hoc imputations.

---

## 2. Leakage-Free Cross-Validation Splits

Apply all out-of-sample data partitioning **before** any model fitting, prior tuning, or hyperparameter selection begins:

* **Leave-One-Cell-Line-Out (LOCLO):** Holds out all records for a single cell line to evaluate parameter transfer to unseen biological cell lines.
* **Leave-One-Radiation-Quality-Out (LOQO):** Holds out specific ion species or $\text{LET}$ ranges to test extrapolation across radiation qualities.
* **Leave-One-Source-Out (LOSO):** Holds out all data from an entire laboratory or publication to evaluate robustness against between-laboratory and batch variability.
* **Leave-One-Dose-Rate-Out (LODRO):** Holds out specific dose-rate regimes to test dose-rate extrapolation where sufficient public data are available.

---

## 3. Quantitative Evaluation Metrics

* **Primary Metric:** Root-mean-square error (**$\text{log RMSE}$**) of the log surviving fraction ($\ln S$ or $\log_{10} S$, defined consistently across all runs) on held-out test partitions.
* **Secondary Metrics:**
  * **Mean Absolute Error ($\text{MAE}$)** on held-out log surviving fraction.
  * **Empirical coverage** of **90% prediction intervals**.
  * **Rank agreement** (e.g., Spearman's $\rho$ / Kendall's $\tau$) of surviving fraction at $2\text{ Gy}$ (**$SF_2$**) across held-out cell lines/conditions.

---

## 4. Primary Hypotheses & Success Criteria

* **Frozen Primary Comparisons:** Restrict confirmatory statistical testing strictly to:
  1. **M5 vs. M2**
  2. **M5 vs. M4**
  Label all other pairwise model comparisons as **exploratory**.
* **Success Criterion:** **M5** achieves lower held-out **$\text{log RMSE}$** than both **M2** and **M4**, with a paired-difference **95% confidence interval**-computed via **cluster bootstrapping** grouped by cell line or source-excluding zero across the majority of splits.
* **Negative-Result Protocol:** If **M5** does not outperform the baselines under these rules, report the outcome as a **pre-specified failure-mode benchmark** without post-hoc modification of evaluation metrics or success thresholds.

---

## 5. Identifiability & Sensitivity Protocols

* **Parameter Identifiability:** Perform **profile likelihood** analysis and/or **Bayesian posterior sampling** (e.g., **PyMC** or **Stan**) to diagnose weakly identified, collinear, or unconstrained parameters.
* **Sensitivity Analyses:** Execute pre-planned robustness checks evaluating:
  * Alternative data exclusion rules
  * Alternative $\text{LET}$ definitions (e.g., dose-averaged vs. track-averaged $\text{LET}$)
  * Literature prior choices
  * Digitization error perturbations
  * Single-source (leave-one-study-out) removals

---

## 6. Data Schema & Objective Exclusion Rules

### Uniform Record Schema
Enforce a standardized schema across every record with the following required fields:

| Field | Description |
| :--- | :--- |
| `record_id` | Unique identifier for each survival observation |
| `cell_line` | Standardized cell line identifier |
| `radiation_type/quality` | Particle/photon species, energy, and $\text{LET}$ / microdosimetric descriptor |
| `dose` | Absorbed radiation dose (and dose rate / delivery metadata where available) |
| `surviving_fraction` | Measured surviving fraction $S$ |
| `source_paper/lab` | Source publication, dataset, or laboratory identifier |
| `extraction_method` | Data provenance/extraction method (e.g., raw table, digitized curve) |
| `quality_flags` | Standardized data-quality and completeness indicators |

### Pre-Defined Exclusion Logic
* Document all inclusion and exclusion rules **prior to fitting** so data filtering is never chosen or adjusted after inspecting fit residuals or benchmark rankings.
