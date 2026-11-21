# Feature impact on "draw"

This document is the per-class SHAP analysis of the three-class model (lose = 0, draw = 1, win = 2). It only describes the model's attribution for the "draw" class; see the corresponding files for the other two classes.

## Analysis setup

- **Data**: `data/match_features.csv`, 208 team-match records, 131 features; 5-fold `StratifiedKFold` (`random_state=42`).
- **Numbering**: feature numbers follow `feature-dictionary.md` (#4–#134) and match the "#" column of the dictionary tables.
- **Model**: `XGBClassifier` (500 trees, max depth 5, learning rate 0.04, `multi:softprob`), matching the configuration in `model/train.ipynb`.
- **Importance**: mean |SHAP| for the "draw" class, computed on each fold's held-out set (about 42 records).
- **Direction**: Spearman correlation between the feature value and that class's SHAP value (pooled over all 208 held-out records). ↑ draw = higher values push toward "draw"; ↓ draw = higher values push away from "draw"; — = |ρ| < 0.15, no clear direction.
- **Stability**: core = mean rank ≤ 3 across the five folds; volatile = top 40 in at least 2 folds and outside the top 80 in at least 2 folds; single-fold extreme = top 40 in only 1 fold with mean rank > 40; stable = mean rank ≤ 20 and rank range ≤ 25; unstable = rank range ≥ 60; moderate = mean rank ≤ 40; otherwise low.
- **Class distribution**: 80 lose, 48 draw, 80 win (208 total).
- **Reference performance**: five-fold accuracy 0.779 ± 0.068, macro-averaged F1 0.747 ± 0.070.

## Key findings

- The top 3 features account for **16.9%** of total SHAP for "draw"; the top 10 account for **36.9%**.
- **2** features are "core", **10** are "volatile", and **25** rely on a single fold.
- SHAP values are conditional attributions: direction describes the marginal push within this model, not a raw correlation coefficient.

## Top 20 features (the 5 with "unstable" stability removed)

| Rank | # | Feature | Mean \|SHAP\| | Rank per fold | Direction | Stability |
|---:|---:|---|---:|---|---|---|
| 1 | 119 | `gk_save_pct` | 0.2928 | 1, 2, 2, 1, 2 | ↑ draw | core |
| 2 | 132 | `physical_zone5_25_plus_km` | 0.2645 | 4, 3, 4, 3, 1 | ↓ draw | core |
| 3 | 99 | `duels_aerial` | 0.2543 | 17, 1, 6, 2, 5 | ↑ draw | stable |
| 4 | 5 | `xg` | 0.1650 | 5, 7, 3, 9, 11 | ↓ draw | stable |
| 5 | 75 | `movement_final_third_in_between` | 0.1532 | 12, 12, 48, 4, 3 | ↑ draw | moderate |
| 6 | 131 | `physical_zone4_20_25_km` | 0.1489 | 31, 19, 1, 15, 6 | ↓ draw | moderate |
| 9 | 56 | `attempts_goals` | 0.1155 | 10, 26, 14, 6, 10 | ↓ draw | stable |
| 10 | 89 | `def_forced_turnovers` | 0.1113 | 23, 4, 15, 26, 7 | ↑ draw | stable |
| 11 | 90 | `def_possession_regained` | 0.0959 | 14, 14, 10, 41, 13 | — | moderate |
| 12 | 108 | `def_lh_low_block_length_norm` | 0.0841 | 7, 23, 30, 7, 39 | ↓ draw | moderate |
| 13 | 57 | `attempts_on_target` | 0.0793 | 11, 44, 11, 18, 36 | ↑ draw | moderate |
| 14 | 6 | `attempts_at_goal` | 0.0763 | 24, 5, 27, 51, 18 | ↑ draw | moderate |
| 15 | 98 | `duels_physical` | 0.0737 | 3, 31, 19, 39, 42 | ↑ draw | moderate |
| 17 | 85 | `movement_defensive_third_in_between` | 0.0708 | 8, 111, 13, 17, 83 | ↓ draw | volatile |
| 20 | 129 | `physical_zone2_7_15_km` | 0.0639 | 15, 33, 37, 21, 19 | ↓ draw | moderate |

## Full ranking (54 features, sorted by mean |SHAP|; features with "low" (40) and "unstable" (37) stability removed)

| Rank | # | Feature | Mean \|SHAP\| | Rank per fold | Direction | Stability |
|---:|---:|---|---:|---|---|---|
| 1 | 119 | `gk_save_pct` | 0.2928 | 1, 2, 2, 1, 2 | ↑ draw | core |
| 2 | 132 | `physical_zone5_25_plus_km` | 0.2645 | 4, 3, 4, 3, 1 | ↓ draw | core |
| 3 | 99 | `duels_aerial` | 0.2543 | 17, 1, 6, 2, 5 | ↑ draw | stable |
| 4 | 5 | `xg` | 0.1650 | 5, 7, 3, 9, 11 | ↓ draw | stable |
| 5 | 75 | `movement_final_third_in_between` | 0.1532 | 12, 12, 48, 4, 3 | ↑ draw | moderate |
| 6 | 131 | `physical_zone4_20_25_km` | 0.1489 | 31, 19, 1, 15, 6 | ↓ draw | moderate |
| 9 | 56 | `attempts_goals` | 0.1155 | 10, 26, 14, 6, 10 | ↓ draw | stable |
| 10 | 89 | `def_forced_turnovers` | 0.1113 | 23, 4, 15, 26, 7 | ↑ draw | stable |
| 11 | 90 | `def_possession_regained` | 0.0959 | 14, 14, 10, 41, 13 | — | moderate |
| 12 | 108 | `def_lh_low_block_length_norm` | 0.0841 | 7, 23, 30, 7, 39 | ↓ draw | moderate |
| 13 | 57 | `attempts_on_target` | 0.0793 | 11, 44, 11, 18, 36 | ↑ draw | moderate |
| 14 | 6 | `attempts_at_goal` | 0.0763 | 24, 5, 27, 51, 18 | ↑ draw | moderate |
| 15 | 98 | `duels_physical` | 0.0737 | 3, 31, 19, 39, 42 | ↑ draw | moderate |
| 17 | 85 | `movement_defensive_third_in_between` | 0.0708 | 8, 111, 13, 17, 83 | ↓ draw | volatile |
| 20 | 129 | `physical_zone2_7_15_km` | 0.0639 | 15, 33, 37, 21, 19 | ↓ draw | moderate |
| 21 | 134 | `physical_sprints` | 0.0606 | 27, 25, 24, 16, 28 | ↑ draw | moderate |
| 22 | 111 | `pressures_direct` | 0.0573 | 18, 40, 47, 47, 12 | ↓ draw | moderate |
| 23 | 19 | `ip_lh_buildup_mid_block_width_norm` | 0.0569 | 48, 58, 23, 29, 8 | ↓ draw | moderate |
| 27 | 130 | `physical_zone3_15_20_km` | 0.0550 | 19, 11, 51, 52, 37 | — | moderate |
| 31 | 46 | `lb_3units_defensive_attempted` | 0.0444 | 34, 6, 105, 76, 82 | — | volatile |
| 33 | 128 | `physical_zone1_0_7_km` | 0.0434 | 43, 39, 55, 27, 16 | ↑ draw | moderate |
| 40 | 105 | `def_lh_mid_block_length_norm` | 0.0365 | 16, 97, 86, 25, 72 | ↓ draw | volatile |
| 42 | 49 | `lb_3units_midfield_completed` | 0.0348 | 20, 103, 46, 23, 102 | ↓ draw | volatile |
| 43 | 65 | `crosses_lofted` | 0.0333 | 13, 65, 39, 104, 117 | ↓ draw | volatile |
| 46 | 59 | `attempts_blocked` | 0.0301 | 47, 21, 118, 72, 43 | ↑ draw | single-fold extreme |
| 47 | 94 | `blocks_passes` | 0.0294 | 114, 32, 20, 46, 87 | ↑ draw | volatile |
| 48 | 82 | `movement_middle_third_in_to_out` | 0.0291 | 113, 102, 17, 63, 26 | ↑ draw | volatile |
| 49 | 102 | `def_lh_high_block_length_norm` | 0.0289 | 125, 15, 50, 64, 104 | ↑ draw | single-fold extreme |
| 50 | 103 | `def_lh_high_block_width_norm` | 0.0277 | 100, 16, 65, 116, 52 | ↑ draw | single-fold extreme |
| 55 | 44 | `lb_4units_attacking_attempted` | 0.0246 | 72, 100, 112, 14, 67 | ↑ draw | single-fold extreme |
| 58 | 36 | `lb_2units_inside_shape` | 0.0236 | 66, 28, 67, 43, 103 | ↓ draw | single-fold extreme |
| 59 | 92 | `def_tackles` | 0.0236 | 28, 38, 62, 82, 91 | — | volatile |
| 60 | 117 | `pressing_direction_outside` | 0.0228 | 107, 60, 93, 99, 17 | ↑ draw | single-fold extreme |
| 61 | 16 | `ip_lh_buildup_low_block_width_norm` | 0.0220 | 37, 45, 49, 111, 50 | ↑ draw | single-fold extreme |
| 62 | 14 | `ip_lh_buildup_low_vertical_distance_norm` | 0.0220 | 77, 34, 45, 45, 105 | ↓ draw | single-fold extreme |
| 63 | 79 | `movement_middle_third_in_front` | 0.0212 | 26, 66, 84, 77, 76 | — | single-fold extreme |
| 64 | 45 | `lb_4units_attacking_completed` | 0.0205 | 22, 95, 88, 108, 66 | ↑ draw | single-fold extreme |
| 65 | 114 | `pushing_on_into_pressing` | 0.0205 | 125, 62, 41, 38, 55 | — | single-fold extreme |
| 66 | 18 | `ip_lh_buildup_mid_block_length_norm` | 0.0200 | 53, 81, 25, 125, 54 | ↑ draw | single-fold extreme |
| 67 | 76 | `movement_final_third_out_to_in` | 0.0196 | 109, 83, 28, 40, 86 | ↑ draw | volatile |
| 68 | 101 | `def_lh_high_block_vertical_distance_norm` | 0.0191 | 88, 59, 77, 96, 24 | ↓ draw | single-fold extreme |
| 69 | 68 | `offers_final_third` | 0.0191 | 106, 112, 94, 32, 33 | ↑ draw | volatile |
| 70 | 55 | `lb_2units_midfield_completed` | 0.0190 | 80, 57, 99, 37, 48 | ↑ draw | single-fold extreme |
| 71 | 83 | `movement_middle_third_in_behind` | 0.0185 | 52, 49, 36, 125, 71 | ↑ draw | single-fold extreme |
| 72 | 67 | `crosses_push_cross` | 0.0185 | 91, 53, 116, 30, 58 | ↓ draw | single-fold extreme |
| 74 | 33 | `lb_3units_inside_shape` | 0.0172 | 44, 93, 57, 94, 34 | ↓ draw | single-fold extreme |
| 76 | 21 | `ip_lh_final_third_block_length_norm` | 0.0168 | 33, 117, 74, 88, 56 | ↑ draw | single-fold extreme |
| 78 | 43 | `lb_4units_advanced_midfield_completed` | 0.0156 | 125, 110, 31, 49, 101 | ↓ draw | single-fold extreme |
| 79 | 30 | `lb_4units_inside_shape` | 0.0152 | 118, 127, 115, 75, 23 | ↑ draw | single-fold extreme |
| 80 | 26 | `lb_around_completed` | 0.0143 | 112, 77, 97, 61, 30 | ↓ draw | single-fold extreme |
| 83 | 35 | `lb_2units_attempted` | 0.0137 | 103, 85, 26, 118, 95 | ↓ draw | single-fold extreme |
| 86 | 96 | `blocks_crosses` | 0.0126 | 115, 75, 32, 81, 110 | ↓ draw | single-fold extreme |
| 88 | 66 | `crosses_cutback` | 0.0119 | 87, 88, 34, 125, 79 | ↑ draw | single-fold extreme |
| 105 | 62 | `crosses_inswing` | 0.0086 | 125, 121, 125, 125, 29 | ↑ draw | single-fold extreme |

## Notes

- `attempts_goals` is itself the goal outcome (see note 1 in `data/report-data-schema.md`), so its dominance is partly circular.
- Each fold's SHAP is based on only about 42 records; importance estimates combine sampling noise with model variation.
- Correlated feature groups (e.g. `pressures_total` / `pressures_direct`, `duels_physical` / `duels_aerial` / `duels_total`, block-geometry features) share their contribution with one another; a member falling outside the top ranks does not mean the corresponding behaviour is unimportant.
- Rankings beyond roughly the top 10 should not be read one by one; see `model/shap-stability-conclusion.md`.
