# $WEST Methodology

> **Version:** v11.3 (Level Fix + Kurtosis Diagnostic)  
> **Last Updated:** 2026-08-01  
> **Status:** Research Core — Frozen

This document describes the complete methodology behind the Mississippi Divide West Index ($WEST). It is written for quantitative researchers, index providers, and anyone who wants to audit or replicate the backtest.

---

## 1. Index Objective

$WEST seeks to capture the structural economic divergence between companies headquartered **West of the Mississippi River** and the broader U.S. equity market, using an equal-weight construction to avoid concentration in mega-cap technology stocks.

---

## 2. Universe Definition

### 2.1 Primary Universe
The index draws from historical S&P 500 constituents. The source is the public dataset maintained by [fja05680/sp500](https://github.com/fja05680/sp500), which provides month-by-month ticker lists including additions and removals.

### 2.2 Data Source
- **Historical composition:** CSV from GitHub (cached locally)
- **Prices & volume:** Yahoo Finance (yfinance)
- **Company info:** Wikipedia (GICS sector, headquarters state)
- **Shares outstanding:** Yahoo Finance `fast_info` (current proxy)

### 2.3 Download Window
Prices are downloaded from **start date minus 365 days** to **end date** to allow for 6-month ADV calculation before the first selection date.

---

## 3. Geographic Classification

### 3.1 The Mississippi Divide
Each U.S. state is classified as **East**, **West**, or **Neutral** based on its position relative to the Mississippi River. The classification map is static and hardcoded.

**West States:** CA, WA, OR, TX, AZ, CO, UT, NV, ID, MT, WY, NM, AK, HI, MN, IA, MO, AR, LA, ND, SD, NE, KS, OK

**East States:** NY, NJ, PA, MA, CT, MD, VA, NC, SC, GA, FL, OH, IL, MI, IN, WI, KY, TN, WV, DE, VT, NH, ME, RI, AL, MS, DC

### 3.2 HQ Mapping
For each ticker, the headquarters state is extracted from Wikipedia. If a ticker has multiple share classes, the primary class is used.

### 3.3 Historical Corrections
Known corporate relocations are corrected with date ranges:

| Ticker | From State | To State | Effective Date |
|--------|------------|----------|----------------|
| TSLA | CA | TX | 2021-12-01 |
| ORCL | CA | TX | 2020-12-01 |
| HPQ | CA | TX | 2020-12-01 |
| SCHW | CA | TX | 2021-01-01 |
| CBRE | CA | TX | 2020-01-01 |
| TMUS | — | WA | Permanent |

### 3.4 Delisted Tickers
For tickers no longer in the S&P 500, historical HQ data is preserved in static lookup tables (`SEDES_HISTORICAS_OESTE` and `SEDES_HISTORICAS_ESTE`) to prevent look-ahead bias.

---

## 4. Selection Criteria

### 4.1 Geographic Filter
Only tickers classified as **West** on the selection date are eligible.

### 4.2 Liquidity Filter
Average Daily Value (ADV) must exceed **$500,000 USD**.

```
ADV = avg(volume over 180 days) × price_on_selection_date
```

Tickers with insufficient volume history are excluded.

### 4.3 Price Filter
The ticker must have a valid closing price on the selection date (no NaN).

### 4.4 Ranking & Selection
Eligible tickers are ranked by **float-adjusted market capitalization** (price × shares outstanding × free-float factor). The top **100** are selected. If fewer than 100 meet all criteria, all qualifying tickers are included.

---

## 5. Weighting Scheme

### 5.1 Base Weight
Selected constituents are weighted **equally (1/N)**.

```
weight_i = 1 / N
```

### 5.2 Position Cap
No single position may exceed **15%** of the index. In practice, with 100 constituents, the base weight is ~1%, so this cap rarely binds.

### 5.3 Sector Cap
No single GICS sector may exceed **25%** of the index. This is enforced via iterative proportional scaling:

1. Calculate sector totals.
2. If any sector > 25%, scale all constituents in that sector down proportionally.
3. Redistribute excess weight to uncapped sectors.
4. Repeat up to **10 iterations** or until convergence.

### 5.4 Free-Float Adjustment
Free-float factors are estimated using public proxy data:

```
free_float = (shares_outstanding - strategic_holdings) / shares_outstanding
```

Strategic holdings (insider, government, PE) are estimated from known public filings. Default assumption: 20% strategic holdings if no data is available.

---

## 6. Rebalance Schedule

### 6.1 Rebalance Dates
Quarterly on the **3rd Friday** of:
- March
- June
- September
- December

### 6.2 Selection Date
**5 trading days before** the rebalance date. This mimics the notice period required by real-world index providers.

### 6.3 Implementation
- New constituents and weights are determined on the selection date.
- The index level transitions to the new basket at the close of the rebalance date.
- Returns between rebalance dates are calculated using the **previous basket's weights**.

---

## 7. Index Calculation

### 7.1 Daily Returns
For each trading day within a rebalance period:

```
index_return_t = Σ (weight_i × return_i,t)
```

Where `return_i,t` is the daily price return of constituent `i` on day `t`.

### 7.2 Cumulative Level
The index level is the cumulative product of daily returns, base 1,000:

```
level_t = 1000 × Π (1 + index_return_t)
```

### 7.3 Turnover Calculation
Turnover at each rebalance is measured as half the sum of absolute weight changes:

```
turnover = 0.5 × Σ |weight_new,i - weight_old,i|
```

---

## 8. Risk Metrics

### 8.1 Return Metrics
- **CAGR:** Compound Annual Growth Rate
- **Total Return:** Cumulative return over full period
- **Yearly Returns:** Calendar-year grouped returns

### 8.2 Risk Metrics
- **Volatility:** Annualized standard deviation of daily returns
- **Sharpe Ratio:** (CAGR - risk_free) / volatility
- **Sortino Ratio:** (CAGR - risk_free) / downside deviation
- **Max Drawdown:** Maximum peak-to-trough decline
- **Calmar Ratio:** CAGR / |max drawdown|

### 8.3 Tail Risk
- **VaR 95%:** 5th percentile of daily returns
- **CVaR 95%:** Mean of returns below the 5th percentile
- **Skewness:** Asymmetry of return distribution
- **Kurtosis:** Tail thickness (diagnostic for outlier-driven alpha)

### 8.4 Relative Metrics
- **Beta:** Covariance($WEST, benchmark) / variance(benchmark)
- **Alpha:** Excess return unexplained by beta
- **Tracking Error:** Standard deviation of active returns
- **Information Ratio:** Alpha / tracking error
- **Upside/Downside Capture:** Ratio of mean returns in up/down markets

---

## 9. Statistical Testing

### 9.1 Alpha Regression
Daily returns are regressed against benchmark daily returns:

```
r_WEST = alpha + beta × r_benchmark + epsilon
```

### 9.2 Newey-West HAC
Standard errors are corrected for autocorrelation using the Newey-West heteroskedasticity-and-autocorrelation-consistent estimator.

**Lag selection:**
```
lags = max(1, floor(4 × (n/100)^(2/9)))
```

Where `n` is the number of daily observations.

### 9.3 Significance Threshold
Alpha is considered statistically significant if **p-value < 0.05** under Newey-West.

### 9.4 Benchmarks Tested
- **RSP:** S&P 500 Equal-Weight Index (primary)
- **QQQ:** Nasdaq-100 (secondary)
- **^GSPC:** S&P 500 Cap-Weight (reference)

---

## 10. Kurtosis Diagnostic

To ensure alpha is not driven by a handful of extreme days, a kurtosis diagnostic is run:

1. Identify top 15 positive and negative daily returns.
2. Test exclusion of known market stress periods (COVID crash, Ukraine invasion, etc.).
3. If kurtosis remains elevated (>8) after exclusion, flag potential data errors.

**Result:** Kurtosis of 15.56 is explained by known market events. No isolated data anomalies detected.

---

## 11. Known Limitations

| # | Limitation | Impact | Mitigation |
|---|------------|--------|------------|
| 1 | Shares outstanding are current proxies, not historical | May distort historical market cap and ADV | Documented; seeking Compustat/CRSP |
| 2 | Free-float is estimated, not measured | Weight calculations carry approximation error | Conservative estimates; institutional validation sought |
| 3 | Transaction costs not modeled | Real-world returns would be lower | On roadmap (~20-50bps annual impact estimated) |
| 4 | Selection uses T+0 prices | Real execution requires T+1/T+2 slippage | Documented |
| 5 | ~11 methodology iterations | Risk of data-mining bias | Walk-forward 2025-2026 as antidote |
| 6 | Yahoo Finance data quality | Possible errors in dividends, splits, corporate actions | Cross-validated against known prices |

---

## 12. Data Sources

| Data | Source | URL |
|------|--------|-----|
| S&P 500 Historical Components | fja05680/sp500 | https://github.com/fja05680/sp500 |
| Prices & Volume | Yahoo Finance (yfinance) | https://finance.yahoo.com |
| Company Info | Wikipedia | https://en.wikipedia.org/wiki/List_of_S%26P_500_companies |
| Shares Outstanding | Yahoo Finance fast_info | Via yfinance API |

---

## 13. Version History

| Version | Date | Key Changes |
|---------|------|-------------|
| v11.3 | 2026-08-01 | Level now derived from equal-weight returns (cumulative product); added kurtosis diagnostic |
| v11.2 | 2026-07-XX | Added Newey-West HAC; walk-forward 2025-2026 |
| v11.1 | 2026-07-XX | Switched from cap-weight to equal-weight |
| v10.x | 2026-06-XX | Earlier iterations with various weighting schemes |

---

## 14. License & Attribution

This methodology is published as open research. The code is available under the MIT License. If you use this methodology in published work, please cite this repository.

---

*For questions or corrections, open an issue on GitHub.*
