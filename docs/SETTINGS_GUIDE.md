# Settings Guide

## Level Detection

### Pivot Lookback (default: 5)
How many candles on each side to confirm a swing high/low. The pivot detection looks `lookback` bars to the left AND right, so a value of 5 means 5 candles must be lower on each side to confirm a swing high.

- **3-5**: More levels detected, good for scalping or intraday on 5m/15m
- **5-10**: Balanced, good for swing trading on 1h/30m (recommended)
- **10-20**: Fewer but stronger levels, good for position trading on daily

### Max Stored Levels (default: 100)
Cap on how many levels are held in memory. When exceeded, the weakest level (by the scoring formula) gets removed. Higher values preserve more history but increase computation. Most users won't need to change this.

### Merge Tolerance (default: 1.5x ATR)
How close two pivot prices must be (measured in multiples of ATR-14) to be merged into a single level rather than creating two separate ones. When merged, the level's price is averaged and its touch count increments.

- **0.5-1.0**: Tight merging, more separate levels
- **1.5-2.0**: Moderate, reduces clutter (recommended)
- **2.0+**: Aggressive merging, very few levels

### Band Cluster % (default: 0.1%)
After levels are selected for display, any that fall within this percentage of each other get visually grouped into a shaded band. The line is drawn at the strongest level in the cluster, and the band spans the full range.

### Min Touches to Display (default: 2)
Levels with fewer total touches (support + resistance combined) than this threshold are hidden. Raise to 3+ to show only the most confirmed levels. Set to 1 to see everything.

### Min Volume (default: 0.0 = off)
Pivots formed on candles with volume below this multiple of the 20-bar average volume are discarded entirely. Useful for filtering out levels formed during low-liquidity periods. Set to 0 to disable.

## Display

### Levels Above/Below Price (default: 4 / 4)
How many levels to show on each side of the current price. Only the closest N above and N below are drawn. Levels beyond this count remain in memory and reappear when price moves toward them.

- **2-3 per side**: Very clean chart, only the most relevant levels
- **4-5 per side**: Good balance (recommended)
- **8-10 per side**: More context, busier chart

### Label Offset (default: 3 bars)
How many bars past the last candle the labels are placed. Lower values keep labels tight to price action. Higher values give more breathing room.

### Show Price Labels (default: on)
Toggle the text labels on/off. Lines still draw when labels are hidden.

### Show Legend (default: on)
Toggle the top-right legend table explaining all visual markers.

### Label Size (default: Normal)
Small, Normal, or Large. Affects all labels including structural ones.

### Label Spacing % (default: 0.3%)
Controls the anti-overlap system. Labels whose prices are closer than this percentage will either merge (structural into pivot) or stagger horizontally. Increase if you still see overlap on tightly-spaced levels.

## Structural Levels

### Previous Day High / Low (default: on)
Draws dashed lines at yesterday's high and low. Thin (1px) lines. These are deterministic, always accurate, and don't require pivot detection.

### Previous Week High / Low (default: on)
Draws dashed lines at last week's high and low. Thicker (2px) lines. Critical for framing the weekly range.

## Colors

### Resistance Color (default: red)
Color for levels above current price and their labels.

### Support Color (default: green)
Color for levels below current price and their labels.

### Structural Level Color (default: white 40% opacity)
Color for PDH/PDL/PWH/PWL lines and labels. Intentionally muted so they don't compete with pivot levels.

## Debug

### Show Debug Info (default: off)
Displays a table in the bottom-left corner with internal counters: how many pivots were found, merged, trimmed, stored, and drawn. Useful for diagnosing why levels aren't appearing.
