# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

A data-analysis repository, not an application: 104 FIFA World Cup 2026 Post-Match Summary Reports (PMSR) as PDFs, the same statistics extracted into per-team-per-match CSVs, and an XGBoost notebook that classifies match outcome. There is no build system, test suite, linter, or package manifest.

All features are **in-match performance metrics**, not pre-match information. The model explains/associates in-match team behaviour with results; it is not a pre-match forecaster.

## Commands

```bash
# Dependencies (already installed in E:\miniconda)
python -m pip install pandas scikit-learn xgboost shap matplotlib

# Run the notebook interactively:
#   open model/train.ipynb in VS Code / a Jupyter UI and run all cells.
# Headless execution (nbconvert is not installed by default):
python -m pip install nbconvert
cd model && python -m nbconvert --to notebook --execute train.ipynb --inplace
```

**The notebook must run with `model/` as the working directory**: it loads `../data/data_all.csv`.

Quick data checks:

```bash
python -c "import pandas as pd; df=pd.read_csv('data/match_features.csv'); print(df.shape); print(df['stage'].value_counts())"
python -c "import pandas as pd; print(pd.read_csv('data/data_all.csv').columns.tolist())"
```

## Data pipeline

```
data/group-stage/group-a..l/*.pdf              72 group matches
data/knock-out/{round-of-32,...,final}/*.pdf   32 knockout matches
        │  manifest.csv: score, source_url, bytes, sha256
        ▼
data/match_features.csv    208 rows × 135 cols (tracked in git)
                           nation, match_no, stage, 131 features, win
        │  drop the 3 identity columns (notebook cell 1)
        ▼
data/data_all.csv          208 rows × 132 cols (131 features + win)
                           ← loaded by model/train.ipynb; git-ignored
```

- **`data/match_features.csv` is the authoritative table.** `data_all.csv`, `data_group_stage.csv` and `data_knock_out.csv` are generated from it and are git-ignored; recreate them by uncommenting the `to_csv` lines in notebook cell 1. They are not currently shuffled.
- There is no PDF-extraction code in the repo. `docs/en/report-data-schema.md` documents the extraction (source page numbers, column order, de-duplication), and `data/schema_page_inventory.csv` records each PDF's page count plus the variable Attempts-at-Goal block length that shifts later pages.
- `docs/en/feature-dictionary.md` is the authoritative description of the 131 feature columns and their units; `docs/zh/feature-dictionary-zh.md` is the Chinese counterpart.
- `data/group-stage/manifest.csv` (72 matches) and `data/knock-out/manifest.csv` (32 matches) hold source URLs, scores, byte sizes, and SHA-256 checksums. The PDFs themselves are git-ignored.

## Notebook structure (`model/train.ipynb`)

One data-loading cell followed by a single classification section:

- Target: `win` mapped to `outcome` as `{0 → 0 lose, 0.5 → 1 draw, 1 → 2 win}`.
- Model: `XGBClassifier` (500 trees, depth 4, lr 0.04, `subsample`/`colsample_bytree` 0.8, `min_child_weight` 3, `multi:softprob`).
- Split: 85/15 stratified, `test_size=0.15`, `random_state=42` (cached run: 176 train / 32 test rows).
- Evaluation: accuracy, classification report, confusion matrix.
- SHAP: `shap.TreeExplainer` on the test rows `(32, 131, 3)`, one beeswarm per class (index 0 = lose, 1 = draw, 2 = win), then a mean-|SHAP| table across the three classes.

Cached outputs are from one split (accuracy 0.75, macro F1 0.68) and may be stale if the CSVs or notebook are regenerated. The 5-fold CV estimate is accuracy 0.779 ± 0.069 and macro F1 0.747 ± 0.070.

## Findings worth knowing

- Cross-validated classification is reasonably stable but variable; draw is the weakest and least stable class.
- SHAP rankings: `attempts_goals`, `gk_save_pct`, `xg` are top 3 in every fold (31% of total mean |SHAP|), but the ordering below the head is not reproducible across folds — report only the head or a thresholded set, not ranks beyond ~10.
- `attempts_goals` is near-circular (it *is* the goals count on the Attempts at Goal page), so treat its dominance with care.
- Details: `docs/en/cross-validation-5fold-conclusion.md`, `docs/en/shap-stability-conclusion.md`, and the per-class `docs/en/feature-impact-{win,draw,lose}.md`.

## Data conventions and pitfalls

- **One row per team per match**, home team first, two rows per `match_no`; 208 rows = 104 matches, 48 nations. There is no match ID beyond `match_no` (parsed from the PDF filename: `M01` → 1).
- `win` is `1` win, `0.5` draw (level after 120 minutes, including matches decided on penalties), `0` loss.
- Percentages (`possession_pct`, `pass_completion_pct`, `gk_save_pct`) are shares in 0–1. Line-height/block values are normalised to 0–1 by pitch dimensions (105 m length, 68 m width). Physical distances are kilometres; durations seconds.
- `stage` is `group stage` or `knock out`: 144 rows (72 matches) and 64 rows (32 matches).
- Semantics worth remembering from the schema docs: `attempts_goals` can be below the cover score (own goals are not logged as attempts); `duels_total` is the donut's "Duels" segment, not physical + aerial; `crosses_total` includes set plays while `crosses_open_play_attempted` does not; possession excludes the contested share printed between the two teams.
- The two rows of a match are correlated, and the random train/test split does not keep them together. Any evaluation of genuine predictive power should split by match, group, or stage rather than by row (`GroupKFold` on `match_no`).

## Documentation layout

`docs/en/` holds the English dictionary, extraction schema, CV/SHAP conclusions and per-class impact analyses; `docs/zh/` holds the Chinese dictionary and impact analyses; `docs/figs/` holds the figures, referenced from subfolders as `../figs/<name>.png`.

## Git

Tracked content: README, CLAUDE.md, docs, notebook, `data/match_features.csv`, `data/schema_page_inventory.csv`, and the two manifests. `.gitignore` excludes the 104 PDFs (573 MB) and the three generated modelling CSVs. Remote: `origin` → `https://github.com/LihongJi-SA/world-cup-factor-analysis.git`.
