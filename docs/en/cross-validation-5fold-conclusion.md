# Cross-Validation Conclusion

## Setup

- **Data:** `data/match_features.csv` — 208 team-rows (104 matches × 2 teams), 131 features.
- **Splitter:** `StratifiedKFold(n_splits=5, shuffle=True, random_state=42)`, stratified on the 3-class outcome.
- **Model:** the same XGBoost classifier configuration used in `train.ipynb`.
- **Note:** the CV uses `match_features.csv` (131 features, including the 18 new line-break distribution features). The single-split section in the notebook still uses the 113-feature `data_all.csv`, so comparisons with the holdout scores are indicative rather than exact.

## Classification

Target: `win` mapped to `2` win, `1` draw, `0` loss.

| Fold | Accuracy | Macro F1 |
|---:|---:|---:|
| 1 | 0.8571 | 0.8211 |
| 2 | 0.7857 | 0.7726 |
| 3 | 0.6905 | 0.6563 |
| 4 | 0.8293 | 0.7951 |
| 5 | 0.7317 | 0.6919 |
| **Mean ± SD** | **0.779 ± 0.069** | **0.747 ± 0.070** |
| **Range** | 0.690–0.857 | 0.656–0.821 |

**Verdict: reasonably stable, with moderate fold variance.** The single-split accuracy of 0.810 is inside the CV range but about half a standard deviation above the mean, so that split was mildly optimistic. One fold dropping to 0.690, together with the weaker holdout F1 for draws (0.737 vs 0.848 for losses and 0.813 for wins), suggests that draw classification is the main source of instability.

## Feature-set check

Repeating the same cross-validation on the original 113-feature `data_all.csv` produced essentially identical means: accuracy 0.779, macro F1 0.747.

The 18 newly added line-break distribution features therefore neither improve nor destabilise performance on this dataset. They are functionally neutral under the current models.

## Caveats and recommendation

- **Rows are not independent.** Each match contributes two correlated team-rows. The CV above mirrors the notebook's row-level split; for match-level generalisation, use `GroupKFold` grouped by `match_no`.
- **The dataset is small.** With 208 rows, each fold has only about 42 test rows, so a few percent of accuracy difference is within noise. Report the CV mean ± SD rather than relying on one split.
- **Overall:** classification performance is acceptable but variable. The model is suitable for explanatory analysis, not as a high-confidence pre-match predictor.
