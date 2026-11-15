# Methodology

Short notes on the choices in each notebook. The README has the reading order; this file says why the code is
written the way it is.

## Data

Every notebook generates synthetic data in its own cells, with a fixed seed (42), so it runs offline and gives the
same numbers each time. The correlation and momentum notebooks can download public prices from Yahoo Finance
instead when `USE_YFINANCE=1`. Synthetic numbers say nothing about real markets.

## Intraday bar resampling

- Filter to the trading session first (09:30 to 16:00, end excluded), then resample. Resampling first can create
  buckets that straddle the open or the close.
- `label="left", closed="left"`: the 09:30 bucket holds the 09:30, 09:35 and 09:40 bars.
- Open is the first open, high the max, low the min, close the last close, volume the sum.
- Count source bars per bucket. Empty buckets are dropped; buckets with fewer than 3 bars are kept but flagged
  `complete = False`.
- Checks at the end: high and low contain open and close, total volume is unchanged, and the last close of each
  day is the same in both series.

## Cross-asset correlation

- Log returns, not prices. Prices trend, and two trending series look correlated even when they are not.
- Sector series are equal-weight averages of the members' returns.
- `corr()` is pairwise: each pair uses the days both have data. With real data this matters for names that
  listed later.
- Full-period correlation with oil is one number. The 63-day rolling correlation shows how much it moves; the
  table of its mean, standard deviation, min and max is the stability check.
- The synthetic model gives each sector a fixed oil exposure (energy 1.0, shipping 0.4, metals 0.3, agriculture
  0.25, insurance 0.05) plus a global factor, a sector factor and noise, so the ordering of the results can be
  checked against what was planted.

## Price momentum

- Month-end prices. Signal at month-end t is `P(t-1) / P(t-12) - 1`: the last month is skipped.
- Top 3 of 15 by signal, equal weight.
- `weights.shift(1)`: the position chosen at the end of month t earns month t+1's return. Without the shift the
  strategy would trade on returns it already knows.
- Turnover is half the sum of absolute weight changes per month.
- No costs, and the synthetic drifts are random, so the equity curve is only there to show the mechanics.

## Implied volatility

- Black-Scholes with a flat rate (3%) and no dividends.
- The chain quotes out-of-the-money options only (puts below spot, calls at and above), as is common in practice.
  Prices get 0.2% noise.
- The solver keeps a bracket `[lo, hi]` and moves it with each guess. It takes a Newton step when vega is large
  enough and the step stays inside the bracket, otherwise it bisects. Prices at or below intrinsic value return
  NaN, since no volatility can produce them.
- The true smile is known, so the error in basis points of vol is reported. On the default seed all 65 options
  are within 4 basis points.
- The last cell compares a 21-day realised vol of a simulated path with the 30-day at-the-money implied level.

## Known limitations

- Synthetic data only by default; the downloads are not cached or cleaned.
- No transaction costs in the momentum notebook.
- The implied-vol chain uses one rate and no dividends.
