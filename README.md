# Market Data and Volatility Notebooks

[![CI](https://github.com/Avinash186200/market-data-and-volatility-notebooks/actions/workflows/ci.yml/badge.svg)](https://github.com/Avinash186200/market-data-and-volatility-notebooks/actions/workflows/ci.yml) ![Python](https://img.shields.io/badge/python-3.11-blue) [![License: MIT](https://img.shields.io/badge/license-MIT-green)](LICENSE)

A small set of research notebooks I keep coming back to: cross-asset correlation, implied volatility, intraday bar resampling and a basic momentum ranking. Each one is self-contained and runs offline on synthetic data; two of them can pull public prices from Yahoo Finance instead.

## Structure

```
notebooks/
  cross_asset_correlation.ipynb   asset and sector correlation with oil, full-period and rolling
  implied_volatility.ipynb        Black-Scholes, an implied-vol solver, smile and term structure
  intraday_bar_resampling.ipynb   5-minute to 15-minute OHLCV with session filters and checks
  price_momentum.ipynb            12-1 cross-sectional momentum mechanics
```

## How to run

```bash
python -m venv .venv && source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt
jupyter lab notebooks/
```

Outputs go to `data/outputs/` (git-ignored). Set `DATA_DIR` to put them somewhere else. To use real public prices in the correlation and momentum notebooks, set `USE_YFINANCE=1` (see `.env.example`).

## What I learned

- Correlations with oil are not one number. A rolling window shows how much they drift, and that drift matters more than the full-period figure when you want a hedge or a proxy.
- A plain Newton solver for implied vol breaks on deep out-of-the-money options where vega is tiny. Keeping a bracket and falling back to bisection makes it reliable.
- Resampling bugs are quiet. Getting `label`/`closed` right, filtering the session first and flagging incomplete buckets saved me from a lot of misaligned bars later.
- In momentum code, the easy mistake is look-ahead: the signal has to be lagged before it is used for the next month's weights.

## Status

Learning and reference notebooks, not a trading system. The synthetic data is there so everything runs; any numbers it produces mean nothing about real markets.

## Original research code

[`original/`](original/README.md) holds the code as I first wrote it for this work, sanitised and nothing else:
client, company and person names replaced, credentials and local paths removed, data files and notebook outputs
dropped. It is kept so the real work can be read alongside the maintained version. Some of it contains the issues
described above; comparing the two shows each fix.

See [CONTRIBUTING.md](CONTRIBUTING.md) and the [CHANGELOG](CHANGELOG.md).
