# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

A data-analysis repository, not an application: 104 FIFA World Cup 2026 Post-Match Summary Reports (PMSR) as PDFs, the same statistics extracted into per-team-per-match CSVs, and an XGBoost notebook that models match outcome. There is no build system, test suite, linter, package manifest, or git repository.

All features are **in-match performance metrics**, not pre-match information. The models explain/associate in-match team behaviour with results; they are not pre-match forecasters.

## Commands

```bash
# Dependencies (already installed in E:\miniconda): pandas, scikit-learn, xgboost, shap, matplotlib
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
group-stage/group-a..l/*.pdf                  72 group matches
knock-out/{round-of-32,...,final}/*.pdf       32 knockout matches
        │  manifests.csv: score, source_url, bytes, sha256
        ▼
data/match_features.csv    208 rows × 117 cols
                           nation, match_no, stage, 113 features, win
        │  drop the 3 identity columns, shuffle rows
        ▼
data/data_all.csv          208 rows × 114 cols (113 features + win)
                           ← loaded by model/train.ipynb
```

- There is no PDF-extraction code in the repo. `data/report-data-schema.md` documents the extraction (source page numbers, column order, de-duplication), and `data/schema_page_inventory.csv` records each PDF's page count plus the variable Attempts-at-Goal block length that shifts later pages.
- `data/feature-dictionary.md` is the authoritative description of the 113 feature columns and their units.
- `group-stage/manifest.csv` (72 matches) and `knock-out/manifest.csv` (32 matches) hold source URLs, scores, byte sizes, and SHA-256 checksums.

### Do not trust the stage split files

`data/data_group_stage.csv` and `data/data_knock_out.csv` are **byte-identical to each other** (144 rows each). They are not a valid group/knockout split. Use `match_features.csv` (which has `stage`) or `data_all.csv`.

`data_all.csv` is also slightly stale relative to `match_features.csv`: only `crosses_lofted` differs, on 57 rows (`match_features.csv` has the corrected values, all ≤ `crosses_open_play_attempted`). The notebook currently trains on the stale `data_all.csv` values. Reconcile before regenerating the modelling matrix.

## Notebook structure (`model/train.ipynb`)

A data-loading cell followed by one classification section: target `outcome` mapped `{lose → 0, draw → 1, win → 2}`, `XGBClassifier` (500 trees, depth 5, lr 0.04, `multi:softprob`). Cached accuracy 0.8095.

It uses an 85/15 stratified split (`test_size=0.15`, `random_state=42`) and `shap.TreeExplainer`. Cached outputs may be stale if the CSVs are regenerated.

## Data conventions and pitfalls

- **One row per team per match**, home team first, two rows per `match_no`; 208 rows = 104 matches, 48 nations. There is no match ID beyond `match_no` (parsed from the PDF filename: `M01` → 1).
- `win` is `1` win, `0.5` draw (level after 120 minutes, including matches decided on penalties), `0` loss.
- Percentages (`possession_pct`, `pass_completion_pct`, `gk_save_pct`) are shares in 0–1. Line-height/block values are normalised to 0–1 by pitch dimensions (105 m length, 68 m width). Physical distances are kilometres; durations seconds.
- `stage` is `group stage` or `knock out`.
- Semantics worth remembering from the schema docs: `attempts_goals` can be below the cover score (own goals are not logged as attempts); `duels_total` is the donut's "Duels" segment, not physical + aerial; `crosses_total` includes set plays while `crosses_open_play_attempted` does not; possession excludes the contested share printed between the two teams.
- The two rows of a match are correlated, and the random train/test split does not keep them together. Any evaluation of genuine predictive power should split by match, group, or stage rather than by row.
