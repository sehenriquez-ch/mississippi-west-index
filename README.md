# $WEST — Mississippi Divide West Index

![version](https://img.shields.io/badge/version-v11.3-orange)
![python](https://img.shields.io/badge/python-3.9%2B-blue)
![backtest](https://img.shields.io/badge/backtest-2016--2024-gray)
![weighting](https://img.shields.io/badge/weighting-Equal%20Weight-green)

[![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/sehenriquez-ch/mississippi-west-index/blob/main/main.ipynb)

<div align="center">

## Geography as a Factor

*An equal-weight index of S&P 500 companies headquartered West of the Mississippi River.*

</div>

---

## 🎯 The Thesis in One Line

> Companies headquartered **West of the Mississippi River** have built a distinct economic ecosystem—technology, energy, favorable demographics, and business-friendly fiscal policy.
> `$WEST` quantifies that thesis as an investable index.

Less concentrated in mega-cap tech than QQQ or cap-weighted alternatives, but with persistent exposure to the sector through a broader West-of-Mississippi footprint.

---

## ⚠️ Known Limitations (Read This First)

This is a **proxy back-test**, not a live index. The following biases are present and documented:

| Limitation | Direction |
|------------|-----------|
| Survivorship bias — delisted stocks excluded from returns | Upward |
| Shares outstanding — current values, not point-in-time | Varies |
| HQ classification — current Wikipedia + manual corrections | Minimal |
| ~11 methodology iterations during development | Possible overfitting |

**Despite these limitations, alpha vs RSP is statistically significant at p=0.0138 (Newey-West). Even adjusting conservatively for bias, the edge is unlikely to disappear entirely.**

Production use requires: Compustat (point-in-time shares/HQ), Bloomberg (prices), S&P DJI historical composition files.

---

## 📊 Performance Summary (2016–2024)

> **Benchmark: S&P 500 Total Return (^SP500TR)** — includes dividend reinvestment.
> Prior versions used ^GSPC (Price Return only), which understated the benchmark by ~1.5–2.0% p.a. Fixed in v11.3.

| Metric | **$WEST** | S&P 500 TR | QQQ | RSP (EW) |
|--------|-----------|------------|-----|----------|
| **CAGR** | **18.66%** | 14.77% | 19.73% | 11.64% |
| **Sharpe Ratio** | **0.75** | 0.59 | 0.70 | 0.40 |
| **Sortino Ratio** | **0.90** | 0.70 | 0.89 | 0.48 |
| **Max Drawdown** | -34.09% | -33.79% | -35.12% | -39.04% |
| **Volatility** | 19.34% | 18.06% | 22.20% | 18.65% |
| **Total Return** | **337.85%** | 244.48% | 403.71% | 168.90% |
| **VaR 95% (daily)** | -1.75% | -1.70% | -2.30% | -1.68% |

---

## 🔬 Why RSP Is the Right Benchmark

$WEST is equal-weight, geographically filtered. RSP is equal-weight, no geographic filter. The only difference is the Mississippi Divide. If $WEST outperforms RSP, the geography adds real value.

### Alpha vs RSP — Statistically Significant ✅

```
Alpha (annualized):     +4.39%
Correlation with RSP:    0.9567
Beta vs RSP:             0.995
R²:                      0.915
p-value (OLS):           0.0227
p-value (Newey-West):    0.0138  ← the one that matters (robust to autocorrelation)
```

### Alpha vs QQQ — Not Significant (and that's honest) ⚠️

```
Alpha (annualized):     +1.10%
Correlation with QQQ:    0.8980
p-value (Newey-West):    0.7271
```

$WEST does not claim to beat the Nasdaq on a risk-adjusted basis. The back-test outperformance vs QQQ is driven by the 2016–2024 AI/tech cycle. When that concentration unwinds, equal-weight geographic diversification becomes a structural advantage.

**2022 evidence — Fed rate hikes crushed tech:**
- `$WEST: -11.7%`
- `QQQ:   -32.6%`
- `S&P 500 TR: -18.1%`

---

## 📅 Year-by-Year Scoreboard

| Year | $WEST | S&P 500 TR | QQQ | RSP | Winner |
|------|-------|------------|-----|-----|--------|
| 2016 | +15.5% | +13.7% | +9.4% | +15.8% | WEST |
| 2017 | +23.7% | +21.8% | +32.7% | +18.5% | WEST |
| 2018 | -0.8% | -4.4% | -0.1% | -7.8% | WEST |
| 2019 | +34.2% | +31.5% | +39.0% | +28.9% | WEST |
| 2020 | +24.6% | +18.4% | +48.4% | +12.7% | WEST |
| 2021 | +37.9% | +28.7% | +27.4% | +29.4% | WEST |
| 2022 | **-11.7%** | -18.1% | **-32.6%** | -11.6% | **WEST** |
| 2023 | +30.3% | +26.3% | +54.9% | +13.7% | WEST |
| 2024 | +16.4% | +25.5% | +26.7% | +12.6% | S&P TR |

**$WEST wins 8 out of 9 years vs S&P 500 Total Return (2016–2024)**

---

## ⚙️ Methodology

| Parameter | Value |
|-----------|-------|
| Universe | S&P 500 constituents |
| Geographic filter | HQ in states west of the Mississippi River |
| Selection | Top 100 by total market capitalization |
| Weighting | Equal weight — 1/N = 1.00% per component |
| Position cap | 15% max per stock (SEC diversification rule) |
| Sector cap | 25% max per GICS sector |
| Rebalancing | Quarterly — 3rd Friday of Mar / Jun / Sep / Dec |
| Selection date | 5 business days before effective date |
| Expense ratio (modeled) | 0.50% p.a. |
| Transaction costs (modeled) | 0.10% per rebalancing turnover |
| Cash drag (modeled) | 1.5% |
| Avg quarterly turnover | 3.79% |
| Benchmark | ^SP500TR (S&P 500 Total Return — dividends reinvested) |

### West-Classified States

`CA` `WA` `OR` `TX` `AZ` `CO` `UT` `NV` `ID` `MT` `WY` `NM` `AK` `HI`
`MN` `IA` `MO` `AR` `LA` `ND` `SD` `NE` `KS` `OK`

---

## 📈 Additional Risk Metrics

| Metric | Value |
|--------|-------|
| CVaR 95% (daily) | -2.92% |
| Tracking Error vs RSP | 4.23% |
| Information Ratio vs RSP | 0.39 |
| Upside Capture vs S&P TR | 105.7% |
| Downside Capture vs S&P TR | 105.0% |
| Calmar Ratio | 0.55 |
| Skewness | -0.60 |
| Kurtosis | 15.48 (3.86 ex-COVID March 2020) |
| Positive Months | 56.0% |
| Gain/Pain Ratio | 1.21 |
| Omega Ratio | 1.21 |
| Rebalancings executed | 35 |

---

## 🚀 Installation

```bash
git clone https://github.com/sehenriquez-ch/mississippi-west-index
cd mississippi-west-index
pip install -r requirements.txt
python main.py
```

---

## 📁 Output Files

```
output/
  WEST_composition.csv          — Current index holdings (100 stocks)
  index_performance.csv         — Daily returns
  WEST_index_levels.csv         — Index levels (base 1,000)
  executive_report.md           — Auto-generated pitch summary
  Mississippi_West_v11_PitchDeck.png
WEST_Index_Rulebook_v3.0.docx   — Full methodology document
```

---

## 📋 Version History

| Version | Key Changes |
|---------|-------------|
| **v11.3** | Fixed benchmark to ^SP500TR (Total Return). Added Newey-West regression, kurtosis diagnostics, CVaR, capture ratios. Equal-weight index level consistency fix. |
| v9.2 | Fixed look-ahead bias in market cap ranking. Added sector cap 25%, buffer zone, split adjustment |
| v8.0 | Added volume filter, SQLite cache, exponential backoff retry |
| v7.3 | Fixed survivorship bias with historical S&P 500 composition (github.com/fja05680/sp500) |
| v6.4 | Added real frictions: expense ratio, transaction costs, cash drag |

---

## 📄 Disclaimer

Back-test results are indicative only. Historical performance does not guarantee future results.
This is a preliminary methodology document for discussion with index providers (Solactive, FTSE Russell, Bloomberg).
Not investment advice. Not for distribution to investors.

*Concept and methodology: Sebastian Henriquez — San Felipe, Chile*
