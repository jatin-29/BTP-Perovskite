# BTP-Perovskite

Computational part of a B.Tech project (DTU, Engineering Physics). The project studies how the band gap of ABX3
halide perovskites divides between geometric and electronic contributions, using small interpretable models and
honest (grouped) validation.

## Contents

| Path | What it is |
|---|---|
| `btp_perovskite.ipynb` | The whole analysis, stages 0 to 10 (and optional stage 9), saved with outputs |
| `data/perovs_data_final.xlsx` | Dataset (Yang, Manganaris, Mannodi-Kanakkithodi, Digital Discovery 2023, CC-BY), copied from `github.com/yjq829/perovskite_dataset` |
| `figures/` | All figures, 300 dpi PNG (`fig01` to `fig21`) |
| `tables/` | CSV table behind every figure, plus `master_table.csv` and `features.csv` |
| `numbers.json` | Every reported number, with the stage that produced it |
| `FINDINGS.md` | Findings labelled CONFIRMED / HYPOTHESIS / RETIRED, discrepancies, open questions |
| `figure_notes.md` | 3 to 5 plain lines per figure |
| `glossary.md` | Plain-language meaning of the technical terms |
| `experimental_template.csv` | Pure compounds with dataset gaps filled and experimental columns empty (to fill in) |
| `experimental/experimental_candidates.csv` | Candidate literature band gaps (18 compounds, with method, phase, reference, DOI, location); status "candidate, student must verify" |
| `data_issues_for_authors.md` | Neutral list of observed data issues that could be sent to the dataset authors |
| `Papers/` | Background papers (not used by the code) |

## How to run

**Locally**
```bash
pip install pymatgen scikit-learn pandas openpyxl matplotlib numpy scipy nbconvert ipykernel
```
```bash
jupyter nbconvert --to notebook --execute --inplace --ExecutePreprocessor.timeout=3000 btp_perovskite.ipynb
```
Run it from the project folder. The notebook uses the current folder as the project folder.

**Google Colab**: upload the notebook (or open it from GitHub) and use `Runtime > Run all`. The first code cell installs
any missing packages. If `data/perovs_data_final.xlsx` is not present, the notebook clones the dataset repository and
copies the file. If the clone fails, it stops with an error and never creates data. Files are written to the Colab
working folder (`/content`). Download `figures/`, `tables/` and `numbers.json` afterwards.

**Experimental comparison (stage 7)**: copy `experimental_template.csv` to `experimental/experimental_verified.csv`,
fill `exp_gap`, `exp_reference` and `exp_phase_and_temperature` from primary papers, and re-run. Until then, stage 7
uses DRAFT values: v1 (approximate values) and v2 (`experimental/experimental_candidates.csv`), each figure stamped as DRAFT.

**Radii sets (stage 8)**: `RADII_SETS` in the config cell holds the dataset radii and a Shannon CN12/CN6 set built from pymatgen
(sources in `tables/radii_sets_sources.csv`). A third set can be added the same way.

## Environment used for the saved run

- Python 3.13.18, numpy 2.5.3, pandas 3.0.5, matplotlib 3.11.2, scipy 1.18.1, scikit-learn 1.9.1, pymatgen 2026.10.31 (Linux container).
- Run time: about 3.5 minutes for the whole notebook (most of it is the RandomForest runs in stage 8). The exact value is `notebook_runtime_seconds` in `numbers.json`.
- Install note: `pip install pymatgen` first failed because the `bibtexparser` wheel could not be built with the
  system's old setuptools. `pip install --upgrade --ignore-installed setuptools` fixed it. No replacement package was needed.
- All random seeds are fixed (`random_state=0`).

## Data download log

- `git clone https://github.com/yjq829/perovskite_dataset.git` succeeded (repository commit b84ed4d, 2023-04-13).
  `perovs_data_final.xlsx` is in the repository root. sha256 of the copied file:
  `5ee941331477bb6038696c98e570846c3411b2e4b7216df5a4e6da0fabf7f862`.
