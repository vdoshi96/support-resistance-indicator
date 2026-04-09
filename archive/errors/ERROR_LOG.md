# Error Log

Every Pine Script error encountered during development, with root cause and fix.

---

## Error 1: PINE_VERSION_OUTDATED
**When:** v1
**Message:** `Version 5 of Pine Script is outdated. We recommend using the current version, which is 6.`
**Fix:** Changed `//@version=5` to `//@version=6`. No code changes needed.

---

## Error 2: request.security() inside conditional
**When:** v2
**Line:** 83
**Message:** `Cannot use request.*() call within loops or conditional structures, when dynamic_requests is false in the indicator() function.`
**Root cause:** `request.security()` was inside an `if htfEnabled` block.
**Fix:** Moved `request.security()` calls to global scope. The result is always computed but only used when `htfEnabled` is true.
**Pine Script rule:** All `request.*()` calls must be at the top level of the script, never inside `if`, `for`, `while`, or function bodies (unless `dynamic_requests=true` is set, which has its own limitations).

---

## Error 3: array.get() out of bounds
**When:** v2
**Line:** 46 (isMerge function)
**Message:** `RE10045: Error on bar 11: In 'array.get()' function. Index 0 is out of bounds, array size is 0.`
**Root cause:** `isMerge()` loop iterated `for i = 0 to array.size(levels) - 1` even when array was empty. `array.size() - 1 = -1`, but the loop still executed once with i=0.
**Fix:** Added `if array.size(levels) > 0` guard before the loop.
**Pine Script rule:** Always guard array access with size checks. `for i = 0 to -1` still executes once in Pine Script.

---

## Error 4: Bar index too far from current
**When:** v4
**Message:** `RE10026: Error on bar 25174: Bar index value of the x1 argument (11512) in line.new() is too far from the current bar index. Try using time instead.`
**Root cause:** Stored `origin` bar index from old pivots was thousands of bars back. `line.new()` has a max distance of ~500 bars.
**Fix:** Clamped origin: `origin = math.max(array.get(lvlBars, i), bar_index - 500)`

---

## Error 5: No levels displayed (silent failure)
**When:** v4
**Symptom:** Script runs without errors but no lines appear on chart.
**Debug approach:** Added comprehensive debug label showing:
- Total pivots found (623 highs, 632 lows)
- Volume rejections, merges, new levels
- Array contents with prices and touch counts
**Root cause:** Discovered in two stages:

**Stage 1: Volume gate too strict.**
Original code: `if not na(ph) and not na(avgVol) and avgVol > 0`
On some instruments, `avgVol` is `na` for the first 20 bars, but the condition silently blocked ALL subsequent pivots in some edge cases.
**Partial fix:** Separated volume filter into a helper function with na/zero fallbacks.

**Stage 2: Old levels hogging storage (the real bug).**
Debug output revealed: levels were at $392-$411 while SPY was at $675. Old levels had accumulated massive `touches * volume` scores over thousands of bars. New levels near current price got immediately trimmed because their 1-touch score was lower than a 12-touch score from 6 months ago.
**Fix:** Distance-weighted trim scoring: `score / (1 + distance%)`. A level 40% away gets its score divided by 41.

**Stage 3: Still failing on 1h timeframe.**
1h loads more history than 15m in TradingView, so even more ancient levels accumulated.
**Final fix:** Changed from purging distant levels to proximity-based display (show nearest N above/below). Levels stay in memory indefinitely but only display when price is nearby.

---

## Error 6: Label overlap
**When:** v8
**Symptom:** PDH label stacked directly on top of a support level label at the same price.
**Not a Pine Script error but a UX bug.**
**Fix:** Unified label system that:
1. Collects ALL labels (structural + pivot) into one array
2. Sorts by price
3. Merges structural into pivot if within 0.3% (appends tag like `[PDH]`)
4. Staggers remaining close labels by offsetting bar position +8

---

## General Pine Script Gotchas Learned

1. **`for i = 0 to -1` executes once.** Always guard with size > 0.
2. **`request.security()` must be global scope.** Cannot be inside if/for/while.
3. **`line.new()` max distance is ~500 bars.** Clamp origin bar indices.
4. **`var` arrays persist across bars.** Without `var`, arrays reset every bar.
5. **`nz()` is essential for ATR/volume early bars.** First 14-20 bars return `na`.
6. **Tables survive redraws.** Use `var table` so it's created once, then update cells.
7. **`color.new()` transparency range is 0-100.** 0 = opaque, 100 = invisible.
8. **`color.r()`, `color.g()`, `color.b()` return 0-255.** Available in Pine v6.
