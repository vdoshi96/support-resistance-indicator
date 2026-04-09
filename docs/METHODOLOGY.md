# Trading Methodology

This document captures the level-drawing methodology extracted from ADR's trading transcripts and the user's own swing trading workflow. This is the "ground truth" the indicator aims to replicate.

## Sources

- ADR YouTube: "2026 Path To Profitability: Market Structure Explained"
- ADR YouTube: "Support and Resistance Didn't work Till I Discovered This SECRET"
- ADR YouTube: "Market Structure Explained" (video course)
- ADR YouTube: "The ONLY Candlestick Video You'll EVER Need"
- ADR YouTube: "Intraday Analysis" (first transcript provided in chat)
- User's own hand-drawn RKLB levels as reference

## Core Principles

### 1. Top-Down Multi-Timeframe Approach

Start on the highest timeframe and work down. The user's workflow:
- Weekly candles first (macro structure, key ranges)
- Daily candles (refine levels, identify key reaction zones)
- 1h / 30m candles (fine-tune entry levels)
- Does NOT use 15m or below for drawing levels

ADR's workflow (intraday-focused):
- Daily first (key S/R zones)
- 30m (last week's price action, major levels)
- 5m (entry timeframe)
- 1m (precision entries, watching for wick reactions)

### 2. Level Quality Hierarchy

**High quality levels (prioritize these):**
- Acted as BOTH support AND resistance in the past
- Multiple touches (3+ minimum for ADR)
- Strong wick reactions (long wicks = strong rejection)
- Recent relevance (within visible chart history)

**Low quality levels (deprioritize/ignore):**
- Only touched once
- No data showing past S/R dual role
- Small pivots inside consolidation with no clear rejection

### 3. Structural Anchors (Always Plot These First)

Before any pivot analysis:
- Previous week high and low
- Previous day high and low
- Pre-market high/low (for futures/intraday)
- All-time high (for equities)
- Gaps (ADR marks these in blue)

These are deterministic (no detection needed) and frame the trading range.

### 4. Wick/Reaction Reading

ADR's key insight: "I don't care if the candlestick is red or green. I only care what it's telling me. How the candlestick formed."

- Long wick at bottom = strong rejection of lower prices, buyers stepping in
- Long wick at top = strong rejection of higher prices, sellers stepping in
- Small body with long wicks = indecision
- Large body = strong trend/sentiment
- Engulfing candle = potential direction change

For the indicator: wick length relative to ATR is the quality metric, not just whether a pivot existed.

### 5. Levels Are Zones, Not Lines

ADR: "It is a zone, it is not just a single line."

The indicator implements this through:
- Band clustering: nearby levels merge into shaded zones
- ATR-scaled merge tolerance for initial pivot grouping

### 6. Market Moves Level by Level

ADR: "The market moves level by level. If I enter at support, my price target is the next resistance."

This means:
- Each level is both an entry zone AND a target zone
- Fewer, higher-conviction levels are better than many weak ones
- ADR typically has 4-6 levels on a chart

### 7. Selectivity Over Quantity

ADR: "I will not start looking at very minute levels. I will not be putting lines all over the chart."

The user's own observation: "My hand-drawn levels are more accurate."

Default display: 4 above, 4 below (8 total visible).

### 8. Color Convention

- Green = support (below current price)
- Red = resistance (above current price)
- White/gray dashed = structural levels (PDH/PDL, PWH/PWL)
- Dynamic: a level's color changes as price crosses it (positional S/R)

## What the Indicator Cannot Replicate

1. **Discretionary judgment.** A human trader evaluates the "quality" of a reaction holistically. The indicator approximates this with wick scoring.

2. **News/data awareness.** ADR waits for economic data before trading. The indicator has no concept of scheduled events.

3. **Context from multiple tickers.** ADR checks SPY, QQQ, and individual stocks together. The indicator only sees the current chart.

4. **Pattern recognition.** Rising wedges, head & shoulders, gap fills. These are separate from S/R detection.

5. **Emotional discipline.** ADR's most important point: following the plan. No indicator solves this.
