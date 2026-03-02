# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

A collection of Pine Script v6 indicators for the TradingView platform. There is no build system, package manager, or test runner — all code executes inside TradingView's Pine Editor. The only way to validate a script is to paste it into the Pine Editor and click "Add to chart."

**Pine Script documentation:**
- User Manual: https://www.tradingview.com/pine-script-docs/
- Language Reference (v6): https://www.tradingview.com/pine-script-reference/v6/

## Project Structure

Three indicator families, each with its own subdirectory and a matching doc in `docs/`:

```
spy_0dte_scalper/   # High-precision signal generation for SPY 0DTE options
market_monitor/     # Directional bias overlays for watchlist monitoring (no signals)
trend_compass/      # Strategic trend assessment for any equity/ETF/index (no signals)
docs/               # Per-script markdown documentation
```

Current scripts (no version suffixes in filenames):
- `spy_0dte_scalper/spy_0dte_scalper_1min.pine`
- `spy_0dte_scalper/spy_0dte_scalper_5min.pine`
- `spy_0dte_scalper/spy_0dte_scalper_15min.pine`
- `market_monitor/market_monitor_1min.pine`
- `market_monitor/market_monitor_5min.pine`
- `trend_compass/trend_compass_daily.pine`
- `trend_compass/trend_compass_4h.pine`

## Common Architecture

Every script follows the same top-to-bottom layout:

1. **Header comment block** — title, timeframe, version, author, component list
2. **`//@version=6` + `indicator()` declaration** — always `overlay=true`, explicit `max_labels_count`, `max_lines_count`, `max_boxes_count`, `max_bars_back=5000`
3. **Constants** — input group names as `var string GP_*`, colors as `var color C_*`
4. **Inputs** — all prefixed `i_`, grouped via `group=GP_*`
5. **Utility functions** — small helpers (`getDashSize()`, `inSession()`, `f2()`, `posnegColor()`)
6. **Core calculations** — EMA ribbon, VWAP, RSI, ADX, ATR, Squeeze, TICK, scoring
7. **Key levels** — pre-market, prior day, opening range, HOD/LOD
8. **Regime / phase classification** — five-mode (Scalper/Monitor) or six-state (Trend Compass)
9. **Signal logic** — family-specific (see below)
10. **Rendering** — plots, fills, labels, lines, boxes
11. **Dashboard** — `table` updated only on `barstate.islast`
12. **Alerts** — static `alertcondition()` + dynamic `alert()` with interpolated context

### Naming Conventions

| Prefix | Used for |
|--------|----------|
| `i_` | All `input.*` variables |
| `C_` | All `var color` constants |
| `GP_` | Input group name string constants |
| `var` | All persistent state (lines, labels, boxes, session tracking, etc.) |

### Signal Engine Differences

| Family | Timeframe | Engine |
|--------|-----------|--------|
| SPY 0DTE Scalper | 1-min | AND-gate: all conditions must pass |
| SPY 0DTE Scalper | 5-min, 15-min | Weighted scoring: conditions contribute points toward configurable threshold |
| Market Monitor | all | Equal-weighted scoring for bias (-5 to +5 general, -11 to +11 weighted 5m) |
| Trend Compass | Daily, 4H | Weighted scoring (-11 to +11): structural 2x + confirmation 1x |

### Repainting Prevention

- Signal logic gated by `barstate.isconfirmed` — signals appear after bar close only
- All `request.security()` calls use `lookahead=barmerge.lookahead_off` except prior-day reference levels (which use `lookahead_on` intentionally)

## Pine Script Constraints to Keep In Mind

- **`request.security()` limit**: 40 per script. Current scripts use 3–6. Keep well within headroom for multi-instance layouts (8+ simultaneous chart tiles).
- **Label/line/box limits**: 500 labels, 500 lines, 200 boxes per script. Declared explicitly in `indicator()`.
- **No loops on hot path**: Percentile ranking is the only loop usage (ATR/BB width lookback, lightweight). All other calculations are vectorized.
- **No arrays or maps** in the simpler scripts (Market Monitor 1m, Scalper 1m) for minimum overhead.
- **Dashboard tables** update only on `barstate.islast` — never inside the per-bar calculation path.
- These are `indicator()` scripts, not `strategy()` — no backtesting capability.

## Color Palette (Dark-Theme Optimized)

All scripts share a consistent color vocabulary:
- Bull green: `#00E676`, Bear red: `#FF1744`, Neutral: `#B0BEC5`
- Yellow: `#FFD600`, Orange: `#FF9100`, Cyan: `#00BCD4`, Purple: `#7C4DFF`
- Dashboard background: `#1A1A2E` or `#1E1E1E` (5–10% transparency)
- Cloud fills use 85–92% transparency over the base bull/bear colors

## Docs

Each `.pine` file has a corresponding `.md` in `docs/` using the same base filename. When modifying a script, update its doc if the change affects inputs, dashboard rows, alert conditions, or behavior.

## Versioning

Version is tracked in the header comment of each `.pine` file (e.g., `Version: 1.1.0`). CHANGELOG.md is updated manually per release. File names do not include version suffixes — a single canonical file per script variant.
