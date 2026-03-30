# Bottom Level Finder

**Platform**: TradingView
**Language**: Pine Script v6
**Chart Timeframe**: Daily (all modes)
**Modes**: 1D, 5D, 1M, 3M, 6M, 1Y, 5Y
**Theme**: Dark theme (vibrant, high-contrast palette)

---

## Overview

A multi-horizon bottom and support level identification overlay designed for correction analysis. A single horizon mode selector adapts the entire indicator — drawdown granularity, moving average suite, signal thresholds, MA cross pair, and VIX sensitivity — to match TradingView's zoom presets (1D, 5D, 1M, 3M, 6M, 1Y, 5Y).

The indicator layers three independent analytical frameworks onto one chart: Fibonacci retracement levels (anchored from ATH to a user-defined or auto-detected cycle low), percentage drawdown levels from ATH, and a mode-adaptive moving average suite with cross detection. A six-factor bottom signal scorecard evaluates structural, momentum, fear, and volume conditions to assess proximity to a durable bottom.

The indicator does not generate buy/sell signals. It provides a structured framework of price levels and condition monitoring to inform re-entry timing decisions during corrections. All reference levels, line styles, and dashboard settings are configurable.

---

## Table of Contents

- [Features](#features)
- [Chart Setup](#chart-setup)
- [Installation](#installation)
- [Indicator Components](#indicator-components)
  - [ATH and Swing Low Detection](#ath-and-swing-low-detection)
  - [Fibonacci Retracement Levels](#fibonacci-retracement-levels)
  - [Drawdown Levels from ATH](#drawdown-levels-from-ath)
  - [Mode-Adaptive Moving Averages](#mode-adaptive-moving-averages)
  - [MA Cross Detection](#ma-cross-detection)
  - [VIX Fear Gauge](#vix-fear-gauge)
  - [Bottom Signal Scorecard](#bottom-signal-scorecard)
  - [Dashboard](#dashboard)
- [Visual Reference](#visual-reference)
  - [Color Map](#color-map)
  - [Chart Element Legend](#chart-element-legend)
  - [Dashboard Fields](#dashboard-fields)
- [Mode Reference](#mode-reference)
  - [Drawdown Levels by Mode](#drawdown-levels-by-mode)
  - [Moving Averages by Mode](#moving-averages-by-mode)
  - [Signal Thresholds by Mode](#signal-thresholds-by-mode)
- [Tuning Guide](#tuning-guide)
  - [Swing Low Reference](#swing-low-reference)
  - [ATH Override](#ath-override)
  - [Fibonacci Line Style](#fibonacci-line-style)
  - [Drawdown Line Style](#drawdown-line-style)
  - [Label Positioning](#label-positioning)
  - [VIX Symbol](#vix-symbol)
  - [Dashboard Position and Size](#dashboard-position-and-size)
- [Interpreting the Chart](#interpreting-the-chart)
  - [Confluence Zones](#confluence-zones)
  - [Zone Classification](#zone-classification)
  - [Bottom Proximity Assessment](#bottom-proximity-assessment)
- [Architecture Notes](#architecture-notes)
- [Known Limitations](#known-limitations)
- [Changelog](#changelog)

---

## Features

- **Seven horizon modes** (1D, 5D, 1M, 3M, 6M, 1Y, 5Y) that reconfigure the entire indicator via a single dropdown — drawdown steps, MA suite, cross pair, and signal thresholds all adapt.
- **Fibonacci retracement levels** (23.6%, 38.2%, 50.0%, 61.8%, 78.6%) computed from ATH to a configurable swing low reference, shared across all modes. Plotted as colored dashed lines with price labels.
- **Percentage drawdown levels** from ATH with mode-adaptive granularity. Five levels per mode, ranging from -0.5% steps (1D) to -10% steps (5Y). Plotted as colored dotted lines.
- **Four mode-adaptive moving averages** plotted on the chart — from 5D/10D/20D/50D SMA (short modes) to 200D/50W/100W/200W SMA (long modes).
- **MA cross detection** with mode-appropriate pair selection: 10/20 (1D–1M), 20/50 (3M), 50/200 (6M–5Y). Dashboard shows bullish/bearish status and bars since last cross.
- **VIX integration** with six-tier classification (LOW through EXTREME) and mode-dependent signal threshold (> 20 for short modes, > 30 for long modes).
- **Six-factor bottom signal scorecard** evaluating RSI oversold, Bollinger %B breach, momentum turn, MA support test, VIX elevation, and volume capitulation. Each factor adapts its data source and threshold to the selected mode.
- **Bottom Proximity assessment** summarizing scorecard state: NONE (0/6), LOW (1/6), MODERATE (3/6), ELEVATED (4/6), EXTREME (5–6/6).
- **Comprehensive dashboard** with level ladder (price + distance), signal status, and summary — padded for clean presentation.
- **ATH auto-detection** from loaded chart data with manual override for custom reference points.

---

## Chart Setup

1. Set the **chart timeframe** to **Daily (D)** in the top toolbar. This controls what each candle represents and what all MA calculations mean.
2. Use the **bottom toolbar zoom presets** (1D, 5D, 1M, 3M, 6M, 1Y, 5Y) to adjust the visible range.
3. Set the indicator's **Horizon Mode** dropdown to match your current zoom preset.

The zoom presets only scroll the viewport — they do not change any indicator math. The Horizon Mode selector controls which parameter set the indicator uses.

---

## Installation

1. Open a chart in [TradingView](https://www.tradingview.com/).
2. Open the Pine Editor (bottom panel).
3. Copy the contents of `bottom-level-finder.pine` and paste it into the editor.
4. Click **Add to chart**.
5. Set the chart timeframe to **Daily (D)**.
6. Open indicator settings and configure:
   - **Horizon Mode**: Match your zoom preset.
   - **Swing Low Reference**: Set to a meaningful cycle low (e.g., 348 for SPY Oct-2022, 218 for COVID-2020 low). Leave at 0 for auto-detect.
   - **ATH Price Override**: Leave at 0 for auto-detection (recommended for most use cases).

---

## Indicator Components

### ATH and Swing Low Detection

The indicator tracks two reference prices used as anchors for all level calculations:

- **All-Time High (ATH)**: Auto-detected as the highest high across all loaded bars. Override via `ATH Price Override` input if needed (e.g., to measure from a local swing high rather than the true ATH).
- **Swing Low Reference**: Auto-detected as the lowest low across all loaded bars. Override via `Swing Low Reference` input — this is strongly recommended for meaningful Fibonacci levels.

Auto-detected values depend on how much history TradingView loads for the current chart. For major tickers like SPY on a daily chart, this typically goes back 20+ years.

### Fibonacci Retracement Levels

Five standard Fibonacci retracement levels computed from ATH down to the swing low reference:

| Level | Calculation | Color | Typical Interpretation |
|-------|-------------|-------|----------------------|
| 23.6% | ATH - span * 0.236 | Light Blue | Shallow pullback |
| 38.2% | ATH - span * 0.382 | Blue | Standard correction |
| 50.0% | ATH - span * 0.500 | Yellow | Deep correction |
| 61.8% | ATH - span * 0.618 | Orange | Bear market territory |
| 78.6% | ATH - span * 0.786 | Red-Orange | Recession / structural breakdown |

Where `span = ATH - swingLow`.

These levels are shared across all modes — the Fibonacci structure represents the full cycle regardless of zoom level.

### Drawdown Levels from ATH

Five percentage drawdown levels computed as `ATH * (1 - pct)`. The percentages adapt per mode to match the granularity most useful at each horizon:

| Slot | 1D | 5D | 1M | 3M | 6M | 1Y / 5Y |
|------|----|----|----|----|----|----|
| Level 1 | -0.5% | -1% | -2% | -3% | -5% | -10% |
| Level 2 | -1% | -2% | -5% | -5% | -10% | -15% |
| Level 3 | -2% | -3% | -8% | -10% | -15% | -20% |
| Level 4 | -3% | -5% | -10% | -15% | -20% | -25% |
| Level 5 | -5% | -8% | -15% | -20% | -25% | -30% |

### Mode-Adaptive Moving Averages

Four MA slots are plotted on the chart, with the specific SMA assigned to each slot depending on the selected mode:

| Slot | 1D | 5D | 1M | 3M | 6M | 1Y | 5Y |
|------|----|----|----|----|----|----|-----|
| MA 1 (thin) | 5D | 5D | 5D | 10D | 20D | 50D | 200D |
| MA 2 | 10D | 10D | 10D | 20D | 50D | 100D | 50W |
| MA 3 | 20D | 20D | 20D | 50D | 100D | 200D | 100W |
| MA 4 (thick) | 50D | 50D | 50D | 100D | 200D | 50W | 200W |

Daily MAs (5D through 200D) are computed directly on chart data. Weekly MAs (50W, 100W, 200W) are fetched via `request.security()` on the "W" timeframe.

### MA Cross Detection

Three cross pairs are tracked simultaneously, with the mode determining which is displayed:

| Modes | Cross Pair | Signal |
|-------|-----------|--------|
| 1D, 5D, 1M | 10/20 SMA | Short-term trend shift |
| 3M | 20/50 SMA | Intermediate trend shift |
| 6M, 1Y, 5Y | 50/200 SMA | Golden Cross / Death Cross |

The dashboard shows the current regime (BULLISH or BEARISH) and bars since the last crossover.

### VIX Fear Gauge

VIX close and its 20-period SMA are fetched via `request.security()` (configurable symbol, default `CBOE:VIX`). The dashboard displays the current VIX value and a six-tier classification:

| Tier | VIX Range |
|------|-----------|
| LOW | < 15 |
| NORMAL | 15–20 |
| ELEVATED | 20–25 |
| HIGH | 25–30 |
| VERY HIGH | 30–35 |
| EXTREME | > 35 |

### Bottom Signal Scorecard

Six independent conditions are evaluated each bar. Each factor adapts its data source, indicator, and threshold based on the selected mode.

**Signal 1 — RSI Oversold:**

| Modes | Source | Threshold |
|-------|--------|-----------|
| 1D | Daily RSI(14) | < 20 |
| 5D, 1M | Daily RSI(14) | < 25 |
| 3M, 6M | Daily RSI(14) | < 30 |
| 1Y, 5Y | Weekly RSI(14) | < 30 |

**Signal 2 — Bollinger %B Below Lower Band:**

| Modes | Source | Threshold |
|-------|--------|-----------|
| 1D–6M | Daily BB(20, 2.0) %B | < 0.0 |
| 1Y, 5Y | Weekly BB(20, 2.0) %B | < 0.0 |

**Signal 3 — Momentum Turn:**

| Modes | Indicator | Condition |
|-------|-----------|-----------|
| 1D, 5D, 1M | Daily MACD(12,26,9) Histogram | Negative and rising (inflecting up) |
| 3M, 6M | Weekly RSI(14) | < 40 (intermediate weakness) |
| 1Y, 5Y | Monthly MACD(12,26,9) Histogram | Negative and rising (cycle inflection) |

**Signal 4 — Price Below Key MA:**

| Mode | Reference MA |
|------|-------------|
| 1D | 10D SMA |
| 5D | 20D SMA |
| 1M | 50D SMA |
| 3M | 100D SMA |
| 6M, 1Y | 200D SMA |
| 5Y | 200W SMA |

**Signal 5 — VIX Elevated:**

| Modes | VIX Threshold |
|-------|--------------|
| 1D, 5D | > 20 |
| 1M, 3M | > 25 |
| 6M, 1Y, 5Y | > 30 |

**Signal 6 — Volume Capitulation:**

Constant across all modes: current bar volume > 2x the 20-period volume SMA.

### Dashboard

Rendered on `barstate.islast` only. 5-column, up to 50-row table with padding spacers on all four sides. Three data columns display level names, values, and distance percentages.

**Sections (top to bottom):**

1. Header (mode, ticker, timeframe)
2. Current status (close, drawdown from ATH)
3. MA cross status (pair, regime, bars since cross)
4. VIX (value + tier classification)
5. Drawdown level ladder (5 rows: label, price, distance %)
6. Fibonacci level ladder (5 rows: label, price, distance %)
7. Moving average ladder (4 rows: name, price, distance %)
8. Bottom signals (6 rows: name, value, FIRED/—)
9. Bottom Proximity assessment (summary)

Color coding: green when price is above a level, the level's native color when price is below. FIRED signals show in red (bearish conditions) or green (bullish momentum turn).

---

## Visual Reference

### Color Map

| Element | Color | Hex |
|---------|-------|-----|
| Bull / Above Level | Green | `#00E676` |
| Bear / Below Level | Red | `#FF1744` |
| Neutral | Silver | `#B0BEC5` |
| Warning | Yellow | `#FFD600` |
| Caution | Orange | `#FF9100` |
| ATH Line | Purple | `#7C4DFF` |
| Fib 23.6% | Light Blue | `#4FC3F7` |
| Fib 38.2% | Blue | `#29B6F6` |
| Fib 50.0% | Yellow | `#FFD600` |
| Fib 61.8% | Orange | `#FF9100` |
| Fib 78.6% | Red-Orange | `#FF5722` |
| Drawdown 1 (shallowest) | Pale Green | `#C8E6C9` |
| Drawdown 2 | Light Green | `#A5D6A7` |
| Drawdown 3 | Gold | `#FFD54F` |
| Drawdown 4 | Amber | `#FFB74D` |
| Drawdown 5 (deepest) | Coral | `#E57373` |
| MA 1 (shortest) | Green | `#81C784` |
| MA 2 | Teal | `#26A69A` |
| MA 3 | Blue | `#42A5F5` |
| MA 4 (longest) | Red | `#EF5350` |
| Dashboard BG | Dark Navy | `#1A1A2E` |
| Dashboard Header | Navy | `#2A2A4E` |

### Chart Element Legend

| Visual | Element | Style |
|--------|---------|-------|
| Purple solid line | ATH reference level | `line.style_solid`, width 2 |
| Colored dashed lines | Fibonacci retracement levels | Configurable style/width |
| Colored dotted lines | Drawdown levels from ATH | Configurable style/width |
| Green thin curve | MA Slot 1 (shortest period) | linewidth 1 |
| Teal curve | MA Slot 2 | linewidth 2 |
| Blue curve | MA Slot 3 | linewidth 2 |
| Red thick curve | MA Slot 4 (longest period) | linewidth 3 |
| Labels at line endpoints | Level name + price | `label.style_label_left` |

### Dashboard Fields

| Field | Description |
|-------|-------------|
| Close | Current bar close price |
| Drawdown | Percentage decline from ATH |
| XX/YY SMA | MA cross pair status (BULLISH/BEARISH) + bars since cross |
| VIX | Current VIX value + tier classification |
| Drawdown rows | Each shows: label, price level, distance from current price |
| Fibonacci rows | Each shows: retracement %, price level, distance from current price |
| MA rows | Each shows: MA name, current value, distance from current price |
| Signal rows | Each shows: condition name, current value, FIRED or — |
| Bottom Proximity | Summary assessment: NONE / LOW / MODERATE / ELEVATED / EXTREME |

---

## Mode Reference

### Drawdown Levels by Mode

| Slot | 1D | 5D | 1M | 3M | 6M | 1Y / 5Y |
|------|----|----|----|----|----|----|
| 1 | -0.5% | -1% | -2% | -3% | -5% | -10% |
| 2 | -1% | -2% | -5% | -5% | -10% | -15% |
| 3 | -2% | -3% | -8% | -10% | -15% | -20% |
| 4 | -3% | -5% | -10% | -15% | -20% | -25% |
| 5 | -5% | -8% | -15% | -20% | -25% | -30% |

### Moving Averages by Mode

| Slot | 1D | 5D | 1M | 3M | 6M | 1Y | 5Y |
|------|----|----|----|----|----|----|-----|
| MA 1 | 5D | 5D | 5D | 10D | 20D | 50D | 200D |
| MA 2 | 10D | 10D | 10D | 20D | 50D | 100D | 50W |
| MA 3 | 20D | 20D | 20D | 50D | 100D | 200D | 100W |
| MA 4 | 50D | 50D | 50D | 100D | 200D | 50W | 200W |

### Signal Thresholds by Mode

| Signal | 1D | 5D | 1M | 3M | 6M | 1Y | 5Y |
|--------|----|----|----|----|----|----|-----|
| RSI source | Daily | Daily | Daily | Daily | Daily | Weekly | Weekly |
| RSI threshold | < 20 | < 25 | < 25 | < 30 | < 30 | < 30 | < 30 |
| BB %B source | Daily | Daily | Daily | Daily | Daily | Weekly | Weekly |
| Momentum signal | D MACD | D MACD | D MACD | W RSI < 40 | W RSI < 40 | M MACD | M MACD |
| Key MA test | 10D | 20D | 50D | 100D | 200D | 200D | 200W |
| VIX threshold | > 20 | > 20 | > 25 | > 25 | > 30 | > 30 | > 30 |
| MA cross pair | 10/20 | 10/20 | 10/20 | 20/50 | 50/200 | 50/200 | 50/200 |

---

## Tuning Guide

### Swing Low Reference

The most impactful setting. This anchors the Fibonacci retracement levels.

- **0 (auto-detect)**: Uses the lowest low in all loaded chart data. For SPY, this typically goes back to the 1990s (~$23), which produces absurdly wide fib levels that are analytically meaningless for current corrections.
- **348**: SPY's October 2022 bear market low — the most relevant cycle anchor for the current bull run. Produces fib levels that cluster meaningfully around correction territory.
- **218**: SPY's March 2020 COVID crash low — useful for analyzing deeper structural support if the correction evolves into a bear market.
- **For SPX**: Use 3491 (Oct-2022) or 2191 (COVID).
- **For other tickers**: Set to the most recent major cycle low visible on the chart.

### ATH Override

Leave at 0 for most use cases. Auto-detection scans all loaded bars and tracks the highest high.

Override if you want to measure drawdowns from a reference point other than the true ATH — for example, a local swing high after a partial recovery, to analyze the subsequent decline structure.

### Fibonacci Line Style

Default: Dashed, width 1. For multi-year zoomed-out views, increase width to 2 for readability. Solid style works well when drawdown levels are hidden (avoids visual confusion between the two level sets).

### Drawdown Line Style

Default: Dotted, width 1. The dotted style visually distinguishes drawdown levels from Fibonacci levels (dashed). Consider hiding drawdown levels entirely (uncheck "Show Drawdown Levels") for cleaner charts when using Fibonacci as the primary support framework, or vice versa.

### Label Positioning

Line labels are positioned at `bar_index - 5` (5 bars to the left of the last bar). To adjust:

- Move labels further left: decrease the offset (e.g., `bar_index - 20`)
- Move labels to the right of the last bar: use a positive offset (e.g., `bar_index + 5`)
- Label transparency: modify the `color.new()` second parameter (0 = opaque, 100 = invisible). Default is 50.
- Label size: change `size.small` to `size.normal` for larger text.

### VIX Symbol

Default: `CBOE:VIX`. Alternative volatility indices can be substituted for different underlyings (e.g., `CBOE:VXN` for NASDAQ, `CBOE:RVX` for Russell 2000). The VIX data is fetched on the chart's timeframe.

### Dashboard Position and Size

Position options: Top Right, Top Left, Bottom Right, Bottom Left. Default: Bottom Left.

Text size options: Tiny, Small, Normal. Default: Small. Use Tiny for multi-chart layouts where screen real estate is constrained.

---

## Interpreting the Chart

### Confluence Zones

The highest-probability support areas are where multiple independent level types cluster within a narrow price range (1-3%). Look for zones where a Fibonacci level, a drawdown level, and a moving average converge. These confluence zones represent areas where multiple analytical frameworks independently identify the same price as significant.

### Zone Classification

When viewing with Fibonacci levels on a cycle-relevant swing low (e.g., SPY 348):

| Zone | Fibonacci Range | Character |
|------|----------------|-----------|
| Shallow Pullback | Above Fib 23.6% | Normal correction; bull market health check |
| Standard Correction | Fib 23.6% → Fib 38.2% | Most common reversal zone in secular bull markets |
| Deep Correction | Fib 38.2% → Fib 50.0% | Half the prior rally retraced; macro catalyst likely |
| Bear Market | Fib 50.0% → Fib 61.8% | Structural trend may be broken; death cross likely |
| Recession / Structural | Below Fib 61.8% | Systemic crisis; repricing toward cycle origin |

### Bottom Proximity Assessment

The scorecard summary reflects how many of the six bottom-associated conditions are currently active:

| Assessment | Signals | Interpretation |
|------------|---------|---------------|
| NONE | 0/6 | No bottom conditions present; trend intact |
| LOW | 1–2/6 | Early stress; correction underway but not extreme |
| MODERATE | 3/6 | Multiple conditions firing; correction is real and sentiment stressed |
| ELEVATED | 4/6 | Significant bottom conditions present; approaching durable support |
| EXTREME | 5–6/6 | Maximum bottom signal confluence; historically marks durable reversals |

Key signal to watch: the **momentum turn** (Signal 3). When MACD histogram or weekly RSI inflects upward while still in negative/oversold territory, it indicates selling momentum is decelerating — the classic bottoming signature. This signal firing alongside 3+ others provides the highest conviction for re-entry timing.

---

## Architecture Notes

- **`request.security()` budget**: 3 calls total, well under the 40-call limit.
  1. Weekly: 50W/100W/200W SMA + RSI(14) + BB %B — 1 tuple call.
  2. Monthly: MACD(12,26,9) histogram current + prior bar — 1 tuple call.
  3. VIX: close + 20-period SMA on chart timeframe — 1 call.
- **`ta.*` global scope**: All `ta.sma()`, `ta.rsi()`, `ta.bb()`, `ta.macd()` calls execute at global scope on every bar, as required by Pine v6. Mode selection happens downstream via `switch` expressions that pick from pre-computed values.
- **Type inference**: Variables assigned from `switch` expressions and `input`-dependent ternaries use type inference (no explicit `int`/`float` annotations) to avoid Pine v6's strict `const` vs `input` vs `series` type qualification conflicts.
- **Object budget**: Up to 12 lines (1 ATH + 5 Fib + 5 Drawdown + headroom) and 12 labels. Well under TradingView's 50-line / 50-label limits. Delete-and-recreate pattern on `barstate.islast`.
- **Dashboard**: 5-column, 50-row table with column 0 (left spacer), columns 1–3 (data), column 4 (right spacer). Row 0 is a top spacer, final row is a bottom spacer. Rendered only on `barstate.islast`.
- **No repainting**: All level calculations are based on confirmed bar data. `request.security()` calls use `lookahead = barmerge.lookahead_off`.
- **MA cross tracking**: Three cross pairs (10/20, 20/50, 50/200) are tracked simultaneously using `var int` bar counters. The mode selects which pair to display, but all three compute on every bar for instant switching.

---

## Known Limitations

- **Daily chart optimized**: While the indicator will technically run on any timeframe, all MA lengths, signal thresholds, and drawdown granularity are calibrated for daily bars. Running on weekly or intraday charts will produce incorrect MA periods and signal behavior.
- **ATH auto-detection depends on loaded history**: If TradingView doesn't load enough bars to capture the true ATH (rare for major tickers on daily charts), the auto-detected value will be incorrect. Use the override input in this case.
- **Swing Low auto-detection is usually wrong**: Auto-detect picks the lowest low across all loaded data, which for long-history tickers like SPY goes back decades. This produces meaningless Fibonacci levels. Manual override to a cycle-relevant low (e.g., 348 for SPY Oct-2022) is strongly recommended.
- **Fibonacci levels are range-based, not pivot-based**: The fib calculation uses a static ATH and swing low reference, not algorithmically detected swing pivots. This is intentional — it provides stable, non-repainting levels, but may not match hand-drawn fibs on intermediate swings.
- **1D and 5D modes show very few candles**: At daily resolution, the 1D zoom shows 1 bar and 5D shows 5 bars. These modes are most useful for checking dashboard readings and level proximity rather than visual chart analysis.
- **VIX data requires the symbol to be available**: If the configured VIX symbol is not available on the user's data plan, VIX-related dashboard fields and signals will show N/A.
- **No alert system**: Unlike other indicators in the suite, the Bottom Level Finder does not include `alertcondition()` or `alert()` calls. It is designed for visual chart analysis, not automated notification workflows.
- **Weekly and monthly indicators may show N/A initially**: The 50W SMA requires 50 weeks (~1 year) of weekly data; the 200W SMA requires ~4 years. If insufficient history is loaded, these values will be N/A and the corresponding bottom signals will not fire.
