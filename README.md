# Dynamic Support and Resistance Zones

A TradingView Pine Script indicator that automatically identifies key support and resistance levels using wick-quality scoring, dual-role tracking, volume weighting, and structural anchors.

Published on TradingView: [Dynamic Support and Resistance Zones](https://www.tradingview.com/)

## Project Structure

```
dynamic-sr-zones/
├── src/
│   └── dynamic_sr_zones.pine       # Current production Pine Script (v8)
├── docs/
│   ├── DEVELOPMENT_LOG.md          # Full history of iterations, bugs, decisions
│   ├── METHODOLOGY.md              # Trading methodology this indicator encodes
│   ├── DESCRIPTION.md              # TradingView publication description
│   ├── INPUTS_REFERENCE.md         # Every input parameter explained
│   └── reference/
│       └── adr_transcripts.docx    # ADR trading methodology transcripts
├── archive/
│   ├── iterations/
│   │   └── ITERATION_HISTORY.md    # Summary of all 8 versions
│   └── errors/
│       └── ERROR_LOG.md            # Every Pine Script error encountered + fix
└── README.md
```

## Setup

1. Open TradingView and navigate to Pine Editor (bottom panel)
2. Paste contents of `src/dynamic_sr_zones.pine`
3. Click "Add to Chart"
4. Adjust settings via the indicator gear icon

## Recommended Settings by Trading Style

| Setting | Scalping (1-5m) | Day Trade (15m-1h) | Swing (4h-D) |
|---------|-----------------|--------------------:|---------------|
| Pivot Lookback | 3 | 5 | 10 |
| Levels Above/Below | 3-4 | 4 | 5-6 |
| Min Touches | 1 | 2 | 2-3 |
| Merge Tolerance | 1.0 | 1.5 | 2.0 |

## Key Features

- Wick-based reaction scoring (not just touch counting)
- Dual-role tracking (levels that acted as both S and R get priority)
- Structural anchors (PDH/PDL, PWH/PWL) always visible
- Smart label anti-overlap (merge + stagger)
- Band clustering for nearby levels
- Recency-weighted trimming so old levels don't crowd out new ones
- Volume weighting for pivot significance
- Built-in debug mode for diagnosing issues

## Keeping your clones in sync

Canonical history lives on **GitHub** (`main`). After you push from any machine or worktree, update every other checkout so `HEAD` matches `origin/main`:

```bash
git fetch origin && git pull --ff-only origin main
```

If you use a second folder (for example iCloud plus a local worktree), run that in each repo root after each merge or push. `src/dynamic_sr_zones.pine` should be byte-identical everywhere once pulls finish.

## Future Roadmap

See `docs/DEVELOPMENT_LOG.md` for planned features including:
- Gap detection and display
- Market structure overlay (higher highs / lower lows trend)
- Engulfing candle markers at key levels
- Multi-timeframe level confluence scoring
- Alert conditions when price approaches key levels
