# Pine Scripts for TradingView

A modular, production-grade suite of Pine Script v6 indicators for
systematic trading on TradingView.

Designed for structured decision workflows across three layers:

1.  **Strategic Trend Context** -- Trend Compass (Daily, 4H, 1H)
2.  **Directional Watchlist Bias** -- Market Monitor
3.  **Execution & Signal Generation** -- SPY 0DTE Scalpers

All indicators share a unified architectural philosophy:

-   EMA ribbon structure
-   Regime / phase classification
-   Weighted scoring engines (where applicable)
-   Controlled `request.security()` usage
-   Non-repainting logic (`barstate.isconfirmed`)
-   Structured dashboards
-   Alert pipeline compatibility

------------------------------------------------------------------------

# Indicator Suite

## Trend Compass (Strategic Context Layer)

Ticker-agnostic strategic trend assessment overlays.

These are **not signal generators**.

They answer:

-   What is the prevailing trend?
-   How strong is it?
-   Is momentum accelerating or exhausting?
-   Are divergences forming?
-   Are higher timeframes aligned?

### Variants

  ---------------------------------------------------------------------------------------------------
  Script                       Timeframe   Composite      HTF Stack  Dashboard   `request.security`
                                           Score                                 
  ---------------------------- ----------- -------------- ---------- ----------- --------------------
  `trend_compass_1h.pine`      1H          -11 → +11      4H50 +     19 rows     7
                                                          D50 +                  
                                                          D200 + W50             

  `trend_compass_4h.pine`      4H          -11 → +11      D50 +      18 rows     6
                                                          D200 + W50             

  `trend_compass_daily.pine`   Daily       -11 → +11      W50        17 rows     3
  ---------------------------------------------------------------------------------------------------

### Architectural Positioning

-   **Daily** → Macro structural bias
-   **4H** → Multi-day swing structure
-   **1H** → Intraday swing validation & fastest divergence detection

The 1H variant uniquely introduces a 4H 50 EMA bridge between local
structure and Daily context.

------------------------------------------------------------------------

## Market Monitor (Reconnaissance Layer)

Directional bias overlays for multi-chart watchlist monitoring.

These do **not** generate trade signals.

They answer one question:

> Which way is the wind blowing right now?

  ----------------------------------------------------------------------------------------------
  Script                       Variant     Bias Range    Dashboard Rows    `request.security`
  ---------------------------- ----------- ------------- ----------------- ---------------------
  `market_monitor_1min.pine`   General     -5 → +5       12                3
                               Purpose                                     

  `market_monitor_5min.pine`   5m          -11 → +11     18                4
                               Optimized   (weighted)                      
  ----------------------------------------------------------------------------------------------

### 5m Optimized Adds

-   Weighted scoring engine
-   NYSE TICK integration
-   TTM Squeeze detection
-   ATR percentile regime
-   Session phase classification
-   Prior Day VWAP
-   15m Anchor EMA

------------------------------------------------------------------------

## SPY 0DTE Scalper (Execution Layer)

Signal generation systems calibrated specifically for SPY 0DTE options.

Each timeframe is independently tuned.

  ----------------------------------------------------------------------------------------------
  Script                          TF    Signal Engine       Target Hold     Key Difference
  ------------------------------- ----- ------------------- --------------- --------------------
  `spy_0dte_scalper_1min.pine`    1m    Strict AND-gate     1--5 min        Precision
                                                                            micro-entries

  `spy_0dte_scalper_5min.pine`    5m    5+4 scoring         5--25 min       Balanced structure

  `spy_0dte_scalper_15min.pine`   15m   Scoring + bias      15--60 min      Session bias framing
  ----------------------------------------------------------------------------------------------

These combine:

-   EMA ribbon
-   VWAP anchor
-   ADX / RSI momentum
-   TTM Squeeze
-   NYSE TICK
-   Key structural levels
-   Structured dashboard
-   Alert system

------------------------------------------------------------------------

# Recommended Workflow

## Full Structured Pipeline

1.  **Trend Compass Daily** → Establish macro bias
2.  **Trend Compass 4H** → Confirm swing structure
3.  **Trend Compass 1H** → Validate intraday strengthening / weakening
4.  **Market Monitor** → Scan watchlist alignment
5.  **0DTE Scalper** → Execute

Each layer narrows the decision space.

------------------------------------------------------------------------

# Installation

1.  Open TradingView.
2.  Navigate to appropriate timeframe.
3.  Open Pine Editor.
4.  Paste full `.pine` file.
5.  Click **Add to Chart**.

------------------------------------------------------------------------

# Performance Profile

  -----------------------------------------------------------------------------------------
  Resource               Pine Limit Scalpers   Monitor GP Monitor 5m TC 1H  TC 4H  TC Daily
  ---------------------- ---------- ---------- ---------- ---------- ------ ------ --------
  `request.security()`   40         4--5       3          4          7      6      3

  Labels                 500        \~100      100        200        500    500    500

  Lines                  500        \~40       200        300        500    500    500

  Boxes                  200        \~10       50         100        100    100    100
  -----------------------------------------------------------------------------------------

Optimized for multi-instance execution.

-   No repainting
-   HTF calls use `lookahead_off`
-   Dashboards render on `barstate.islast`
-   Percentile loops lightweight

------------------------------------------------------------------------

# Known Constraints

-   No cumulative delta / order flow
-   No native volume profile
-   Scalper parameters calibrated for SPY
-   Indicators (not strategies) --- no backtest engine
-   Divergence detection requires pivot confirmation
-   52-week calculations require sufficient historical data

------------------------------------------------------------------------

# Repository Structure

    ├── market_monitor/
    ├── spy_0dte_scalper/
    ├── trend_compass/
    ├── docs/
    ├── README.md
    ├── CHANGELOG.md
    └── LICENSE

------------------------------------------------------------------------

# Roadmap

-   Market internals dashboard
-   Volume profile approximation
-   QQQ / IWM scalper variants
-   Strategy conversion for backtesting
-   Session statistics overlay
-   Options-aware enhancements
-   Multi-asset correlation monitor
-   Alert webhook templates

------------------------------------------------------------------------

# License

MIT. See `LICENSE`.
