# Statistical Arbitrage in the EMU Equity Market — PCA-Based Mean Reversion

A systematic, market-neutral statistical arbitrage strategy on Euro Stoxx 50 constituents, built from scratch in Python. The strategy follows Avellaneda & Lee (2008), *"Statistical Arbitrage in the U.S. Equities Market"*: PCA-based factor models isolate idiosyncratic residuals, which are modeled as Ornstein-Uhlenbeck mean-reverting processes and traded via a modified s-score signal.


## Overview

At every daily rebalance date, the strategy:

1. Estimates a statistical factor model via PCA on the trailing 252-day correlation matrix of stock returns, keeping the first 4 principal components as systematic risk factors.
2. Runs a rolling OLS regression to extract each stock's idiosyncratic residual and drift term.
3. Fits an AR(1) / Ornstein-Uhlenbeck process to the cumulative residuals over a trailing 60-day window, estimating mean-reversion speed (κ), equilibrium level (m), and volatility (σ).
4. Filters for stocks with fast mean reversion (κ > 8.4) and computes a modified s-score that accounts for residual drift.
5. Generates long/short signals from the s-score (entry at ±1.25, exit at ±0.5), sized with equal weighting under a 100% gross exposure constraint.
6. Backtests the strategy with a one-day execution lag (no look-ahead bias), both with and without transaction costs (5 bps), and benchmarks it against an equal-weight portfolio.

## Key Results (2013–2024 backtest, no trading costs)

| Metric | StatArb Strategy | Equal-Weight Benchmark |
|---|---|---|
| Total Return | 49.23% | 124.96% |
| Annualized Return | 4.40% | 9.10% |
| Annualized Volatility | 6.06% | 19.52% |
| Sharpe Ratio | **0.73** | 0.47 |
| Maximum Drawdown | **-11.59%** | -39.93% |

The strategy trades off raw return for a materially better risk-adjusted profile: roughly a third of the volatility and drawdown of a passive equal-weight portfolio, by isolating idiosyncratic mean-reversion rather than taking market beta.

A full sensitivity analysis (entry threshold, fixed vs. variable number of PCA factors, trading-time/volume-adjusted returns) is included in the report and notebook — see [`Report_Assignment4_BS_Group2.pdf`](Report_Assignment4_BS_Group2.pdf).

## Repository Structure

```
.
├── ex4a_notebook_Group2.ipynb          # Main notebook: end-to-end analysis, backtest, plots
├── utilities/
│   ├── principal_component_analysis.py # PCA and eigenvector alignment across rebalances
│   ├── covariance_utilities.py         # Rolling estimation windows, covariance validation
│   ├── statistical_arbitrage.py        # Factor model, OU estimation, s-score computation
│   └── backtest.py                     # Portfolio construction, PnL, turnover, transaction costs
├── data/
│   ├── sx5e_underlyings.csv            # Euro Stoxx 50 total return prices (not included, see Data below)
│   ├── volume.csv                      # Daily trading volumes (not included, see Data below)
│   └── ticker_details.csv              # Ticker → company name / sector mapping
├── Report_Assignment4_BS_Group2.pdf    # Written report with full results and discussion
├── BS4a_Presentation.pdf               # Summary presentation
└── Assignment_4a_BS.pdf                # Original assignment brief
```

## Methodology

**Factor model.** Returns are standardized and projected onto the top-K eigenvectors of the trailing correlation matrix (K = 4), giving statistical risk factors purely data-driven, with no macro or fundamental input.

**Residual dynamics.** The idiosyncratic residual of each stock is cumulated and modeled as an OU process `dX_t = κ(m - X_t)dt + σ dW_t`, discretized as an AR(1) and estimated by least squares over a 60-day window.

**Signal.** The modified s-score `s_mod = (X_t - m)/σ_eq - α/(κ·σ_eq)` measures deviation from equilibrium net of drift. Positions open at |s_mod| > 1.25 and close at |s_mod| < 0.5.

**Backtest.** Weights are shifted forward one day to avoid look-ahead bias (a signal computed at the close of day *t* is only tradable on day *t+1*). Both frictionless and cost-adjusted (5 bps all-in) scenarios are evaluated.

## Getting Started

```bash
git clone <your-repo-url>
cd statistical-arbitrage-pca
pip install -r requirements.txt
jupyter notebook ex4a_notebook_Group2.ipynb
```

### Requirements

- Python 3.12+
- numpy, pandas, matplotlib, jupyter

## Data

`data/sx5e_underlyings.csv` (total return prices) and `data/volume.csv` (daily volumes) are **not included** in this repository: they come from a licensed market data feed (Refinitiv/LSEG) provided for coursework use and cannot be redistributed publicly. `data/ticker_details.csv` (ticker → name/sector mapping) is included, since it contains no licensed price data.

To reproduce the results, source daily total-return prices and volumes for the Euro Stoxx 50 constituents (tickers listed in `ticker_details.csv`) from your own data provider and save them in the same format/column layout as the two files above.

## Reference

Avellaneda, M., & Lee, J.-H. (2008). *Statistical Arbitrage in the U.S. Equities Market*. Journal Not for redistribution here (third-party copyrighted material) — available via any academic search engine.

## License

Academic project — shared for portfolio purposes. Market data is provided for coursework use only.
