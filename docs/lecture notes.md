# **Cell Survival Models** for Photon and Heavy-Ion Benchmarking

Six interlocking topics define a defensible benchmark: the LQ baseline and its parameter drift, microdosimetric RBE scaling, DSB complexity partitioning, repair-misrepair ODEs, public datasets, and leakage-free validation. Each is summarized below with its governing equations and typical parameter bounds.

## LQ Baseline and Parameter Drift

The standard model is S = exp(-α D - β D²), with α (Gy⁻¹) and β (Gy⁻²) representing single-track and two-track lethal events (Van Leeuwen et al., 2018; McMahon, 2018; Dalalas et al., 2024). Its assumptions are Poisson-distributed lethal lesions and linear dose production of biologically important lesions below ~20 Gy (Curtis, 1988; Loan & Bhat, 2022). Kinetic derivations show almost every repair/misrepair model collapses to LQ at low-to-intermediate dose, with two-track (β) lethality largely from dicentric aberrations and one-track (α) lethality from other mechanisms (Sachs et al., 1997).

Parameters describe one specific condition only; full characterization would require dependence on phenotype, cell cycle, microenvironment, radiation type, and delivery (McMahon, 2018). Empirically, α rises with LET while β shows no systematic change (Andō & Goodhead, 2016; Zanni et al., 2024), and α and β are anti-correlated during photon fits, so different LQ fits yield different RBE predictions (Van Leeuwen et al., 2018; Zanni et al., 2024). At high dose the LQ overestimates effect (Qin et al., 2026), and at low dose it underestimates survival because lesions are over-dispersed rather than Poisson (Loan & Bhat, 2022). A quasi-mechanistic alternative redefines α as critical damages per Gy, β as repair probability, and m as the no-repair threshold, fitting 431 datasets best at m = 6 (Qin et al., 2026).

| Parameter | Typical Range | Notes |
|---|---|---|
| α (Gy⁻¹) | 0.05–0.5 (photons); rises with LET | Most LET-sensitive term (Andō & Goodhead, 2016) |
| β (Gy⁻²) | 0.01–0.08; ~constant or falling at high LET | Anti-correlated with α (Zanni et al., 2024) |
| α/β (Gy) | 1.5–10 (tissue-dependent) | High heterogeneity, I² > 75% (Van Leeuwen et al., 2018) |
| m (threshold) | 5–7 optimal | Quasi-mechanistic model (Qin et al., 2026) |

**Figure 1:** Typical LQ parameter bounds for Python implementation

## MKM and Microdosimetric RBE Scaling

The MKM splits the nucleus into domains (~0.2–0.4 µm) where type I (lethal) and type II (potentially lethal) lesions form proportionally to specific energy z (Dalalas et al., 2024). The linear term becomes α = α₀ + z_Dβ₀ with β = β₀, where z_D is dose-mean specific energy (Dalalas et al., 2024). Sublethal lesions follow four fates: monomolecular conversion to lethal, pairwise combination, repair, or persistence-then-fixation (Dalalas et al., 2024; Curtis, 1988). Domain radius controls LET dependence of α; nucleus radius controls overkill (Abolfath et al., 2020).

Microdosimetric quantities are preferred over LET because they capture energy-loss straggling and finite δ-ray range (Dalalas et al., 2024). Empirical models parameterize RBE via dose-averaged LET, while biophysical models use dose-averaged lineal energy y_D (Dalalas et al., 2024; Qi et al., 2026). At 2 Gy, proton RBE ranges 1.0–1.2 proximally but reaches 1.3–1.5 distally for low α/β tissues (Dalalas et al., 2024). Empirical models are limited by their training cell lines (e.g., V79-only), whereas McNamara and Tian models span broader datasets (Dalalas et al., 2024). GPU dose engines like FRoG implement both LEM and MKM for p, ⁴He, ¹²C, and ¹⁶O (Stewart, 2020).

## DSB Complexity and Yields

DSB yields rise with LET (Zanni et al., 2024; Kundrát & Stewart, 2006), with prompt DSB yields of 70–120 Gy⁻¹ cell⁻¹ for protons and enzymatic DSBs rising from ~40 to 70 Gy⁻¹ cell⁻¹ (Kundrát & Stewart, 2006). Apparent DSB rates of 5–16 DSBs/cell/Gy decrease with LET as measured by foci (Nikitaki et al., 2016). Lethal events correlate with DSBs containing three or more elementary lesions, not with other damage classes (Kundrát & Stewart, 2006). Minimum LETs of 9.4 and 11.5 keV/µm are needed for single-track clustered damage by protons and alphas (Rezaee & Adhikary, 2024). Qi et al. modeled survival as a third-order polynomial of DSB complexity, achieving RMSE 0.218 (H460) and 0.170 (H1437) across 19 conditions (Qi et al., 2026). Clustered DSBs resist canonical NHEJ due to short DNA fragments, shifting repair toward HR and alternative NHEJ (Nickoloff et al., 2020). Local DSB density (cluster density) linearly predicts misrepair regardless of ion species, with 7.3% residual DSBs at 24 h (Henthorn et al., 2018).

## Repair-Misrepair ODEs and Lethality Mapping

Curtis's LPL model posits lethal (irreparable) and potentially lethal (repairable) lesions, with correct repair (first-order), binary misrepair (second-order), and linear fixation competing (Curtis, 1986). Two repair subclasses exist: fast (t₁/₂ 10–15 min) and slow (t₁/₂ 1–2 h) (Curtis, 1988). The governing ODEs are:

dN_PL/dt = η_PLḊ - ε_PLN_PL - ε₂ N_PL² - κ N_PL

dN_L/dt = η_LḊ + κ N_PL + ε₂ N_PL²

where η are production rates, ε_PL repair, ε₂ binary misrepair, and κ fixation (Curtis, 1988). The RMR model shares the binary misrepair assumption (Curtis, 1988). Lethality maps via S = exp(-N_L) at mitosis, with RaDRI showing near-LQ dose dependence arising almost exclusively from DSB misjoining (Loan & Bhat, 2022). The Medras model uses 11 shared mechanistic parameters, predicting survival, aberrations, and mutation with R² > 0.9 for mean inactivation dose (McMahon et al., 2016). A Feller square-root SDE framework links single-cell γ-H2AX kinetics to LQ via Feynman-Kac (Kara & Tuğrul, 2026).

## Benchmarks and Validation

PIDE contains >1,100 photon–ion curve pairs with raw data and uniformly re-derived LQ parameters (Loan & Bhat, 2022). Its caveats include missing SOBP LET metadata and large inter-experiment variability (Zanni et al., 2024; Sachs et al., 1997). RadPhysBio adds ML-predicted α with LET as top feature, though β remains poorly predicted (Zanni et al., 2024). Individual cell lines show ~20% survival variability under identical setup (McMahon, 2018). For validation, LOCLO/LOQO/LOSO splits must hold out entire cell lines, radiation qualities, or source laboratories respectively; re-optimizing the DSB rejoining range to 0.0335 µm lowered RMSD by >70% (Nikitaki et al., 2016). Cross-endpoint constraints (e.g., using repair kinetics to inform dose-rate effects) reduce overfitting risk (Rezaee & Adhikary, 2024).

## Conclusion

Fixed-radiosensitivity LQ models are parsimonious but condition-specific, with α drifting systematically with LET while β remains comparatively stable. Damage-complexity models (MKM, Medras, Qi polynomial) capture the mechanistic basis of RBE but require careful parameterization and leakage-free validation against PIDE and RadPhysBio, with log-RMSE reporting and LOCLO/LOQO/LOSO splits as the appropriate standards.
 
_These search results were found and analyzed using Consensus, an AI-powered search engine for research. Try it at https://consensus.app. © 2026 Consensus NLP, Inc. Personal, non-commercial use only; redistribution requires copyright holders’ consent._
 
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
 
