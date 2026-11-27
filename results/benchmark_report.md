# Out-of-Sample Benchmark Report: Mechanistic Damage-Complexity & Biphasic Repair Models vs. Empirical LQ Baselines on PIDE 3.4

## 1. Dataset & Stage 1 Curation Audit
* **Source Database:** GSI Particle Irradiation Data Ensemble (`PIDE 3.4`, `PIDE3.4_IonRawData.dat`, `PIDE3.4_PhotonRawData.dat`, and `PIDEMasterfile`).
* **Total Validated Survival Points:** **9,403** `(Dose, SurvivingFraction)` observations normalized to the frozen 8-field `SurvivalRecord` schema (`data/curated/pide_survival_records.csv`):
  * **Ion Irradiation Points:** 7,205
  * **Photon Reference Points:** 2,198
* **Audit Logging:** All curves lacking raw `(Dose, SF)` pairs (`N/A`) or violating physical bounds ($D < 0$ or $SF \le 0$) were deterministically excluded and logged in `data/curated/stage1_exclusion_log.csv`.

## 2. Leakage-Free Validation Splits (Stage 2)
All grouping manifests were frozen in `data/splits/` prior to model fitting:
* **LOCLO (Leave-One-Cell-Line-Out):** 93 held-out cell-line folds ($\ge 20$ observations each; 152 unique cell lines total).
* **LOQO_ION (Leave-One-Ion-Species-Out):** 22 held-out ion species folds.
* **LOQO_LET (Leave-One-LET-Regime-Out):** 3 held-out radiation quality regimes (`Low_LET < 20 keV/µm`, `Mid_LET 20–100 keV/µm`, `High_LET > 100 keV/µm`).
* **LOSO (Leave-One-Source-Out):** 107 held-out publication/laboratory folds ($\ge 20$ observations each; 129 unique publications total).

## 3. Held-Out Predictive Performance (`M1–M5`)
All models were fitted under strict non-negativity constraints ($\alpha \ge 0, \beta \ge 0$) via `L-BFGS-B` on identical training folds.

| Validation Split | Folds | M1 (Fixed LQ) | M2 (Empirical LET) | M3 (Saturation MKM) | M4 (Static Complexity) | M5 (Complexity + Biphasic ODE) |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| **LOCLO** (Cell Line) | 93 | 2.23 | 1.85 | 2.22 | 11.60 | **1.67** |
| **LOQO_ION** (Ion Species) | 22 | 3.41 | **2.38** | 3.43 | 5.20 | 2.69 |
| **LOQO_LET** (LET Regime) | 3 | **3.59** | 536.00 | 3.67 | 13.00 | 10.70 |
| **LOSO** (Laboratory) | 107 | 2.05 | 1.75 | 2.05 | 14.70 | **1.65** |

## 4. Primary Confirmatory Comparisons (Cluster-Bootstrapped 95% CIs)
Paired differences ($\Delta \text{log-RMSE} = \text{RMSE}_{M5} - \text{RMSE}_{\text{Baseline}}$; negative values favor **M5**) estimated across 2,000 cluster bootstrap resamples:

### Comparison A: M5 vs. M2 (Empirical LET Scaling)
| Split | Mean $\Delta$ Log-RMSE | 95% Bootstrap CI | Fold Win Rate (M5 < M2) | Confirmatory Verdict |
| :--- | :---: | :---: | :---: | :--- |
| **LOCLO** | -0.181 | [-0.388, +0.090] | 79.6% (74/93) | Inconclusive (CI crosses 0) |
| **LOQO_ION** | +0.310 | [-0.311, +1.350] | 68.2% (15/22) | Inconclusive (CI crosses 0) |
| **LOQO_LET** | -525.00 | [-1574.0, -0.433] | 100.0% (3/3) | Favors M5 (prevents M2 high-LET blow-up) |
| **LOSO** | -0.100 | [-0.278, +0.125] | 68.2% (73/107) | Inconclusive (CI crosses 0) |

### Comparison B: M5 vs. M4 (Static Damage Complexity Without Repair Kinetics)
| Split | Mean $\Delta$ Log-RMSE | 95% Bootstrap CI | Fold Win Rate (M5 < M4) | Confirmatory Verdict |
| :--- | :---: | :---: | :---: | :--- |
| **LOCLO** | -9.93 | [-13.10, -7.72] | 95.7% (89/93) | **Significant improvement** |
| **LOQO_ION** | -2.51 | [-6.05, +0.654] | 68.2% (15/22) | Inconclusive (CI crosses 0) |
| **LOQO_LET** | -2.33 | [-26.10, +26.50] | 66.7% (2/3) | Inconclusive (small $N=3$) |
| **LOSO** | -13.00 | [-16.70, -10.30] | 98.1% (105/107) | **Significant improvement** |

## 5. Key Conclusions & Failure-Mode Synthesis
1. **Biphasic Repair Kinetics Are Essential for Mechanistic Models:** Adding fast/slow DSB repair kinetics (`M5`) dramatically stabilizes out-of-sample cell survival predictions compared to static damage-complexity yields (`M4`), reducing mean log-RMSE by **9.93** on `LOCLO` and **13.00** on `LOSO` with 95% CIs strictly below zero.
2. **Unbounded Empirical LET Scaling Fails High-LET Extrapolation:** While `M2` performs competitively in-domain, linear LET scaling suffers catastrophic extrapolation failure when `High_LET (>100 keV/µm)` is held out (`LOQO_LET` log-RMSE = `536.0`), whereas saturation-bounded mechanistic formulations remain finite.
3. **Mechanistic ODE Complexity Does Not Uniformly Beat Empirical LET Scaling Without Cell-Line Priors:** Despite winning 68–80% of individual `LOCLO`, `LOSO`, and `LOQO_ION` folds, `M5`'s cluster-bootstrapped 95% CIs against `M2` cross zero, demonstrating that global mechanistic ODE parameters without cell-line-specific repair-capacity priors cannot statistically dominate empirical LET scaling on mean held-out log-RMSE.
