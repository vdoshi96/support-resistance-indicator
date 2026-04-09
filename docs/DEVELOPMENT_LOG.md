# Development Log

## Overview

This indicator went through 8 major iterations from a simple pivot-based S/R drawer to a methodology-driven system incorporating wick scoring, dual-role tracking, structural anchors, and smart label management. Development occurred in a single Claude.ai session on 2026-04-09.

---

## Version 1: Basic Pivot S/R (Pine v5)

**What it did:** Simple `ta.pivothigh()` / `ta.pivotlow()` with `extend.right` lines.
**Errors:**
- None, but too basic to be useful.

## Version 2: Zones, Touch Merging, ATR Scaling (Pine v5)

**What was added:**
- Nearby pivots merge into one level, incrementing touch count
- ATR-scaled zone boxes instead of thin lines
- Min touch filter (default 2)
- Multi-timeframe overlay (optional daily pivots on intraday)
- Max levels cap (default 10)

**Errors encountered:**
1. `Cannot use request.*() call within loops or conditional structures` (line 83). `request.security()` was inside an `if htfEnabled` block. Fix: moved to global scope, only used result conditionally.
2. `PINE_VERSION_OUTDATED` warning. Fix: bumped to `@version=6`.
3. `RE10045: array.get() Index 0 out of bounds, array size is 0` in `isMerge()`. Fix: added `if array.size(levels) > 0` guard before loop.

**User feedback:** "These labels are too far from the candles" (lineExtend=30 was too much). Changed default to 5, then 10, then finally 3 in v8.

## Version 3: HTF Distance Filter

**Problem:** HTF weekly pivots showing levels thousands of points from current price (e.g., $3500 levels on ES at $6800).
**Fix:** Added `htfRange` input (% distance), purged levels beyond threshold.
**Still problematic:** lookback=3 on weekly needs 3 weeks each side, so recent pivots invisible.

## Version 4: Flip Logic + Volume Weighting

**What was added:**
- Unified array (S and R in one pool with type flag)
- Auto-flip: support broken becomes resistance (yellow dotted line)
- Volume filter: pivots below 0.5x 20-bar avg volume ignored
- Volume scaling: high-volume pivots get thicker lines + star marker
- Smart trimming: weakest by `touches * volume` score dropped first

**Major bug: NO LEVELS DISPLAYED**
- Added debug label to diagnose. Output showed:
  ```
  Pivot highs found: 623, vol rejected: 10, merged: 51, new: 562
  Pivot lows found: 632, vol rejected: 8, merged: 50, new: 574
  Trimmed: 1124, Array size: 12, Drawn: 12
  First 5 levels: 392.21, 409.23, 398.98, 388.97, 411.87
  ```
- ROOT CAUSE: Old levels from SPY at $400 had massive touch counts (11x, 12x) from accumulating over thousands of bars. New levels near $675 got immediately trimmed because score was lower. Levels were being drawn but off-screen at $400.

**Fix (v5):** Added distance-weighted trim scoring: `score / (1 + distance%)`. Level 40% away gets score divided by 41.

**Other bug:** `RE10026: Bar index value of x1 argument too far from current bar index`. Fix: clamped `origin` with `math.max(stored_bar, bar_index - 500)`.

## Version 5: Distance-Aware Trimming

**Fix applied:** Trim formula changed to `(touches * volume) / (1 + distPct)`.
**Still broken on 1h timeframe.** The 15m worked because TradingView loads less history, so fewer ancient levels accumulated.

## Version 6: Proximity-Based Display

**Key insight from user:** "I'd rather not purge them, just hide them till price gets closer."
**Solution:** Increased storage to 100, only display closest N above and N below current price. Old levels survive in memory, reappear when price revisits.
**Inputs added:** `showAbove`, `showBelow` (default 10 each).

**Other additions:**
- Label size selector (Small/Normal/Large)
- Dynamic text color using luminance calculation for legibility on any background
- Legend explaining line colors, thickness, markers

## Version 7: Band Clustering + Flip Removal

**Band logic:** After selecting visible levels, sort by price. Levels within 0.1% of each other cluster into a band. Anchor = highest-scoring level in cluster. Shaded box spans min to max price of cluster.
**Removed:** Flip logic (user preferred pure positional S/R: above price = red, below = green).
**Removed:** Manual zone toggle (bands are now automatic).

## Version 8: Ground-Up Rebuild (Current)

**Methodology sources:** ADR trading transcripts (4 videos) + user's own swing trading workflow.

**New scoring system:**
- Wick-based reaction quality: `wick_length / ATR` at each pivot. Long-wick shooting star scores much higher than tiny consolidation pivot.
- Dual-role tracking: separate `supTouches` and `resTouches` arrays. Levels that acted as both get 2x scoring bonus + diamond marker.
- Recency weighting: `1 / (1 + bars_since_last_touch / 500)`. Recent levels favored.
- Full trim formula: `reaction * dual_bonus * recency * volume / (1 + distance%)`

**Structural anchors:**
- Previous Day High/Low (dashed, thin, white)
- Previous Week High/Low (dashed, thicker, white)
- Always drawn, toggleable independently

**Smart label anti-overlap:**
- All labels (structural + pivot) unified into one sorted list
- Structural label within 0.3% of pivot label merges: `68.61 S (27s/18r) ◆ ★ [PDH]`
- Remaining close labels stagger horizontally (+8 bars offset)
- Spacing threshold adjustable

**Defaults calibrated from ADR transcripts:**
- lookback=5, filterTouch=2, showAbove/Below=4, lineExtend=3

**Label legend uses Pine `table` fixed to `position.top_right`** so it doesn't scroll with the chart.

---

## Known Limitations / Future Work

1. **No gap detection.** ADR marks gaps in blue as magnetic zones. Could detect gaps between daily close/open and add as structural levels.

2. **No market structure overlay.** Higher highs / lower lows trend identification. Would help contextualize whether to favor long or short setups.

3. **No engulfing candle detection at levels.** ADR uses engulfing candles at key levels as entry confirmation. Could add markers.

4. **No multi-timeframe confluence.** A level confirmed on weekly + daily should outrank a current-TF-only level. Would require multiple `request.security()` calls.

5. **No alert conditions.** Price approaching a key level could trigger TradingView alerts.

6. **Sorting uses bubble sort.** Fine for 8-20 items but could be optimized if level counts increase.

7. **Session-aware structural levels.** For futures: pre-market high/low, regular session open. Currently only prev day/week.

8. **Previous month high/low.** For swing traders using weekly+ timeframes.
