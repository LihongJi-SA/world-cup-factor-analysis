# FIFA World Cup 2026 PMSR — extraction schema

One CSV row per team per match. Column 1 is the nation name, column 2 is the
match number taken from the PDF filename (1–104), column 3 is the stage
(`group stage` or `knock out`), and the last column is the result:
`1` = win, `0.5` = draw, `0` = loss. Every value between is numeric
(`int` or `float`). Floats are rounded to 2 decimals.

Percentages are stored as shares from 0 to 1 (not 0–100%). Line height and team
length distances are normalised to 0–1 by the pitch dimensions (105 m length,
68 m width). Physical distances are in kilometres.

This file was de-duplicated: any metric printed in more than one section is kept
only once. See "Duplicates resolved" below.

## Duplicates resolved

| Metric | Printed in | Kept in |
|---|---|---|
| Goals | Key statistics; Attempts at Goal | Attempts at Goal |
| Forced turnovers | Key statistics; Defensive Actions; Defensive Pressure | Defensive Actions |
| Direct pressures | Key statistics; Defensive Pressure | Defensive Pressure |
| Defensive pressures applied / total pressures | Key statistics; Defensive Pressure | Defensive Pressure |
| Total distance covered | Key statistics (km); Physical Data (sum of zones, m) | Physical Data (sum of zones, km) |
| Zone 4 distance | Key statistics (km); Physical Data zone 4 (m) | Physical Data (zone 4, km) |

Not duplicates, despite similar names:

- **`crosses_total`** (Key statistics) counts every cross, including set plays.
  **`crosses_open_play_attempted`** counts open-play crosses only. Both are kept.
- **`completed_line_breaks`** is the only printed total; the Line Breaks charts
  give a Through/Around/Over breakdown, not a second total.
- **`attempts_at_goal`** is the only printed attempt total; the Attempts at Goal
  page gives the outcome breakdown.

## Columns (in CSV order)

### Match identity & key statistics (page 3)

| # | Column | Type |
|---|---|---|
| 1 | nation | str |
| 2 | match_no | int |
| 3 | stage | str |
| 4 | possession_pct | float |
| 5 | xg | float |
| 6 | attempts_at_goal | int |
| 7 | total_passes | int |
| 8 | pass_completion_pct | float |
| 9 | completed_line_breaks | int |
| 10 | receptions_final_third | int |
| 11 | crosses_total | int |
| 12 | ball_progressions | int |
| 13 | second_balls | int |

### In Possession — Line Height & Team Length (pages 6–7)

Order per phase: vertical distance to own goal line → block length → block width.
Vertical distance and block length are divided by 105 (pitch length); block
width is divided by 68 (pitch width). Values are fractions from 0 to 1.

| # | Column | Type |
|---|---|---|
| 14 | ip_lh_buildup_low_vertical_distance_norm | float |
| 15 | ip_lh_buildup_low_block_length_norm | float |
| 16 | ip_lh_buildup_low_block_width_norm | float |
| 17 | ip_lh_buildup_mid_vertical_distance_norm | float |
| 18 | ip_lh_buildup_mid_block_length_norm | float |
| 19 | ip_lh_buildup_mid_block_width_norm | float |
| 20 | ip_lh_final_third_vertical_distance_norm | float |
| 21 | ip_lh_final_third_block_length_norm | float |
| 22 | ip_lh_final_third_block_width_norm | float |

### In Possession — Line Breaks (pages 8–9)

Top charts in order: Through → Around → Over. Bottom panels: 4 Units → 3 Units → 2 Units.

| # | Column | Type |
|---|---|---|
| 23 | lb_through_attempted | int |
| 24 | lb_through_completed | int |
| 25 | lb_around_attempted | int |
| 26 | lb_around_completed | int |
| 27 | lb_over_attempted | int |
| 28 | lb_over_completed | int |
| 29 | lb_4units_attempted | int |
| 30 | lb_4units_inside_shape | int |
| 31 | lb_4units_outside_shape | int |
| 32 | lb_3units_attempted | int |
| 33 | lb_3units_inside_shape | int |
| 34 | lb_3units_outside_shape | int |
| 35 | lb_2units_attempted | int |
| 36 | lb_2units_inside_shape | int |
| 37 | lb_2units_outside_shape | int |

### In Possession — Line Break Distribution by Units (pages 8–9)

Each bottom panel has one row per unit. Row order, top to bottom: 4 Units — Defensive → Midfield → Advanced Midfield → Attacking; 3 Units — Defensive → Midfield → Attacking; 2 Units — Defensive → Midfield. Within each row the first value is Attempted and the second is Completed.

| # | Column | Type |
|---|---|---|
| 38 | lb_4units_defensive_attempted | int |
| 39 | lb_4units_defensive_completed | int |
| 40 | lb_4units_midfield_attempted | int |
| 41 | lb_4units_midfield_completed | int |
| 42 | lb_4units_advanced_midfield_attempted | int |
| 43 | lb_4units_advanced_midfield_completed | int |
| 44 | lb_4units_attacking_attempted | int |
| 45 | lb_4units_attacking_completed | int |
| 46 | lb_3units_defensive_attempted | int |
| 47 | lb_3units_defensive_completed | int |
| 48 | lb_3units_midfield_attempted | int |
| 49 | lb_3units_midfield_completed | int |
| 50 | lb_3units_attacking_attempted | int |
| 51 | lb_3units_attacking_completed | int |
| 52 | lb_2units_defensive_attempted | int |
| 53 | lb_2units_defensive_completed | int |
| 54 | lb_2units_midfield_attempted | int |
| 55 | lb_2units_midfield_completed | int |

For each panel the per-line attempted values sum to the matching `lb_4units_attempted`, `lb_3units_attempted` or `lb_2units_attempted`. `inside_shape` and `outside_shape` split that attempted total by location; they do not split the Completed values.

### In Possession — Attempts at Goal (summary page)

| # | Column | Type |
|---|---|---|
| 56 | attempts_goals | int |
| 57 | attempts_on_target | int |
| 58 | attempts_off_target | int |
| 59 | attempts_blocked | int |
| 60 | attempts_incomplete | int |

### In Possession — Crosses, open play (pages 18–19)

| # | Column | Type |
|---|---|---|
| 61 | crosses_open_play_attempted | int |
| 62 | crosses_inswing | int |
| 63 | crosses_outswing | int |
| 64 | crosses_driven | int |
| 65 | crosses_lofted | int |
| 66 | crosses_cutback | int |
| 67 | crosses_push_cross | int |

### In Possession — Offering to Receive (pages 20–21)

| # | Column | Type |
|---|---|---|
| 68 | offers_final_third | int |
| 69 | offers_middle_third | int |
| 70 | offers_defensive_third | int |
| 71 | offers_received | int |
| 72 | offers_inside_shape | int |
| 73 | offers_outside_shape | int |

### In Possession — Movement to Receive (pages 22–23)

Values come from the "Movement Types Pitch Third" panel (right-hand side), so
the third is the area of the pitch where the movement happened, not the
possession phase shown in the left-hand donuts (Final Third Phase /
Progression Phase / Build Up Phase).

Order per third: In Front → In Between → Out to In → In to Out → In Behind.

| # | Column | Type |
|---|---|---|
| 74 | movement_final_third_in_front | int |
| 75 | movement_final_third_in_between | int |
| 76 | movement_final_third_out_to_in | int |
| 77 | movement_final_third_in_to_out | int |
| 78 | movement_final_third_in_behind | int |
| 79 | movement_middle_third_in_front | int |
| 80 | movement_middle_third_in_between | int |
| 81 | movement_middle_third_out_to_in | int |
| 82 | movement_middle_third_in_to_out | int |
| 83 | movement_middle_third_in_behind | int |
| 84 | movement_defensive_third_in_front | int |
| 85 | movement_defensive_third_in_between | int |
| 86 | movement_defensive_third_out_to_in | int |
| 87 | movement_defensive_third_in_to_out | int |
| 88 | movement_defensive_third_in_behind | int |

### Out of Possession — Defensive Actions (pages 25–26)

| # | Column | Type |
|---|---|---|
| 89 | def_forced_turnovers | int |
| 90 | def_possession_regained | int |
| 91 | def_interceptions | int |
| 92 | def_tackles | int |
| 93 | possession_actions_per_defensive_action | float |
| 94 | blocks_passes | int |
| 95 | blocks_attempts_at_goal | int |
| 96 | blocks_crosses | int |
| 97 | blocks_clearances | int |
| 98 | duels_physical | int |
| 99 | duels_aerial | int |
| 100 | duels_total | int |

### Out of Possession — Defensive Line Height & Team Length (pages 27–28)

Order per block: High Block/Press → Mid Block → Low Block.
Within each: vertical distance to own goal line → block length → block width.
Vertical distance and block length are divided by 105 (pitch length); block
width is divided by 68 (pitch width). Values are fractions from 0 to 1.

| # | Column | Type |
|---|---|---|
| 101 | def_lh_high_block_vertical_distance_norm | float |
| 102 | def_lh_high_block_length_norm | float |
| 103 | def_lh_high_block_width_norm | float |
| 104 | def_lh_mid_block_vertical_distance_norm | float |
| 105 | def_lh_mid_block_length_norm | float |
| 106 | def_lh_mid_block_width_norm | float |
| 107 | def_lh_low_block_vertical_distance_norm | float |
| 108 | def_lh_low_block_length_norm | float |
| 109 | def_lh_low_block_width_norm | float |

### Out of Possession — Defensive Pressure (page 29)

| # | Column | Type |
|---|---|---|
| 110 | pressures_total | int |
| 111 | pressures_direct | int |
| 112 | pressure_avg_duration_s | float |
| 113 | ball_recovery_time_s | float |
| 114 | pushing_on_into_pressing | int |
| 115 | pushing_on | int |
| 116 | pressing_direction_inside | int |
| 117 | pressing_direction_outside | int |

### Goalkeeping (pages 34–37)

| # | Column | Type |
|---|---|---|
| 118 | gk_attempts_faced | int |
| 119 | gk_save_pct | float |
| 120 | aerial_interventions | int |

### Set Plays (pages 39–40)

| # | Column | Type |
|---|---|---|
| 121 | set_plays_free_kicks | int |
| 122 | set_plays_penalties | int |
| 123 | set_plays_corners | int |
| 124 | set_plays_corner_inswing | int |
| 125 | set_plays_corner_outswing | int |
| 126 | set_plays_throw_ins | int |

### Individual Data — Physical (pages 50–51)

All eight are sums across the players listed on the team's page. Distances in
kilometres; `physical_high_speed_runs` and `physical_sprints` are counts.

| # | Column | Type |
|---|---|---|
| 127 | physical_total_distance_km | float |
| 128 | physical_zone1_0_7_km | float |
| 129 | physical_zone2_7_15_km | float |
| 130 | physical_zone3_15_20_km | float |
| 131 | physical_zone4_20_25_km | float |
| 132 | physical_zone5_25_plus_km | float |
| 133 | physical_high_speed_runs | float |
| 134 | physical_sprints | float |

### Result

| # | Column | Type |
|---|---|---|
| 135 | win | float (1 win, 0.5 draw, 0 loss) |

## Notes

1. `attempts_goals` is the Goals outcome on the Attempts at Goal page. It is
   usually equal to the team's goals on the cover, but not always: own goals
   and goal events not logged as an attempt are not counted there. In this
   dataset 19 of 208 team-rows have `attempts_goals` below the cover score
   (20 goals total); the key-statistics goals figure equals the cover score on
   all 208 rows.
2. `crosses_total` ≠ `crosses_open_play_attempted` + corner crosses; the key
   statistics figure includes all crosses (open play and set plays).
3. `physical_total_distance_km` equals the sum of zones 1–5 to within rounding.
4. Goalkeeper rows are included in the physical sums. Subtract them if
   outfield-only totals are needed.
5. The result flag is `1` win, `0.5` draw, `0` loss. A match level after
   120 minutes counts as a draw even if it was decided by a penalty shootout;
   the cover score is the 120-minute score.
6. `duels_physical`, `duels_aerial` and `duels_total` are the three segments of
   the Possession Contests donut. `duels_total` is the segment labelled
   "Duels" — it is **not** the sum of physical + aerial duels.
7. `possession_pct`, `pass_completion_pct` and `gk_save_pct` keep their names
   but are shares in 0–1. Possession excludes the contested share printed
   between the two team values.
8. `match_no` is parsed from the report filename: `M01 …` → 1, `M104 …` → 104.
9. `stage` is the competition stage taken from the report folder:
   `group stage` (group-stage reports) or `knock out` (knock-out reports).
10. The per-unit line break columns (38–55) come from the three bottom panels of
    the Line Breaks pages. In each panel the Attempted values for the individual
    units sum to the panel total (`lb_4units_attempted`, `lb_3units_attempted`
    or `lb_2units_attempted`); this holds for all 208 team-rows. Completed
    values are the chart's Complete series and are not split by inside/outside
    shape.
