# Cell Survival Models for Photon and Heavy-Ion Benchmarking

> **Core Scope:** Six interlocking topics define a defensible benchmark: the LQ baseline and its parameter drift, microdosimetric RBE scaling, DSB complexity partitioning, repair-misrepair ODEs, public datasets, and leakage-free validation.

---

## 1. LQ Baseline and Parameter Drift

The standard Linear-Quadratic (LQ) model is defined as:

$$
S = \exp(-\alpha D - \beta D^2)
$$

where $\alpha\text{ (Gy}^{-1}\text{)}$ and $\beta\text{ (Gy}^{-2}\text{)}$ represent single-track and two-track lethal events (Van Leeuwen et al., 2018; McMahon, 2018; Dalalas et al., 2024).

* **Core Assumptions:** Poisson-distributed lethal lesions and linear dose production of biologically important lesions below $\sim 20\text{ Gy}$ (Curtis, 1988; Loan & Bhat, 2022). Kinetic derivations show almost every repair/misrepair model collapses to LQ at low-to-intermediate dose, with two-track ($\beta$) lethality largely from dicentric aberrations and one-track ($\alpha$) lethality from other mechanisms (Sachs et al., 1997).
* **Condition Specificity & Drift:** Parameters describe one specific condition only; full characterization depends on phenotype, cell cycle, microenvironment, radiation type, and delivery (McMahon, 2018). Empirically, $\alpha$ rises with LET while $\beta$ shows no systematic change (Andō & Goodhead, 2016; Zanni et al., 2024). Because $\alpha$ and $\beta$ are anti-correlated during photon fits, different LQ fits yield different RBE predictions (Van Leeuwen et al., 2018; Zanni et al., 2024).
* **Dose Extremes:** At high dose, LQ overestimates biological effect (Qin et al., 2026); at low dose, it underestimates survival because lesions are over-dispersed rather than Poisson-distributed (Loan & Bhat, 2022).
* **Quasi-Mechanistic Alternative:** Redefines $\alpha$ as critical damages per Gy, $\beta$ as repair probability, and $m$ as the no-repair threshold, fitting 431 datasets best at $m = 6$ (Qin et al., 2026).

| Parameter | Typical Range | Notes |
| :--- | :--- | :--- |
| $\alpha\text{ (Gy}^{-1}\text{)}$ | $0.05\text{–}0.5$ (photons); rises with LET | Most LET-sensitive term (Andō & Goodhead, 2016) |
| $\beta\text{ (Gy}^{-2}\text{)}$ | $0.01\text{–}0.08$; $\sim$constant or falling at high LET | Anti-correlated with $\alpha$ (Zanni et al., 2024) |
| $\alpha/\beta\text{ (Gy)}$ | $1.5\text{–}10$ (tissue-dependent) | High heterogeneity, $I^2 > 75\%$ (Van Leeuwen et al., 2018) |
| $m$ (threshold) | $5\text{–}7$ optimal | Quasi-mechanistic model (Qin et al., 2026) |

*Figure 1: Typical LQ parameter bounds for Python implementation.*

---

## 2. MKM and Microdosimetric RBE Scaling

The Microdosimetric Kinetic Model (MKM) splits the nucleus into domains ($\sim 0.2\text{–}0.4\text{ }\mu\text{m}$) where type I (lethal) and type II (potentially lethal) lesions form proportionally to specific energy $z$ (Dalalas et al., 2024):

$$
\alpha = \alpha_0 + z_D \beta_0, \qquad \beta = \beta_0
$$

where $z_D$ is the dose-mean specific energy (Dalalas et al., 2024).

* **Lesion Fates & Geometry:** Sublethal lesions follow four fates: monomolecular conversion to lethal, pairwise combination, repair, or persistence-then-fixation (Dalalas et al., 2024; Curtis, 1988). Domain radius controls the LET dependence of $\alpha$, while nucleus radius controls overkill (Abolfath et al., 2020).
* **Microdosimetry vs. LET:** Microdosimetric quantities are preferred over LET because they capture energy-loss straggling and finite $\delta$-ray range (Dalalas et al., 2024). Empirical models parameterize RBE via dose-averaged LET, whereas biophysical models use dose-averaged lineal energy $y_D$ (Dalalas et al., 2024; Qi et al., 2026).
* **Proton RBE & Engines:** At $2\text{ Gy}$, proton RBE ranges from $1.0\text{–}1.2$ proximally to $1.3\text{–}1.5$ distally for low $\alpha/\beta$ tissues (Dalalas et al., 2024). Empirical models are limited by training cell lines (e.g., V79-only), whereas McNamara and Tian models span broader datasets (Dalalas et al., 2024). GPU dose engines like **FRoG** implement both LEM and MKM for $p$, $^4\text{He}$, $^{12}\text{C}$, and $^{16}\text{O}$ (Stewart, 2020).

---

## 3. DSB Complexity and Yields

* **Yields vs. Foci:** DSB yields rise with LET (Zanni et al., 2024; Kundrát & Stewart, 2006), with prompt DSB yields of $70\text{–}120\text{ Gy}^{-1}\text{ cell}^{-1}$ for protons and enzymatic DSBs rising from $\sim 40$ to $70\text{ Gy}^{-1}\text{ cell}^{-1}$ (Kundrát & Stewart, 2006). Conversely, apparent DSB rates of $5\text{–}16\text{ DSBs/cell/Gy}$ decrease with LET when measured via foci (Nikitaki et al., 2016).
* **Lethality & LET Thresholds:** Lethal events correlate specifically with DSBs containing $\ge 3$ elementary lesions (Kundrát & Stewart, 2006). Minimum LETs of $9.4$ and $11.5\text{ keV/}\mu\text{m}$ are required for single-track clustered damage by protons and $\alpha$-particles, respectively (Rezaee & Adhikary, 2024).
* **Complexity Modeling & Repair Shift:** Qi et al. modeled survival as a third-order polynomial of DSB complexity, achieving $\text{RMSE} = 0.218$ (H460) and $0.170$ (H1437) across 19 conditions (Qi et al., 2026). Clustered DSBs resist canonical NHEJ due to short DNA fragments, shifting repair toward HR and alternative NHEJ (Nickoloff et al., 2020). Local DSB cluster density linearly predicts misrepair regardless of ion species, leaving $7.3\%$ residual DSBs at $24\text{ h}$ (Henthorn et al., 2018).

---

## 4. Repair-Misrepair ODEs and Lethality Mapping

Curtis's Lethal and Potentially Lethal (LPL) model posits lethal (irreparable) and potentially lethal (repairable) lesions, with first-order correct repair, second-order binary misrepair, and linear fixation competing across two repair subclasses: **fast** ($t_{1/2} \approx 10\text{–}15\text{ min}$) and **slow** ($t_{1/2} \approx 1\text{–}2\text{ h}$) (Curtis, 1986; Curtis, 1988).

The governing ODEs are:

$$
\frac{dN_{PL}}{dt} = \eta_{PL}\dot{D} - \varepsilon_{PL}N_{PL} - \varepsilon_2 N_{PL}^2 - \kappa N_{PL}
$$

$$
\frac{dN_L}{dt} = \eta_L\dot{D} + \kappa N_{PL} + \varepsilon_2 N_{PL}^2
$$

where $\eta$ denotes production rates, $\varepsilon_{PL}$ linear repair, $\varepsilon_2$ binary misrepair, and $\kappa$ fixation (Curtis, 1988).

* **RMR & Lethality Mapping:** The RMR model shares the binary misrepair assumption (Curtis, 1988). Lethality maps via $S = \exp(-N_L)$ at mitosis, with RaDRI showing near-LQ dose dependence arising almost exclusively from DSB misjoining (Loan & Bhat, 2022).
* **Mechanistic & Stochastic Extensions:** The **Medras** model uses 11 shared mechanistic parameters to predict survival, aberrations, and mutation with $R^2 > 0.9$ for mean inactivation dose (McMahon et al., 2016). A Feller square-root SDE framework links single-cell $\gamma$-H2AX kinetics to LQ via Feynman-Kac (Kara & Tuğrul, 2026).

---

## 5. Benchmarks and Validation

* **Public Datasets:** **PIDE** contains $>1{,}100$ photon–ion curve pairs with raw data and uniformly re-derived LQ parameters (Loan & Bhat, 2022), though it lacks SOBP LET metadata and exhibits large inter-experiment variability (Zanni et al., 2024; Sachs et al., 1997). **RadPhysBio** adds ML-predicted $\alpha$ with LET as the top feature, though $\beta$ remains poorly predicted (Zanni et al., 2024).
* **Leakage-Free Splits:** Individual cell lines show $\sim 20\%$ survival variability under identical experimental setups (McMahon, 2018). Validation must use **LOCLO / LOQO / LOSO** splits to hold out entire cell lines, radiation qualities, or source laboratories, respectively. Re-optimizing the DSB rejoining range to $0.0335\text{ }\mu\text{m}$ lowered RMSD by $>70\%$ (Nikitaki et al., 2016), and cross-endpoint constraints (using repair kinetics to inform dose-rate effects) reduce overfitting risk (Rezaee & Adhikary, 2024).

---

## 6. Conclusion

> Fixed-radiosensitivity LQ models are parsimonious but condition-specific, with $\alpha$ drifting systematically with LET while $\beta$ remains comparatively stable. Damage-complexity models (**MKM**, **Medras**, **Qi polynomial**) capture the mechanistic basis of RBE but require careful parameterization and leakage-free validation against **PIDE** and **RadPhysBio**, using **log-RMSE** reporting and **LOCLO/LOQO/LOSO** splits as the standard.
 
## References
 
Abolfath, R., Helo, Y., Carlson, D. J., Stewart, R., Grosshans, D., & Mohan, R. (2020). A new approach to modeling the microdosimetry of proton therapy beams. *Medical Physics, 47*, 3184-3190. https://doi.org/10.1002/mp.14165
 
Andō, K., & Goodhead, D. T. (2016). Dependence and independence of survival parameters on linear energy transfer in cells and tissues. *Journal of Radiation Research, 57*, 596-606. https://doi.org/10.1093/jrr/rrw058
 
Curtis, S. B. (1988). The Lethal and Potentially Lethal Model — A Review and Recent Development. *Springer eBooks*, 137-146. https://doi.org/10.1007/978-3-642-46656-4_13
 
Curtis, S. B. (1986). Lethal and Potentially Lethal Lesions Induced by Radiation --- A Unified Repair Model. *Radiation Research, 106*, 252-252. https://doi.org/10.2307/3576798
 
Dalalas, D., Papadopoulos, A., Kyriakou, I., Stewart, R. D., Karaiskos, P., & Emfietzoglou, D. (2024). Relative Biological Effectiveness (RBE) of Monoenergetic Protons: Comparison of Empirical and Biophysical Models. *Applied Sciences, 14*, 11981-11981. https://doi.org/10.3390/app142411981
 
Henthorn, N. T., Warmenhoven, J.-W., Sotiropoulos, M., Mackay, R. I., Kirkby, N. F., Kirkby, K. J., & Merchant, M. J. (2018). In Silico Non-Homologous End Joining Following Ion Induced DNA Double Strand Breaks Predicts That Repair Fidelity Depends on Break Density. *Scientific Reports, 8*, 2654-2654. https://doi.org/10.1038/s41598-018-21111-8
 
Kara, M. T., & Tuğrul, M. (2026). Stochastic Biophysics of Cellular Radiosensitivity: From Molecular Noise and Repair Kinetics to Evolutionary Demographics. *bioRxiv*. https://doi.org/10.64898/2026.08.30.748070
 
Kundrát, P., & Stewart, R. D. (2006). On the biophysical interpretation of lethal DNA lesions induced by ionising radiation. *Radiation Protection Dosimetry, 122*, 169-172. https://doi.org/10.1093/rpd/ncl439
 
Loan, M., & Bhat, A. (2022). Effect of overdispersion of lethal lesions on cell survival curves. *Biomedical Physics & Engineering Express, 8*, 045011-045011. https://doi.org/10.1088/2057-1976/ac7667
 
McMahon, S. J. (2018). The linear quadratic model: usage, interpretation and challenges. *Physics in Medicine and Biology, 64*, 01TR01-01TR01. https://doi.org/10.1088/1361-6560/aaf26a
 
McMahon, S. J., Schuemann, J., Paganetti, H., & Prise, K. M. (2016). Mechanistic Modelling of DNA Repair and Cellular Survival Following Radiation-Induced DNA Damage. *Scientific Reports, 6*, 33290-33290. https://doi.org/10.1038/srep33290
 
Nickoloff, J. A., Sharma, N., & Taylor, L. E. (2020). Clustered DNA Double-Strand Breaks: Biological Effects and Relevance to Cancer Radiotherapy. *Genes, 11*, 99-99. https://doi.org/10.3390/genes11010099
 
Nikitaki, Z., Nikolov, V., Mavragani, I. V., Mladenov, E., Mangelis, A., Laskaratou, D., Fragkoulis, G. I., Hellweg, C. E., Martin, O. A., Emfietzoglou, D., Hatzi, V. I., Terzoudi, G. I., Iliakis, G. E., & Georgakilas, A. G. (2016). Measurement of complex DNA damage induction and repair in human cellular systems after exposure to ionizing radiations of varying linear energy transfer (LET). *Free Radical Research, 50*, S64-S78. https://doi.org/10.1080/10715762.2016.1232484
 
Qi, M., Lai, Y., Guan, F., Xing, M., Bronk, L., Mohan, R., Li, H., Chi, Y., & Jia, X. (2026). DNA damage complexity as a predictor of cell survival: a microscopic Monte Carlo-based modeling framework for photon, proton and carbon ion irradiation. *Physics in Medicine and Biology, 71*, 125026-125026. https://doi.org/10.1088/1361-6560/ae7baa
 
Qin, X., Li, Y., Yuan, J., You, P., Kong, F., & Jin, J. (2026). A quasi-mechanistic cell survival model and its comparison to the linear-quadratic model in radiobiology. *International Conference on Biomedicine and Bioinformatics Engineering (ICBBE)*, 474-481. https://doi.org/10.1145/3822183.3822257
 
Rezaee, M., & Adhikary, A. (2024). The Effects of Particle LET and Fluence on the Complexity and Frequency of Clustered DNA Damage. *DNA, 4*, 34-51. https://doi.org/10.3390/dna4010002
 
Sachs, H. A. B. P. D. J. R. K., Hahnfeld, P., & Brenner, D. J. (1997). Review The link between low-LET dose-response relations and the underlying kinetics of damage production/repair/misrepair. *International Journal of Radiation Biology, 72*, 351-374. https://doi.org/10.1080/095530097143149
 
Stewart, M. (2020). FRoG: a fast robust analytical dose engine on GPU for p, 4He, 12C and 16O particle therapy. *heiDOK (Heidelberg University)*. https://doi.org/10.11588/heidok.00027888
 
Van Leeuwen, C. M., Oei, A. L., Crezee, J., Bel, A., Franken, N. A. P., Stalpers, L. J. A., & Kok, H. P. (2018). The alfa and beta of tumours: a review of parameters of the linear-quadratic model, derived from clinical radiotherapy studies. *Radiation Oncology, 13*, 96-96. https://doi.org/10.1186/s13014-018-1040-z
 
Zanni, V., Papakonstantinou, D., Kalospyros, S. A., Karaoulanis, D. E., Biz, G. M., Manti, L., Adamopoulos, A. V., Pavlopoulou, A., & Georgakilas, A. G. (2024). RadPhysBio: A Radiobiological Database for the Prediction of Cell Survival upon Exposure to Ionizing Radiation. *International Journal of Molecular Sciences, 25*, 4729-4729. https://doi.org/10.3390/ijms25094729
 
