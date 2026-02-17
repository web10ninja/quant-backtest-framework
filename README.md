# Quantitative Backtesting Framework

A Python backtesting framework for quantitative trading strategies with walk-forward optimization, Monte Carlo validation, and stress testing. Built for validating strategies before deploying real capital.

---

## Components

### VectorBT Engine (655 LOC)
High-performance vectorized backtesting with parameter optimization grid search, multi-asset support, NumPy/Numba acceleration. Sweeps 1000+ parameter combinations efficiently using vectorized operations rather than loop-based simulation.

### Walk-Forward Optimizer (509 LOC)
Anchored walk-forward testing with GO/NO-GO criteria, out-of-sample validation to detect overfitting. Splits the historical period into expanding in-sample windows and fixed out-of-sample windows, then evaluates whether optimized parameters generalize.

### Monte Carlo Simulator (522 LOC)
Trade sequence shuffling (1000+ iterations), random skip simulation, confidence intervals, equity curve distributions. Determines whether strategy performance is attributable to skill or the specific ordering of trades.

### Stress Tester (643 LOC)
Drawdown scenarios, tail-risk analysis, regime-specific stress tests. Tests strategies against historical market crises including the 2008 GFC, 2020 COVID crash, 2022 crypto winter, and the 2010 flash crash, as well as parameterized synthetic shocks.

### Kalshi Backtester (753 LOC)
Prediction market-specific backtesting with settlement verification. Accounts for the binary payoff structure, Kalshi fee schedule, and settlement timing unique to event-contract markets.

### Validation Framework (815 LOC)
GO/NO-GO gate requiring 100+ trades, 45%+ win rate, 1.0+ Sharpe before live deployment. Aggregates results from all upstream modules into a single binary pass/fail decision with a full audit trail of which criteria were met or failed.

### Report Generator (538 LOC)
Automated backtest reports with performance metrics and visualizations. Produces equity curves, monthly returns heatmaps, drawdown charts, and trade-level analysis in PDF/HTML format.

---

## Tech Stack

- Python 3.9+
- VectorBT
- NumPy
- pandas
- SciPy
- matplotlib

---

## Validation Gate

No strategy goes live without passing quantitative thresholds.

The gate requires a minimum trade count, win rate, Sharpe ratio, and maximum drawdown limit before a strategy is permitted to trade real capital. Strategies are blocked from production if they do not have enough trades to establish statistical significance, if their win rate falls below the threshold needed to overcome typical fee drag, if their risk-adjusted return (Sharpe) is below 1.0, or if historical drawdown exceeds the acceptable limit.

This prevents overfitted or lucky strategies from reaching production. A strategy that looks exceptional on a 20-trade backtest may simply have hit a favorable sequence. The Monte Carlo module quantifies exactly this risk by testing whether the performance holds across thousands of alternative trade orderings. The walk-forward optimizer checks whether parameters that worked in-sample still work out-of-sample. Only strategies that clear all three checks — statistical robustness, parameter stability, and absolute performance thresholds — are marked GO.

---

Extracted from a larger algorithmic trading platform — see [algo-trading-platform](https://github.com/yakub268/algo-trading-platform) for the full system.
