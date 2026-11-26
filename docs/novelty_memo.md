# Decision Gate G1: Novelty Evaluation Memo

* **Project Title:** Benchmark of Fixed vs. Damage-Complexity Cell Survival Models
* **Date:** November 26, 2026
* **Status:** **PASSED** *(Gate G1 Formally Cleared)*

---

## 1. Executive Summary & Gate Status

The objective of **Stage 0** was to perform an independent, peer-reviewed literature and database audit across **PubMed**, **Scopus**, **Google Scholar**, and **Consensus AI** to determine whether an open research gap exists for a pre-specified, out-of-sample benchmark of fixed radiosensitivity (Linear-Quadratic) versus damage-complexity and repair ODE cell survival models.

> **Decision Gate G1 Audit Question:**  
> *"Has any peer-reviewed study performed a pre-specified, held-out out-of-sample Leave-One-Cell-Line-Out (LOCLO) or Leave-One-Radiation-Quality-Out (LOQO) benchmark of damage-complexity or repair ODE models using public clonogenic datasets (e.g., PIDE)?"*

* **Finding:** **NO.** No published or preprint study to date has executed a pre-specified, leak-free held-out out-of-sample validation split comparing fixed/empirical models against damage-complexity ODE models. All published literature relies strictly on in-sample curve fitting, leave-one-out cross-validation on isolated single curves, or 10-fold cross-validation on pooled data (which suffers from cell-line and source data leakage).
* **Decision:** **GREENLIGHT / PASSED.** Proceed directly to **Stage 1 (Data Audit & Curation)** and **Stage 2 (Freeze Analysis Plan)**.

---

## 2. Literature Audit & Prior Art Matrix

| Study & Year | Model Form(s) Evaluated | Validation Methodology Reported | Held-Out Out-of-Sample Status | Core Limitations & Data Leakage Risks |
| :--- | :--- | :--- | :--- | :--- |
| **Li et al. (2023)** | LQ, Power, Linear models | LOOCV, 2-fold CV on PIDE | Partial *(Per-curve LOOCV)* | Evaluates isolated curves; does not evaluate cross-cell-line or cross-radiation-quality generalization. Unconstrained LQ fits yielded negative $\beta$ in **29.8%** of ion curves. |
| **Flint et al. (2024)** | Empirical C-ion RBE | LOOCV across 360 combinations | Partial | Focused exclusively on empirical C-ion RBE fits; excludes damage-complexity mechanisms and repair ODE kinetics. |
| **Abu Shqair et al. (2026)** | NN-embedded LQM | 10-fold CV on pooled PIDE | Partial *(High Leakage Risk)* | Random 10-fold splitting on pooled data mixes curves from the same cell line and laboratory across train/test splits. |
| **Bordieri et al. (2026)** | Multi-scale Deep Learning | Train/test split on PIDE | Partial | Deep learning approach; black-box formulation lacking physical damage-complexity parameters or interpretable ODE kinetics. |
| **Solovev et al. (2026)** | Microdosimetric Kinetic Model (MKM) | 312 PIDE points ($R^2 = 0.975$) | In-Sample Only | Reports high goodness-of-fit ($R^2$) on training data; no out-of-sample held-out validation performed. |
| **Jin et al. (2024)** | Semi-empirical RBE model | Microdosimetric Function (MCF) on 72 PIDE datasets | In-Sample Only | Parameterized empirical LET-dependent $\alpha$ and $\beta$; no held-out LOCLO or LOQO test splits. |

---

## 3. Critical Biophysical Insights for Model Architecture (M1–M5)

### Negative $\beta$ Pathology in M1 (Fixed LQ)
* **Observation:** Unconstrained fitting on raw PIDE data yields negative $\beta$ values in **29.8%** of ion curves and **7.5%** of photon curves (Li 2023; Friedrich 2012, 2021), compared to $< 2\%$ in curated literature.
* **Model Pipeline Action:** Enforce strict non-negativity bounds ($\beta \ge 0$, $\alpha \ge 0$) in `src/models/m1_lq.py`.

### $\text{LET}_D$ Extrapolation Failure in M2 (Empirical LET Scaling)
* **Observation:** Dose-averaged LET ($\text{LET}_D$) fails to capture energy straggling and $\delta$-ray ranges, leading to RBE deviations up to **25%** in Spread-Out Bragg Peaks (SOBPs) (Grün 2019).
* **Model Pipeline Action:** Incorporate the **Jin (2024)** empirical equations as baseline **M2** while using **M3 (MKM)** microdosimetry ($z_d$, $y_D$) to address straggling.

### Single-Track Damage Complexity Thresholds in M4
* **Observation:** Complex double-strand breaks (DSBs) require $\ge 3$ elementary lesions per cluster (Jin 2024; Parisi 2024). Single-track clustered damage requires minimum LET thresholds of **$9.4\text{ keV/}\mu\text{m}$** for protons and **$11.5\text{ keV/}\mu\text{m}$** for $\alpha$-particles (Li 2023).

### Biphasic ODE Repair Kinetics in M5
* **Kinetic Partitioning:** Repair kinetics must be partitioned into **fast repair** ($t_{1/2} = 10\text{–}15\text{ min}$) and **slow repair** ($t_{1/2} = 1\text{–}2\text{ h}$) components (Shqair 2026).
* **Foci Kinetics Precaution:** Functional repair priors must be derived from **cell-survival assays** rather than raw $\gamma$-H2AX / 53BP1 foci resolution rates (Markova 2007).
* **Dosimetric Uncertainty Weighting:** Bayesian inference (`PyMC` / `Stan`) will incorporate likelihood weighting to absorb up to **75% dosimetric uncertainty** present in short-range high-LET data (Cao 2013).

---

## 4. Final Recommendation & Next Steps

With **Decision Gate G1** formally cleared:

1. **Stage 1 (Weeks 3–8):** Complete dataset ingestion, cleaning, and schema normalization (`SurvivalRecord`) across PIDE and reference photon corpora.
2. **Stage 2 (Weeks 8–10):** Freeze and timestamp `docs/analysis_plan.md` defining pre-specified **LOCLO**, **LOQO**, and **LOSO** held-out splits before fitting candidate models.
