# FINDINGS

Status (2026-11-22): all stages 0 to 10 are done and committed, including optional stage 9 (end of file).
All numbers below come from `btp_perovskite.ipynb` (executed with outputs). The source of each number is in
`numbers.json` (key and stage) and in the `tables/` CSV files named next to it.

Labels: **CONFIRMED** = reproduced here and validated honestly. **HYPOTHESIS** = plausible but not tested (or tested only
weakly). **RETIRED** = was wrong or contaminated.

"Section 9" means the earlier exploratory values listed in CLAUDE.md section 9.

---

## A. Data (stage 0)

**A1. CONFIRMED. The dataset file and all stage-0 facts reproduce exactly.**
- Source: `git clone https://github.com/yjq829/perovskite_dataset.git` worked (commit b84ed4d, 2023-04-13). The file
  `perovs_data_final.xlsx` is in the repository root and was copied to `data/`.
- Outer merge on Formula: 490 compounds, 90 pure. Gap coverage: PBE 488, HSErel 299, HSErel_SOC 282, HSE-PBE-SOC 244.
- No PBE gap (2, both mixed): `RbGeBr_1.125Cl_1.875` and `Rb_0.375Cs_0.625GeBr_3`.
- 0 parse errors; A, B and X site sums are 1, 1 and 3 for every row. Sites: A = Cs, K, Rb, MA, FA; B = Ge, Sn, Pb, Ca, Sr, Ba; X = Cl, Br, I.
- 16 of 16 checks pass (`tables/stage0_checks.csv`).
- Count note: the published paper is said to mention 495 compositions. This file gives 490 after the merge, which
  matches the count expected in CLAUDE.md. I could not open the paper to check the 495 (see section F).
- Strict groups (from g2_frac): Group 14 = 180 compounds (76 with HSE-PBE-SOC), Group 2 = 187 (77), mixed B = 123 (91).

**A2. CONFIRMED. The recomputed tolerance factor and octahedral factor match the dataset columns.**
Max |difference| = 2.2e-16 (t) and 1.1e-16 (mu) over 488 rows. This uses the radii set in section 4 of CLAUDE.md
(the "dataset" set in the config cell). Mixed compounds use composition-weighted radii, which reproduces the dataset columns.

**A3. CONFIRMED. The dataset's "Bartel's Tolerance Factor" column is only the term (rA/rB)/ln(rA/rB)** (max difference
1.8e-14). It is not used anywhere. Bartel tau is recomputed as tau = rX/rB - nA (nA - (rA/rB)/ln(rA/rB)) with nA = 1.
- Formula check (done in session 2, see "Bartel tolerance factor check" below): the notebook formula is identical to Eq. (2)
  of Bartel et al. Nothing needed fixing, so no tau result changed.
- Bartel's paper uses Shannon radii (A closest to coordination 12, B closest to coordination 6). This project uses the
  dataset's radii set, so tau here is "Bartel's formula with the dataset radii". With these radii 230 of 490 compounds
  have tau < 4.18. The second radii set (task 4) is closer to Bartel's choice.

**A4. Known data issues (documented, not removed).**
- `equal_flag`: 41 of 244 rows have identical HSErel_SOC and HSE-PBE-SOC gaps (37 strict Group 2, 4 mixed B, 0 strict
  Group 14). Cause unknown. Results with and without these rows are in stage 8 (item E4).
- HSErel anomaly: the MAGe and MASn pure halides break the halide trend at HSErel and HSErel_SOC (Br above Cl).
  MAGe: Cl 3.02, Br 3.32, I 2.56 eV (HSErel). MASn: Cl 3.18, Br 3.53, I 2.57 eV (HSErel). At PBE and HSE-PBE-SOC all
  (A, B) series fall from Cl to I (`tables/stage6_consistency_checks.csv`).
- FAPbI3 and FAPbCl3 have no HSE-PBE-SOC gap (checked).
- `RbGeBr_1.125Cl_1.875` has HSE-PBE-SOC 1.34 eV but HSErel 4.31 eV and HSErel_SOC 4.28 eV, and it has no PBE row
  (so it has no deviation columns). It is a strict Group 14 compound, so it is one of the 76 rows in stage 4.
- Outliers (robust z > 3 within each B category for level-minus-PBE differences; `tables/stage1_outliers.csv`):
  2 rows for HSE-PBE-SOC minus PBE (FACa0.5Sr0.5Cl3, FASr0.5Ba0.5Cl3), 10 for HSErel minus PBE and 4 for HSErel_SOC minus PBE.
  MASrI3 is the only single-B outlier: its HSErel gap (3.47 eV) is BELOW its PBE gap (3.76 eV), while its HSE-PBE-SOC
  is 4.57 eV. Nothing was dropped.
- The points "near PBE 3 to 3.5 eV" are mostly mixed-B (30) and strict Group 2 (34) compounds, with 3 strict Group 14
  (FAGeCl3, FA0.5K0.375Cs0.125GeCl3, K0.125FA0.875SnCl3). The points near 4.2 to 4.4 eV are strict Group 2 (22) and mixed B (4). Only one of
  them is flagged by the robust rule (`tables/stage1_pbe_windows.csv`). By this rule they are ordinary members of their
  groups, not outliers.

## Dataset paper checks (session 2)

Source read: arXiv 2302.04896v1 (9 Feb 2023, the only version on arXiv), "A High-Throughput Computational Dataset of
Halide Perovskite Alloys", J. Yang, P. Manganaris, A. Mannodi-Kanakkithodi. 23-page PDF (main text pp. 1-15, supplement
pp. 16-23), downloaded from arxiv.org and read as text. Page numbers are PDF pages. The published Digital Discovery
version was not readable (publisher HTTP 403), so small differences from the published version are possible.

**P1. Band gap RMSE versus experiment (Table 2, p. 5, Section 3.1).** "Band Gap RMSE vs Exp (eV)": PBE 0.78, HSE-rel 0.93,
HSE-rel-SOC 0.74, HSE-PBE-SOC 0.70. Text (p. 5): "We find that on average, HSE-PBE-SOC is the best approach out of the
four for reproducing band gaps". On p. 6 the paper notes that the PBE RMSE "is not significantly different" from the
HSE-PBE-SOC RMSE ("accidental accuracy of semi-local functionals without SOC for hybrid organic-inorganic perovskites").
- Number of compounds: NOT stated. Table 2 has no count column, and the text gives no number. The experimental values are
  "collected from Tao el al and Almora et al" (p. 5). In the arXiv text these two references are not resolved (printed as "?").
  The points are only shown in Fig. 3(b) (p. 5). So the size and composition of the experimental set behind Table 2 are unknown.
- Lattice constants (p. 5): PBE RMSE 0.27 Å, HSE RMSE 0.31 Å versus experiment.
- This confirms the section 9 item 9 statement ("about 0.70 eV RMSE for HSE-PBE-SOC; HSErel worst") -> CONFIRMED.

**P2. Supplementary table for cubic compounds: does NOT exist in the arXiv version.** The supplement (pp. 16-23) has
Figs. S1-S12, Table SI (p. 22, list of 50 descriptor labels without values), Table SII (p. 23, 32 compounds screened at
PBE with PBE gap, decomposition energy and SLME) and Table SIII (p. 23, 14 compounds screened at HSE-PBE-SOC).
No table compares cubic-structure calculations with experiment. The paper (p. 6) says that "non-cubic phases may affect
the accuracy of the computational results here".
- Check of SII/SIII against the xlsx (notebook stage 0e2, `tables/stage0_paper_SI_gap_check.csv`): 42 of 46 printed gaps
  agree with the file within 0.01 eV. 4 rows of Table SII do not. CsSnI1.875Br1.125: paper 2.47, file 0.547;
  CsPbI0.75Br2.25: paper 2.49, file 1.619; CsPbBr2.625Cl0.375: paper 1.98, file 1.780; MAPbI2.25Br0.75: paper 1.53, file 1.829 eV.
  These are the last four rows of the table. The cause is not known (possibly a row shift in the printed table). The
  notebook uses the xlsx values.

**P3. Ionic radii source: NOT given in the paper.** The paper gives no radii values and cites no radii source.
Section 3.6 (p. 11) defines the octahedral factor, the Goldschmidt factor and "tBartel = rX/rB - [1 - (rA/rB)/ln(rA/rB)]"
(Eqs. 9-11) and states that "weighted averages of A-site (rA), B-site (rB), and X-site (rX) radii are considered" for mixed
compounds. The only citation there is ref. 7 (Bartel et al. 2019). The radii in this project were recovered from the
dataset's own t and mu columns (CLAUDE.md section 4). Comparison with pymatgen's Shannon table (task 4) shows that these
radii are a mixture: K 1.51, Rb 1.61, Cs 1.74, Sr 1.26 and Ba 1.42 equal Shannon ionic radii at coordination VIII.
Pb 1.19, Ca 1.00 and Cl 1.81, Br 1.96, I 2.20 equal Shannon ionic radii at coordination VI. Ge 0.87 equals the Shannon
CRYSTAL radius at coordination VI (the ionic radius is 0.73). Sn 1.15 is not in pymatgen's Shannon table (no Sn2+ entry).
MA 2.17 and FA 2.53: source not stated in the paper (see task 4).
- The paper's "tBartel" (Eq. 11) is Bartel's tau with nA = 1 written out. The dataset column, however, holds only the
  (rA/rB)/ln(rA/rB) term (item A3), not the full expression.

**P4. 495 versus 490 compositions.** The paper states 495 in the abstract (p. 1), Section 1 (p. 2), Table 1 (p. 4) and the
Conclusions (p. 14). Section 2.1 (p. 4) gives "90 pure, 126 A-site mixed, 151 B-site mixed, and 127 X-site mixed" =
494, not 495, so the paper's own breakdown is one short of its total. Table 1 (p. 4): PBE 495, HSE-rel 299, HSE-rel-SOC 282, HSE-PBE-SOC 244.
The xlsx file (stage 0e2, `tables/stage0_paper_counts.csv`): PBE sheet 488 rows (7 fewer than Table 1). By mixing type
among the PBE rows: pure 90 (paper 90), A-mixed 124 (126), B-mixed 151 (151), X-mixed 123 (127). The three HSE counts
match exactly (299, 282, 244). The outer merge has 490 compounds because 2 compounds appear only in the HSE sheets.
The paper does not explain the difference, and it says nothing about excluded or failed calculations. So: the
public file lacks 7 PBE entries (2 A-mixed, 4 X-mixed, and 1 more that also cannot be placed because the paper's own
breakdown sums to 494). Which compounds are missing cannot be determined from the paper.
- Data availability (p. 15): "https://github.com/yjq829/perovskite_dataset.git" (the repository used here). The title-block
  ESI footnote points to a different repository (github.com/PanayotisManganaris/manuscript--PIP-data-manifest), which
  was not checked.

## Bartel tolerance factor check (session 2)

**T1. CONFIRMED. The notebook formula equals Bartel's Eq. (2).** Source read: arXiv 1801.07700v2 (revised 6 Jan 2019),
PDF p. 2, and the Science Advances version (Sci. Adv. 2019, 5, eaav0693, PMC6368436), where it is also Eq. (2).
Plain text: tau = rX/rB - nA * ( nA - (rA/rB) / ln(rA/rB) )   (Eq. 2)
"where nA is the oxidation state of A, ri is the ionic radius of ion i, rA > rB by definition and tau < 4.18 indicates
perovskite" (arXiv p. 2). The discussion of Fig. 4 writes it for nA = 1 as tau = rX/rB + (rA/rB)/ln(rA/rB) - 1, which is
the same expression. Radii: Shannon radii, with the A and B radii "chosen to be closest to the coordination number of 12
and 6" (Materials and Methods, "Radii assignment"). No coordination number is given for X. Bartel gives no radii for MA or FA.
For double perovskites rB is the arithmetic mean of the two B radii. Bartel gives no rule for mixed A or X sites (the
weighted average used here is this project's choice, as in the dataset paper). The notebook already used this formula,
so no result changed.

## B. Descriptive physics (stages 1 and 2)

**B1. CONFIRMED. SOC shift (HSErel_SOC minus HSErel), mean over pure compounds (15 per B element, Sr 14).**
Ge -0.098, Sn -0.235, Pb -0.961 eV; Ca -0.107, Sr -0.095, Ba -0.106 eV (`tables/fig04_soc_shift_pure_by_B.csv`).
Matches section 9 item 1 (within 0.01 eV). In Group 14 the shift is negative for all 45 pure compounds, and its size
grows Ge < Sn < Pb for 14 of 15 (A, X) combinations. The exception is MA-iodide: MAGeI3 -0.12, MASnI3 -0.10.
There is no Z trend in Group 2. Without the equal_flag rows: Ca -0.088 (n=6), Sr -0.060 (n=7), Ba -0.134 (n=9). Group 14 has no equal_flag rows.

**B2. CONFIRMED. The SOC shift depends on the A site** (pure compounds, `tables/fig05_soc_shift_per_A.csv`).
PbI3: Cs -1.157, K -1.157, Rb -1.157, MA -0.666, FA -0.639. SnI3: Cs -0.352, MA -0.104, FA -0.197 eV. All match
section 9 item 1 within 0.01 eV. For every Group 14 B x X cell the organic (MA/FA) mean shift is smaller in size than the
inorganic (Cs/K/Rb) mean. Organic minus inorganic: +0.07 (GeCl3) to +0.50 eV (PbI3).
Within Group 2 the shift is controlled mainly by the halide: Cl about -0.02, Br about -0.09 to -0.10, I about -0.18 to -0.21 eV.

**B3. CONFIRMED. PBE versus hybrid error cancellation for Pb** (single-B compounds, `tables/fig02_level_minus_pbe_by_B.csv`).
HSE-PBE-SOC minus PBE is negative for all 22 Pb compounds (mean -0.306 eV). It is positive for every Ge (+0.59), Sn (+0.35),
Ca (+1.11), Sr (+1.03) and Ba (+1.04) compound. For HSErel_SOC minus PBE, 9 of 25 Pb compounds are below zero
(mean +0.33 eV).

**B4. CONFIRMED (with a correction to section 9 item 2). The gap falls from Cl to Br to I for every B element** at PBE
and HSE-PBE-SOC (same-compound means, `tables/stage2_halide_monotonic.csv`). The section 9 numbers are reproduced to
within 0.01 eV when each level is averaged over ALL its available compounds. On the SAME compounds (the 4 or 5 pure
compounds that have both PBE and HSE-PBE-SOC), the PBE means for Sn-Cl and Pb change:
Sn-Cl 1.10 (vs 1.37), Pb-Cl 2.27 (vs 2.34), Pb-I 1.56 (vs 1.64) eV. This happens because FASnCl3, FAPbCl3 and FAPbI3 drop
out. The Group 14 order is Pb > Ge > Sn at PBE and Ge > Pb > Sn at HSE-PBE-SOC for all three halides, also on the same compounds.

**B5. HYPOTHESIS (small n). Halide bowing has opposite signs in the two groups** (`tables/fig20_bowing_summary.csv`).
Deviation from linear interpolation between the pure end members (same A, B, level):
- PBE: strict Group 2 is negative in every pair: I-Br -0.19 (n=18), Br-Cl -0.18 (n=23), I-Cl -0.32 (n=22).
  Strict Group 14 is small or positive: I-Br +0.03 (n=26), Br-Cl +0.13 (n=30), I-Cl +0.14 (n=4).
- HSE-PBE-SOC: few points (Group 14: 9 I-Br and 6 Br-Cl; Group 2: 5 I-Br and 3 Br-Cl). No I-Cl compound has both end members.
- The Group 14 Br-Cl spread is large (std 0.29 eV at PBE), driven by a few Ge/Sn compounds with deviations up to +0.7 eV.
  These may be structural effects, not bowing. Not investigated.

**B6. CONFIRMED (descriptive). The geometric factors correlate with the gap differently in each group**
(HSE-PBE-SOC, `tables/fig07_gap_vs_geometry_correlations.csv`). Tolerance factor: Pearson r = 0.78 in strict Group 14,
0.43 in strict Group 2, 0.01 over all 244. rX: r = -0.91 in strict Group 2. Over all compounds the largest |r| are for
dEN (0.93), g2_frac (0.89), EN_B (-0.87) and IE_B (-0.81) (`tables/fig08_corr_with_gap_all_compounds.csv`).

## C. Variance decomposition (stage 3)

**C1. CONFIRMED (descriptive). One-way variance fractions, HSE-PBE-SOC** (`tables/fig09_variance_explained.csv`).
- All 244: A 0.015, X 0.125, B element 0.659, B group (g2_frac, 3 levels) 0.731. The sum of the one-way fractions is
  1.53, so they overlap and are not additive. Additive one-hot A+X+B model, in-sample R2 0.788.
- Strict Group 14 (n=76): A 0.487, X 0.162, B element 0.097, additive R2 0.801.
- Strict Group 2 (n=77): A 0.097, X 0.811, B element 0.064, additive R2 0.912.
- The dataset is not balanced (main-label counts in `tables/stage3_label_balance.csv`).
- Without the 41 equal_flag rows (n=203), the A-site fraction over all compounds rises from 0.015 to 0.108. B group: 0.700.

**C2. RETIRED. Section 9 item 3** ("B group 0.63", "A site only 1 percent"). The group was defined by idxmax. With the
g2_frac groups, the B-group fraction is 0.731 (difference +0.10). The 1 percent A-site value is reproduced (0.015) over all
compounds, but it hides A as the largest single label inside strict Group 14 (0.487), and it changes to 0.108 without
the equal_flag rows. It must not be used as "the A site does not matter".

## D. Strict Group 14 composition models (stage 4)

Data: g2_frac == 0 and HSE-PBE-SOC present. n = 76, 43 groups (group = main A + main B + main X). GroupKFold(5) test
folds: 16, 15, 15, 15, 15 rows. Mean baseline: grouped R2 -0.04 (MAE 0.58 eV).

**D1. CONFIRMED (small n). The linear model on composition fractions** gives grouped R2 0.738 (MAE 0.285 eV) and random
R2 0.739 (`tables/fig12_random_vs_grouped.csv`). Same as section 9 (0.74 / 0.74). Over 10 random-KFold seeds, random R2 ranges 0.739 to 0.794.

**D2. HYPOTHESIS. Ridge with polynomial products** (degree 2, standardized). Pooled out-of-fold R2, random / grouped:
alpha 0.1: 0.924 / 0.794; alpha 1: 0.912 / 0.866; alpha 10: 0.857 / 0.864; alpha 100: 0.658 / 0.685.
Nested grouped CV (alpha picked by an inner GroupKFold(4) on training folds only, so honest): R2 0.864, MAE 0.213 eV.
It chose alpha = 10 in all 5 outer folds. Random KFold overstates the score most at low alpha (0.924 vs 0.794 at
alpha 0.1). Over 10 random seeds, the alpha 1 random R2 ranges 0.684 to 0.923, so one random split is not reliable.

**D3. CONFIRMED (small n). Linear coefficients** (eV per full site substitution; reference CsPbCl3-like, intercept 1.777;
95 percent row bootstrap / group bootstrap, 500 resamples; `tables/fig11_coefficients.csv`):
K -0.112 [-0.32, 0.07] / [-0.28, 0.11]; Rb -0.106 [-0.26, 0.05] / [-0.28, 0.12]; MA +0.820 [0.67, 1.00] / [0.61, 1.00];
FA +1.190 [0.89, 1.54] / [0.80, 1.65]; Br -0.464 [-0.67, -0.29] / [-0.68, -0.22]; I -0.958 [-1.14, -0.79] / [-1.18, -0.74];
Ge +0.223 [0.03, 0.43] / [-0.04, 0.47]; Sn -0.321 [-0.47, -0.16] / [-0.50, -0.14].
All match section 9 within 0.005 eV. K and Rb intervals include zero. Ge is above zero with the row bootstrap, but its
interval includes zero with the group bootstrap.

**D4. HYPOTHESIS (association only). Deviation of cubicity versus organic fraction.**
Same 75 rows (RbGeBr_1.125Cl_1.875 has no deviation values). Grouped R2 (linear): B+X only 0.179; + organic 0.773;
+ deviation (5 columns) 0.786; + both 0.822; all fractions 0.780; all fractions + deviation 0.854
(`tables/fig14_organic_vs_deviation.csv`).
- Coefficient changes when the deviation columns are added: MA 0.82 -> 0.25, FA 1.19 -> -0.37, Ge 0.24 -> -0.15
  (sign flips show collinearity).
- Inorganic A compounds have deviation near 0 (max dev_b 0.021; max dev_alpha, dev_beta, dev_gamma 0.0002 or less).
  Correlation of organic fraction with the deviation columns: r = 0.43 to 0.57.
- Residual after removing B and X effects: inorganic A (dev_b < 0.01) mean -0.47 eV (n=37). Organic A with
  dev_b < 0.01: +0.31 eV (n=6). Organic A with dev_b >= 0.01: +0.64 eV (n=22).
  So at near-zero deviation, organic compounds still sit above inorganic ones. Deviation and "organic A" cannot be
  separated with these data. No mediation claim.

## E. Honest generalization (stage 5) and robustness (stage 8)

Target HSE-PBE-SOC (244 rows). Clean leave-one-B-out: test = frac_b == 1, train = frac_b == 0.
Train / test sizes: Ge 174/22, Sn 172/24, Pb 154/22, Ca 168/25, Sr 170/22, Ba 162/19. Rows with a partial amount of
the held-out element are left out: 48 to 68 per split.

**E1. CONFIRMED (as a result). Clean leave-one-B-out, average MAE over 6 elements** (`tables/fig15_clean_leave_one_B_out_average.csv`):
mean baseline 2.059; geometry only: RF 1.847, Ridge 2.126; geometry + Z: RF 1.954, Ridge 1.725;
geometry + electronic: RF 0.447, Ridge 0.698 eV. RF per element (geometry + electronic): Ge 0.62, Sn 0.71, Pb 0.52,
Ca 0.48, Sr 0.16, Ba 0.19. Ridge Pb 1.63. All within 0.007 eV of section 9 item 7.
- Robustness: RF over 10 seeds gives 0.447 to 0.463 (geometry + electronic). 95 percent bootstrap range of the average
  MAE (test rows resampled): RF 0.39 to 0.50, Ridge 0.65 to 0.76, mean baseline 1.94 to 2.17 eV.
- Geometry alone does not beat the mean baseline in a meaningful way (RF 1.85 vs 2.06; Ridge is worse than the baseline).

**E2. CONFIRMED (new). The electronic descriptors mainly act as a Group 14 / Group 2 label** (ablation,
`tables/fig16_ablation.csv`, average MAE, RF / Ridge):
geometry 1.847 / 2.126; + g14_flag only 0.361 / 0.343; + EN_B only 0.420 / 0.811; + IE_B only 0.658 / 0.521;
+ EN_X only 1.871 / 2.126; + all electronic 0.447 / 0.698; + all electronic + g14_flag 0.446 / 0.512.
A single 0/1 group label added to geometry does at least as well as all four electronic descriptors, for both models.
EN_X adds nothing. The same holds for the PBE target (g14_flag 0.455 / 0.549 vs electronic 0.527 / 0.808) and for
HSErel_SOC (0.392 / 0.514 vs 0.455 / 0.717) (`tables/stage8_ablation_other_targets.csv`). This turns the section 9 item 7
interpretation ("electronic features act as a group label") from HYPOTHESIS into a tested result. Note that the test
only shows that the label explains as much. It does not show that EN_B or IE_B carry no further physics within a group.

**E3. CONFIRMED (negative result). Leave-one-group-out fails completely** (`tables/fig17_leave_one_group_out.csv`).
- Train strict Group 14 (76) -> test strict Group 2 (77): MAE 3.50 (baseline), 3.54 to 3.66 (RF), 3.70 to 5.03 (Ridge) eV.
- Train strict Group 2 (77) -> test strict Group 14 (76): baseline 3.50; best Ridge geometry + electronic 2.27; RF 3.28 to 3.57 eV.
- Every prediction is on the wrong side: 100 percent too low for Group 2 and 100 percent too high for Group 14, for
  every model and feature set (`tables/fig17_error_side.csv`). No model extrapolates the gap level across the two groups.
- Without the equal_flag rows: RF geometry + electronic, G14 -> G2 MAE 3.49 (n_test 40), G2 -> G14 3.33.

**E4. CONFIRMED. Other targets and the equal_flag rows do not change the picture** (`tables/fig21_*`).
- PBE target (488 rows): baseline 1.493, RF geometry + electronic 0.527, Ridge 0.808.
- HSErel_SOC target (282): baseline 1.660, RF 0.455, Ridge 0.717.
- Without the 41 equal_flag rows (203 rows): RF geometry + electronic 0.501 (vs 0.447), Ridge 0.729 (vs 0.698),
  g14_flag RF 0.359 / Ridge 0.354.
- Strict Group 14 linear model, grouped R2: PBE 0.736 (n=178), HSErel 0.544 (88), HSErel_SOC 0.684 (82), HSE-PBE-SOC 0.738 (76).
- Second radii set: NOT RUN. None was supplied. The code path exists: add a set to `RADII_SETS` in the config cell.

## F. Cross-fidelity (stage 6)

**F1. CONFIRMED. PBE plus simple descriptors predicts HSE-PBE-SOC well under clean leave-one-B-out** (average MAE,
`tables/fig18_cross_fidelity_lobo.csv`, 243 rows): PBE gap as is 0.737; linear on PBE 0.405; Ridge on PBE + geometry
+ electronic 0.173; RF on the same 0.350; RF without PBE 0.457. For HSErel_SOC (281 rows), the best is RF on PBE +
descriptors 0.414, and linear on PBE gives 0.476. For HSErel (297), linear on PBE + g2_frac gives 0.308.
HSE-PBE-SOC is computed on the PBE structures, so a close link to the PBE gap is expected.

**F2. CONFIRMED, with ranges wider than section 9 item 6. Organic minus Cs offset, pure Pb/Sn iodides and bromides**
(`tables/fig18_organic_minus_Cs_offsets.csv`): PBE +0.20 to +0.71 (8 pairs); HSE-PBE-SOC +0.31 to +0.89 (7 pairs; FAPbI3 missing);
HSErel_SOC +0.96 to +2.31 (8 pairs); HSErel +0.47 to +2.14. The section 9 ranges (+0.2 to +0.6, +0.3 to +0.7, +1.0 to +1.9)
fit when the SnBr3 pairs are left out. The SnBr3 pairs give the largest offsets.
All 13 quoted HSE-PBE-SOC values and both quoted HSErel_SOC values (MASnI3 2.47, FASnI3 2.67) agree with the file
within 0.005 eV (`tables/stage6_quoted_values_check.csv`).

**F3. CONFIRMED. Consistency checks** (`tables/stage6_consistency_checks.csv`): SOC shift < 0 for 45/45 pure Group 14;
|SOC| Pb > Sn > Ge in 14/15 (A, X). Gap Cl > Br > I for each (A, B): PBE 30/30, HSE-PBE-SOC 24/24, HSErel 28/30,
HSErel_SOC 27/29. The violations are MAGe and MASn at HSErel and HSErel_SOC (item A4).

## G. Experiment (stage 7)

**G1. NOT AVAILABLE (DRAFT only).** `experimental/experimental_verified.csv` does not exist. `experimental_template.csv`
was written (90 pure compounds, dataset gaps filled, experimental columns empty). The stage ran on the approximate
section 9 item 8 values. Every figure is stamped "DRAFT, unverified experimental values".
How the DRAFT values were made: ranges were replaced by their midpoints (Pb bromides 2.25, MAPbCl3 3.0), and "Pb iodides
about 1.6" and "Pb bromides" were applied to the Cs, MA and FA compounds only.
DRAFT numbers, for orientation only: HSE-PBE-SOC n=11, MAE 0.45, mean error -0.45 eV (Cs -0.73, n=3; organic -0.34, n=8);
PBE n=13, MAE 0.48; HSErel_SOC n=13, MAE 0.57, mean error +0.28 eV.
**HYPOTHESIS (not tested):** forced-cubic small A cations give gaps that are too small. This needs verified values.
**Not verified:** the claim that the source paper reports about 0.70 eV RMSE for HSE-PBE-SOC against experiment and
that HSErel is the worst level (section 9 item 9). The paper's web page refused automated access (HTTP 403), and
`papers/` does not contain the dataset paper.

---

## Discrepancies with CLAUDE.md section 9 (more than about 0.05)

| Item | Section 9 | This run | Difference | Reason |
|---|---|---|---|---|
| 3: variance, B group | 0.63 | 0.731 | +0.10 | group now from g2_frac (earlier idxmax) -> RETIRED |
| 4: products Ridge alpha 1, grouped R2 | 0.80 | 0.866 | +0.066 | exact earlier pipeline unknown (here: degree-2 products + scaling + Ridge, GroupKFold(5) without shuffling) |
| 2: PBE mean Sn-Cl on the SAME compounds | 1.37 | 1.102 | -0.27 | earlier means were over different compound sets; on all available compounds, 1.374 (match) |
| 2: PBE Pb-Cl / Pb-I on the SAME compounds | 2.34 / 1.64 | 2.274 / 1.560 | -0.07 / -0.08 | same reason |
| 5: "+ deviation" grouped R2 (read from a bar chart) | about 0.72 | 0.786 | +0.07 | bar-chart reading; same 75 rows here |
| 5: "grouped R2 0.73 to 0.82" with deviation | 0.73 -> 0.82 | 0.780 -> 0.854 | +0.05 / +0.03 | here n=75 (one row without deviation); earlier row set unknown |
| 6: organic minus Cs ranges | PBE 0.2-0.6, HPS 0.3-0.7, HSErel_SOC 1.0-1.9 | 0.20-0.71, 0.31-0.89, 0.96-2.31 | upper ends +0.1 to +0.4 | the SnBr3 pairs fall outside the earlier ranges |
| Stage 7 refers to "section 9 item 10" | - | item 8 used | - | fixed in CLAUDE.md in session 2 (now reads item 8) |

Within 0.05 (reproduced): SOC shifts (item 1), all-compound halide means (item 2), A-site and X-site variance
fractions (item 3), linear R2 and coefficients (item 4), MA/FA/Ge coefficient changes (item 5), every clean
leave-one-B-out number (item 7), all quoted dataset values (item 6).

## Open questions and things not done

1. RESOLVED in session 2 through the arXiv version (see "Dataset paper checks"). Still open: the size of the experimental
   set behind the paper's Table 2, and which 7 PBE compounds are missing from the public file.
2. No second radii set was supplied, so robustness test (1) was skipped.
3. Experimental comparison is DRAFT only, until `experimental/experimental_verified.csv` is filled from primary papers.
4. Cause of the 41 equal_flag rows is unknown. A question for the dataset authors.
5. The MASrI3 HSErel < PBE anomaly and the RbGeBr_1.125Cl_1.875 HSErel 4.31 vs HSE-PBE-SOC 1.34 eV gap are unexplained.
6. Group 14 Br-Cl "bowing" up to +0.7 eV for a few Ge/Sn compounds was not investigated.
7. Main-label grouping (group_key) breaks ties by list order (for example FA0.5Cs0.5 counts as Cs). This affects only the
   grouping of a few mixed-A rows.
8. Mixed compounds are described by composition-weighted averages of site properties. This is a simplification; for
   example, it ignores local distortions and ordering.
9. Package note: `pymatgen` failed to install at first (the bibtexparser build failed with the old setuptools). After
   `pip install --upgrade --ignore-installed setuptools` it installed (version 2026.10.31). No alternative package was needed.

## Stage 9 (optional): compact formula

**H1. HYPOTHESIS. A 3-term linear formula does about as well as the 8-term composition model in strict Group 14, but
the chosen formula is not stable.** PySR was not used, because it needs Julia. Instead, every linear formula with
1 to 3 terms from a library of 21 descriptors (1561 formulas) was searched. The formula was chosen inside each training
fold with an inner GroupKFold(4) (nested, honest). Outer splits: the same GroupKFold(5) as stage 4 (`tables/stage9_compact_formula.csv`).
- Nested grouped R2 0.749, MAE 0.267 eV (n = 76), against 0.738 / 0.285 eV for the 8-term linear fraction model.
- The five outer folds chose different formulas: rA + rB + t; rX + t + Z_B; frac_Sn + organic_frac + Z_X; mu + t + IE_B (twice).
  The tolerance factor appears in 4 of 5.
- Chosen on all 76 rows: gap = -20.251 + 7.231 mu + 6.284 t + 0.540 IE_B (eV), grouped R2 0.805. This number is
  optimistic, because the choice saw all rows. Within strict Group 14, IE_B only takes the Ge, Sn and Pb values, so it
  works like a B-element label here.
