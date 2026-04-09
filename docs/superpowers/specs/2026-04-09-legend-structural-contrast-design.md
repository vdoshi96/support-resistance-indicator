# Dynamic SR Zones: Legend and Structural Contrast Design

## Goal

Address two user-reported visual issues in `Dynamic Support and Resistance Zones`:

1. Legend blends into bright/white chart backgrounds.
2. Structural labels like `PDL` and `PWH` become hard to read when close together.

The design keeps current behavior stable, adds strong defaults, and introduces optional controls for users who want customization.

## Scope

In scope:

- Improve legend readability with stronger default contrast.
- Add hybrid configurability for legend contrast.
- Add structural-label-specific proximity handling so close labels are staggered.
- Enforce hard cap: structural label positive bar offset must never exceed `+7`.

Out of scope:

- Rewriting full label architecture.
- Theme auto-detection beyond current color logic.
- Any trading logic or level-detection changes.

## Current State

The script currently:

- Renders legend table with `bgcolor=color.new(color.gray, 85)`, which can look too faint on light backgrounds.
- Uses a unified anti-overlap mechanism (`lblOverlap`) that staggers labels by horizontal bar offset only.
- Treats structural labels and pivot labels mostly the same in spacing logic, except structural-to-pivot merge tagging.

This works in many cases, but not reliably for clustered structural labels.

## Design Principles

- Preserve line/level truth: never move actual structural level lines.
- Improve readability first with safe defaults.
- Keep tuning optional and minimal.
- Limit offset behavior to avoid unexpected visual drift (`<= +7` bars).

## Proposed Inputs

Add these inputs under display/visual controls:

1. `legendBgAlpha` (`int`, default around `68`, range `0..95`)
   - Lower alpha value increases opacity in Pine color model.
   - Replaces fixed `85` for legend background only.

2. `legendHighContrastText` (`bool`, default `true`)
   - When true, explanatory legend rows use strong foreground color (`color.white`) for readability.
   - Support/Resistance rows still use their semantic colors.

3. `enableStructAutoStagger` (`bool`, default `true`)
   - Enables structural-label-specific spacing behavior.

4. `structLblOverlapPct` (`float`, default around `0.20`, min `0.05`, step `0.05`)
   - Threshold for detecting structural label proximity.

5. `structLblOffsetAtrMult` (`float`, default around `0.08`, min `0.01`, step `0.01`)
   - Vertical spacing offset as ATR fraction for close structural labels.

## Legend Contrast Design

### Behavior

- Legend table uses `legendBgAlpha` instead of hardcoded `85`.
- Header and explanatory lines are rendered with high-contrast text when `legendHighContrastText=true`.
- Keep semantic color coding for:
  - Support line (`supColor`)
  - Resistance line (`resColor`)
  - Structural legend line (`structColor`)

### Rationale

- This preserves meaning while ensuring the legend remains legible on white/light backgrounds.
- Hybrid control supports users who prefer softer styling.

## Structural Label Proximity Design

### Detection

During sorted label pass, identify close structural neighbors:

- both labels are structural (`allLblStruct=true`)
- relative price distance is below `structLblOverlapPct`

### Placement Strategy

When a close structural pair is found:

- Keep both labels (no skipping).
- Apply horizontal offsets to separate callouts.
- Apply vertical offsets in opposite directions:
  - label A: `+priceOffset`
  - label B: `-priceOffset`

Vertical offset:

- `priceOffset = clamp(atrVal * structLblOffsetAtrMult, minTickFloor, maxSafeOffset)`
- Use conservative clamp bounds to prevent excessive shifts.

Horizontal offset:

- Derived from existing bar staggering flow.
- Hard constrain final positive offset: `finalBarOffset = math.min(finalBarOffset, 7)`.

### Cluster Fallback (3+ close structural labels)

For dense groups, use alternating pattern with progressive but bounded horizontal shifts:

- direction pattern: `+,-,+,-`
- horizontal progression increases per label but clamps at `+7`
- maintain vertical ATR-based shift with alternating sign

## Data Flow Changes

No structural schema rewrite required. Reuse existing unified arrays:

- `allLblPrice`
- `allLblBar`
- `allLblStruct`
- `skipLbl`

Add temporary per-label vertical adjustment storage during draw phase:

- e.g. `allLblYAdj` (`float[]`) initialized to `0.0`, applied only at `label.new(...)` call.

Lines continue to draw at original level price `p` without modification.

## Error Handling and Guards

- Guard all loops with size checks (Pine array safety pattern already used).
- If ATR is `na` or zero, fallback to minimum tick-based vertical offset.
- Never let structural staggering produce bar offset beyond `+7`.
- If overlap detection fails or inputs are disabled, behavior falls back to current logic.

## Testing Plan (Manual TradingView)

Symbols:

- `RKLB`
- `ES1!`
- `SPY`

Timeframes:

- `1h`, `4h`, `Daily`

Checks:

1. **Legend readability on light theme**
   - Legend text remains clear; no washed-out rows.
2. **Legend readability on dark theme**
   - Contrast remains acceptable without becoming too heavy.
3. **Close structural pair**
   - `PDL` and `PWH` both visible with top/bottom separation.
4. **Multi-cluster case**
   - 3+ structural labels remain readable and bounded.
5. **Offset bound enforcement**
   - no structural label horizontal shift exceeds `+7`.
6. **Regression**
   - standard S/R labels and band rendering stay unchanged.

## Implementation Touchpoints

File: `src/dynamic_sr_zones.pine`

Edit areas:

- Inputs block (new legend + structural stagger controls)
- Unified label overlap/stagger section
- Label draw call section (apply y-adjusted display value)
- Legend table creation/cell color usage

## Risks and Mitigations

1. **Risk:** Over-staggering creates visual clutter.
   - **Mitigation:** low default ATR multiplier and strict `+7` cap.

2. **Risk:** Theme variance still affects perceived contrast.
   - **Mitigation:** hybrid override inputs for user tuning.

3. **Risk:** Extra logic increases complexity in overlap block.
   - **Mitigation:** isolate structural-specific logic with clear guards and comments.

## Success Criteria

- Legend is readable on white backgrounds without manual tweaking.
- Close structural labels do not visually collide.
- Structural staggering respects `+7` max horizontal nudge.
- No regressions in existing pivot S/R rendering.

