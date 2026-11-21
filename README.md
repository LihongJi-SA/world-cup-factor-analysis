# FIFA World Cup 2026 — In-Match Performance Analysis

This repository holds the 104 FIFA World Cup 2026 Post-Match Summary Reports (PMSR), the statistics extracted from them into per-team-per-match tables, and XGBoost models that relate **in-match** team behaviour to match outcome.

Every feature is measured during the match. The models describe and explain performance; they are **not pre-match forecasters**.

## Repository layout

| Path | Contents |
|---|---|
| `data/group-stage/` | 72 group-stage PMSR PDFs (groups A–L) plus `manifest.csv` (source URL, score, bytes, SHA-256) |
| `data/knock-out/` | 32 knockout PMSR PDFs (round of 32 → final) plus `manifest.csv` |
| `data/` | Extracted feature table `match_features.csv`, the shuffled modelling matrix `data_all.csv`, schema inventory |
| `docs/en/`, `docs/zh/` | English and Chinese documentation (dictionaries, schema, conclusions) |
| `docs/figs/` | Figures referenced by the feature dictionaries |
| `model/train.ipynb` | Three-class classification model with SHAP analysis |

The PDFs are git-ignored; only the manifests are committed. The manifests preserve each report's provenance and checksums.

## Data

- **104 matches**: 72 group stage + 32 knockout; 48 nations.
- **One row per team per match**, home team first: **208 rows**.
- **135 columns**: `nation`, `match_no`, `stage`, **131 in-match features**, `win`.
- `win` is `1` win, `0.5` draw (level after 120 minutes, including matches decided on penalties), `0` loss.
- Percentages are shares in 0–1; line-height/block values are normalised to pitch dimensions (105 m × 68 m); physical distances are kilometres; durations are seconds.

The authoritative descriptions are [`docs/en/feature-dictionary.md`](docs/en/feature-dictionary.md) (columns and units) and [`docs/en/report-data-schema.md`](docs/en/report-data-schema.md) (extraction, page numbers, de-duplication).

## Models

`model/train.ipynb` trains an XGBoost classifier on an 85/15 stratified split (`test_size=0.15`, `random_state=42`) and explains it with `shap.TreeExplainer`:

| Target | Configuration | Single-split result |
|---|---|---|
| lose=0, draw=1, win=2 | 500 trees, depth 5, lr 0.04, `multi:softprob` | accuracy 0.810 |

## Cross-validation results

5-fold `StratifiedKFold` (`random_state=42`) on `data/match_features.csv`:

| Metric | Mean ± SD | Fold range |
|---|---|---|
| Accuracy | 0.779 ± 0.069 | 0.690–0.857 |
| Macro F1 | 0.747 ± 0.070 | 0.656–0.821 |

- **Classification is reasonably stable but variable.** The single-split accuracy of 0.810 sits about half a standard deviation above the CV mean. Draws are the main source of instability: one fold drops to 0.690, and draw F1 is weaker than the win/lose F1 scores.
- **The 18 added line-break distribution features are neutral.** Repeating CV on the original 113-feature table gave essentially identical means (accuracy 0.779, macro F1 0.747).
- **Caveats.** The two rows of a match are correlated, and the CV mirrors the notebook's row-level split; match-level generalisation needs `GroupKFold` on `match_no`. With 208 rows each test fold holds only ~42 rows, so a few accuracy points are noise. Treat the result as explanatory, not as a high-confidence predictor.

Full details: [`docs/en/cross-validation-5fold-conclusion.md`](docs/en/cross-validation-5fold-conclusion.md).

## SHAP importance stability

Across the same five folds:

- **Stable head.** `attempts_goals`, `gk_save_pct` and `xg` are in the top 3 in **every** fold, holding 31% of total mean |SHAP|. `gk_attempts_faced`, `total_passes` and `crosses_outswing` are the next most reproducible names.
- **Unstable body.** Mean pairwise Spearman between folds is only 0.58; 79% of features move more than 30 rank places. Correlated substitutes (`pressures_total`/`pressures_direct`, the `duels_*` block, block-geometry features) swap credit from fold to fold.
- **Reading.** The *set* of important features is roughly reproducible; the *ordering* within it is not. Report only the top 3–5, or a thresholded set that clears a mean-|SHAP| cut in all five folds — not a ranked list beyond ~10.
- **Circularity.** `attempts_goals` *is* the goals outcome on the Attempts at Goal page, so its dominance is near-circular. The substantive stable signal is shot quality (`xg`), goalkeeping (`gk_save_pct`, `gk_attempts_faced`) and ball circulation (`total_passes`, `crosses_outswing`).

Full details: [`docs/en/shap-stability-conclusion.md`](docs/en/shap-stability-conclusion.md).

## Per-class feature impact

The three-class model was analysed per class (5-fold, mean |SHAP| on held-out rows):

| Class | Leading features | Share of total SHAP (top 3 / top 10) | Document |
|---|---|---|---|
| **Win** | `attempts_goals`, `xg`, `gk_attempts_faced`, `gk_save_pct`, `crosses_outswing` | 44.4% / 65.4% | [`feature-impact-win.md`](docs/en/feature-impact-win.md) |
| **Draw** | `gk_save_pct`, `physical_zone5_25_plus_km`, `duels_aerial`, `xg` | 16.9% / 36.9% | [`feature-impact-draw.md`](docs/en/feature-impact-draw.md) |
| **Lose** | `gk_save_pct`, `xg`, `attempts_goals`, `pressures_direct`, `ball_recovery_time_s` | 36.0% / 56.0% | [`feature-impact-lose.md`](docs/en/feature-impact-lose.md) |

- SHAP directions describe marginal push **within this model**, not raw correlations.
- Draw is the least concentrated and least stable class, consistent with its weaker classification scores.
- Correlated feature groups share their contribution; a group member missing from the top ranks does not make the behaviour irrelevant.
- Rankings below roughly the top 10 are single-fold artefacts and should not be read one by one.

Chinese versions of the dictionaries and class impact documents are in [`docs/zh/`](docs/zh/).