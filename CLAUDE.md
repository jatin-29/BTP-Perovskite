# CLAUDE.md: BTP project brief (physics-oriented study of ABX3 halide perovskite band gaps)

## 0. What you are asked to do

Build the complete computational part of a B.Tech project (BTP) from scratch, as ONE clean, reproducible notebook
(`btp_perovskite.ipynb`, must run top to bottom on Google Colab and locally), plus the supporting files listed in
section 10. Read this whole file before writing any code. Work stage by stage (section 6). Do not stop to ask for
approval between stages unless you are blocked. Execute the notebook so that it is saved WITH outputs.

The student will read your outputs and notes to learn the material. Clarity matters more than cleverness.

## 1. Project context

- Student: Jatin, Engineering Physics, Delhi Technological University (DTU).
- 15-day project, started 2026-11-18, target finish 2026-12-02. The supervisor wants a higher-level paper that is
  physics-first rather than pure machine learning.
- Direction chosen: how the band gap of ABX3 halide perovskites divides between GEOMETRIC contributions
  (ionic sizes, tolerance factor, lattice distortion) and ELECTRONIC contributions (B-site chemistry, halide,
  spin-orbit coupling, hybrid-functional opening), using small interpretable models, honest (grouped) validation,
  physics-consistency checks, cross-fidelity checks and comparison with literature experimental gaps.
- Working title: "Geometric versus Electronic Contributions to the Band Gap of ABX3 Halide Perovskites: An
  Interpretable, Physics-Constrained Study with Honest Cross-Validation". It may change with the results.
- Own DFT calculations are only a stretch goal. Do not attempt them.

## 2. Rules (follow strictly)

1. Do NOT write paper prose (no abstract, introduction, discussion, conclusions). The student writes the paper
   in his own words. You produce code, numbers, figures, plain notes (sections 10).
2. Do NOT invent numbers, references, experimental values or dataset facts. Every number you report must come
   from code you ran. If something is not in the data, say so.
3. Label every finding in `FINDINGS.md` as CONFIRMED (reproduced here, validated honestly), HYPOTHESIS (plausible,
   not tested) or RETIRED (was wrong or contaminated).
4. Validation must be honest: never report random-split scores as the headline. Always include a trivial baseline
   (predict the training mean). Report grouped or held-out splits, and state their size.
5. Code style: simple and readable, small steps, plain English comments only, fixed random seeds
   (`random_state=0`), no clever one-liners. The student is a beginner in Python and ML.
6. Figures: 300 dpi, labelled axes with units (eV), consistent style, colour-blind-safe colours, readable legends,
   saved to `figures/` with the names in section 7. Where sensible, make multi-panel figures (the paper has a
   page limit). Do not use the same colour for different meanings across figures.
7. If a number you obtain differs from section 9 by more than about 0.05 eV (or 0.05 in R2), do not silently
   adjust. Write the discrepancy in `FINDINGS.md`.
8. Do not use the dataset's own "Bartel's Tolerance Factor" column (see section 3). Recompute from the radii.
9. Never create, simulate or fill in missing dataset values. If the data cannot be obtained, stop and say so.
10. Do not fabricate experimental gaps. The experimental comparison reads a CSV the student fills in himself
   (section 6, stage 7).

## 3. Data

Source: Mannodi-Kanakkithodi group, Digital Discovery 2023 (Yang, Manganaris, Mannodi-Kanakkithodi, "A
high-throughput computational dataset of halide perovskite alloys", CC-BY) halide perovskite dataset.

### Getting the data (do this yourself; the student does NOT upload it)
1. If `data/perovs_data_final.xlsx` already exists, use it.
2. Otherwise run `git clone https://github.com/yjq829/perovskite_dataset.git` into a temporary folder. This is the
   repository named in the paper's data availability statement (the paper prints the URL with a space, so this
   exact URL is an inference; if it fails, search for the repository named in the paper). Find
   `perovs_data_final.xlsx` anywhere inside it (search recursively) and copy it to `data/`.
3. If the clone or download fails (for example because of limited network access), do NOT create, simulate or guess
   data. Stop the analysis, write exactly what failed into `README.md` and `FINDINGS.md`, and report to the
   student that the file must be uploaded to `data/`.
4. After getting the file, run the assertions listed below. The published paper mentions 495 compositions, while the
   merged table of this project had 490 rows: a slightly different version is possible. If your counts differ from
   section 3, report both sets of counts in `FINDINGS.md` and continue with the file you have.

Expected file: `data/perovs_data_final.xlsx`. Four sheets, one per level of theory:

| Sheet | Meaning |
|---|---|
| `PBE` | PBE relaxation and PBE gap (cheap, underestimates gaps) |
| `HSErel` | HSE06 with full HSE geometry relaxation |
| `HSErel_SOC` | HSErel plus spin-orbit coupling (SOC) |
| `HSE-PBE-SOC` | static HSE+SOC on PBE-relaxed structures (best versus experiment according to the source paper; verify against its main Table 2) |

PBE sheet columns include: Formula, Decomposition Energy (eV p.f.u.), Band Gap (eV), SLME @ 5um, deviation of
cubicity columns (b axis, c axis, alpha, beta, gamma; find them by searching for "Deviation"), Octaehdral Factor
(this spelling is in the file), Tolerance Factor, Bartel's Tolerance Factor.

Facts to verify with assertions in stage 0:
- Outer merge on Formula over the four sheets gives 490 compounds, of which 90 are pure (formula contains exactly one "_").
- Gap coverage: PBE 488, HSErel 299, HSErel_SOC 282, HSE-PBE-SOC 244.
- Two hybrid-only mixed compounds have no PBE gap.
- Sites: A = Cs, K, Rb, MA, FA; B = Pb, Sn, Ge, Ca, Sr, Ba; X = I, Br, Cl. Structures are pseudo-cubic.
- Group 14 B (Ge, Sn, Pb) have an ns2 lone pair; Group 2 B (Ca, Sr, Ba) are noble-gas-like.

Formula format examples: `CsPbI_3`, `MAPbI_3`, `FAGeI_3`, `RbGeBr_1.125Cl_1.875`, `Rb_0.375Cs_0.625GeBr_3`.
Parsing: replace "MA" by "Ma" and "FA" by "Fa" first, then use the regex `([A-Z][a-z]?)(?:_([0-9.]+))?` to get
element counts (missing count means 1.0). A-site counts sum to 1, B-site counts sum to 1, halide counts sum to 3
(use halide fraction = count / 3). Expect 0 parse errors. The `Pearson_Correlation/*` csv files have no header and
are NOT used.

Mixed compounds use weighted-average features. This is a stated simplification; mention it in the notes.

### Known data issues (handle and document, do not hide)
- 41 of 244 rows have identical HSErel_SOC and HSE-PBE-SOC values, concentrated in Ca/Sr/Ba compounds. Cause unknown.
  Group 14 is not affected. Keep an `equal_flag` column and report results with and without those rows where relevant.
- HSErel anomaly: MAGeBr3 gap is larger than MAGeCl3 (against the halide trend).
- Some outlier points near PBE gap of about 3 to 3.5 eV and 4.3 eV. Identify them, do not silently drop.
- FAPbI3 and FAPbCl3 have no HSE-PBE-SOC gap.
- Different levels have different coverage. When comparing levels, always compare on the SAME compounds
  (drop rows missing in either level).

## 4. Features

Radii used (angstrom, taken from the dataset by exact recovery, std 0):
A: K 1.51, Rb 1.61, Cs 1.74, MA 2.17, FA 2.53. B: Ge 0.87, Ca 1.00, Sn 1.15, Pb 1.19, Sr 1.26, Ba 1.42.
X: Cl 1.81, Br 1.96, I 2.20. Z_B: Ge 32, Sn 50, Pb 82, Ca 20, Sr 38, Ba 56. Z_X: Cl 17, Br 35, I 53.
Literature radii sets conflict, so state the set used in the notes. Put the radii in ONE config dictionary so a
second radii set can be swapped in later for the robustness test.

For mixed compounds use composition-weighted averages (A, B, Z_B weighted by site fractions; X radius and Z_X
averaged with halide fraction).

- rA, rB, rX, Z_B, Z_X
- octahedral factor mu = rB / rX
- Goldschmidt tolerance factor t = (rA + rX) / (sqrt(2) * (rB + rX)). Must reproduce the dataset's tolerance and
  octahedral columns to about 1e-12.
- Bartel tau = rX/rB - nA * (nA - (rA/rB) / ln(rA/rB)), with nA = oxidation state of A (= 1). The formula was
  written from memory in an earlier session. Check it against the original Bartel et al. 2019 paper and state the
  result in the notes. The dataset's own Bartel column holds only the term (rA/rB)/ln(rA/rB): do not use it.
- Electronic descriptors from `pymatgen` (do not type values from memory): electronegativity of B and X
  (`Element(el).X`), B ionization energy IE_B = IE1 + IE2 (`Element(el).ionization_energies[:2]`),
  EN_B, EN_X, IE_B, dEN = EN_X - EN_B (weighted for mixed compounds).
- g2_frac = fraction of Ca+Sr+Ba on the B site (0 = only Group 14, 1 = only Group 2, in between = mixed B site).
  g14_flag = 1 - g2_frac. USE g2_frac TO DEFINE GROUPS. Do not define groups by the largest B element (`idxmax`):
  that put Group 2 atoms inside the "Group 14" set in an earlier attempt.
- Composition fractions for linear models: A (K, Rb, Ma, Fa; Cs is the reference), halide (Br, I; Cl is the
  reference), B (Ge, Sn; Pb is the reference).

## 5. Lessons already learned (do not repeat these mistakes)

1. Random splits inflate scores strongly. Leave-one-element-out with a CLEAN split is needed: test rows are
   compounds whose B site is ONLY the held-out element (`tbl[b] == 1`); training rows have ZERO trace of it
   (`tbl[b] == 0`). Mixed compounds with a trace of the held-out element in training leak information.
2. Compounds with B_main in Ge/Sn/Pb can still contain Ca/Sr/Ba (mixed B site) with gaps up to about 4.7 eV.
   An earlier "Group 14" analysis built from `idxmax` labels was contaminated and is RETIRED.
3. RandomForest cannot extrapolate outside the training range. Ridge can, but fails on non-linear trends
   (for example SOC against Z) and when a held-out element lies outside the training range of a feature.
4. In the held-out-element test, Ge/Sn/Pb siblings of the held-out element stay in training, so electronic
   features can simply act as a Group 14 versus Group 2 label. Test this explicitly (stage 5).
5. A flexible model with a random split can score R2 0.92 while the grouped score is 0.76. Choosing a
   hyperparameter on the same grouped CV that is reported makes the result slightly optimistic: say so, or use
   nested CV.
6. Deviation of cubicity and "organic A-site" are almost the same variable in this dataset (inorganic compounds
   have deviation about 0). Do not claim distortion "mediates" the A-site effect; at best they are associated.
7. Variable-name clashes cost time in the earlier notebook (a features table overwrote the halide list). Use
   distinct, descriptive names.

## 6. Pipeline (build in this order)

### Stage 0: get the data, setup and checks
First make sure `data/perovs_data_final.xlsx` exists (section 3, "Getting the data"). Install `pymatgen`, `scikit-learn`, `pandas`, `openpyxl`, `matplotlib`, `numpy`, `scipy`. Put paths in one config
cell (`DATA_PATH`, `FIG_DIR`). Load, merge, parse, assert all facts in section 3. Reproduce dataset t and mu.
Save `master_table.csv`, `features.csv`.

### Stage 1: descriptive physics
Gap histograms per level; PBE versus HSE+SOC scatter coloured by B (show the PBE-versus-hybrid error
cancellation: for Pb the HSE+SOC gap falls below PBE); hybrid shift (HSE-PBE-SOC minus PBE) and SOC shift
(HSErel_SOC minus HSErel) by B element; SOC shift versus Z_B for Ge/Sn/Pb with a mean line (include the
`equal_flag` check); a 3x3 table and heatmap (Ge/Sn/Pb by Cl/Br/I) of mean and std of the SOC shift on pure
compounds, plus a second version showing how the SOC shift depends on the A-site (Cs/K/Rb versus MA/FA).

### Stage 2: halide trends and mixing
Mean gap of pure compounds against halide (Cl, Br, I) for each B, PBE and HSE+SOC side by side, on the same
compounds. Mixed-halide compounds: gap against halide fraction, and the deviation from linear interpolation
(bowing) where data allow. Report counts per cell.

### Stage 3: variance decomposition
Fraction of gap variance explained by one label at a time (A, X, B element, B group via g2_frac), then the
same inside strict Group 14 (g2_frac == 0) and strict Group 2 (g2_frac == 1), plus an additive-model R2.
State that the one-way fractions overlap and are not additive, and that the dataset is not balanced.

### Stage 4: interpretable composition models in strict Group 14
Strict Group 14 means g2_frac == 0 (HSE-PBE-SOC target, about 76 compounds, about 43 groups where a group is
main A + main B + main X). Linear model on composition fractions with GroupKFold (5 folds) and random KFold side
by side; Ridge with polynomial products at several alpha values; bootstrap (500 resamples) 95 percent intervals
for linear coefficients, plotted in eV against the reference CsPbCl3-like compound. Add the deviation-of-cubicity
columns and an "organic fraction" (Ma + Fa) and compare grouped R2 for: B+X only, + organic, + deviation,
+ both. Plot residual (after removing B and X effects) against each deviation column, coloured by A-site type.

### Stage 5: honest generalization (the core result)
Target: HSE-PBE-SOC gap. Models: mean baseline, RandomForest (200 trees), Ridge (alpha 1, standardized).
Feature sets: geometry only (rA, rB, rX, octahedral, tolerance); geometry + Z (adds Z_B, Z_X);
geometry + electronic (adds EN_B, EN_X, IE_B, dEN).
(a) Clean leave-one-B-element-out over Ge, Sn, Pb, Ca, Sr, Ba (rule in section 5, item 1). Report MAE per
element and the average, with train and test sizes.
(b) Ablation: geometry only, + g14_flag only, + EN_B only, + IE_B only, + EN_X only, + all electronic.
(c) Leave-one-GROUP-out: train on strict Group 14, test on strict Group 2, and the reverse. Report MAE and the mean
signed error. This is a genuine extrapolation test; report it even if the result is bad.

### Stage 6: cross-fidelity
How well do PBE gap and simple features predict the hybrid gaps? Level-to-level differences for the pure compounds
Pb/Sn with I/Br, A = Cs/MA/FA (the organic minus Cs offset at PBE, HSE-PBE-SOC and HSErel_SOC). Check
consistency: SOC shift sign and magnitude increase with Z_B in Group 14; gap decreases Cl to I.

### Stage 7: comparison with experiment
Read `experimental/experimental_verified.csv` (columns: Formula, exp_gap, exp_reference, exp_phase_and_temperature)
if it exists. The student fills it from primary papers. Compute error (dataset minus experiment) for PBE,
HSErel_SOC and HSE-PBE-SOC: n, MAE, RMSE, mean error, parity plots coloured by A type (Cs versus organic), and
error versus halide. If the file is missing or rows are empty, build `experimental_template.csv` with the dataset
gaps filled and the experimental columns empty, and run the stage with the DRAFT values in section 9 item 10 ONLY
if every figure is stamped "DRAFT, unverified experimental values" in the title.

### Stage 8: robustness
(1) A second radii set through the config dictionary (the student will supply one; if none is given, skip and say
so). (2) Other targets: PBE gap and HSErel_SOC. (3) Several random seeds and bootstrap ranges for Stage 5.
(4) Results with and without the `equal_flag` rows.

### Stage 9: optional
Only if all earlier stages are done: a compact symbolic-regression formula (PySR or similar) for the strict Group 14
gap, validated with the same grouped splits, compared with the linear model.

### Stage 10: export
See section 10.

## 7. Figures (file names)

`fig01_gap_hist`, `fig02_pbe_vs_hybrid`, `fig03_shifts_by_B`, `fig04_soc_vs_ZB`, `fig05_soc_grid`,
`fig06_halide_trend`, `fig07_gap_vs_geometry`, `fig08_corr_heatmap`, `fig09_variance_explained`,
`fig10_composition_parity`, `fig11_coefficients`, `fig12_random_vs_grouped`, `fig13_deviation_residuals`,
`fig14_organic_vs_deviation`, `fig15_clean_leave_one_B_out`, `fig16_ablation`, `fig17_leave_one_group_out`,
`fig18_cross_fidelity_offset`, `fig19_vs_experiment`, `fig20_bowing`, `fig21_robustness`.
Combine into multi-panel figures if two or three belong together, but keep these base names plus `_panel` suffix.

## 8. Environment and deliverable format

- Notebook runs top to bottom with `Runtime > Run all` on Colab (pip installs in the first cell) and locally with
  `jupyter nbconvert --execute`. Fixed seeds. No hidden state between cells.
- Each stage starts with a markdown cell: what question it answers, in two plain sentences.
- Repeated code goes into small functions defined once near the top.

## 9. Results from the earlier exploratory session (re-derive and compare; treat with caution)

Status labels are the earlier ones; re-label them after your own run.

1. SOC shift (HSErel_SOC minus HSErel), mean over pure compounds: Ge -0.10, Sn -0.24, Pb -0.96 eV (15 each).
   Group 2 about -0.06 to -0.13, no clear Z trend. SOC shift also depends on the A site: PbI3 Cs -1.15, K -1.16,
   Rb -1.16, MA -0.67, FA -0.63; SnI3 Cs -0.36, MA -0.10, FA -0.19. (CONFIRMED on the pure compounds shown.)
2. Halide trend, mean over A, pure compounds, Cl/Br/I. PBE: Pb 2.34/1.87/1.64, Sn 1.37/0.87/0.64, Ge 1.59/1.16/0.91,
   Ca 5.01/4.20/3.49, Sr 4.76/4.04/3.43, Ba 4.53/3.91/3.36. HSE+SOC: Pb 2.04/1.55/1.07, Sn 1.50/1.21/0.81,
   Ge 2.27/1.73/1.28, Ca 6.37/5.32/4.29, Sr 6.06/5.16/4.23, Ba 5.78/5.01/4.20. Gap falls Cl to I for every B.
   Caveat: PBE and HSE+SOC averages may be over different compound sets. In HSE+SOC the Group 14 order is
   Ge > Pb > Sn, in PBE it is Pb > Ge > Sn. (Needs same-compound recheck.)
3. Variance explained, HSE-PBE-SOC, all 244: A 0.01, X 0.12, B element 0.66, B group 0.63. (CONTAMINATED:
   the group was defined by idxmax. Redo with g2_frac. Also the "A-site is only 1 percent" claim must not be
   used.)
4. Strict Group 14 (g2_frac == 0; n = 76, 43 groups), HSE-PBE-SOC: linear fractions grouped CV R2 0.74 (random 0.74);
   products Ridge: alpha 0.1: 0.92 random, 0.76 grouped; alpha 1: 0.91 / 0.80; alpha 10: 0.86 / 0.82;
   alpha 100: 0.66 / 0.65. (HYPOTHESIS-level: small n.) Linear coefficients (eV, reference CsPbCl3-like,
   intercept 1.78): K -0.11, Rb -0.11, Ma +0.82, Fa +1.19, Br -0.46, I -0.96, Ge +0.22, Sn -0.32;
   95 percent intervals: K and Rb include zero; Ge is barely above zero.
5. With deviation-of-cubicity columns added (rows with all deviation values present): grouped R2 0.73 to 0.82,
   Ma 0.82 to 0.25, Fa 1.19 to -0.37, Ge +0.24 to -0.15 (sign flips = collinearity). On strict Group 14 with grouped
   CV (values read from a bar chart): B+X only about 0.15, + organic about 0.76, + deviation about 0.72, both
   about 0.78. Residual-versus-deviation plots: at deviation 0, MA/FA points sit above inorganic ones. (HYPOTHESIS:
   association only.)
6. Organic (MA/FA) minus Cs gap, pure compounds: PBE +0.2 to +0.6, HSE-PBE-SOC +0.3 to +0.7,
   HSErel_SOC +1.0 to +1.9 eV. HSErel_SOC for MASnI3 is 2.47 and FASnI3 2.67 eV.
   Dataset values (HSE-PBE-SOC): CsPbI3 0.94, MAPbI3 1.53, CsPbBr3 1.42, MAPbBr3 1.72, FAPbBr3 1.87,
   CsSnI3 0.61, MASnI3 1.14, FASnI3 1.32, CsGeI3 0.93, MAGeI3 1.80, FAGeI3 1.98, MAPbCl3 2.42, MASnBr3 1.69.
7. Clean leave-one-B-out, HSE-PBE-SOC target, average MAE (eV). Mean baseline 2.06. Geometry only: RF 1.85,
   Ridge 2.13. Geometry + Z: RF 1.95, Ridge 1.73. Geometry + electronic: RF 0.45, Ridge 0.70.
   RF per element (geometry + electronic): Ba 0.19, Sr 0.16, Ca 0.49, Pb 0.51, Ge 0.62, Sn 0.71; Ridge fails on Pb
   (1.63). Test sizes about 19 to 25 per element, training 154 to 172. (CONFIRMED as a result; interpretation
   as group label is a HYPOTHESIS until stage 5b and 5c are run.) An older version of this test that used B_main
   labels gave RF 1.83 and Ridge 1.41 average MAE: RETIRED.
8. Experimental values mentioned in an earlier session, APPROXIMATE, from review-type sources, NOT verified:
   Pb iodides about 1.6, Pb bromides about 2.2 to 2.3, MAPbCl3 about 2.9 to 3.1, FAPbCl3 about 3.0,
   MASnI3 1.21, MASnBr3 about 2.2, CsGeI3 1.63, MAGeI3 2.00, FAGeI3 2.35 eV. Observed pattern:
   dataset minus experiment about -0.7 to -0.8 for Cs compounds, about -0.1 for MA iodides, about -0.5 to -0.7
   for MA bromides and chlorides. (HYPOTHESIS: forced-cubic small A cations give too small gaps. Not tested.)
9. The source paper reports about 0.70 eV RMSE for HSE-PBE-SOC versus experiment and HSErel as the worst level.
   Verify against its main Table 2 before quoting.

## 10. Deliverables (all in the project folder)

- `btp_perovskite.ipynb`: executed, with outputs.
- `figures/*.png` at 300 dpi (names in section 7), plus `figure_notes.md`: for each figure, 3 to 5 plain English
  lines: what is plotted, what to look for, one neutral observation sentence. No interpretation beyond the data.
- `tables/*.csv` for every table behind a figure.
- `numbers.json`: every number you report, with the cell or stage that produced it.
- `FINDINGS.md`: each finding with label (CONFIRMED / HYPOTHESIS / RETIRED), the numbers, the comparison with
  section 9, and a list of discrepancies, open questions and things you could not do.
- `glossary.md`: plain-language meaning of HSE, PBE, SOC, tolerance factor, octahedral factor, lone pair, grouped
  cross-validation, leakage, MAE, R2, bootstrap. Two or three sentences each.
- `README.md`: how to run, environment, run time.
- `experimental_template.csv` (see stage 7).

## 11. Definition of done

All stages 0 to 8 run top to bottom without errors; all assertions in stage 0 pass; every figure exists;
`FINDINGS.md` labels every finding; discrepancies against section 9 are listed; no paper prose, no invented
numbers or references anywhere.
