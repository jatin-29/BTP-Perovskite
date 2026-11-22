# Figure notes

Each entry gives what is plotted, what to look for, and one neutral observation. Tables behind the figures are in
`tables/` under the same base name. Colours keep one meaning across all figures: B elements (Ge orange, Sn sky blue,
Pb green, Ca vermillion, Sr pink, Ba blue), strict groups (Group 14 indigo, mixed B grey, Group 2 wine), A type
(inorganic black circle, organic sand triangle, mixed teal square), levels of theory, models and CV types each have
their own colours.

**fig01_gap_hist.png**
- Histograms of the band gap at each of the four levels of theory, stacked by strict group (from g2_frac).
- Look for: the separation of Group 14 and Group 2 and the different coverage of each level (n in each title).
- Observation: at every level, strict Group 2 compounds occupy the high-gap end (HSE-PBE-SOC mean 5.10 eV vs 1.60 eV for strict Group 14).

**fig02_pbe_vs_hybrid_panel.png**
- (a) PBE versus HSE-PBE-SOC gap, (b) PBE versus HSErel_SOC gap, on the same compounds. Single-B compounds coloured by
  B element, mixed-B compounds grey, black rings = flagged outliers, dashed line y = x.
- Look for: Pb points below the diagonal in (a).
- Observation: all 22 Pb compounds have HSE-PBE-SOC below PBE (mean -0.31 eV); every other single-B compound lies above.

**fig03_shifts_by_B.png**
- (a) Hybrid shift (HSE-PBE-SOC minus PBE) and (b) SOC shift (HSErel_SOC minus HSErel) for each single B element and for mixed-B compounds (box plus points, n above).
- Look for: the Pb box in both panels.
- Observation: the mean SOC shift is -0.84 eV for Pb (single-B, n=25) and between -0.08 and -0.20 eV for the other single B elements.

**fig04_soc_vs_ZB.png**
- SOC shift of pure compounds against the atomic number of B, (a) Group 14 and (b) Group 2, with the mean per element (black line). Rings mark equal_flag rows.
- Look for: the rise in size of the shift from Ge to Pb in (a), and the lack of a trend in (b).
- Observation: Group 14 means are -0.10 (Ge), -0.24 (Sn), -0.96 eV (Pb); Group 2 means are -0.11, -0.10, -0.11 eV.

**fig05_soc_grid_panel.png**
- (a, b) Mean SOC shift (std, n) of pure compounds for each B element and halide; (c) the shift for each Group 14 B-X pair and each A cation.
- Look for: in (c), the lighter (smaller) values in the MA and FA columns.
- Observation: for PbI3 the shift is -1.16 eV with Cs, K or Rb and -0.67 (MA) and -0.64 eV (FA).

**fig06_halide_trend.png**
- Mean gap of pure compounds against halide for each B element, PBE (dashed) and HSE-PBE-SOC (solid). Both levels use the SAME compounds in each cell; error bars = std over A; n per cell under each panel.
- Look for: the downward slope from Cl to I and the change of order of Ge and Pb between the two levels.
- Observation: the gap falls from Cl to I for every B element at both levels; the Group 14 order is Pb > Ge > Sn at PBE and Ge > Pb > Sn at HSE-PBE-SOC.

**fig07_gap_vs_geometry.png**
- HSE-PBE-SOC gap against tolerance factor, octahedral factor, weighted halide radius and Bartel tau (recomputed), coloured by strict group.
- Look for: different trends inside each colour versus the pooled cloud.
- Observation: the tolerance factor has Pearson r = 0.78 inside strict Group 14 and r = 0.01 over all 244 compounds.

**fig08_corr_heatmap.png**
- Pearson correlation of each descriptor with the gap at each level, (a) all compounds, (b) strict Group 14 (g2_frac is constant there, so blank). n under each column.
- Look for: which descriptors change sign or size between (a) and (b).
- Observation: EN_B has r = -0.87 with the HSE-PBE-SOC gap over all compounds and r = 0.04 inside strict Group 14.

**fig09_variance_explained.png**
- Fraction of gap variance explained by one label at a time (A, X, B element, B group) and the in-sample R2 of an additive A+X+B model, for PBE and HSE-PBE-SOC, in (a) all compounds, (b) strict Group 14, (c) strict Group 2.
- Look for: how the largest bar changes between the three subsets. One-way fractions overlap and do not add up.
- Observation: for HSE-PBE-SOC, the largest one-way fraction is B group (0.73) over all compounds, A site (0.49) in strict Group 14 and X site (0.81) in strict Group 2.

**fig10_composition_parity.png**
- Grouped out-of-fold predictions against true HSE-PBE-SOC gaps in strict Group 14 for (a) the linear fraction model and (b) products Ridge with nested alpha.
- Look for: systematic offsets by A type.
- Observation: grouped R2 = 0.74 (MAE 0.29 eV) for (a) and 0.86 (MAE 0.21 eV) for (b), n = 76.

**fig11_coefficients.png**
- Linear coefficients (eV per full substitution, relative to CsPbCl3) with 95 percent row-bootstrap and group-bootstrap intervals.
- Look for: intervals that cross zero.
- Observation: the K and Rb intervals include zero under both bootstraps; the Ge interval includes zero only under the group bootstrap.

**fig12_random_vs_grouped.png**
- Out-of-fold R2 with random 5-fold and grouped 5-fold CV for the mean baseline, the linear model and products Ridge at four fixed alphas, plus nested alpha (grouped only), strict Group 14.
- Look for: the gap between the two bars for each model. The headline for the products model is the nested bar; the fixed-alpha bars are for comparison only.
- Observation: at alpha 0.1 the random R2 is 0.92 and the grouped R2 is 0.79; the nested grouped R2 is 0.86; for the linear model both CV types give 0.74.

**fig13_deviation_residuals.png**
- Residual of the HSE-PBE-SOC gap after a linear fit on B and X fractions, against each deviation-of-cubicity column, coloured by A type (strict Group 14, n = 75).
- Look for: points at deviation near 0.
- Observation: at deviation near 0, inorganic-A compounds have negative residuals (mean -0.47 eV, dev_b < 0.01, n=37) and organic-A compounds positive ones (mean +0.31 eV, n=6).

**fig14_organic_vs_deviation.png**
- (a) Random and grouped R2 of linear models with different feature sets (B+X, + organic fraction, + deviation, + both, all fractions, all fractions + deviation); (b) organic fraction against deviation of the b axis.
- Look for: how close the "+ organic" and "+ deviation" bars are, and the clustering in (b).
- Observation: grouped R2 is 0.18 (B+X), 0.77 (+ organic), 0.79 (+ deviation) and 0.82 (+ both), n = 75.

**fig15_clean_leave_one_B_out.png**
- Test MAE for each held-out B element (clean split) and the average, for three models and three feature sets. Test/train sizes under the figure.
- Look for: which feature set brings MAE well below the mean baseline (grey).
- Observation: average MAE is 2.06 eV for the baseline, 1.85 (RF) for geometry only and 0.45 (RF) for geometry + electronic.

**fig16_ablation.png**
- Average clean leave-one-B-out MAE when single descriptors are added to the geometry features, for RF and Ridge; dashed line = mean baseline.
- Look for: the "+ g14_flag only" bars compared with "+ all electronic".
- Observation: adding only the 0/1 group label gives 0.36 (RF) and 0.34 eV (Ridge), lower than all electronic descriptors (0.45 and 0.70 eV).

**fig17_leave_one_group_out.png**
- (a, b) Test MAE (mean signed error in brackets) for training on one strict group and testing on the other; (c, d) parity plots for geometry + electronic.
- Look for: the sign of the errors and the position of the points relative to the diagonal.
- Observation: every prediction is too low for Group 2 test compounds and too high for Group 14 test compounds; MAEs are 2.3 to 5.0 eV.

**fig18_cross_fidelity_offset_panel.png**
- (a) Average clean leave-one-B-out MAE for predicting HSE-PBE-SOC, HSErel_SOC and HSErel from the PBE gap and descriptors; (b) organic minus Cs gap for pure Pb/Sn iodides and bromides at each level (n/a = missing gap).
- Look for: in (b), the size of the HSErel and HSErel_SOC bars compared with PBE and HSE-PBE-SOC.
- Observation: Ridge on PBE + descriptors reaches 0.17 eV for HSE-PBE-SOC; the offsets range 0.20 to 0.71 (PBE), 0.31 to 0.89 (HSE-PBE-SOC) and 0.96 to 2.31 eV (HSErel_SOC).

**fig19_vs_experiment.png** (DRAFT v1, unverified experimental values)
- Parity of dataset gaps (PBE, HSErel_SOC, HSE-PBE-SOC) against the approximate DRAFT values of CLAUDE.md section 9 item 8, Cs versus organic A, and the error against halide.
- Look for: nothing final. Superseded in practice by the candidate version below; both must be redone with verified values.
- Observation: with these DRAFT values, the HSE-PBE-SOC mean error is -0.73 eV for the 3 Cs compounds and -0.34 eV for the 8 organic compounds.

**fig19_vs_experiment_candidates.png** (DRAFT v2, candidate experimental values, not verified)
- (a-d) Parity of the dataset gap at each of the four levels against one candidate literature value per compound (colour = Cs or organic A; marker = phase class of the measurement: cubic, non-cubic, unknown); (e) mean signed error by halide; (f) mean signed error by phase class.
- Look for: the black (Cs) points below the diagonal in (a), (c) and (d), and the small n in each phase class.
- Observation: at HSE-PBE-SOC the mean signed error is -0.93 eV for 7 Cs compounds and -0.27 eV for 9 organic compounds; MAE over 16 compounds is 0.56 eV.

**fig20_bowing.png**
- (a-c) PBE gap against halide fraction for mixed-halide compounds, with straight lines between the pure end members; (d-i) deviation from that line at PBE and HSE-PBE-SOC. Black rings = flagged outliers.
- Look for: the sign of the deviation for Group 2 (Ca, Sr, Ba) compared with Group 14.
- Observation: at PBE the mean deviation is negative for strict Group 2 in all three pairs (-0.18 to -0.32 eV) and +0.03 to +0.14 eV for strict Group 14. No I-Cl compound has both end members at HSE-PBE-SOC.

**fig21_robustness.png**
- (a) Average clean leave-one-B-out MAE of RandomForest over 10 seeds for geometry only, + g14_flag only and + all electronic; (b) per-element MAE with 95% ranges from 1000 bootstrap resamples of the test rows (RandomForest); (c) the same test for PBE and HSErel_SOC targets; (d) with and without the 41 equal_flag rows; (e) stage 5 numbers with the dataset radii and with Shannon CN12/CN6 radii; (f) Pearson r of the geometric descriptors with the HSE-PBE-SOC gap for both radii sets.
- Look for: whether the "+ g14_flag only" and "+ all electronic" results stay far below "geometry only" in every panel, and which bars move in (e) and (f).
- Observation: RandomForest average MAE over 10 seeds is 1.869 +/- 0.015 (geometry only), 0.357 +/- 0.006 (+ g14_flag) and 0.455 +/- 0.005 eV (+ all electronic); with Shannon radii the geometry-only RF MAE rises from 1.85 to 3.03 eV.
