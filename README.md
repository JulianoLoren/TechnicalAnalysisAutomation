# TechnicalAnalysisAutomation

A collection of Python modules for automating common technical analysis tasks on OHLCV price data. Includes chart pattern detection, harmonic patterns, support/resistance levels, trendline fitting, and more.

---

## Table of Contents

- [Installation](#installation)
- [Data](#data)
- [Modules](#modules)
  - [directional_change.py](#directional_changepy)
  - [perceptually_important.py](#perceptually_importantpy)
  - [rolling_window.py](#rolling_windowpy)
  - [trendline_automation.py](#trendline_automationpy)
  - [head_shoulders.py](#head_shoulderspy)
  - [flags_pennants.py](#flags_pennantspy)
  - [harmonic_patterns.py](#harmonic_patternspy)
  - [mp_support_resist.py](#mp_support_resistpy)
  - [pip_pattern_miner.py](#pip_pattern_minerpy)
  - [wf_pip_miner.py](#wf_pip_minerpy)
  - [retracement_ratios.py](#retracement_ratiospy)
- [Tests](#tests)
- [License](#license)

---

## Installation

Install required dependencies:

```bash
pip install pandas numpy matplotlib mplfinance scipy pandas_ta pyclustering
```

> **Important:** The `pyclustering` library does not work with recent versions of NumPy. Use:
> ```bash
> pip install numpy==1.23.1
> ```

---

## Data

Two sample BTCUSDT datasets are included for testing and experimentation:

| File | Description |
|---|---|
| `BTCUSDT3600.csv` | BTC/USDT OHLCV data at 1-hour intervals |
| `BTCUSDT86400.csv` | BTC/USDT OHLCV data at 1-day intervals |

---

## Modules

### directional_change.py

Identifies significant tops and bottoms in price data using the **Directional Change** algorithm. A new extreme is only confirmed once price retraces by a given percentage (`sigma`) from the previous extreme.

**Functions:**

- `directional_change(close, high, low, sigma)` — Returns arrays of confirmed tops and bottoms.
- `get_extremes(ohlc, sigma)` — Convenience wrapper that returns a labelled DataFrame of extremes from an OHLCV DataFrame.

---

### perceptually_important.py

Implements the **Perceptually Important Points (PIP)** algorithm, which identifies the most visually significant price points in a series.

**Functions:**

- `find_pips(data, n_pips, dist_measure)` — Returns the indices and prices of the `n_pips` most important points. Supports three distance measures: Euclidean (1), Perpendicular (2), and Vertical (3).

---

### rolling_window.py

Detects local extrema using a **rolling window** approach.

**Functions:**

- `rw_top(data, curr_index, order)` — Returns `True` if `curr_index` is a local top within the window defined by `order`.
- `rw_bottom(data, curr_index, order)` — Returns `True` if `curr_index` is a local bottom within the window.
- `rw_extremes(data, order)` — Returns a DataFrame of all local tops and bottoms.

---

### trendline_automation.py

Fits **support and resistance trendlines** to price data by optimising the slope to minimise violations.

**Functions:**

- `check_trend_line(support, pivot, slope, y)` — Checks the quality of a trendline given a pivot and slope.
- `optimize_slope(support, pivot, init_slope, y)` — Optimises the slope of a trendline.
- `fit_trendlines_single(data)` — Fits support and resistance trendlines to a 1D price array.
- `fit_trendlines_high_low(high, low, close)` — Fits trendlines using separate high and low arrays.

---

### head_shoulders.py

Detects **Head and Shoulders** (H&S) and **Inverse Head and Shoulders** (IHS) patterns.

**Classes:**

- `HSPattern` — Dataclass holding the indices and prices of each part of an H&S or IHS pattern.

**Functions:**

- `find_hs_patterns(data, order, early_find)` — Scans a price array for H&S patterns using a rolling-window extrema detector.
- `check_hs_pattern(extrema_indices, data, i)` — Validates a candidate H&S pattern.
- `check_ihs_pattern(extrema_indices, data, i)` — Validates a candidate IHS pattern.
- `compute_pattern_r2(data, pat)` — Computes an R² score describing how cleanly the pattern fits the data.
- `get_pattern_return(data, pat, log_prices)` — Estimates the return generated after pattern confirmation.
- `plot_hs(candle_data, pat, pad)` — Plots the detected pattern on a candlestick chart.

---

### flags_pennants.py

Detects **Bull/Bear Flag** and **Pennant** chart patterns using either PIP-based or trendline-based methods.

**Classes:**

- `FlagPattern` — Dataclass holding pole and flag/pennant geometry.

**Functions:**

- `find_flags_pennants_pips(data, order)` — Detects flags/pennants using Perceptually Important Points.
- `find_flags_pennants_trendline(data, order)` — Detects flags/pennants by fitting trendlines to the consolidation zone.
- `plot_flag(candle_data, pattern, pad)` — Plots a detected flag or pennant on a candlestick chart.

---

### harmonic_patterns.py

Detects **Harmonic Patterns** (Gartley, Bat, Butterfly, Crab, Deep Crab, Cypher, Shark) using XABCD retracement ratios.

**Classes:**

- `XABCD` — Defines the retracement ratio ranges for a harmonic pattern type.
- `XABCDFound` — Stores a detected harmonic pattern instance.

**Predefined patterns:** `GARTLEY`, `BAT`, `BUTTERFLY`, `CRAB`, `DEEP_CRAB`, `CYPHER`, `SHARK`

**Functions:**

- `find_xabcd(ohlc, extremes, err_thresh)` — Searches for all defined harmonic patterns within a set of price extremes.
- `get_error(actual_ratio, pattern_ratio)` — Computes the percentage error between an observed ratio and the ideal ratio.
- `plot_pattern(ohlc, pat, pad)` — Plots a detected harmonic pattern on a candlestick chart.

---

### mp_support_resist.py

Identifies **Support and Resistance levels** using a **Market Profile** (kernel density estimation) approach.

**Functions:**

- `find_levels(price, atr, first_w, atr_mult, prom_thresh)` — Estimates S/R levels from a kernel density estimate of closing prices, weighted towards recent prices.
- `support_resistance_levels(ohlc, ...)` — High-level function that computes ATR and returns S/R levels for an OHLCV DataFrame.
- `sr_penetration_signal(data, levels)` — Generates a buy/sell signal array based on price penetration of S/R levels.
- `get_trades_from_signal(data, signal)` — Converts a signal array into a list of trade entries and exits.

---

### pip_pattern_miner.py

Uses **K-Means clustering on PIP sequences** (via `pyclustering`) to mine recurring price patterns from historical data.

**Classes:**

- `PIPPatternMiner` — Mines and clusters PIP-based patterns, returning representative pattern templates.

> Requires `numpy==1.23.1` due to a `pyclustering` compatibility issue.

---

### wf_pip_miner.py

**Walk-Forward PIP Pattern Miner** — extends `PIPPatternMiner` with a walk-forward validation framework to avoid look-ahead bias when evaluating mined patterns.

**Classes:**

- `WFPIPMiner` — Splits data into in-sample and out-of-sample windows, mines patterns on in-sample data, and evaluates on out-of-sample data.

---

### retracement_ratios.py

Utilities for computing **Fibonacci and other retracement ratios** between price swings, used internally by the harmonic pattern detection module.

---

## Tests

Two test scripts are included:

| File | Description |
|---|---|
| `test_hs_patterns.py` | Tests for the Head and Shoulders pattern detector |
| `test_flag_patterns.py` | Tests for the Flag and Pennant pattern detector |

Run tests with:

```bash
python test_hs_patterns.py
python test_flag_patterns.py
```

---

## License

See [LICENSE](LICENSE) for details.
