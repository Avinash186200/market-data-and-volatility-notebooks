# Market Data and Volatility Notebooks

[![CI](https://github.com/Avinash186200/market-data-and-volatility-notebooks/actions/workflows/ci.yml/badge.svg)](https://github.com/Avinash186200/market-data-and-volatility-notebooks/actions/workflows/ci.yml) ![Python](https://img.shields.io/badge/python-3.11-blue) [![License: MIT](https://img.shields.io/badge/license-MIT-green)](LICENSE)

A small set of research notebooks I keep coming back to: cross-asset correlation, implied volatility, intraday bar resampling and a basic momentum ranking. Each one is self-contained and runs offline on synthetic data; two of them can pull public prices from Yahoo Finance instead.

## What problem this solves

Market data work goes wrong in quiet ways: a correlation that drifts, a volatility solver that fails on cheap options, 15-minute bars that are off by one bucket, a momentum signal that peeks at next month. Each notebook takes one of these everyday jobs and shows a correct way to do it, with checks you can see.

## What you will learn

- How to measure correlation between assets and sectors, and why a rolling window tells you more than one number.
- How Black-Scholes prices an option, and how to back out implied volatility reliably.
- How to turn 5-minute bars into 15-minute bars without misaligned or partial buckets.
- How to build a monthly momentum ranking with no look-ahead, and how to measure turnover.
- How to make a notebook run offline on synthetic data, with real data as an option.

## How it works

Read the notebooks in this order. Each one is self-contained and runs offline on synthetic data.

1. [`intraday_bar_resampling.ipynb`](notebooks/intraday_bar_resampling.ipynb): start here, it is the simplest. It builds 20 days of synthetic 5-minute bars with about 1% of bars missing, filters to the 09:30-16:00 session, resamples to 15 minutes and flags buckets with fewer than 3 source bars. It ends with assertions: high and low contain open and close, volume adds up, daily closes match. On the default seed it turns 1,544 5-minute bars into 520 15-minute bars, 16 of them incomplete.
2. [`cross_asset_correlation.ipynb`](notebooks/cross_asset_correlation.ipynb): log returns for energy, metals, agriculture, shipping and insurance names, asset and sector correlation heatmaps, correlation with oil, and a 63-day rolling correlation with oil per sector. The synthetic prices come from a factor model with a known oil exposure per sector, so you can check the output against what was planted.
3. [`price_momentum.ipynb`](notebooks/price_momentum.ipynb): a 12-1 momentum ranking on 15 ETFs. Rank on the return from 12 months ago to 1 month ago, hold the top 3 for the next month, compare with an equal-weight basket and report turnover (27.6% a month one-way on the synthetic data). The point is the timing of signal and position.
4. [`implied_volatility.ipynb`](notebooks/implied_volatility.ipynb): Black-Scholes price and vega, an implied-vol solver (Newton with a bisection fallback), a synthetic option chain built from a known smile, the smile by expiry and the at-the-money term structure. Because the true vol is known, you can see the solver error: on the default seed all 65 options are recovered within 4 basis points of vol.

Each notebook writes its tables to `data/outputs/` at the repo root (git-ignored).

## Key concepts

- **OHLCV bar:** open, high, low, close and volume over a fixed interval.
- **`label` / `closed` in resampling:** which edge names a bucket and which edge belongs to it. Here both are `left`, so the 09:30 bar covers 09:30 to 09:45.
- **Incomplete bucket:** a resampled bar built from fewer source bars than expected. Flag it rather than trust it.
- **Log return:** `log(P_t / P_{t-1})`. Adds up over time, which makes it convenient for correlation work.
- **Correlation on common days:** `DataFrame.corr()` uses only the days both series have a value. With real data, names with shorter history are compared over fewer days.
- **Rolling correlation:** the same correlation over a moving window (63 trading days, about a quarter), to see if the relationship is stable.
- **12-1 momentum:** rank assets on their return from 12 months ago to 1 month ago; the last month is skipped because short-term returns tend to reverse.
- **No look-ahead:** weights decided at month-end t earn the returns of month t+1 (`weights.shift(1)`).
- **Turnover:** how much of the portfolio changes at each rebalance; half the sum of absolute weight changes.
- **Implied volatility:** the volatility that makes the Black-Scholes price equal the market price.
- **Vega:** how much the option price moves for a change in volatility. When it is tiny, Newton steps blow up, so the solver falls back to bisection inside a bracket.
- **Smile and term structure:** implied vol across strikes for one expiry, and at-the-money vol across expiries.

## Repo layout

```
notebooks/
  intraday_bar_resampling.ipynb   5-minute to 15-minute OHLCV with session filters and checks
  cross_asset_correlation.ipynb   asset and sector correlation with oil, full-period and rolling
  price_momentum.ipynb            12-1 cross-sectional momentum mechanics
  implied_volatility.ipynb        Black-Scholes, an implied-vol solver, smile and term structure
tests/
  test_notebooks.py   every notebook is valid, output-free, with no credentials or local paths
docs/
  README.md           index of the docs
  methodology.md      method notes and the choices behind each notebook
original/             the original research notebooks (read-only record)
.github/workflows/ci.yml   runs the tests on every push
.env.example · requirements.txt · CHANGELOG.md · CONTRIBUTING.md · LICENSE
```

## How to run

```bash
python -m venv .venv && source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt
python -m pytest -q
jupyter lab notebooks/
```

Outputs go to `data/outputs/` at the repo root (git-ignored), whether you start Jupyter from the repo root or from `notebooks/`. Set `DATA_DIR` to put them somewhere else. To use real public prices in the correlation and momentum notebooks, set `USE_YFINANCE=1` (see `.env.example`); that needs a network connection. Notebooks are committed without outputs; run them to see the tables and charts.

## What I learned

- Correlations with oil are not one number. A rolling window shows how much they drift, and that drift matters more than the full-period figure when you want a hedge or a proxy.
- A plain Newton solver for implied vol breaks on deep out-of-the-money options where vega is tiny. Keeping a bracket and falling back to bisection makes it reliable.
- Resampling bugs are quiet. Getting `label`/`closed` right, filtering the session first and flagging incomplete buckets saved me from a lot of misaligned bars later.
- In momentum code, the easy mistake is look-ahead: the signal has to be lagged before it is used for the next month's weights.

## Status

Learning and reference notebooks, not a trading system. The synthetic data is there so everything runs; any numbers it produces mean nothing about real markets.

## Original research code

[`original/`](original/README.md) holds the code as I first wrote it for this project. It is kept so you can
read the first version alongside the maintained one; some of it contains the issues in the audit table above,
and comparing the two shows each fix.

See [CONTRIBUTING.md](CONTRIBUTING.md) and the [CHANGELOG](CHANGELOG.md).
