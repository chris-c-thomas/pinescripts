# SPX Intraday Analyst

**Platform**: TradingView
**Language**: Pine Script v6
**Chart**: SPX (CBOE) — 1-minute, 5-minute, 10-minute, 15-minute (adaptive)
**Theme**: Dark theme (vibrant, high-contrast palette)

---

## Overview

A single-overlay indicator purpose-built for intraday trading of the S&P 500 Cash Index (SPX) from CBOE on TradingView. SPX has **no volume data** on TradingView — this indicator is architected entirely around that constraint. Every component is either price-derived or pulled from external companion symbols (VIX, NYSE TICK).

Volume-dependent tools used in the rest of the indicator suite (VWAP, OBV, relative volume, footprint) are replaced with purpose-built alternatives:

| Replaced Component | SPX Substitute | Rationale |
|:-------------------|:---------------|:----------|
| Session-anchored VWAP | **Session TWAP** (time-weighted avg price) with std dev bands | Provides session fair-value reference without volume. Equal time weighting instead of volume weighting. |
| OBV trend confirmation | **VIX context module** (level + slope + intraday change) | VIX is SPX's implied volatility — provides fear/complacency context that OBV gives on volume-capable instruments. |
| Relative volume | **ATR percentile + BB width percentile** | Volatility regime classification replaces volume regime classification. |
| Volume footprint | **Stochastic K/D** added as fast momentum confirmation | Fills the confirmation slot that footprint delta occupied in SPY variants. |

The indicator auto-detects the chart timeframe and adapts opening range duration, ATR lookbacks, divergence sensitivity, signal cooldown, and HTF anchor mapping accordingly.

---

## Table of Contents

- [Features](#features)
- [Design Philosophy](#design-philosophy)
- [Installation](#installation)
- [Indicator Components](#indicator-components)
  - [EMA Ribbon](#ema-ribbon)
  - [HTF Anchor EMA](#htf-anchor-ema)
  - [Session TWAP and Bands](#session-twap-and-bands)
  - [RSI](#rsi)
  - [MACD](#macd)
  - [ADX / DMI](#adx--dmi)
  - [Stochastic K/D](#stochastic-kd)
  - [ATR / Volatility Regime](#atr--volatility-regime)
  - [Bollinger Band Width](#bollinger-band-width)
  - [TTM Squeeze](#ttm-squeeze)
  - [NYSE TICK Index](#nyse-tick-index)
  - [VIX Context Module](#vix-context-module)
  - [Key Price Levels](#key-price-levels)
  - [Session Phase Detection](#session-phase-detection)
  - [Regime Classifier](#regime-classifier)
  - [Scoring Engine](#scoring-engine)
  - [Signal Engine](#signal-engine)
  - [Dashboard](#dashboard)
  - [Alerts](#alerts)
- [Timeframe Adaptation](#timeframe-adaptation)
- [Tuning Guide](#tuning-guide)
  - [Scoring Mode](#scoring-mode)
  - [Minimum Signal Score](#minimum-signal-score)
  - [VIX Thresholds](#vix-thresholds)
  - [TWAP Band Multiplier](#twap-band-multiplier)
  - [Signal Cooldown](#signal-cooldown)
  - [Tuning Profiles](#tuning-profiles)
- [Architecture Notes](#architecture-notes)
- [Known Limitations](#known-limitations)
- [Changelog](#changelog)

---

## Features

- **EMA Ribbon** (9/21/50) with dynamic cloud fill.
- **Session-anchored TWAP** with configurable standard deviation bands — the volume-free substitute for VWAP.
- **HTF Anchor EMA** pulled via `request.security()` with auto-adaptive timeframe mapping.
- **VIX Context Module** — real-time fear gauge integration with level tiers, slope analysis, and intraday % change. Unique to this SPX-specific indicator.
- **RSI with pivot-based divergence detection** — bullish/bearish divergence scanning with timeframe-adaptive sensitivity.
- **MACD momentum** — histogram direction and signal line crossover state.
- **ADX / DMI** with slope analysis and 4-tier strength classification.
- **Stochastic K/D** — fast momentum confirmation filling the slot vacated by volume footprint.
- **TTM Squeeze** (BB/KC compression) — pure price-based, no volume dependency.
- **NYSE TICK Index** — 6-tier market breadth classification with SMA smoothing.
- **ATR percentile ranking** — LOW / NORMAL / HIGH / EXTREME regime classification.
- **Bollinger Band width percentile** — compression detection for breakout anticipation.
- **Session levels**: Prior Day H/L/C, Pre-Market H/L, Opening Range, Session HOD/LOD.
- **Session phase detection** — 6 phases from Open Drive through Close.
- **Weighted scoring engine** (-13 to +13) with 3 structural + 7 confirmation conditions.
- **Signal generation** (CALLS / PUTS) with configurable cooldown and ADX gate.
- **Regime classifier** (BULLISH / BEARISH / RANGING / NO TRADE).
- **24-row heads-up dashboard** with color-coded indicators.
- **10 static alert conditions** + dynamic context-rich alerts.

---

## Design Philosophy

SPX is the cash index — it represents the actual S&P 500 value, not an ETF wrapper. The critical difference from SPY is the **complete absence of volume data**. This isn't a limitation to work around; it's a fundamental constraint that shapes the entire indicator architecture.

Rather than simply disabling volume-based features, this indicator **substitutes** them with SPX-native context:

1. **VIX replaces OBV**: VIX is the implied volatility of SPX options. Where OBV tells you whether volume confirms or contradicts the trend, VIX tells you whether the options market's fear level confirms or contradicts the trend. A rising SPX with falling VIX is the classic "healthy rally" confirmation. A rising SPX with rising VIX is a warning signal.

2. **TWAP replaces VWAP**: Without volume, we can't volume-weight the average price. But a session-anchored time-weighted average price (running mean of typical price since session open) serves the same structural role — a dynamic fair-value line around which price oscillates during the session. The standard deviation bands provide overbought/oversold context relative to the session mean.

3. **Stochastic replaces footprint**: The fast stochastic K/D provides the quick momentum read that footprint delta provided in SPY variants — a "who's in control right now" signal.

4. **ATR/BB width replaces relative volume**: Volatility regime (percentile-ranked ATR) tells you whether conditions are conducive to directional moves, filling the role that relative volume classification played.

---

## Installation

1. Open TradingView and navigate to an SPX chart (symbol: `SP:SPX` or `CBOE:SPX`).
2. Open the Pine Editor (bottom panel).
3. Delete the default script content.
4. Paste the entire `spx_intraday_analyst.pine` script.
5. Click "Add to chart."
6. Set the chart timeframe to 1-minute, 5-minute, 10-minute, or 15-minute.
7. Enable Extended Trading Hours in chart settings for pre-market level detection.

---

## Indicator Components

### EMA Ribbon

Three EMAs (default 9/21/50) with a cloud fill between fast and slow.

- **Bullish alignment**: Fast > Mid > Slow (green cloud)
- **Bearish alignment**: Fast < Mid < Slow (red cloud)
- **Mixed**: Any other configuration (yellow cloud)

The 50 EMA slope (5-bar rate of change) is tracked and contributes to the scoring engine.

### HTF Anchor EMA

A higher-timeframe EMA plotted on the chart via `request.security()`. The HTF timeframe auto-adapts:

| Chart TF | HTF Anchor TF | Reasoning |
|:---------|:--------------|:----------|
| 1-minute | 5-minute | Immediate structure |
| 5-minute | 15-minute | Standard intraday anchor |
| 10-minute | 30-minute | Intermediate structure |
| 15-minute | 60-minute | Session-level structure |

Plotted as a purple step line. Serves as a structural condition in the scoring engine (price above/below).

### Session TWAP and Bands

**Time-Weighted Average Price** — a running mean of typical price (HLC/3) anchored to each RTH session start. Resets daily.

- **TWAP line**: session fair-value reference (white, 2px)
- **Upper band**: TWAP + N * session standard deviation
- **Lower band**: TWAP - N * session standard deviation

The standard deviation is computed incrementally using Welford's online algorithm (via cumulative sum of squares), so it's efficient and accurate even on 1-minute charts with 390 bars per session.

**Interpretation**: price above TWAP = net buyers have controlled the session. Price below = net sellers. The bands provide extension context — similar to VWAP bands but time-weighted.

### RSI

Standard 14-period RSI. Dashboard shows the value with overbought (>70) and oversold (<30) labels. Contributes to scoring at the >55 (bullish) / <45 (bearish) thresholds for trend confirmation rather than reversal detection.

### MACD

Standard 12/26/9 MACD. The signal line crossover state (MACD above or below signal) and histogram value are both displayed. The crossover state is a 1x confirmation condition in the scoring engine.

### ADX / DMI

14-period ADX with DI+/DI- directional indicators. Enhanced with:

- **Slope analysis**: 5-bar rate of change classifies ADX direction as RISING / FLAT / FALLING.
- **Tier classification**: EXTREME (>45), STRONG (>30), MODERATE (>20), WEAK (<20).

ADX + DI direction contributes to scoring only when ADX exceeds the no-trend threshold (default 18), ensuring trend strength must be present before directionality counts.

### Stochastic K/D

14-period stochastic with 3-period K smoothing and 3-period D signal. Provides fast momentum confirmation that the SPY variants get from volume footprint data.

Displayed in the dashboard with overbought (>80) and oversold (<20) labels. Not currently in the scoring engine but provides visual momentum context.

### ATR / Volatility Regime

14-period ATR with percentile ranking over an adaptive lookback window:

| Chart TF | Default Lookback | Calendar Coverage |
|:---------|:----------------|:-----------------|
| 1-minute | 390 bars | ~1 full session |
| 5-minute | 200 bars | ~2.5 sessions |
| 10-minute | 150 bars | ~3.8 sessions |
| 15-minute | 100 bars | ~3.8 sessions |

**Regime tiers**: LOW (<30th pctl), NORMAL (30-70), HIGH (70-90), EXTREME (>90).

### Bollinger Band Width

20-period BB with 2x multiplier. Width percentile is ranked over the same adaptive lookback as ATR.

**States**: COMPRESSED (<10th pctl), NARROW (<30th), NORMAL (30-70), WIDE (>70th), EXPANDED (>90th). Compression is a precursor to breakout moves.

### TTM Squeeze

Detects when Bollinger Bands compress inside Keltner Channels — the classic John Carter TTM Squeeze:

| State | Condition | Dashboard Color |
|:------|:----------|:---------------|
| **SQUEEZE ON** | BB inside KC | Orange |
| **FIRED** | BB just expanded outside KC | Green |
| **OFF** | Normal volatility | Gray |

Squeeze momentum (linear regression of price deviation from BB midline) direction contributes to the scoring engine when a squeeze has recently fired.

### NYSE TICK Index

Real-time NYSE TICK pulled from `USI:TICK` on the 1-minute timeframe. SMA(5) smoothing applied for noise reduction.

| Range | Label | Dashboard Color |
|:------|:------|:---------------|
| > 1000 | EXTREME BULL | Green |
| > 500 | BULLISH | Green (30% transparent) |
| > 200 | LEAN BULL | Green (60% transparent) |
| -200 to 200 | NEUTRAL | Gray |
| < -200 | LEAN BEAR | Red (60% transparent) |
| < -500 | BEARISH | Red (30% transparent) |
| < -1000 | EXTREME BEAR | Red |

Contributes ±1 to the bias score when TICK exceeds the bull/bear thresholds.

### VIX Context Module

**Unique to this SPX indicator.** VIX is the CBOE Volatility Index — the market's real-time implied volatility gauge derived from SPX options prices. It provides context that is not available through price action alone.

**Three VIX dimensions are tracked:**

1. **Level** — current VIX value classified into tiers:

| VIX Value | Tier | Dashboard Color | Market Interpretation |
|:----------|:-----|:---------------|:---------------------|
| < 15 | LOW | Green | Complacency / calm markets |
| 15 - 20 | NORMAL | Gray | Typical volatility |
| 20 - 25 | ELEVATED | Yellow | Increasing uncertainty |
| 25 - 30 | HIGH | Orange | Significant fear |
| > 30 | EXTREME | Red | Panic / crisis conditions |

2. **Slope** — VIX rate of change over a configurable lookback (default 5 bars):
   - RISING (>+0.5): Fear is increasing — bearish context for SPX
   - FLAT: Stable volatility expectations
   - FALLING (<-0.5): Fear is decreasing — bullish context for SPX

3. **Intraday change %** — today's VIX movement vs the daily open. Displayed in the dashboard as a signed percentage.

**Scoring integration**: VIX contributes ±1 to the bias score. Bullish: VIX below the elevated threshold AND slope falling. Bearish: VIX above elevated AND slope rising. This captures the "healthy rally" vs "fear-driven decline" dynamic that is unique to SPX.

### Key Price Levels

#### Pre-Market High / Low

Tracks the highest high and lowest low during the pre-market session (default 0400-0930 ET). Drawn as horizontal lines once RTH begins. Requires Extended Trading Hours enabled in chart settings.

#### Prior Day High / Low / Close

Pulled from the daily timeframe via `request.security()` with `[1]` offset. No repainting risk. Plotted as step lines.

#### Opening Range

Captures the high and low of the first N minutes of RTH. Duration auto-adapts:

| Chart TF | Default OR Duration | Bars |
|:---------|:-------------------|:-----|
| 1-minute | 5 minutes | 5 |
| 5-minute | 15 minutes | 3 |
| 10-minute | 20 minutes | 2 |
| 15-minute | 30 minutes | 2 |

Drawn as horizontal lines once the OR window closes.

#### Session HOD / LOD

Real-time session high and low tracked from RTH open. Dashboard shows position-in-range classification: AT HOD, AT LOD, UPPER, LOWER, MID.

### Session Phase Detection

Classifies current time-of-day into behavioral phases:

| Phase | Time (ET) | Dashboard Color | Character |
|:------|:----------|:---------------|:----------|
| OPEN DRIVE | 9:30 - 10:00 | Orange | High volatility, directional push |
| MORNING | 10:00 - 11:30 | Green | Trend continuation, best signal quality |
| MIDDAY | 11:30 - 14:00 | Gray | Low energy, chop risk |
| POWER HOUR | 14:00 - 15:30 | Cyan | Institutional repositioning |
| CLOSE | 15:30 - 16:00 | Yellow | Final positioning, 0DTE gamma |
| PRE/POST | Outside RTH | Gray | Extended hours |

### Regime Classifier

Composite classifier combining ADX trend strength, EMA alignment, and TWAP position:

| Regime | Conditions | Implication |
|:-------|:----------|:-----------|
| **BULLISH** | ADX >= threshold, EMAs bullish, price above TWAP | Favor long entries |
| **BEARISH** | ADX >= threshold, EMAs bearish, price below TWAP | Favor short entries |
| **RANGING** | ADX >= threshold but mixed EMA/TWAP alignment | Caution — both directions possible |
| **NO TRADE** | ADX below threshold | Low conviction, avoid directional bets |

### Scoring Engine

Weighted composite scoring engine. Architecturally consistent with the Market Monitor 5m and Trend Compass families.

**Structural Conditions (2x weight when weighted mode enabled):**

| # | Condition | Bullish (+2) | Bearish (-2) |
|:--|:----------|:------------|:------------|
| 1 | EMA Stack | Fast > Mid > Slow | Fast < Mid < Slow |
| 2 | Price vs Session TWAP | Above | Below |
| 3 | Price vs HTF Anchor EMA | Above | Below |

**Confirmation Conditions (1x weight always):**

| # | Condition | Bullish (+1) | Bearish (-1) |
|:--|:----------|:------------|:------------|
| 4 | ADX + DI Direction | ADX > threshold AND DI+ > DI- | ADX > threshold AND DI- > DI+ |
| 5 | RSI Position | RSI > 55 | RSI < 45 |
| 6 | MACD vs Signal | MACD above signal | MACD below signal |
| 7 | NYSE TICK | TICK SMA > bull threshold | TICK SMA < bear threshold |
| 8 | Squeeze Momentum | Positive momentum (recently fired) | Negative momentum |
| 9 | VIX Context | VIX < elevated AND slope falling | VIX > elevated AND slope rising |
| 10 | 50 EMA Slope | Positive | Negative |

**Score Ranges (weighted mode, max ±13):**

| Score | Label | Interpretation |
|:------|:------|:--------------|
| +7 to +13 | STRONG BULL | High-conviction bullish. All major components aligned. |
| +4 to +6 | LEAN BULL | Moderate bullish lean. |
| +1 to +3 | WEAK BULL | Slight bullish tilt. Low conviction. |
| 0 | NEUTRAL | No directional bias. |
| -1 to -3 | WEAK BEAR | Slight bearish tilt. |
| -4 to -6 | LEAN BEAR | Moderate bearish lean. |
| -7 to -13 | STRONG BEAR | High-conviction bearish. |

**Score Trend**: Compares current score to its 5-bar SMA. IMPROVING / STABLE / FADING.

### Signal Engine

Generates CALLS and PUTS labels when the composite score exceeds the minimum threshold. Gated by:

1. **`barstate.isconfirmed`** — anti-repaint gate. Signals only fire on confirmed (closed) bars.
2. **Signal time window** — default 09:45 - 15:45 ET. Avoids the opening volatility flush and closing auction.
3. **Cooldown** — adaptive minimum bars between signals (1m=5, 5m=3, 10m=2, 15m=2).
4. **ADX gate** — optional requirement for ADX above the no-trend threshold.
5. **TWAP distance filter** — rejects signals when price is extended beyond N * ATR from TWAP (prevents chasing).

### Dashboard

Up to 24 rows of real-time data in a configurable table overlay. Only rendered on `barstate.islast`.

### Alerts

**10 static `alertcondition()` entries:**
- CALLS Signal / PUTS Signal
- Strong Bullish / Bearish Bias
- Squeeze Fired
- VIX Extreme
- RSI Bullish / Bearish Divergence
- New Session HOD / LOD

**Dynamic `alert()` calls** on signal bars with interpolated context: score, regime, ATR regime, VIX level, TICK value.

---

## Timeframe Adaptation

The indicator auto-detects `timeframe.in_seconds()` and adapts parameters marked with "0 = auto" in the inputs:

| Parameter | 1-minute | 5-minute | 10-minute | 15-minute |
|:----------|:---------|:---------|:----------|:----------|
| Opening Range | 5 min | 15 min | 20 min | 30 min |
| Signal Cooldown | 5 bars | 3 bars | 2 bars | 2 bars |
| ATR Pctl Lookback | 390 bars | 200 bars | 150 bars | 100 bars |
| BB Width Pctl Lookback | 390 bars | 200 bars | 150 bars | 100 bars |
| Divergence Pivot L/R | 5/3 | 4/2 | 3/2 | 3/2 |
| Divergence Lookback | 30 bars | 20 bars | 15 bars | 10 bars |
| HTF Anchor TF | 5m | 15m | 30m | 60m |

All auto values can be overridden by setting the input to a non-zero value.

---

## Tuning Guide

### Scoring Mode

**Weighted (default)**: Structural conditions score 2 points each, confirmation conditions score 1 point each. This reflects the reality that price being above the session TWAP and aligned with the HTF anchor is structurally more significant than a single oscillator reading.

**Equal**: All conditions score 1 point. Max ±10. Simpler to interpret.

### Minimum Signal Score

Default 6 (weighted mode). This requires all 3 structural conditions aligned (+6) plus at least one confirmation, or 2 structural + 2 confirmations. Increase to 8+ for high-conviction-only signals. Decrease to 4 for more frequent signals on slower timeframes.

### VIX Thresholds

The default tiers (15/20/25/30) are calibrated for the post-2020 VIX regime. If VIX has been persistently elevated (e.g., during a macro crisis), consider shifting all thresholds up by 5 points.

### TWAP Band Multiplier

Default 1.5 standard deviations. Increase to 2.0 for wider bands (fewer extension signals) on volatile sessions. Decrease to 1.0 for tighter bands on low-volatility days.

### Signal Cooldown

Auto-adapts to timeframe. On 1-minute charts, the 5-bar cooldown prevents signal clustering during rapid moves. Override to 10+ for very conservative signal spacing.

### Tuning Profiles

#### Profile: Conservative / Sniper

For traders wanting 1-3 high-conviction trades per session.

```
Min Signal Score:      8
Signal Cooldown:       10 (1m) / 5 (5m) / 3 (15m)
ADX No-Trend:          20
Signal Window:         1000-1530
TWAP Distance Filter:  2.0
Require ADX:           ON
TICK Threshold:        800 / -800
```

#### Profile: Balanced (Default)

For traders wanting moderate signal frequency with reasonable accuracy.

```
Min Signal Score:      6
Signal Cooldown:       auto
ADX No-Trend:          18
Signal Window:         0945-1545
TWAP Distance Filter:  2.5
Require ADX:           ON
TICK Threshold:        500 / -500
```

#### Profile: Aggressive

For experienced traders managing risk through position sizing.

```
Min Signal Score:      4
Signal Cooldown:       3 (1m) / 2 (5m) / 1 (15m)
ADX No-Trend:          15
Signal Window:         0935-1555
TWAP Distance Filter:  3.5
Require ADX:           OFF
TICK Threshold:        300 / -300
```

---

## Architecture Notes

**`request.security()` budget**: The script makes up to 8 `request.*()` calls:

| # | Call | Purpose |
|:--|:-----|:--------|
| 1 | Prior Day High (Daily) | Key level |
| 2 | Prior Day Low (Daily) | Key level |
| 3 | Prior Day Close (Daily) | Key level |
| 4 | HTF Anchor EMA | Structural scoring condition |
| 5 | NYSE TICK close (1m) | Breadth |
| 6 | NYSE TICK SMA (1m) | Smoothed breadth |
| 7 | VIX close (current TF) | VIX level |
| 8 | VIX daily open | VIX intraday change |

This is well within TradingView's 40-call limit. Disabling TICK saves 2 calls. Disabling VIX saves 2 calls. Disabling HTF saves 1 call.

**Anti-repaint**: All signal logic is gated by `barstate.isconfirmed`. TICK and VIX data use `lookahead=barmerge.lookahead_off`. Prior day levels use `[1]` offset with `lookahead_on` (accessing only completed bars).

**Performance**: Two loops per bar (ATR percentile ranking and BB width percentile ranking) over configurable lookbacks. On 1-minute charts with 390-bar lookback, this is the primary computation cost. Reduce lookbacks if running multiple instances.

**Dashboard**: Rendered only on `barstate.islast`. Uses `var table` declaration to avoid reallocation.

**TWAP computation**: Uses incremental mean and variance via cumulative sums. O(1) per bar, no loops.

---

## Known Limitations

1. **No volume data**: SPX from CBOE provides no volume. TWAP, not VWAP. No OBV, no relative volume, no footprint. This is by design.

2. **TWAP is not VWAP**: TWAP treats every bar equally regardless of participation. In reality, more volume transacts at certain prices. TWAP is a reasonable proxy for session fair value but lacks the volume-weighted precision of true VWAP.

3. **VIX intraday granularity**: VIX updates frequently but is derived from SPX options prices. During very fast SPX moves, VIX may lag slightly. The slope lookback (default 5 bars) smooths this.

4. **Pre-market levels require extended hours**: SPX pre-market data availability depends on your TradingView plan and data feed. If extended hours are not enabled, PM High/Low will not populate.

5. **TICK data RTH only**: NYSE TICK is only available during regular trading hours (9:30-16:00 ET). Pre-market and after-hours, TICK returns N/A.

6. **Single-instrument calibration**: Defaults are calibrated for SPX. While the indicator will technically run on other symbols, the VIX integration is SPX-specific and TWAP substitution is designed for the no-volume constraint. Use the SPY scalper variants for SPY.

7. **No backtest capability**: This is an `indicator()`, not a `strategy()`. Signals are for discretionary trading context, not automated execution.

8. **Divergence detection sensitivity**: Pivot-based divergence detection depends on clean pivots forming. In choppy, range-bound conditions, divergence detection may produce false positives. The adaptive pivot parameters help but cannot eliminate this.

9. **10-minute chart**: While fully supported, 10-minute is a non-standard TradingView timeframe. Some adaptation math may produce slightly different results than the primary 1m/5m/15m targets.

---

## Changelog

### v1.0.0 (2025-03-12)

- Initial release.
- Session-anchored TWAP with standard deviation bands (VWAP substitute).
- VIX context module with level tiers, slope analysis, intraday % change.
- Weighted scoring engine: 3 structural (2x) + 7 confirmation (1x), max ±13.
- Timeframe-adaptive parameters for 1m/5m/10m/15m charts.
- Signal generation (CALLS/PUTS) with cooldown, ADX gate, TWAP distance filter.
- Full component stack: EMA ribbon, HTF anchor, RSI (with divergence), MACD, ADX/DMI, Stochastic, TTM Squeeze, NYSE TICK, ATR regime, BB width percentile.
- Session levels: prior day H/L/C, pre-market H/L, opening range, HOD/LOD.
- Session phase detection (6 phases).
- Regime classifier (4 states).
- 24-row dashboard with configurable position and size.
- 10 static alert conditions + dynamic context alerts.
