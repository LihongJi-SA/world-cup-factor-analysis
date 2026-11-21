# Feature impact on "win"

This document is the per-class SHAP analysis of the three-class model (lose = 0, draw = 1, win = 2). It only describes the model's attribution for the "win" class; see the corresponding files for the other two classes.

## Analysis setup

- **Data**: `data/match_features.csv`, 208 team-match records, 131 features; 5-fold `StratifiedKFold` (`random_state=42`).
- **Numbering**: feature numbers follow `feature-dictionary.md` (#4–#134) and match the "#" column of the dictionary tables.
- **Model**: `XGBClassifier` (500 trees, max depth 5, learning rate 0.04, `multi:softprob`), matching the configuration in `model/train.ipynb`.
- **Importance**: mean |SHAP| for the "win" class, computed on each fold's held-out set (about 42 records).
- **Direction**: Spearman correlation between the feature value and that class's SHAP value (pooled over all 208 held-out records). ↑ win = higher values push toward "win"; ↓ win = higher values push away from "win"; — = |ρ| < 0.15, no clear direction.
- **Stability**: core = mean rank ≤ 3 across the five folds; volatile = top 40 in at least 2 folds and outside the top 80 in at least 2 folds; single-fold extreme = top 40 in only 1 fold with mean rank > 40; stable = mean rank ≤ 20 and rank range ≤ 25; unstable = rank range ≥ 60; moderate = mean rank ≤ 40; otherwise low.
- **Class distribution**: 80 lose, 48 draw, 80 win (208 total).
- **Reference performance**: five-fold accuracy 0.779 ± 0.068, macro-averaged F1 0.747 ± 0.070.

## Key findings

- The top 3 features account for **44.4%** of total SHAP for "win"; the top 10 account for **65.4%**.
- **2** features are "core", **4** are "volatile", and **34** rely on a single fold.
- SHAP values are conditional attributions: direction describes the marginal push within this model, not a raw correlation coefficient.

## Top 20 features (the 4 with "unstable" stability removed)

| Rank | # | Feature | Mean \|SHAP\| | Rank per fold | Direction | Stability |
|---:|---:|---|---:|---|---|---|
| 1 | 56 | `attempts_goals` | 1.3013 | 1, 1, 1, 1, 1 | ↑ win | core |
| 2 | 5 | `xg` | 0.5427 | 2, 2, 2, 2, 2 | ↑ win | core |
| 3 | 118 | `gk_attempts_faced` | 0.3455 | 6, 3, 3, 3, 3 | ↓ win | stable |
| 4 | 119 | `gk_save_pct` | 0.2995 | 3, 4, 4, 4, 4 | ↑ win | stable |
| 5 | 63 | `crosses_outswing` | 0.2309 | 5, 5, 5, 5, 7 | ↓ win | stable |
| 6 | 7 | `total_passes` | 0.1741 | 4, 6, 6, 7, 8 | ↑ win | stable |
| 7 | 92 | `def_tackles` | 0.1059 | 8, 25, 20, 8, 5 | ↓ win | stable |
| 8 | 127 | `physical_total_distance_km` | 0.1053 | 9, 8, 7, 10, 11 | ↑ win | stable |
| 9 | 78 | `movement_final_third_in_behind` | 0.0596 | 10, 11, 15, 13, 26 | ↓ win | stable |
| 10 | 4 | `possession_pct` | 0.0594 | 19, 16, 10, 32, 12 | ↑ win | stable |
| 11 | 22 | `ip_lh_final_third_block_width_norm` | 0.0588 | 11, 7, 16, 17, 27 | ↓ win | stable |
| 12 | 112 | `pressure_avg_duration_s` | 0.0545 | 7, 12, 19, 47, 33 | ↓ win | moderate |
| 15 | 64 | `crosses_driven` | 0.0498 | 39, 9, 18, 20, 15 | ↓ win | moderate |
| 16 | 79 | `movement_middle_third_in_front` | 0.0452 | 78, 42, 63, 91, 6 | ↑ win | single-fold extreme |
| 18 | 99 | `duels_aerial` | 0.0385 | 44, 17, 14, 39, 16 | ↓ win | moderate |
| 20 | 69 | `offers_middle_third` | 0.0358 | 15, 26, 45, 15, 55 | ↑ win | moderate |

## Full ranking (59 features, sorted by mean |SHAP|; features with "low" (39) and "unstable" (33) stability removed)

| Rank | # | Feature | Mean \|SHAP\| | Rank per fold | Direction | Stability |
|---:|---:|---|---:|---|---|---|
| 1 | 56 | `attempts_goals` | 1.3013 | 1, 1, 1, 1, 1 | ↑ win | core |
| 2 | 5 | `xg` | 0.5427 | 2, 2, 2, 2, 2 | ↑ win | core |
| 3 | 118 | `gk_attempts_faced` | 0.3455 | 6, 3, 3, 3, 3 | ↓ win | stable |
| 4 | 119 | `gk_save_pct` | 0.2995 | 3, 4, 4, 4, 4 | ↑ win | stable |
| 5 | 63 | `crosses_outswing` | 0.2309 | 5, 5, 5, 5, 7 | ↓ win | stable |
| 6 | 7 | `total_passes` | 0.1741 | 4, 6, 6, 7, 8 | ↑ win | stable |
| 7 | 92 | `def_tackles` | 0.1059 | 8, 25, 20, 8, 5 | ↓ win | stable |
| 8 | 127 | `physical_total_distance_km` | 0.1053 | 9, 8, 7, 10, 11 | ↑ win | stable |
| 9 | 78 | `movement_final_third_in_behind` | 0.0596 | 10, 11, 15, 13, 26 | ↓ win | stable |
| 10 | 4 | `possession_pct` | 0.0594 | 19, 16, 10, 32, 12 | ↑ win | stable |
| 11 | 22 | `ip_lh_final_third_block_width_norm` | 0.0588 | 11, 7, 16, 17, 27 | ↓ win | stable |
| 12 | 112 | `pressure_avg_duration_s` | 0.0545 | 7, 12, 19, 47, 33 | ↓ win | moderate |
| 15 | 64 | `crosses_driven` | 0.0498 | 39, 9, 18, 20, 15 | ↓ win | moderate |
| 16 | 79 | `movement_middle_third_in_front` | 0.0452 | 78, 42, 63, 91, 6 | ↑ win | single-fold extreme |
| 18 | 99 | `duels_aerial` | 0.0385 | 44, 17, 14, 39, 16 | ↓ win | moderate |
| 20 | 69 | `offers_middle_third` | 0.0358 | 15, 26, 45, 15, 55 | ↑ win | moderate |
| 21 | 11 | `crosses_total` | 0.0357 | 26, 45, 38, 19, 13 | ↓ win | moderate |
| 22 | 107 | `def_lh_low_block_vertical_distance_norm` | 0.0335 | 36, 22, 48, 11, 49 | ↓ win | moderate |
| 27 | 71 | `offers_received` | 0.0274 | 55, 21, 31, 18, 65 | ↑ win | moderate |
| 29 | 28 | `lb_over_completed` | 0.0236 | 32, 30, 69, 48, 19 | ↓ win | moderate |
| 30 | 25 | `lb_around_attempted` | 0.0232 | 18, 44, 33, 34, 34 | ↑ win | moderate |
| 31 | 46 | `lb_3units_defensive_attempted` | 0.0231 | 123, 90, 86, 112, 9 | ↑ win | single-fold extreme |
| 36 | 116 | `pressing_direction_inside` | 0.0220 | 46, 13, 44, 93, 70 | ↓ win | single-fold extreme |
| 39 | 93 | `possession_actions_per_defensive_action` | 0.0204 | 28, 32, 24, 44, 59 | ↑ win | moderate |
| 42 | 121 | `set_plays_free_kicks` | 0.0199 | 87, 36, 32, 125, 18 | ↑ win | volatile |
| 43 | 83 | `movement_middle_third_in_behind` | 0.0196 | 56, 112, 93, 63, 14 | ↑ win | single-fold extreme |
| 44 | 14 | `ip_lh_buildup_low_vertical_distance_norm` | 0.0187 | 58, 20, 72, 49, 45 | ↑ win | single-fold extreme |
| 51 | 125 | `set_plays_corner_outswing` | 0.0152 | 97, 31, 106, 23, 92 | ↓ win | volatile |
| 53 | 128 | `physical_zone1_0_7_km` | 0.0150 | 21, 49, 41, 71, 73 | ↑ win | single-fold extreme |
| 55 | 111 | `pressures_direct` | 0.0131 | 77, 74, 25, 46, 43 | ↓ win | single-fold extreme |
| 56 | 123 | `set_plays_corners` | 0.0125 | 115, 29, 83, 38, 90 | ↓ win | volatile |
| 57 | 27 | `lb_over_attempted` | 0.0124 | 90, 126, 55, 24, 44 | ↓ win | single-fold extreme |
| 59 | 45 | `lb_4units_attacking_completed` | 0.0118 | 27, 72, 49, 51, 83 | — | single-fold extreme |
| 60 | 115 | `pushing_on` | 0.0113 | 82, 39, 50, 50, 76 | ↓ win | single-fold extreme |
| 61 | 109 | `def_lh_low_block_width_norm` | 0.0111 | 59, 93, 66, 92, 22 | ↓ win | single-fold extreme |
| 62 | 131 | `physical_zone4_20_25_km` | 0.0108 | 80, 56, 71, 40, 41 | ↑ win | single-fold extreme |
| 63 | 61 | `crosses_open_play_attempted` | 0.0106 | 123, 91, 54, 21, 121 | ↓ win | single-fold extreme |
| 64 | 51 | `lb_3units_attacking_completed` | 0.0104 | 40, 71, 124, 41, 50 | — | single-fold extreme |
| 65 | 67 | `crosses_push_cross` | 0.0101 | 69, 120, 124, 22, 109 | ↑ win | single-fold extreme |
| 66 | 106 | `def_lh_mid_block_width_norm` | 0.0097 | 123, 126, 110, 85, 20 | ↓ win | single-fold extreme |
| 67 | 124 | `set_plays_corner_inswing` | 0.0096 | 33, 43, 103, 98, 72 | ↓ win | single-fold extreme |
| 68 | 13 | `second_balls` | 0.0096 | 62, 33, 85, 106, 67 | ↓ win | single-fold extreme |
| 69 | 62 | `crosses_inswing` | 0.0092 | 52, 69, 89, 113, 31 | ↓ win | single-fold extreme |
| 71 | 26 | `lb_around_completed` | 0.0090 | 71, 53, 35, 89, 66 | ↑ win | single-fold extreme |
| 73 | 34 | `lb_3units_outside_shape` | 0.0089 | 112, 65, 26, 75, 91 | ↑ win | single-fold extreme |
| 74 | 9 | `completed_line_breaks` | 0.0087 | 35, 126, 47, 66, 86 | — | single-fold extreme |
| 76 | 29 | `lb_4units_attempted` | 0.0083 | 93, 68, 27, 125, 62 | ↑ win | single-fold extreme |
| 77 | 104 | `def_lh_mid_block_vertical_distance_norm` | 0.0083 | 24, 92, 109, 86, 75 | ↓ win | single-fold extreme |
| 78 | 38 | `lb_4units_defensive_attempted` | 0.0082 | 23, 115, 81, 109, 121 | ↑ win | single-fold extreme |
| 79 | 81 | `movement_middle_third_out_to_in` | 0.0080 | 38, 98, 76, 54, 121 | ↑ win | single-fold extreme |
| 80 | 97 | `blocks_clearances` | 0.0079 | 109, 63, 28, 90, 121 | ↑ win | single-fold extreme |
| 81 | 95 | `blocks_attempts_at_goal` | 0.0077 | 29, 109, 124, 125, 39 | ↑ win | volatile |
| 82 | 30 | `lb_4units_inside_shape` | 0.0076 | 123, 126, 21, 107, 101 | — | single-fold extreme |
| 83 | 60 | `attempts_incomplete` | 0.0075 | 123, 126, 68, 33, 80 | ↓ win | single-fold extreme |
| 84 | 130 | `physical_zone3_15_20_km` | 0.0074 | 51, 126, 116, 37, 121 | — | single-fold extreme |
| 85 | 88 | `movement_defensive_third_in_behind` | 0.0072 | 103, 55, 124, 64, 40 | ↑ win | single-fold extreme |
| 86 | 65 | `crosses_lofted` | 0.0072 | 25, 121, 87, 116, 106 | ↓ win | single-fold extreme |
| 93 | 35 | `lb_2units_attempted` | 0.0061 | 75, 105, 34, 96, 121 | ↑ win | single-fold extreme |
| 105 | 98 | `duels_physical` | 0.0049 | 123, 126, 36, 125, 48 | ↓ win | single-fold extreme |

## Notes

- `attempts_goals` is itself the goal outcome (see note 1 in `data/report-data-schema.md`), so its dominance is partly circular.
- Each fold's SHAP is based on only about 42 records; importance estimates combine sampling noise with model variation.
- Correlated feature groups (e.g. `pressures_total` / `pressures_direct`, `duels_physical` / `duels_aerial` / `duels_total`, block-geometry features) share their contribution with one another; a member falling outside the top ranks does not mean the corresponding behaviour is unimportant.
- Rankings beyond roughly the top 10 should not be read one by one; see `model/shap-stability-conclusion.md`.
