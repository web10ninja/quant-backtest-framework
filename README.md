# Quantitative Backtesting Framework

A rigorous Python framework for validating algorithmic trading strategies before deploying real capital. Built around the principle that a strategy must prove statistical robustness — not just historical profitability — before touching a live account.

![Python 3.9+](https://img.shields.io/badge/python-3.9%2B-blue)
![License: MIT](https://img.shields.io/badge/license-MIT-green)
![CI](https://github.com/yakub268/quant-backtest-framework/actions/workflows/ci.yml/badge.svg)

---

## Key Features

- **Walk-Forward Optimization** — Anchored and rolling windows with per-window GO/NO-GO assessment; parameters optimized in-sample, evaluated strictly out-of-sample
- **Monte Carlo Validation** — Trade sequence shuffling, random-skip simulation, and bootstrap resampling across 1000+ iterations to distinguish skill from luck
- **Historical Stress Testing** — Replay strategies against the 2008 GFC, 2020 COVID crash, 2022 crypto winter, 2010 flash crash, and parameterized synthetic shocks
- **Combinatorial Purged Cross-Validation (CPCV)** — Lopez de Prado's method for leak-free cross-validation on time-series data, with configurable embargo periods
- **Deflated Sharpe Ratio (DSR)** — Adjusts reported Sharpe for the number of parameter trials tested, penalizing strategies discovered through brute-force search
- **Kalshi Prediction Market Backtester** — Purpose-built for binary event contracts: binary payoff structure, Kalshi fee schedule, theta decay, Brier score tracking across Fed/weather/sports/economic markets
- **HTML Report Generation** — Equity curves, monthly returns heatmaps, drawdown charts, and trade-level analysis rendered to standalone HTML/PDF

---

## Architecture

```
quant-backtest-framework/
├── backtest/
│   ├── walk_forward.py        509 LOC  Anchored WFO, GO/NO-GO criteria, OOS efficiency
│   ├── monte_carlo.py         522 LOC  Shuffle / skip / bootstrap, confidence intervals
│   ├── stress_test.py         643 LOC  6 crisis scenarios + synthetic shock generator
│   ├── validation_framework.py 815 LOC CPCV, DSR, permutation tests, final GO gate
│   ├── kalshi_backtester.py   753 LOC  Prediction market sim, Brier scoring, fee model
│   ├── vectorbt_engine.py     655 LOC  Vectorized grid search, multi-asset, Numba accel
│   └── report_generator.py    538 LOC  HTML/PDF reports, equity curves, heatmaps
└── requirements.txt
```

| Module | Primary Output | Key Algorithm |
|---|---|---|
| `walk_forward` | `WalkForwardSummary` | Anchored OOS splits, profit factor, OOS efficiency ratio |
| `monte_carlo` | `MonteCarloResult` | Permutation p-value, 5th/95th percentile return bands |
| `stress_test` | `StressTestResult` | Regime-conditioned drawdown, tail loss, recovery time |
| `validation_framework` | `ValidationReport` | CPCV Sharpe distribution, DSR, composite GO/NO-GO |
| `kalshi_backtester` | `KalshiBacktestResult` | Binary payoff sim, fee-adjusted P&L, Brier score |
| `vectorbt_engine` | `BacktestResult` | Vectorized parameter grid, Sortino, Calmar, CAGR |
| `report_generator` | HTML/PDF file | matplotlib equity + drawdown + monthly heatmap |

---

## Installation

```bash
git clone https://github.com/yakub268/quant-backtest-framework.git
cd quant-backtest-framework
pip install -r requirements.txt
```

Optional dependencies for the VectorBT engine and data download:

```bash
pip install vectorbt yfinance
```

TA-Lib requires platform-specific build tools — see the [official install guide](https://github.com/TA-Lib/ta-lib-python).

---

## Quick Start

### Walk-Forward Optimization

```python
import pandas as pd
from backtest import walk_forward_test, print_walk_forward_report

# strategy_func(data, params) -> DataFrame with 'pnl' column
# optimize_func(train_data) -> dict of best params
summary = walk_forward_test(
    data=ohlcv_df,
    strategy_func=my_strategy,
    optimize_func=my_optimizer,
    train_window=252,   # 1 year in-sample
    test_window=63,     # 1 quarter out-of-sample
    anchored=True,      # Expanding window (vs rolling)
    strategy_name="RSI Mean Reversion"
)

print(print_walk_forward_report(summary))
# OVERALL STATUS: GO
# OOS Efficiency: 73.4%  |  Avg Sharpe: 1.24  |  Worst DD: -11.2%
```

### Monte Carlo Validation

```python
from backtest.monte_carlo import MonteCarloSimulator, quick_monte_carlo

mc = MonteCarloSimulator(n_iterations=2000)
result = mc.simulate_from_trades(trades_df, method='shuffle')

print(f"Prob. profitable:    {result.prob_profitable:.1%}")
print(f"5th pct return:      {result.return_5th:.1%}")
print(f"Median Sharpe:       {result.sharpe_50th:.2f}")

# One-liner for quick checks
check = quick_monte_carlo(trade_returns_list)
print(check['passes_validation'])  # True / False
```

### Deflated Sharpe Ratio (DSR)

```python
from backtest.validation_framework import calculate_deflated_sharpe_ratio

# Penalizes the observed Sharpe for the number of parameter combinations tested
dsr = calculate_deflated_sharpe_ratio(
    observed_sharpe=1.45,
    n_trials=50,          # parameter combinations attempted
    n_observations=252,
    skewness=-0.3,
    kurtosis=4.1
)
print(f"Deflated Sharpe: {dsr:.3f}")  # materially lower than 1.45 if n_trials is large
```

---

## Validation Pipeline: GO / NO-GO

Every strategy must clear a sequential gate before live deployment:

```
Raw Backtest
     |
     v
[1] Walk-Forward OOS Check
     Sharpe >= 1.0  |  Win Rate >= 45%  |  Max DD >= -15%
     |
     v
[2] Monte Carlo Permutation
     Prob. Profitable >= 75%  |  5th pct Sharpe > 0
     |
     v
[3] CPCV + Deflated Sharpe
     DSR > 0  |  CPCV Sharpe distribution mean > 0.5
     |
     v
[4] Stress Test
     Survives 2008 GFC scenario with DD < threshold
     |
     v
  GO / NO-GO
```

The gate exists because optimized backtests are systematically misleading. Quantopian's study of 888 strategies found near-zero correlation between backtest Sharpe ratio and live performance. The more parameters tuned, the worse the live result. CPCV and DSR quantify exactly how much of the observed Sharpe is explained by the number of trials.

---

## Academic References

- **Lopez de Prado, M.** (2018). *Advances in Financial Machine Learning*. Wiley. — CPCV construction, Combinatorial Purging, feature importance for time-series
- **Bailey, D., Borwein, J., Lopez de Prado, M., Zhu, Q.** (2014). Pseudo-Mathematics and Financial Charlatanism: The Effects of Backtest Overfitting on Out-of-Sample Performance. *Notices of the AMS*, 61(5). — Deflated Sharpe Ratio derivation
- **Bailey, D., Lopez de Prado, M.** (2012). The Sharpe Ratio Efficient Frontier. *Journal of Risk*, 15(2). — OOS efficiency and walk-forward methodology

---

## Tech Stack

| Layer | Library |
|---|---|
| Vectorized backtesting | VectorBT, NumPy, Numba |
| Data manipulation | pandas |
| Statistics | SciPy |
| Visualization | matplotlib |
| Market data (examples) | yfinance |
| Technical indicators | TA-Lib (optional) |

---

Extracted from a larger algorithmic trading platform — see [algo-trading-platform](https://github.com/yakub268/algo-trading-platform) for the full system.
