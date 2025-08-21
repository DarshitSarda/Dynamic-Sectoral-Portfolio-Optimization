# Dynamic Sectoral Portfolio Optimization

> **A research-grade pipeline** for dynamic sector selection, portfolio construction and rebalancing using hierarchical risk parity (HRP), volatility-aware sector selection, dynamic leverage and optimized stop-loss rules. Built as a reproducible Jupyter notebook + utilities for backtesting, evaluation and result export.

---

## 🚀 Project Overview

This project implements a **dynamic sectoral portfolio optimization** framework that aims to improve risk-adjusted returns by:

* Selecting the top *N* sectors every quarter based on a robust combination of volatility and recent returns.
* Selecting best-in-class stocks within those sectors using momentum, liquidity and correlation-aware filters.
* Constructing portfolios using **Hierarchical Risk Parity (HRP)** to gain the benefits of diversification without unstable inverse-covariance estimates.
* Applying **dynamic leverage** and **optimized stop-loss** mechanisms to enhance returns while controlling drawdowns.
* Simulating and reporting returns for both dynamic (HRP + sector selection) and benchmark strategies (equally-weighted sector, buy-and-hold).

This repository contains a single running notebook (with supporting code embedded) that documents the entire pipeline, including data fetching (yfinance), preprocessing, model functions, simulation loops, evaluation metrics and plotting utilities.

---

## ✅ Key Features

* **Quarterly Rebalancing:** Selects top sectors and rebalances every 3 months by default.
* **Sector Selection:** Uses volatility + trailing returns ranking to choose sectors; configurable lookback windows.
* **Stock Selection within Sectors:** Filters by liquidity, market cap (optional), and cross-correlation to pick diversified names.
* **HRP Portfolio Construction:** Computes hierarchical clustering and risk-parity weights for stability and interpretability.
* **Dynamic Leverage:** Scales gross exposure based on historical volatility targets.
* **Stop-Loss & Trailing Stop:** Per-position stop-loss rules and portfolio-level drawdown protection.
* **Comparative Backtests:** Run against equally-weighted and buy-and-hold baselines; outputs performance tables and plots.
* **Export Outputs:** Saves detailed CSVs for each rebalance period, cumulative returns, trade logs and summary reports.

---

## 📁 Repository Structure (what you will see)

```
BE_FINAL_CODE/
├─ BE_FINAL_CODE with explanation.ipynb     
├─ data/                                    
│  ├─ prices/                               
│  └─ scheme_master.csv                     
├─ outputs/
│  ├─ backtests/                            
│  ├─ plots/                                
│  └─ reports/                              
├─ utils/                                   
├─ requirements.txt
└─ README.md                                
```

> *Note:* The notebook embeds many utility functions; for production use extract them to `utils/` and create a CLI or pipeline.

---

## 💡 Workflow / Pipeline (high-level)

1. **Configuration** — Set experiment parameters (start/end dates, rebalance frequency, top N sectors, lookback windows, risk target, stop-loss thresholds).
2. **Data Fetching** — Download OHLCV for the universe using `yfinance` (or load from local CSVs).
3. **Preprocessing** — Align daily returns, handle missing data, winsorize outliers, and compute sector-level returns and volatility.
4. **Sector Ranking** — For each rebalance date compute scores (e.g., inverse volatility + trailing returns) and pick the top `K` sectors.
5. **Stock Selection** — From each selected sector, filter stocks by liquidity and volatility and pick the top `M` per sector.
6. **Weighting** — Build correlation matrix, perform hierarchical clustering and compute HRP weights across selected stocks.
7. **Leverage & Risk Targeting** — Scale weights so portfolio volatility matches the target (or apply leverage cap).
8. **Stop-Loss Application** — During the simulation apply per-position and portfolio stop-loss rules, with optional trailing stop.
9. **Performance Calculation** — Track P\&L, drawdowns, turnover, Sharpe, CAGR, volatility, and other metrics.
10. **Reporting & Visualization** — Export trade logs, per-rebalance reports, cumulative returns and diagnostic plots.

---

## ⚙️ Configuration (important parameters)

* `START_DATE`, `END_DATE` — backtest horizon
* `REBALANCE_FREQ = 'Q'` — quarterly by default (can use monthly or custom date list)
* `TOP_SECTORS = 3 to 10` — number of sectors to select per rebalance
* `STOCKS_PER_SECTOR = 5` — top stocks selected within each sector
* `STOP_LOSS_PCT = 0.05` — 5% stop-loss for the portfolio (configurable)
* `ENABLE_PLOTS = True` — toggle plotting

All these appear at the top of the notebook so they are easy to tune for experiments.

---

## 📚 Requirements

The notebook uses standard scientific Python libraries. Example `requirements.txt` (use the versioning you prefer):

```
pandas
numpy
matplotlib
scipy
scikit-learn
yfinance
networkx
seaborn
joblib
tqdm
PyYAML
```

> Add `torch`/`tensorflow` only if you extend with ML models for forecasting.

---

## 🔧 Key Functions (what the notebook implements)

The notebook contains well-documented functions for:

* `get_historical_data(tickers, start, end, interval)` — fetch and cache OHLCV from yfinance.
* `compute_sector_returns(prices, sector_map)` — aggregate stock returns to sectors.
* `rank_sectors(sector_returns, lookback_vol, lookback_ret)` — compute sector scores and select top K.
* `select_diversified_sectors(universe, sector_map, K)` — return selected sector tickers.
* `get_top_stocks(prices, sector, n, liquidity_filter=True)` — choose best stocks per sector.
* `get_hrp_weights(cov_matrix)` — compute HRP weights using hierarchical clustering and inverse-variance allocation.
* `simulate_backtest(config)` — orchestrates the simulation loop (rebalance, apply trades, mark-to-market, stop-losses).
* `calculate_performance_metrics(nav_series)` — computes Sharpe, CAGR, max drawdown, volatility.

Each function is accompanied by docstrings and example usage in the notebook cells.

---

## 🧪 How to run (quickstart)

1. Clone the repository and open the notebook: `BE_FINAL_CODE with explanation.ipynb`.
2. Create a virtualenv and install requirements: `pip install -r requirements.txt`.
3. Edit configuration at the top of the notebook (dates, lookbacks, TOP\_SECTORS etc.).
4. Execute the notebook cells in order (or run the entire notebook in one go).
5. Inspect `outputs/` for CSVs and plots.

For headless execution, extract `simulate_backtest` and call it from a script `run_backtest.py` that loads a YAML config and writes outputs to `outputs/`.

---

## 📈 Evaluation & Benchmarks

The project includes comparison against:

* **Equally Weighted (sector-level)** — allocate equal weight across selected sectors and equally across stocks inside sector.
* **Buy-and-Hold Benchmark** — the benchmark index or equal-weight universe.

Metrics tracked: CAGR, Annualized Volatility, Sharpe Ratio (annualized), Max Drawdown, Calmar Ratio, Turnover, Average Position Size.

---

## 🧭 Practical tips & caveats

* **Data quality matters:** Use cleaned, adjusted close prices for return calculations.
* **Delistings / corporate actions:** Backtest must handle missing tickers across history (drop or replace with proxy).
* **Lookahead bias:** Avoid using future information when selecting sectors/stocks — ensure all signals are computed using data available at the rebalance date.
* **Transaction costs & slippage:** Add realistic fees to the simulate\_backtest function for robust results.
* **Out-of-sample testing:** Reserve the last portion of the dataset (e.g., final 12 months) as a holdout to test parameter stability.

---

## 🐞 Troubleshooting

* If yfinance throttles, cache price CSVs locally and re-run tests from cached data.
* If performance metrics are NaN, ensure NAV does not contain zeros or constant values; examine trade log for missed rebalances.

---

## 📄 Contact

For questions or to collaborate, contact **Darshit Sarda**.
