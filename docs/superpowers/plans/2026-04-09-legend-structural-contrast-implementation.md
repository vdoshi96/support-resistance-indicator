# Legend + Structural Contrast Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Make the legend readable on bright charts and prevent close structural labels (`PDL`/`PWH`) from colliding, with smart defaults and optional user overrides.

**Architecture:** Extend the existing single-file Pine v6 pipeline in `src/dynamic_sr_zones.pine` by adding a small set of display inputs, then layering structural-specific stagger logic into the existing unified anti-overlap section. Keep structural line geometry unchanged and apply offsets only at label rendering time. Reuse current arrays and barstate-last rendering model.

**Tech Stack:** TradingView Pine Script v6, single-file indicator architecture, manual verification in TradingView chart UI.

---

## File Structure

- Modify: `src/dynamic_sr_zones.pine`
  - Add new display inputs for legend contrast and structural staggering.
  - Add a per-label vertical adjustment array for label-only offsets.
  - Add structural-cluster detection and stagger rules in the existing overlap pass.
  - Enforce horizontal nudge cap (`+7`) before label draw.
  - Apply legend contrast settings to `table.new` and legend `table.cell` rows.
- Manual verification reference: `docs/superpowers/specs/2026-04-09-legend-structural-contrast-design.md`

No new runtime files are required.

### Task 1: Add New Inputs and Defaults

**Files:**
- Modify: `src/dynamic_sr_zones.pine` (inputs block near `grpDisplay`)
- Test: Manual TradingView input panel check

- [ ] **Step 1: Add legend and structural stagger inputs**

```pine
legendBgAlpha = input.int(68, "Legend Background Opacity", minval=0, maxval=95, group=grpDisplay, tooltip="Lower = less transparent, more contrast")
legendHighContrastText = input.bool(true, "Legend High Contrast Text", group=grpDisplay)
enableStructAutoStagger = input.bool(true, "Auto Stagger Close Structural Labels", group=grpDisplay)
structLblOverlapPct = input.float(0.20, "Structural Label Spacing (%)", minval=0.05, step=0.05, group=grpDisplay)
structLblOffsetAtrMult = input.float(0.08, "Structural Label Vertical Offset (ATR)", minval=0.01, step=0.01, group=grpDisplay)
```

- [ ] **Step 2: Validate compile in TradingView**

Run: Paste script in Pine Editor and click `Update on chart`  
Expected: No compile error; five new controls appear under display section.

- [ ] **Step 3: Commit**

```bash
git add src/dynamic_sr_zones.pine
git commit -m "feat: add legend contrast and structural stagger inputs"
```

### Task 2: Add Label-Only Vertical Offset Storage

**Files:**
- Modify: `src/dynamic_sr_zones.pine` (unified label system arrays + draw section)
- Test: Manual compile and baseline visual parity

- [ ] **Step 1: Add `allLblYAdj` array to unified label arrays**

```pine
var float[] allLblYAdj = array.new_float()
array.clear(allLblYAdj)
```

- [ ] **Step 2: Push default `0.0` adjustment alongside every label insert**

```pine
array.push(allLblYAdj, 0.0)
```

Apply this at each label insertion site:
- structural label pushes (`PDH`, `PDL`, `PWH`, `PWL`)
- pivot/zone label pushes (`S`/`R` labels)

- [ ] **Step 3: Validate compile and baseline**

Run: `Update on chart` with `enableStructAutoStagger=false`  
Expected: Visual output matches previous behavior (no unintended drift).

- [ ] **Step 4: Commit**

```bash
git add src/dynamic_sr_zones.pine
git commit -m "refactor: track per-label vertical adjustment for label placement"
```

### Task 3: Implement Structural Proximity Detection and Pair Stagger

**Files:**
- Modify: `src/dynamic_sr_zones.pine` (overlap section after label sorting)
- Test: Manual scenario with close `PDL`/`PWH`

- [ ] **Step 1: Add structural proximity condition using dedicated threshold**

```pine
isCloseStructPair = enableStructAutoStagger and array.get(allLblStruct, i) and array.get(allLblStruct, i - 1) and p1 > 0 and math.abs(p2 - p1) / p1 * 100 < structLblOverlapPct
```

- [ ] **Step 2: Compute bounded vertical offset with ATR fallback**

```pine
tick = syminfo.mintick
baseOffset = na(atrVal) or atrVal == 0 ? tick * 5.0 : atrVal * structLblOffsetAtrMult
minOffset = tick * 3.0
maxOffset = tick * 50.0
priceOffset = math.max(minOffset, math.min(baseOffset, maxOffset))
```

- [ ] **Step 3: Apply top/bottom vertical separation to the pair**

```pine
array.set(allLblYAdj, i - 1, priceOffset)
array.set(allLblYAdj, i, -priceOffset)
```

- [ ] **Step 4: Add horizontal stagger with hard cap (`+7`)**

```pine
prevBar = array.get(allLblBar, i - 1)
curBar = array.get(allLblBar, i)
array.set(allLblBar, i - 1, math.min(prevBar + 4, bar_index + lineExtend + 7))
array.set(allLblBar, i, math.min(curBar + 7, bar_index + lineExtend + 7))
```

- [ ] **Step 5: Validate collision fix**

Run: On a chart where `PDL` and `PWH` are close, `Update on chart`  
Expected: both labels remain visible, one above and one below, and rightward nudge does not exceed `+7`.

- [ ] **Step 6: Commit**

```bash
git add src/dynamic_sr_zones.pine
git commit -m "feat: stagger close structural labels with capped horizontal offset"
```

### Task 4: Handle 3+ Structural Label Clusters

**Files:**
- Modify: `src/dynamic_sr_zones.pine` (same overlap pass)
- Test: Manual cluster validation on volatile symbols/timeframes

- [ ] **Step 1: Extend pair logic into progressive alternating pattern**

```pine
// clusterIndex increments for consecutive close structural labels
dir = clusterIndex % 2 == 0 ? 1.0 : -1.0
stepBars = math.min(2 + clusterIndex * 2, 7)
array.set(allLblBar, i, math.min(array.get(allLblBar, i) + stepBars, bar_index + lineExtend + 7))
array.set(allLblYAdj, i, dir * priceOffset)
```

- [ ] **Step 2: Keep generic overlap logic for non-structural labels unchanged**

```pine
if not isCloseStructPair
    // existing lblOverlap bar-stagger behavior stays here
```

- [ ] **Step 3: Validate cluster behavior**

Run: Test on `RKLB`, `ES1!`, `SPY` across `1h`, `4h`, `Daily`  
Expected: 3+ close structural labels remain readable; no label with positive horizontal shift beyond `+7`.

- [ ] **Step 4: Commit**

```bash
git add src/dynamic_sr_zones.pine
git commit -m "feat: add bounded alternating stagger for structural label clusters"
```

### Task 5: Apply Legend Contrast Controls

**Files:**
- Modify: `src/dynamic_sr_zones.pine` (`legTbl` and legend `table.cell` rows)
- Test: Manual light/dark theme readability check

- [ ] **Step 1: Use `legendBgAlpha` for legend table background**

```pine
var table legTbl = table.new(position.top_right, 1, 8, bgcolor=color.new(color.gray, legendBgAlpha), border_width=0)
```

- [ ] **Step 2: Route explanatory row colors through contrast toggle**

```pine
legendBodyText = legendHighContrastText ? color.white : structTxtCol
table.cell(legTbl, 0, 0, "── LEGEND ──", text_color=legendBodyText, text_size=size.small, text_halign=text.align_left)
// keep support/resistance rows semantic
table.cell(legTbl, 0, 1, "━ Support (S): level below price", text_color=supColor, text_size=size.small, text_halign=text.align_left)
table.cell(legTbl, 0, 2, "━ Resistance (R): level above price", text_color=resColor, text_size=size.small, text_halign=text.align_left)
```

- [ ] **Step 3: Update remaining explanatory lines to use `legendBodyText`**

```pine
table.cell(legTbl, 0, 4, "◆ = dual role (acted as both S and R)", text_color=legendBodyText, text_size=size.small, text_halign=text.align_left)
table.cell(legTbl, 0, 5, "★ = high volume pivot (1.5x+ avg)", text_color=legendBodyText, text_size=size.small, text_halign=text.align_left)
table.cell(legTbl, 0, 6, "Line width = reaction strength + touches", text_color=legendBodyText, text_size=size.small, text_halign=text.align_left)
table.cell(legTbl, 0, 7, "[PWH] = coincides with structural level", text_color=legendBodyText, text_size=size.small, text_halign=text.align_left)
```

- [ ] **Step 4: Validate legend readability**

Run: Toggle between light and dark chart themes, then `Update on chart`  
Expected: legend remains readable on light theme and not overly heavy on dark theme.

- [ ] **Step 5: Commit**

```bash
git add src/dynamic_sr_zones.pine
git commit -m "feat: improve legend contrast with smart defaults and override"
```

### Task 6: Render Labels Using Y-Adjusted Display Price

**Files:**
- Modify: `src/dynamic_sr_zones.pine` (`label.new` draw loop)
- Test: Manual visual correctness and line integrity

- [ ] **Step 1: Use adjusted display price at label draw only**

```pine
lblY = array.get(allLblPrice, i) + array.get(allLblYAdj, i)
label.new(array.get(allLblBar, i), lblY, array.get(allLblText, i), style=label.style_label_left, color=array.get(allLblColor, i), textcolor=array.get(allLblTxtCol, i), size=labelSz)
```

- [ ] **Step 2: Ensure line draw still uses raw level price**

```pine
// keep existing line.new(..., p, ..., p, ...) using original p
```

- [ ] **Step 3: Validate no level drift**

Run: Visual compare labels vs lines on structural and pivot levels  
Expected: lines remain at true level prices; only label boxes shift for readability.

- [ ] **Step 4: Commit**

```bash
git add src/dynamic_sr_zones.pine
git commit -m "fix: apply structural staggering to labels without moving level lines"
```

### Task 7: Full Manual Verification Sweep

**Files:**
- Modify: none (verification-only)
- Test: TradingView chart matrix from spec

- [ ] **Step 1: Run symbol/timeframe matrix**

Run:
- `RKLB` on `1h`, `4h`, `Daily`
- `ES1!` on `1h`, `4h`, `Daily`
- `SPY` on `1h`, `4h`, `Daily`

Expected:
- legend readable on bright charts
- close structural labels stagger cleanly
- no clipping or extreme displacement

- [ ] **Step 2: Verify hard `+7` cap**

Run: Inspect visually in clustered regions and confirm maximum rightward shift does not exceed seven bars from base label anchor (`bar_index + lineExtend`).  
Expected: constraint holds in all tested contexts.

- [ ] **Step 3: Regression check**

Run:
- toggle `showLabels`
- toggle `showLegend`
- toggle `enableStructAutoStagger`

Expected: no runtime errors; disabling stagger restores near-baseline layout behavior.

- [ ] **Step 4: Final commit**

```bash
git add src/dynamic_sr_zones.pine
git commit -m "test: verify contrast and structural label staggering across symbols"
```

