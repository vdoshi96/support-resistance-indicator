# Changelog

## v8.0 - Current Production (2026-04-09)

Full rebuild from scratch incorporating ADR methodology analysis.

### Added
- Wick-based reaction scoring (wick length / ATR at each pivot)
- Dual-role tracking with separate support/resistance touch counters
- Structural anchors: Previous Day High/Low, Previous Week High/Low
- Anti-overlap label system (merge coinciding, stagger close labels)
- Recency factor in trim scoring formula
- Grouped inputs (Level Detection, Display, Structural, Colors, Debug)
- [PWH]/[PDH] tags when structural levels coincide with pivot levels
- Diamond (◆) marker for dual-role levels
- Star (★) marker for high-volume pivots
- Label size selector (Small/Normal/Large)
- Label spacing control for anti-overlap threshold
- Fixed legend using Pine `table` at position.top_right

### Changed
- Defaults: lookback=5, showAbove/Below=4, lineExtend=3
- S/R determined by position relative to price (not tracked type)
- Scoring formula: reaction x dual x recency x volume / (1 + distance%)
- Line width driven by reaction quality + touches (not just touches)
- Label text shows dual-role breakdown (3s/2r) or simple (5x)

### Removed
- Flip logic and flip color
- HTF pivot overlay (structural anchors replaced it)
- Manual zone toggle (bands are now automatic)
- Flip buffer input

## v7.0 (2026-04-09)

### Added
- Dynamic text color (luminance-based: black on light, white on dark)
- Label size dropdown
- Legend label

## v6.0 (2026-04-09)

### Added
- Automatic band clustering (0.1% threshold)
- Removed manual zone toggle

### Changed
- Removed flip tracking entirely
- S/R purely positional (above=R, below=S)

## v5.0 (2026-04-09)

### Added
- Show nearest N above + N below instead of purging distant levels
- maxLevels bumped to 100 for storage
- Distance-penalized trim scoring

### Fixed
- Zombie levels at old prices hogging all storage slots

## v4.0 (2026-04-09)

### Added
- Debug mode with comprehensive counters
- Volume filter toggle

### Fixed
- Volume gate blocking all pivots when avgVol was na
- Diagnosed root cause: old levels with high touch x volume scores surviving trim

## v3.0 (2026-04-09)

### Added
- Unified S/R arrays with type flag
- Auto-flip (support breaks become resistance)
- Volume filter and volume-weighted line thickness
- Flip buffer (ATR-based threshold)
- Smart trimming (weakest score removed, not oldest)

### Fixed
- line.new() crash when origin bar too far back (clamped to 500)

## v2.0 (2026-04-09)

### Added
- ATR-scaled zones
- Touch merging (nearby pivots combine)
- Higher timeframe overlay
- Configurable colors, lookback, max levels

### Fixed
- request.security() inside if block (moved to global scope)
- array.get() on empty array (added size guard)
- Upgraded Pine v5 to v6

## v1.0 (2026-04-09)

Initial version. Simple pivot detection with `extend.right` lines.
