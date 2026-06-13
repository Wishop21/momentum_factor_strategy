# Momentum + Multi-Factor Equity Strategy

A systematic long-only equity strategy combining four return factors (Momentum, Value, Quality, and Low Volatility) applied monthly across 200 S&P 500 stocks. The project's main focus is validation rigour: a strict in-sample / out-of-sample split, walk-forward analysis, and block-bootstrap Monte Carlo simulation, rather than just reporting a backtest Sharpe.

---

## What it does

Each month, stocks are scored on a composite signal built from four cross-sectionally z-scored factors:

| Factor | Definition | Weight |
|--------|-----------|--------|
| Momentum | 12-1 month price return | 35% |
| Value | Inverse of P/B ratio | 25% |
| Quality | ROE + gross margin composite | 25% |
| Low Volatility | Inverse of 12-month realised vol | 15% |

The top quintile by composite score is held equally weighted, rebalanced monthly with 10 bps one-way transaction costs applied to turnover only. No leverage, no short selling.

---

## Validation pipeline

The analysis is structured to make overfitting hard to hide:

1. In-sample (2010–2018) — factor weights and portfolio rules calibrated here, then frozen
2. Out-of-sample (2019–present) — primary test; parameters untouched since 2018
3. Walk-forward analysis — rolling 36-month Sharpe to assess stability across regimes
4. Monte Carlo bootstrap — 5,000 block-resampled paths to frame the outcome distribution
5. Factor attribution — single-factor portfolios to identify what's actually driving returns
6. Regime analysis — performance conditional on bull/bear market environment

---

## Key results

> These numbers run from live notebook output and will update each time the notebook is run against current data. The figures below are representative of results as of mid-2025.

| Metric | Strategy (OOS) | SPY (OOS) |
|--------|---------------|-----------|
| CAGR | ~14–16% | ~13–15% |
| Sharpe Ratio | ~0.85–1.0 | ~0.75–0.90 |
| Max Drawdown | Lower than SPY | Benchmark |
| Win Rate | ~58–62% | ~58–60% |

Rolling walk-forward shows positive Sharpe in the majority of 36-month windows, with the weakest periods concentrated around 2022–2023 (rate shock regime).

---

## Honest limitations

This is a research prototype, not a production system. Three issues materially affect the headline numbers:

- Survivorship bias — the universe is the current S&P 500 constituent list, not a historical one. Bankrupt and acquired companies are excluded, which inflates returns by roughly 1–2% p.a.
- Look-ahead bias in fundamentals — Value and Quality use a static fundamental snapshot pre-2021, not point-in-time quarterly data. This is genuine look-ahead bias; the Quality factor's strong single-factor Sharpe is likely inflated by it.
- Universe concentration — with 200 similar-quality large-caps, all four factors tend to select overlapping stocks. Factor portfolio return correlations are 0.77–0.88 despite near-zero cross-sectional signal correlations.

The price-based factors (Momentum and Low Volatility) are unaffected by the data quality issues and represent the more credible alpha sources.

---

## Project structure

```
.
├── momentum_factor_strategy.ipynb   # Main notebook — run top to bottom
├── data_cache/                      # Auto-created on first run (gitignored)
│   ├── prices.parquet
│   ├── fundamentals_static.parquet
│   └── fundamentals_ts.parquet
└── *.png                            # Charts saved on each run
```

---

## How to run

Install dependencies:
```bash
pip install numpy pandas matplotlib seaborn scipy tqdm yfinance pyarrow
```

Run the notebook:
```bash
jupyter notebook momentum_factor_strategy.ipynb
```

Run cells top to bottom. On first run, data is downloaded from Yahoo Finance and cached locally. Subsequent runs load from the Parquet cache and are fast.

To force a fresh data download, delete the `data_cache/` directory:
```bash
rm -rf data_cache/
```

---

## Dependencies

| Package | Purpose |
|---------|---------|
| `yfinance` | Price and fundamental data |
| `pandas` / `numpy` | Data manipulation |
| `matplotlib` / `seaborn` | Visualisation |
| `scipy` | Regression (alpha/beta calculation) |
| `tqdm` | Progress bars during data fetch |
| `pyarrow` | Parquet cache read/write |

Python 3.9+ recommended.

---

## Further work

The highest-priority improvement is replacing the static fundamental snapshot with a true point-in-time database (Compustat or similar). Until that's done, Value and Quality results should be treated as illustrative. Beyond that, the walk-forward analysis points to dynamic factor weighting as the most promising extension: the macro regime clearly affects which factors are in favour, and a static weight scheme leaves that information unexploited.
