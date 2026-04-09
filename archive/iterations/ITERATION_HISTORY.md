# Iteration History

## v1 - Basic Pivot Lines
- Pine v5, simple `ta.pivothigh/low` with `extend.right`
- No merging, no filtering, no scoring

## v2 - Zones + Touch Merging + HTF
- Pine v6
- ATR-scaled zone boxes
- Nearby pivots merge, touch count increments
- Optional higher timeframe overlay
- Bugs: request.security in conditional, array bounds

## v3 - HTF Distance Filter
- Added `htfRange` to filter out HTF levels too far from price
- Separate `htfLookback` input
- Still problematic: HTF levels stale

## v4 - Unified Arrays + Flip Logic + Volume
- Single array pool with type flag (+1 support, -1 resistance)
- Auto-flip: broken support becomes resistance (yellow dotted)
- Volume filter + volume-scaled line width
- Score-based trimming (touches * volume)
- MAJOR BUG: no levels displayed due to old levels hogging storage

## v5 - Distance-Weighted Trimming
- Trim formula: `(touches * volume) / (1 + distance%)`
- Fixed the $400-levels-on-$675-SPY problem
- Still failed on 1h (more history loaded)

## v6 - Proximity Display
- Storage increased to 100
- Display only nearest N above/below current price
- Old levels preserved in memory, reappear when price revisits
- Added label size selector, dynamic text color, legend

## v7 - Band Clustering + Simplification
- Removed flip logic (user preferred positional S/R)
- Removed manual zone toggle
- Auto band clustering: levels within 0.1% merge into shaded band
- Anchor = highest-scoring level in cluster
- Removed HTF entirely

## v8 - Ground-Up Rebuild (CURRENT)
- Wick-based reaction scoring (wick_length / ATR)
- Dual-role tracking (separate supTouches / resTouches)
- Structural anchors (PDH/PDL, PWH/PWL)
- Recency-weighted trim formula
- Smart label anti-overlap (merge + stagger)
- Legend as fixed Pine table (top-right)
- Debug as fixed Pine table (bottom-left)
- Defaults from ADR methodology: lookback=5, showAbove/Below=4, lineExtend=3
- Grouped inputs for cleaner settings panel
