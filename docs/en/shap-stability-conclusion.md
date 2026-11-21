# SHAP Importance Stability Across CV Folds

Companion to `cross-validation-5fold-conclusion.md`. Answers: **do the SHAP feature-importance rankings agree between folds, or do they vary a lot?**

## Setup

- **Data:** `data/match_features.csv` — 208 team-rows, 131 features.
- **Splitter:** the same `StratifiedKFold(n_splits=5, shuffle=True, random_state=42)` used in the CV block.
- **Procedure:** for each fold, fit the model on the training rows, then run `shap.TreeExplainer` on that fold's **held-out** rows only. Importance = mean |SHAP| per feature across those rows, averaged over the three classes.
- **Comparison:** Spearman/Kendall correlation of the full importance vector between every pair of folds, per-feature rank spread, and top-k membership overlap.
- **Caveat:** each fold explains only ~42 rows, which itself adds sampling noise to the per-fold importance estimates.

## Stable head

| Rank | Feature | Mean \|SHAP\| | Ranks over folds 1–5 |
|---:|---|---:|---|
| 1 | `attempts_goals` | 0.582 | 1,1,2,1,1 |
| 2 | `gk_save_pct` | 0.527 | 2,2,1,2,2 |
| 3 | `xg` | 0.405 | 3,3,3,3,3 |

These three are in the top 3 in **every** fold and hold 31% of total mean |SHAP|. `gk_attempts_faced` (ranks 4–8), plus `total_passes` and `crosses_outswing`, are the next most reproducible names.

## Unstable body

| Measure | Value |
|---|---|
| Mean pairwise Spearman (range) | 0.579 (0.489–0.694) |
| Mean pairwise Kendall | 0.421 |
| Spearman with top 5 removed | 0.530 |
| Mean SD of a feature's rank (of 131) | 22.0 |
| Mean rank range across folds | 53 places |
| Features moving >30 rank places | 104/131 (79%) |
| Features moving >60 rank places | 48/131 (37%) |
| Top-5 mean Jaccard overlap | 0.61 |
| Top-10 mean Jaccard overlap | 0.50 |
| Top-20 mean Jaccard overlap | 0.44 |
| Features in the top 10 in all 5 folds | 4/10 |
| Features in the top 30 in all 5 folds | 11/30 |
| Features with mean \|SHAP\| > 0.01 in every fold | 42 |

### Genuinely unstable features (fold ranks 1–5)

**A single extreme rank is not evidence of instability.** Ranks `114, 104, 126, 45, 99` describe a feature that is consistently unimportant and happened to rank 45th once — its importance did not change. A feature is counted as unstable here only when **at least two folds place it in the top 40 and at least two folds place it outside the top 80**. A lone stray rank at either end is treated as an outlier and ignored.

By this criterion 10 features qualify. They are ranked by range (largest first). For comparison, 100 features move at least 35 rank places, but almost all of those swings rest on a single fold.

| Rank | Feature | Mean rank | Ranks 1–5 | Top-40 folds | Outside-80 folds | Range |
|---|---:|---|---:|---:|---:|---:|
| 1 | `lb_4units_advanced_midfield_attempted` | 67.6 | 26, 46, 124, 13, 129 | 2 | 2 | 116 |
| 2 | `set_plays_corner_outswing` | 60.6 | 91, 20, 122, 19, 51 | 2 | 2 | 103 |
| 3 | `movement_defensive_third_in_between` | 49.2 | 13, 84, 28, 17, 104 | 3 | 2 | 91 |
| 4 | `def_lh_mid_block_length_norm` | 66.4 | 24, 80, 109, 23, 96 | 2 | 2 | 86 |
| 5 | `lb_3units_defensive_attempted` | 63.0 | 63, 22, 104, 99, 27 | 2 | 2 | 82 |
| 6 | `def_lh_mid_block_vertical_distance_norm` | 73.2 | 36, 90, 112, 31, 97 | 2 | 3 | 81 |
| 7 | `offers_final_third` | 65.4 | 83, 116, 39, 52, 37 | 2 | 2 | 79 |
| 8 | `offers_received` | 65.4 | 93, 14, 90, 39, 91 | 2 | 3 | 79 |
| 9 | `lb_4units_attacking_completed` | 75.6 | 37, 113, 93, 96, 39 | 2 | 3 | 76 |
| 10 | `set_plays_free_kicks` | 65.2 | 86, 73, 37, 98, 32 | 2 | 2 | 66 |

Two patterns in this list are worth naming. `lb_4units_advanced_midfield_attempted` is the clearest case: rank 26 and 46 in two folds, 124, 13 and 129 in the others. And the block-length/geometry features (`def_lh_mid_block_length_norm`, `def_lh_mid_block_vertical_distance_norm`) recur, consistent with the collinear-block effect.

## Reading

Rank stability tracks importance magnitude. The head is stable because it is an order of magnitude larger than the tail. Below that, the model distributes nearly equal credit among collinear substitutes — `pressures_total`/`pressures_direct`, the `duels_*` block, the block-geometry features — and does so differently on each training subset.

So: **the set of important features is roughly reproducible; the ordering within it is not.** A feature's rank beyond ~10 should not be treated as a finding.

The top-ranked feature, `attempts_goals`, is near-circular: per `report-data-schema.md` note 1 it *is* the goals outcome on the Attempts at Goal page, not an independent behaviour metric. The substantive stable signal is shot quality (`xg`), goalkeeping (`gk_save_pct`, `gk_attempts_faced`), and ball circulation (`total_passes`, `crosses_outswing`).

## Recommendation

- Report only the head (top 3–5) as a stable finding, with its worst rank across folds.
- For anything below the head, present a thresholded set instead of a ranking: features with mean \|SHAP\| above a chosen cut in **all five** folds (42 features clear this cut for the three-class model), rather than "the top N".
- If a stable ordering is the goal, average SHAP importances over many resamples (or fit one model per fold on all training data and average SHAP over the union) instead of reading one split's ranking.
- These results are separate from, but consistent with, the performance conclusion: the models predict consistently, yet the attribution among correlated features is not identifiable at this sample size.

Script: `E:/tmp/cv_shap_stability.py` (detail pass: `E:/tmp/cv_shap_detail.py`).
