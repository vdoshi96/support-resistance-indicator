# Architecture

## Data Flow

```
Candle Data
  |
  v
Pivot Detection (ta.pivothigh / ta.pivotlow)
  |
  v
Wick Quality Measurement (wick length / ATR)
  |
  v
Volume Check (optional filter)
  |
  v
Merge or Add to Level Arrays
  |
  v
Trim to maxLevels (distance-penalized scoring)
  |
  v
Select nearest N above + N below current price
  |
  v
Touch filter (min touches to display)
  |
  v
Sort by price, cluster into bands
  |
  v
Build unified label list (structural + pivot)
  |
  v
Anti-overlap: merge coinciding labels, stagger close ones
  |
  v
Draw lines, boxes, labels
```

## Level Arrays (parallel arrays)

All level data is stored in synchronized parallel arrays. Pine Script does not support structs/objects, so each property is a separate array indexed by the same position.

| Array | Type | Description |
|-------|------|-------------|
| `lvlPrice` | float[] | Average price of the level (weighted by merges) |
| `lvlSupTouches` | int[] | Number of times price bounced UP from this level (swing lows) |
| `lvlResTouches` | int[] | Number of times price rejected DOWN from this level (swing highs) |
| `lvlReaction` | float[] | Accumulated wick quality score (sum of wickLength/ATR at each touch) |
| `lvlBars` | int[] | Bar index of most recent touch (for recency scoring) |
| `lvlVol` | float[] | Max relative volume across all touches (pivotVol / avgVol) |

## Scoring Formula

### Trim Score (determines which levels survive when storage exceeds maxLevels)

```
score = reaction * dualBonus * recency * volume / (1 + distancePct)
```

**Components:**
- `reaction`: Accumulated wick quality. Higher = stronger rejections at this level.
- `dualBonus`: 2.0 if level has both support AND resistance touches, 1.0 otherwise.
- `recency`: `1.0 / (1.0 + (currentBar - lastTouchBar) / 500.0)`. Decays slowly. A level touched 500 bars ago gets score halved.
- `volume`: Max relative volume at any touch. `pivotVolume / SMA(volume, 20)`.
- `distancePct`: `abs(levelPrice - close) / close * 100`. Dividing by `(1 + distPct)` means a level 40% away has score divided by 41.

The weakest-scoring level gets removed first. This naturally favors recent, nearby, strongly-reacted, dual-role, high-volume levels.

### Display Line Width

```
totalTouches = supTouches + resTouches
baseWidth = totalTouches >= 6 ? 3 : totalTouches >= 3 ? 2 : 1
volumeBoost = relativeVolume >= 2.0 ? 1 : 0
lineWidth = min(baseWidth + volumeBoost, 4)
```

### Label Text Construction

```
For dual-role levels:  "68.61 S (27s/18r) ◆ ★"
For single-role:       "73.98 R (5x) ★"
With structural merge: "68.61 S (27s/18r) ◆ ★ [PDH]"
```

## Structural Levels

Fetched via `request.security()` at global scope (Pine requirement):
- `pdHigh / pdLow`: Previous day high/low (`"D"` timeframe, `high[1]`/`low[1]`)
- `pwHigh / pwLow`: Previous week high/low (`"W"` timeframe, `high[1]`/`low[1]`)

These use `lookahead=barmerge.lookahead_on` to get the confirmed previous period values, not the current forming period.

## Band Clustering

After selecting visible levels and filtering by touch count:
1. Sort candidates by price ascending
2. Walk through sorted list
3. If next level is within `bandPct%` of current cluster anchor, absorb it
4. Cluster inherits the anchor price of the highest-scoring member
5. Band box spans from lowest to highest price in the cluster
6. Single-level "clusters" draw a line only (no box)

## Anti-Overlap Label System

1. All labels (structural + pivot) collected into one unified array
2. Sorted by price ascending
3. **Merge pass**: If a structural label is within `lblOverlap%` of a pivot label, the structural tag (PDH/PDL/PWH/PWL) gets appended to the pivot label as `[tag]`, and the structural label is skipped
4. **Stagger pass**: Any remaining adjacent labels closer than `lblOverlap%` get their bar position offset by +8 to spread horizontally

## Pine Script Constraints

- `request.security()` must be at global scope (not inside if/for)
- `line.new()` x1 must be within ~500 bars of current bar_index
- Max 500 lines, 500 labels, 500 boxes per indicator
- No structs/objects -- parallel arrays only
- No string interpolation -- concatenation with `+` operator
- Array size must be checked before accessing index 0
- `var` keyword for persistent arrays (survive across bars)
