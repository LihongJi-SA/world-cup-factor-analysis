# Feature impact on "lose"

This document is the per-class SHAP analysis of the three-class model (lose = 0, draw = 1, win = 2). It only describes the model's attribution for the "lose" class; see the corresponding files for the other two classes.

## Analysis setup

- **Data**: `data/match_features.csv`, 208 team-match records, 131 features; 5-fold `StratifiedKFold` (`random_state=42`).
- **Numbering**: feature numbers follow `feature-dictionary.md` (#4–#134) and match the "#" column of the dictionary tables.
- **Model**: `XGBClassifier` (500 trees, max depth 5, learning rate 0.04, `multi:softprob`), matching the configuration in `model/train.ipynb`.
- **Importance**: mean |SHAP| for the "lose" class, computed on each fold's held-out set (about 42 records).
- **Direction**: Spearman correlation between the feature value and that class's SHAP value (pooled over all 208 held-out records). ↑ lose = higher values push toward "lose"; ↓ lose = higher values push away from "lose"; — = |ρ| < 0.15, no clear direction.
- **Stability**: core = mean rank ≤ 3 across the five folds; volatile = top 40 in at least 2 folds and outside the top 80 in at least 2 folds; single-fold extreme = top 40 in only 1 fold with mean rank > 40; stable = mean rank ≤ 20 and rank range ≤ 25; unstable = rank range ≥ 60; moderate = mean rank ≤ 40; otherwise low.
- **Class distribution**: 80 lose, 48 draw, 80 win (208 total).
- **Reference performance**: five-fold accuracy 0.779 ± 0.068, macro-averaged F1 0.747 ± 0.070.

## Key findings

- The top 3 features account for **36.0%** of total SHAP for "lose"; the top 10 account for **56.0%**.
- **3** features are "core", **5** are "volatile", and **27** rely on a single fold.
- SHAP values are conditional attributions: direction describes the marginal push within this model, not a raw correlation coefficient.

## Top 20 features (the 3 with "unstable" stability removed)

| Rank | # | Feature | Mean \|SHAP\| | Rank per fold | Direction | Stability |
|---:|---:|---|---:|---|---|---|
| 1 | 119 | `gk_save_pct` | 0.9892 | 1, 1, 1, 1, 1 | ↓ lose | core |
| 2 | 5 | `xg` | 0.5069 | 2, 2, 4, 2, 2 | ↓ lose | core |
| 3 | 56 | `attempts_goals` | 0.3282 | 4, 3, 2, 3, 3 | ↓ lose | core |
| 4 | 111 | `pressures_direct` | 0.1750 | 7, 9, 10, 4, 5 | ↑ lose | stable |
| 5 | 113 | `ball_recovery_time_s` | 0.1575 | 8, 15, 5, 10, 4 | ↑ lose | stable |
| 6 | 126 | `set_plays_throw_ins` | 0.1471 | 5, 7, 20, 8, 12 | ↑ lose | stable |
| 7 | 22 | `ip_lh_final_third_block_width_norm` | 0.1422 | 9, 4, 9, 12, 7 | ↑ lose | stable |
| 8 | 128 | `physical_zone1_0_7_km` | 0.1411 | 6, 8, 7, 7, 26 | ↓ lose | stable |
| 9 | 7 | `total_passes` | 0.1265 | 3, 6, 21, 15, 28 | ↓ lose | stable |
| 10 | 118 | `gk_attempts_faced` | 0.1252 | 29, 5, 12, 9, 8 | ↑ lose | stable |
| 11 | 110 | `pressures_total` | 0.1174 | 12, 16, 3, 45, 23 | ↑ lose | moderate |
| 12 | 92 | `def_tackles` | 0.0999 | 10, 30, 17, 6, 15 | ↑ lose | stable |
| 13 | 75 | `movement_final_third_in_between` | 0.0973 | 16, 12, 47, 5, 16 | ↓ lose | moderate |
| 14 | 4 | `possession_pct` | 0.0846 | 27, 34, 6, 21, 10 | ↓ lose | moderate |
| 15 | 51 | `lb_3units_attacking_completed` | 0.0677 | 19, 28, 24, 19, 11 | ↑ lose | moderate |
| 19 | 116 | `pressing_direction_inside` | 0.0490 | 14, 38, 45, 48, 13 | ↑ lose | moderate |
| 20 | 89 | `def_forced_turnovers` | 0.0469 | 24, 18, 28, 17, 35 | ↓ lose | moderate |

## Full ranking (57 features, sorted by mean |SHAP|; features with "low" (39) and "unstable" (35) stability removed)

| Rank | # | Feature | Mean \|SHAP\| | Rank per fold | Direction | Stability |
|---:|---:|---|---:|---|---|---|
| 1 | 119 | `gk_save_pct` | 0.9892 | 1, 1, 1, 1, 1 | ↓ lose | core |
| 2 | 5 | `xg` | 0.5069 | 2, 2, 4, 2, 2 | ↓ lose | core |
| 3 | 56 | `attempts_goals` | 0.3282 | 4, 3, 2, 3, 3 | ↓ lose | core |
| 4 | 111 | `pressures_direct` | 0.1750 | 7, 9, 10, 4, 5 | ↑ lose | stable |
| 5 | 113 | `ball_recovery_time_s` | 0.1575 | 8, 15, 5, 10, 4 | ↑ lose | stable |
| 6 | 126 | `set_plays_throw_ins` | 0.1471 | 5, 7, 20, 8, 12 | ↑ lose | stable |
| 7 | 22 | `ip_lh_final_third_block_width_norm` | 0.1422 | 9, 4, 9, 12, 7 | ↑ lose | stable |
| 8 | 128 | `physical_zone1_0_7_km` | 0.1411 | 6, 8, 7, 7, 26 | ↓ lose | stable |
| 9 | 7 | `total_passes` | 0.1265 | 3, 6, 21, 15, 28 | ↓ lose | stable |
| 10 | 118 | `gk_attempts_faced` | 0.1252 | 29, 5, 12, 9, 8 | ↑ lose | stable |
| 11 | 110 | `pressures_total` | 0.1174 | 12, 16, 3, 45, 23 | ↑ lose | moderate |
| 12 | 92 | `def_tackles` | 0.0999 | 10, 30, 17, 6, 15 | ↑ lose | stable |
| 13 | 75 | `movement_final_third_in_between` | 0.0973 | 16, 12, 47, 5, 16 | ↓ lose | moderate |
| 14 | 4 | `possession_pct` | 0.0846 | 27, 34, 6, 21, 10 | ↓ lose | moderate |
| 15 | 51 | `lb_3units_attacking_completed` | 0.0677 | 19, 28, 24, 19, 11 | ↑ lose | moderate |
| 19 | 116 | `pressing_direction_inside` | 0.0490 | 14, 38, 45, 48, 13 | ↑ lose | moderate |
| 20 | 89 | `def_forced_turnovers` | 0.0469 | 24, 18, 28, 17, 35 | ↓ lose | moderate |
| 21 | 83 | `movement_middle_third_in_behind` | 0.0437 | 13, 52, 33, 55, 19 | ↓ lose | moderate |
| 23 | 85 | `movement_defensive_third_in_between` | 0.0416 | 17, 41, 120, 11, 83 | ↑ lose | volatile |
| 24 | 63 | `crosses_outswing` | 0.0412 | 44, 27, 19, 28, 20 | ↑ lose | moderate |
| 25 | 42 | `lb_4units_advanced_midfield_attempted` | 0.0397 | 15, 78, 120, 14, 103 | ↑ lose | volatile |
| 28 | 117 | `pressing_direction_outside` | 0.0379 | 28, 13, 29, 34, 66 | ↑ lose | moderate |
| 29 | 8 | `pass_completion_pct` | 0.0370 | 74, 24, 16, 38, 27 | ↓ lose | moderate |
| 30 | 112 | `pressure_avg_duration_s` | 0.0358 | 32, 22, 18, 59, 30 | ↑ lose | moderate |
| 32 | 107 | `def_lh_low_block_vertical_distance_norm` | 0.0338 | 60, 55, 100, 94, 9 | ↑ lose | single-fold extreme |
| 33 | 93 | `possession_actions_per_defensive_action` | 0.0314 | 57, 25, 22, 26, 51 | ↓ lose | moderate |
| 36 | 14 | `ip_lh_buildup_low_vertical_distance_norm` | 0.0289 | 56, 40, 62, 22, 18 | ↓ lose | moderate |
| 37 | 64 | `crosses_driven` | 0.0278 | 106, 11, 82, 43, 126 | ↑ lose | single-fold extreme |
| 39 | 115 | `pushing_on` | 0.0272 | 43, 36, 27, 23, 58 | ↑ lose | moderate |
| 40 | 108 | `def_lh_low_block_length_norm` | 0.0229 | 61, 17, 93, 35, 91 | ↑ lose | volatile |
| 41 | 21 | `ip_lh_final_third_block_length_norm` | 0.0228 | 21, 29, 120, 84, 34 | ↓ lose | volatile |
| 43 | 36 | `lb_2units_inside_shape` | 0.0213 | 86, 33, 32, 111, 25 | — | volatile |
| 48 | 20 | `ip_lh_final_third_vertical_distance_norm` | 0.0188 | 25, 58, 43, 41, 43 | ↑ lose | single-fold extreme |
| 49 | 74 | `movement_final_third_in_front` | 0.0175 | 45, 77, 84, 65, 24 | ↓ lose | single-fold extreme |
| 53 | 43 | `lb_4units_advanced_midfield_completed` | 0.0160 | 98, 126, 14, 113, 126 | ↑ lose | single-fold extreme |
| 54 | 123 | `set_plays_corners` | 0.0156 | 81, 48, 120, 25, 46 | ↓ lose | single-fold extreme |
| 55 | 84 | `movement_defensive_third_in_front` | 0.0151 | 93, 19, 95, 95, 73 | ↑ lose | single-fold extreme |
| 56 | 45 | `lb_4units_attacking_completed` | 0.0148 | 119, 120, 86, 97, 17 | ↓ lose | single-fold extreme |
| 57 | 96 | `blocks_crosses` | 0.0134 | 26, 126, 55, 61, 47 | ↓ lose | single-fold extreme |
| 58 | 17 | `ip_lh_buildup_mid_vertical_distance_norm` | 0.0134 | 90, 111, 25, 78, 126 | ↓ lose | single-fold extreme |
| 59 | 26 | `lb_around_completed` | 0.0133 | 49, 56, 120, 46, 38 | ↑ lose | single-fold extreme |
| 60 | 131 | `physical_zone4_20_25_km` | 0.0121 | 55, 105, 35, 57, 55 | — | single-fold extreme |
| 61 | 79 | `movement_middle_third_in_front` | 0.0117 | 39, 73, 58, 44, 98 | ↓ lose | single-fold extreme |
| 63 | 121 | `set_plays_free_kicks` | 0.0113 | 82, 92, 120, 70, 29 | — | single-fold extreme |
| 64 | 46 | `lb_3units_defensive_attempted` | 0.0112 | 73, 67, 63, 81, 36 | ↓ lose | single-fold extreme |
| 65 | 48 | `lb_3units_midfield_attempted` | 0.0110 | 75, 70, 40, 51, 62 | ↑ lose | single-fold extreme |
| 66 | 65 | `crosses_lofted` | 0.0110 | 22, 99, 85, 90, 126 | ↑ lose | single-fold extreme |
| 67 | 127 | `physical_total_distance_km` | 0.0107 | 63, 88, 36, 60, 64 | ↓ lose | single-fold extreme |
| 68 | 58 | `attempts_off_target` | 0.0105 | 78, 35, 120, 92, 41 | ↓ lose | single-fold extreme |
| 70 | 130 | `physical_zone3_15_20_km` | 0.0102 | 38, 81, 46, 80, 89 | ↓ lose | single-fold extreme |
| 71 | 29 | `lb_4units_attempted` | 0.0100 | 34, 69, 105, 58, 107 | ↓ lose | single-fold extreme |
| 73 | 73 | `offers_outside_shape` | 0.0099 | 31, 89, 79, 124, 56 | — | single-fold extreme |
| 75 | 61 | `crosses_open_play_attempted` | 0.0093 | 107, 64, 34, 67, 96 | ↑ lose | single-fold extreme |
| 76 | 120 | `aerial_interventions` | 0.0093 | 52, 126, 76, 86, 37 | ↓ lose | single-fold extreme |
| 91 | 60 | `attempts_incomplete` | 0.0067 | 115, 39, 97, 93, 90 | ↓ lose | single-fold extreme |
| 104 | 87 | `movement_defensive_third_in_to_out` | 0.0052 | 120, 104, 38, 124, 84 | ↑ lose | single-fold extreme |
| 107 | 97 | `blocks_clearances` | 0.0049 | 117, 119, 37, 124, 85 | — | single-fold extreme |

## Notes

- `attempts_goals` is itself the goal outcome (see note 1 in `data/report-data-schema.md`), so its dominance is partly circular.
- Each fold's SHAP is based on only about 42 records; importance estimates combine sampling noise with model variation.
- Correlated feature groups (e.g. `pressures_total` / `pressures_direct`, `duels_physical` / `duels_aerial` / `duels_total`, block-geometry features) share their contribution with one another; a member falling outside the top ranks does not mean the corresponding behaviour is unimportant.
- Rankings beyond roughly the top 10 should not be read one by one; see `model/shap-stability-conclusion.md`.
