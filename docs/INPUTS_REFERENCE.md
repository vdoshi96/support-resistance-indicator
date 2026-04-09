# Input Parameters Reference

## Level Detection

| Parameter | Default | Range | Description |
|-----------|---------|-------|-------------|
| Pivot Lookback | 5 | 2-50 | Candles on each side to confirm a swing high/low. Lower = more levels (scalping). Higher = fewer, stronger levels (swing). |
| Max Stored Levels | 100 | 10-500 | Total levels held in memory. Only nearest N are displayed. Old levels reactivate when price revisits. |
| Merge Tolerance | 1.5 | 0.1+ | ATR multiplier. Pivots within this distance merge into one level. Higher = more aggressive merging, fewer lines. |
| Band Cluster (%) | 0.1 | 0.01+ | Visible levels within this % of each other form a shaded band. |
| Min Touches | 2 | 1-10 | Levels with fewer total touches (support + resistance) are hidden. Raise for only the strongest zones. |
| Min Volume (x avg) | 0.0 | 0.0+ | 0 = off. Filters out pivots formed on volume below this multiple of the 20-bar average. |

## Display

| Parameter | Default | Range | Description |
|-----------|---------|-------|-------------|
| Levels Above Price | 4 | 1-20 | How many resistance levels to show above current price. |
| Levels Below Price | 4 | 1-20 | How many support levels to show below current price. |
| Label Offset (bars) | 3 | 1-50 | How far right of the last candle labels are placed. |
| Show Price Labels | true | - | Toggle the price/touch labels on/off. |
| Show Legend | true | - | Toggle the top-right legend table. |
| Label Size | Normal | Small/Normal/Large | Size of all labels on chart. |
| Label Spacing (%) | 0.3 | 0.05+ | Labels closer than this % apart get staggered or merged. |
| Legend Background Opacity | 68 | 0-95 | Lower = more opaque gray behind the legend (better on white charts). |
| Legend High Contrast Text | true | - | When on, legend body text uses high-contrast color. |
| Auto Stagger Close Structural Labels | true | - | Spreads PDH/PDL/PWH/PWL labels when they sit on nearby prices. |
| Structural Label Spacing (%) | 1.25 | 0.05+ | Relative gap threshold for treating two structural labels as overlapping. |
| Structural Near (x ATR) | 0.5 | 0.05+ | Also stagger when absolute price gap is within this multiple of ATR (helps PWH vs PDL on mid-priced names). |
| Structural Label Vertical Offset (ATR) | 0.10 | 0.01+ | How far labels shift vertically when staggered (scaled by ATR). |

## Structural Levels

| Parameter | Default | Description |
|-----------|---------|-------------|
| Previous Day High / Low | true | Dashed lines at yesterday's high and low. Thin (width 1). |
| Previous Week High / Low | true | Dashed lines at last week's high and low. Thicker (width 2). |

## Colors

| Parameter | Default | Description |
|-----------|---------|-------------|
| Resistance | Red | Color for levels above current price. |
| Support | Green | Color for levels below current price. |
| Structural Levels | White (40% transparency) | Color for PDH/PDL/PWH/PWL lines. |

## Debug

| Parameter | Default | Description |
|-----------|---------|-------------|
| Show Debug Info | false | Shows a table in the bottom-left with pivot counts, merge/trim stats, and the top 5 visible levels with their scores. |

## Label Markers Explained

| Marker | Meaning |
|--------|---------|
| S | Support (level below current price) |
| R | Resistance (level above current price) |
| (3x) | Level has been touched 3 times total |
| (5s/3r) | 5 support touches + 3 resistance touches (dual-role) |
| ◆ | Dual-role level (acted as both support and resistance) |
| ★ | High-volume pivot (formed on 1.5x+ average volume) |
| [PDH] | Coincides with Previous Day High |
| [PDL] | Coincides with Previous Day Low |
| [PWH] | Coincides with Previous Week High |
| [PWL] | Coincides with Previous Week Low |

## Scoring Formula

Used for trimming and prioritization:

```
score = reaction_quality * dual_bonus * recency * volume / (1 + distance%)
```

Where:
- `reaction_quality` = accumulated wick_length/ATR across all touches
- `dual_bonus` = 2.0 if both supTouches > 0 and resTouches > 0, else 1.0
- `recency` = 1 / (1 + bars_since_last_touch / 500)
- `volume` = max relative volume across all touches
- `distance%` = % distance from current close
