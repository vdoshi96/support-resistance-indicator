# CLAUDE.md

## Project Context

This is a TradingView Pine Script v6 indicator called "Dynamic Support and Resistance Zones". The main source file is `src/dynamic_sr_zones.pine`. It cannot be run locally -- it must be pasted into TradingView's Pine Editor to test.

## Key Architecture Decisions

- **Single-file Pine Script.** TradingView does not support multi-file projects. Everything lives in one `.pine` file.
- **Parallel arrays instead of UDTs.** Pine Script v6 supports user-defined types but arrays of UDTs have performance issues with large datasets. We use parallel arrays (lvlPrice, lvlSupTouches, lvlResTouches, etc.) indexed by the same integer.
- **Draw only on `barstate.islast`.** All visual elements (lines, labels, boxes) are created on the last bar only. This avoids the "too many drawings" runtime error and is more performant.
- **Unified label system.** All labels (structural + pivot) go into one array, get sorted by price, then merged/staggered to prevent overlap.

## Pine Script Constraints

1. `request.security()` MUST be at global scope. Cannot be inside if/for/while.
2. `for i = 0 to array.size(arr) - 1` executes once even if array is empty. Always guard with `if array.size(arr) > 0`.
3. `line.new()` max bar distance is ~500. Clamp stored bar indices.
4. Max drawing objects: 500 lines, 500 labels, 500 boxes per indicator instance.
5. No string interpolation. Use `str.tostring()` + concatenation.
6. `var` keyword makes variables persist across bars. Without it, they reset every bar.
7. Pine Script has no `null` -- use `na` for missing values.

## Testing Workflow

1. Edit `src/dynamic_sr_zones.pine`
2. Copy entire file contents
3. Paste into TradingView Pine Editor
4. Click "Add to chart" or "Update on chart"
5. Test on: RKLB (equity), ES1! (futures), SPY (high volume ETF)
6. Test timeframes: 15m, 1h, 4h, Daily
7. Toggle debug mode ON to inspect level counts and scores

## Common Modifications

- **Add structural level type** (e.g., prev month H/L): Add `request.security()` at global scope, add input toggle, add to unified label array in draw section.
- **Change scoring:** Edit the trimming loop `sc = ...` line.
- **Add label marker:** Edit `lblText` construction and add to legend table.

## Owner

Vishal. Trades futures (ES) and equities. Uses extended hours. Swing trading style (intra-week). Does NOT use 15m or below for level drawing. Prefers clean charts with 4-6 levels visible.

## Do Not

- Use em dashes in responses
- Use "it's not X, it's Y" framing
- Give trading advice or recommendations
