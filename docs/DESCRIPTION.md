# TradingView Publication

## Title
Dynamic Support and Resistance Zones

## Description

This indicator automatically identifies key price levels where an asset has historically bounced or been rejected, using a methodology inspired by professional intraday and swing trading workflows.

Levels are detected through pivot-based swing analysis with wick-quality scoring. Rather than simply counting how many times price touched a level, the script measures how strongly price reacted at each pivot by analyzing wick length relative to volatility. A long-wick rejection at a level scores significantly higher than a small pivot inside a consolidation zone, producing levels that more closely match what an experienced trader would draw by hand.

Each level tracks support and resistance touches separately. Levels that have acted as both support and resistance in the past are flagged with a diamond marker and receive priority in the scoring system. These dual-role levels are the highest quality zones because the market has confirmed their importance from both directions.

The script automatically plots previous day high and low and previous week high and low as dashed structural reference lines. These anchors frame the key range before any pivot analysis begins. When a structural level coincides with a detected pivot level, the labels merge into a single tag to keep the chart clean.

A smart label system prevents overlapping text. Labels that land too close together either merge (if one is structural and the other is a pivot at the same price) or stagger horizontally so every level remains readable. The spacing threshold is adjustable.

Scoring prioritizes recency, reaction quality, volume, and dual-role history. Levels far from the current price are retained in memory but hidden until price approaches them again. Only the nearest levels above and below the current price are displayed at any time, keeping charts uncluttered while preserving the full history.

Nearby levels within a configurable percentage automatically cluster into shaded bands, reflecting the reality that support and resistance are zones rather than exact lines. Line thickness scales with the combined reaction strength and touch count so the most significant levels stand out visually. High-volume pivots are marked with a star.

All settings are fully customizable including pivot sensitivity, merge tolerance, visible level count, label size, band clustering threshold, volume filtering, and colors. A built-in legend explains every visual element. Works on any ticker, any timeframe, and adapts automatically using Average True Range.
