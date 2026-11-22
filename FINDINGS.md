# FINDINGS

Status (2026-11-22, session 2): all stages 0 to 10 done; session 2 tasks 1-7 (paper checks, Bartel check, candidate
experimental values, second radii set, suspicious-row sensitivity, tidy-up, uncertainty) are being added below.
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
- Second radii set: done in session 2, see "Second radii set (session 2)" below.

## Second radii set (session 2, stage 8 part 1)

Set `shannon_CN12_CN6` (config cell). Sources per radius: `tables/radii_sets_sources.csv`.
- K 1.64, Rb 1.72, Cs 1.88 Å: Shannon ionic radii, coordination XII, read from pymatgen (`Species.get_shannon_radius`).
  The dataset set uses 1.51 / 1.61 / 1.74 Å (coordination VIII).
- Ge 0.73, Ca 1.00, Pb 1.19, Sr 1.18, Ba 1.35 Å: Shannon ionic radii, coordination VI (pymatgen). The dataset set uses
  Ge 0.87 (the coordination-VI crystal radius), Sr 1.26 and Ba 1.42 Å (coordination VIII).
- Cl 1.81, Br 1.96, I 2.20 Å: Shannon ionic radii, coordination VI (pymatgen). Identical to the dataset set.
- MA 2.17 and FA 2.53 Å: effective radii of Kieslich, Sun, Cheetham, Chem. Sci. 2014, 5, 4712, as quoted
  (217 pm and 253 pm) in Kieslich et al., Chem. Sci. 2015, 6, 3430 (read via Europe PMC, PMC5492664). These are the SAME
  values as in the dataset set. The 2014 paper itself could not be read (RSC HTTP 403; not in Europe PMC).
- Sn2+ 1.15 Å: NOT changed. pymatgen's Shannon table has no Sn2+ entry, and Travis et al. 2016 (which proposes revised
  B2+ radii) could not be read (RSC 403; the Europe PMC full text returned HTTP 500). So Sn keeps the dataset value,
  and its source remains unknown.

**R1. CONFIRMED. The main conclusions of stage 5 survive the second radii set; the geometry-only numbers do not.**
(`tables/fig21_second_radii_comparison.csv`, fig21 panels e, f)

| quantity (HSE-PBE-SOC, clean leave-one-B-out average MAE, eV) | dataset radii | Shannon radii | change |
|---|---|---|---|
| geometry only, RF | 1.847 | 3.025 | +1.178 |
| geometry only, Ridge | 2.126 | 2.350 | +0.224 |
| geometry + Z, RF | 1.954 | 2.839 | +0.885 |
| geometry + Z, Ridge | 1.725 | 1.488 | -0.237 |
| geometry + electronic, RF | 0.447 | 0.452 | +0.005 |
| geometry + electronic, Ridge | 0.698 | 1.045 | +0.347 |
| + g14_flag only, RF | 0.361 | 0.306 | -0.055 |
| + g14_flag only, Ridge | 0.343 | 0.341 | -0.002 |
| + EN_B only, RF / Ridge | 0.420 / 0.811 | 0.375 / 0.862 | -0.045 / +0.051 |
| mean baseline | 2.059 | 2.059 | 0 |

What survives: (i) geometry alone never beats the mean baseline in a useful way (with Shannon radii RF is even worse,
3.03 eV against 2.06); (ii) adding only g14_flag does at least as well as all electronic descriptors (RF 0.31 versus 0.45;
Ridge 0.34 versus 1.05); (iii) leave-one-group-out still fails in both directions. G14 -> G2 (Shannon radii): RF 3.60-3.67, Ridge
3.67-4.64 eV. G2 -> G14: RF 3.30-3.56, Ridge 1.72-3.69 eV, against a mean baseline of 3.50 eV (all values in the comparison table).
What changes: the size of the geometry-only and Ridge numbers, which depend on how the radii place Ge, Sr and Ba
relative to the other elements. They are therefore NOT robust numbers.
- Tolerance/octahedral analyses (Pearson r with the HSE-PBE-SOC gap, dataset -> Shannon). t: strict Group 14 0.78 -> 0.73,
  strict Group 2 0.43 -> 0.45, all 0.01 -> -0.01. mu: Group 14 0.00 -> -0.09, Group 2 0.25 -> 0.32, all 0.41 -> 0.36.
  Bartel tau: Group 14 -0.58 -> -0.36, Group 2 -0.30 -> -0.46, all 0.15 -> -0.06. rX is unchanged (same halide radii).
  The t correlation inside Group 14 is robust. The tau correlations are not: they change by up to 0.22.
- Counts over 490 compounds: tau < 4.18: 230 -> 263; 0.8 <= t <= 1.0: 328 -> 386. Mean t 0.879 -> 0.910; mean mu 0.579 -> 0.556.

## Suspicious rows: sensitivity (session 2, stage 8 part 5)

No row was deleted from the data. Each group of suspicious rows was left out in turn and the key results of stages 3, 4
and 5 were recomputed (HSE-PBE-SOC target; `tables/stage8_suspicious_rows_sensitivity.csv`). Rows in each scenario:
MASrI3 1 (it has an HSE-PBE-SOC gap), RbGeBr1.125Cl1.875 1, equal_flag 41, all together 43.

**S1. CONFIRMED. No headline conclusion depends on these rows; three numbers move by more than 0.05.**

| quantity | all rows | without MASrI3 | without RbGeBr1.125Cl1.875 | without 41 equal_flag | without all 43 |
|---|---|---|---|---|---|
| n rows (HSE-PBE-SOC) | 244 | 243 | 243 | 203 | 201 |
| eta2 A site | 0.015 | 0.015 | 0.014 | 0.108 | 0.105 |
| eta2 X site | 0.125 | 0.129 | 0.132 | 0.126 | 0.139 |
| eta2 B element | 0.659 | 0.658 | 0.659 | 0.621 | 0.619 |
| eta2 B group (g2_frac) | 0.731 | 0.731 | 0.730 | 0.700 | 0.698 |
| strict G14 linear grouped R2 | 0.738 | 0.738 | 0.780 | 0.738 | 0.780 |
| strict G14 products Ridge, nested grouped R2 | 0.864 | 0.864 | 0.874 | 0.864 | 0.874 |
| coefficient MA / FA / I / Ge (eV) | 0.820 / 1.190 / -0.958 / 0.223 | same | 0.821 / 1.188 / -0.972 / 0.239 | same as all | 0.821 / 1.188 / -0.972 / 0.239 |
| LOBO avg MAE, geometry only, RF / Ridge | 1.847 / 2.126 | 1.875 / 2.130 | 1.853 / 2.129 | 1.867 / 1.957 | 1.851 / 1.976 |
| LOBO avg MAE, + g14_flag only, RF / Ridge | 0.361 / 0.343 | 0.367 / 0.345 | 0.361 / 0.343 | 0.359 / 0.354 | 0.384 / 0.357 |
| LOBO avg MAE, + all electronic, RF / Ridge | 0.447 / 0.698 | 0.451 / 0.698 | 0.457 / 0.695 | 0.501 / 0.729 | 0.510 / 0.728 |
| leave-one-group-out MAE, RF geom + elec, G14->G2 / G2->G14 | 3.663 / 3.282 | 3.670 / 3.312 | 3.669 / 3.277 | 3.489 / 3.325 | 3.505 / 3.269 |

- Changes above 0.05: the A-site eta2 (+0.09 without the equal_flag rows); strict G14 linear grouped R2 (+0.042 without
  RbGeBr1.125Cl1.875, just under 0.05, but this single row moves the score most); Ridge geometry-only LOBO (-0.17 without
  equal_flag); RF geometry + electronic LOBO (+0.054 without equal_flag); leave-one-group-out G14 -> G2 (-0.17 without
  equal_flag, still about 3.5 eV).
- The ordering g14_flag <= all electronic << geometry only holds in every scenario, and leave-one-group-out fails in every scenario.
- Text the student could send to the dataset authors: `data_issues_for_authors.md` (7 neutral observations, including these
  three issues, the 495-vs-488 count and the four Table SII rows).

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

**G1. DRAFT v1 (session 1).** `experimental/experimental_verified.csv` still does not exist. The stage ran on the
approximate section 9 item 8 values; every figure is stamped "DRAFT, unverified experimental values". Ranges were
replaced by their midpoints, and "Pb iodides/bromides" were applied to the Cs, MA and FA compounds only. DRAFT v1 numbers,
HSE-PBE-SOC: n=11, MAE 0.45, mean error -0.45 eV (Cs -0.73, n=3; organic -0.34, n=8).

**G2. DRAFT v2 (session 2): candidate literature values, NOT verified** (`experimental/experimental_candidates.csv`,
status "candidate, student must verify"; figure `fig19_vs_experiment_candidates.png`, stamped "DRAFT, candidate
experimental values (not verified)"; tables `fig19_vs_experiment_candidates_*.csv`).
- 23 rows for the 18 compounds. One value per compound is selected for the analysis: room temperature, perovskite phase,
  single crystal preferred over powder over film. Each row gives method, phase, temperature, sample type, reference,
  DOI, location in the paper and how it was read.
- How the values were read. Publisher pages (ACS, RSC, Science) refused automated access (HTTP 403), so the values come from:
  (i) the HybriD3 experimental database at Duke University, which stores each value with its primary reference, method,
  phase and in-paper location (downloaded dataset files); (ii) abstracts via Europe PMC and OpenAlex;
  (iii) open-access full texts (arXiv, Nature Communications, OSTI accepted manuscript, Springer Open).
  Values taken from the database were NOT checked in the paper itself. The student must check each one in the primary paper.
- Phase classes of the 18 selected values: cubic 5, non-cubic 8, unknown 5. Several phases are only "as recorded" by the database.
- Results (dataset minus experiment; all 18 selected compounds, 16 at HSE-PBE-SOC because FAPbI3 and FAPbCl3 have no value there):

| level | n | MAE (eV) | RMSE (eV) | mean signed error (eV) | paper Table 2 RMSE (eV) |
|---|---|---|---|---|---|
| PBE | 18 | 0.627 | 0.761 | -0.539 | 0.78 |
| HSErel | 18 | 0.849 | 0.968 | +0.624 | 0.93 |
| HSErel_SOC | 18 | 0.723 | 0.811 | +0.094 | 0.74 |
| HSE-PBE-SOC | 16 | 0.559 | 0.696 | -0.558 | 0.70 |

  The paper's Table 2 RMSEs are for an unknown set of compounds, so the agreement in size is not a check of the same numbers.
- Split by A site, mean signed error (eV), Cs (n=7) / organic MA, FA (n=11; 9 at HSE-PBE-SOC): PBE -0.92 / -0.30;
  HSErel -0.17 / +1.13; HSErel_SOC -0.81 / +0.67; HSE-PBE-SOC -0.93 / -0.27.
- Split by halide, HSE-PBE-SOC mean signed error: Cl -1.06 (n=3), Br -0.62 (n=5), I -0.33 (n=8) eV.
- Split by phase class, HSE-PBE-SOC mean signed error: cubic -0.82 (n=4), non-cubic -0.51 (n=7), unknown -0.41 (n=5) eV.
  The cubic group is small and contains CsSnCl3, whose candidate value is for nanocrystals (error -1.58 eV).
- On the 16 compounds with all four levels: HSE-PBE-SOC MAE 0.56; PBE 0.66; HSErel 0.80; HSErel_SOC 0.75 eV.
  `tables/fig19_vs_experiment_candidates_common_set.csv`.

**G3. HYPOTHESIS (pattern survives, not verified). Cs compounds are much too low at HSE-PBE-SOC.** With compound-specific
candidate values the HSE-PBE-SOC mean signed error is -0.93 eV for 7 Cs compounds (DRAFT v1: -0.73, n=3) and -0.27 eV for
9 organic compounds (v1: -0.34, n=8) (`tables/stage7_cs_pattern_v1_vs_v2.csv`). The gap between the two groups
survives and is larger. The same holds at PBE and HSErel_SOC. At HSErel the Cs compounds are close (-0.17 eV) and
the organic ones too high (+1.13 eV). The cause is not tested: it may be the forced cubic structure, the phase of the
measurement (the Cs compounds are measured in orthorhombic or unknown phases), or the value choice (for example, the
CsPbCl3 gap is 3.03 eV by exciton subtraction but its absorption onset is 2.88 eV). Must be redone with verified values.

**Not verified:** every candidate value. The paper's own RMSE (Table 2) is now confirmed from the arXiv version
(see "Dataset paper checks", P1).

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
2. Second radii set: done in session 2 (Shannon CN12/CN6). Still open: a documented Sn2+ radius (none in pymatgen;
   Travis et al. 2016 not readable here), and a check of Kieslich 2014 itself for MA/FA.
3. Experimental comparison is DRAFT only (v1: section 9 values; v2: candidate literature values), until the student
   creates `experimental/experimental_verified.csv` after checking the candidates in the primary papers.
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
