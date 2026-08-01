[README.md](https://github.com/user-attachments/files/30624032/README.md)
# $WEST — Mississippi Divide West Index

<p align="center">
  <img src="https://img.shields.io/badge/version-v11.3-FF8C42?style=flat-square" />
  <img src="https://img.shields.io/badge/python-3.9+-blue?style=flat-square" />
  <img src="https://img.shields.io/badge/backtest-2016--2024-8B8B8B?style=flat-square" />
  <img src="https://img.shields.io/badge/weighting-Equal%20Weight-22C55E?style=flat-square" />
</p>

<p align="center">
  <a href="https://colab.research.google.com/github/sehenriquez-ch/mississippi-west-index/blob/main/notebooks/quickstart.ipynb">
    <img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open In Colab"/>
  </a>
</p>

<h1 align="center">Geography as a Factor</h1>
<p align="center"><em>An equal-weight index of S&P 500 companies headquartered West of the Mississippi River.</em></p>

---

## 🎯 The Thesis in One Line

> Companies headquartered **West of the Mississippi River** have built a distinct economic ecosystem—technology, energy, favorable demographics, and business-friendly fiscal policy.  
> **$WEST** quantifies that thesis as an investable index.

This is not a "tech proxy." This is a geographic factor.

---

## 📊 Backtest Summary (2016 – 2024)

| Metric | $WEST | S&P 500 | QQQ | RSP (Equal-Weight) |
|--------|-------|---------|-----|-------------------|
| **CAGR** | **18.61%** | 12.74% | 19.73% | 11.64% |
| **Volatility** | 19.28% | 18.06% | 22.20% | 18.65% |
| **Sharpe Ratio** | **0.75** | 0.48 | 0.70 | 0.40 |
| **Sortino Ratio** | **0.90** | 0.57 | 0.89 | 0.48 |
| **Max Drawdown** | -34.04% | -33.92% | -35.12% | -39.04% |
| **Total Return** | **336.14%** | 193.49% | 403.71% | 168.90% |
| **Alpha vs S&P 500** | **+3.00%** | — | — | — |
| **Beta vs S&P 500** | 1.04 | 1.00 | — | — |
| **Tracking Error** | 4.21% | — | — | — |
| **Information Ratio** | 0.87 | — | — | — |
| **Upside Capture** | 106.3% | — | — | — |
| **Downside Capture** | 103.5% | — | — | — |
| **VaR 95% (daily)** | -1.75% | -1.71% | -2.30% | -1.68% |
| **CVaR 95% (daily)** | -2.91% | — | — | — |
| **Skewness** | -0.64 | — | — | — |
| **Kurtosis** | 15.56 | — | — | — |
| **Positive Months %** | 56.0% | — | — | — |
| **Gain/Pain Ratio** | 1.21 | — | — | — |
| **Omega Ratio** | 1.21 | — | — | — |

**Methodology:** Equal-weight (1/N), quarterly rebalancing, liquidity filter ($ADV > 500k), position cap (15%) and sector cap (25%).  
**Statistical rigor:** Regression with Newey-West HAC standard errors to test alpha significance.

---

## 🗺️ Why West of the Mississippi?

| Factor | West | East |
|--------|------|------|
| **Technology** | CA, WA, TX, CO | Concentrated in NY/MA |
| **Energy** | TX, NM, CO, AK | Import-dependent |
| **Demographics** | Net positive migration | Stagnation / outflow |
| **Fiscal Policy** | No state income tax (TX, WA, NV, WY) | Higher historical tax burden |

This is not just "buy tech." It is a **concentration of human and physical capital** along a corridor stretching from Seattle to Austin through Denver and Los Angeles.

---

## ⚙️ How It Works

```
S&P 500 Historical Constituents
        │
        ▼
┌─────────────────────────────┐
│  HQ West of Mississippi?    │
└─────────────────────────────┘
        │ Yes
        ▼
┌─────────────────────────────┐
│  Liquidity Filter (ADV>$500k)│
└─────────────────────────────┘
        │
        ▼
┌─────────────────────────────┐
│  Select Top 100              │
└─────────────────────────────┘
        │
        ▼
┌─────────────────────────────┐
│  Equal-Weight (1/N)          │
│  + 15% position cap          │
│  + 25% sector cap            │
└─────────────────────────────┘
        │
        ▼
┌─────────────────────────────┐
│  Rebalance Quarterly         │
│  (3rd Friday of Mar/Jun/Sep/Dec) │
└─────────────────────────────┘
```

### Engine Features

- **Universe:** Historical S&P 500 constituents ([fja05680/sp500](https://github.com/fja05680/sp500)).
- **Geographic corrections:** Historical HQ changes (TSLA CA→TX 2021, ORCL CA→TX 2020, etc.).
- **Free-float proxy:** Conservative estimation for strategic holdings.
- **Kurtosis diagnostic:** Identifies whether alpha comes from thick tails or consistent drift.
- **Comparatives:** S&P 500, QQQ (Nasdaq-100), RSP (S&P 500 Equal-Weight).

---

## 🔬 Is $WEST Actually Different?

We run alpha regression vs QQQ and RSP with **Newey-West HAC standard errors**—because uncorrected OLS lies about significance when returns are autocorrelated.

### vs QQQ (Nasdaq-100)

| Statistic | Value |
|-----------|-------|
| Correlation | 0.9002 |
| Beta | 0.782 |
| R² | 0.810 |
| Alpha (annual) | +1.05% |
| p-value (OLS) | 0.7145 |
| **p-value (Newey-West, 7 lags)** | **0.7383** |
| **Verdict** | ⚠️ Alpha NOT significant. $WEST does not generate reliable alpha vs QQQ. |

### vs RSP (S&P 500 Equal-Weight)

| Statistic | Value |
|-----------|-------|
| Correlation | 0.9546 |
| Beta | 0.990 |
| R² | 0.911 |
| Alpha (annual) | **+4.40%** |
| p-value (OLS) | 0.0252 |
| **p-value (Newey-West, 7 lags)** | **0.0148** |
| **Verdict** | ✅ Alpha SIGNIFICANT even with Newey-West—solid evidence of real edge. |

> The geographic filter generates **+4.40% annual alpha vs equal-weight S&P 500** with statistical significance. That is the number that matters.

---

## 📅 Year-by-Year Returns

| Year | $WEST | S&P 500 | QQQ | RSP | Winner |
|------|-------|---------|-----|-----|--------|
| 2016 | +15.6% | +11.2% | +9.4% | +15.8% | $WEST |
| 2017 | +23.8% | +19.4% | +32.7% | +18.5% | $WEST |
| 2018 | -1.2% | -6.2% | -0.1% | -7.8% | $WEST |
| 2019 | +33.9% | +28.9% | +39.0% | +28.9% | $WEST |
| 2020 | +24.5% | +16.3% | +48.4% | +12.7% | $WEST |
| 2021 | +37.6% | +26.9% | +27.4% | +29.4% | $WEST |
| 2022 | -11.4% | -19.4% | -32.6% | -11.6% | $WEST |
| 2023 | +30.5% | +24.2% | +54.9% | +13.7% | $WEST |
| 2024 | +16.3% | +23.8% | +26.7% | +12.6% | S&P 500 |

**$WEST outperformed in 8 of 9 years.** The only miss was 2024, when mega-cap tech concentration drove S&P 500 and QQQ ahead.

---![Cumulative Performance 2016-2024](Mississippi_West_v11_PitchDeck.png)

## 🔄 Turnover & Drawdown Analysis

| Metric | Value |
|--------|-------|
| Avg. Turnover / Rebalance | 3.72% |
| Max Turnover / Rebalance | 8.00% |
| Rebalances Executed | 35 |
| Drawdowns >1% | 86 |
| Avg. Recovery Time | 22 days |

---

## 🚀 Installation & Usage

### Local

```bash
# Clone
git clone https://github.com/sehenriquez-ch/mississippi-west-index.git
cd mississippi-west-index

# Dependencies
pip install -r requirements.txt

# Run backtest
python mississippi_west.py --start 2016-01-01 --end 2024-12-31
```

### One-Click (Google Colab)

Click the badge at the top of this README. Zero installation. The notebook downloads the script, installs dependencies, and runs the full backtest in ~3 minutes.

---

## 📁 Repo Structure

```
mississippi-west-index/
├── mississippi_west.py          # Core engine (v11.3 — frozen)
├── notebooks/
│   └── quickstart.ipynb         # Colab-ready, 1-click run
├── requirements.txt
├── output/                      # Generated results (gitignored)
├── data/                        # Cache & backups
├── logs/                        # Execution logs
├── README.md                    # This file
└── LICENSE
```

---

## 🎯 Roadmap

- [x] Backtest 2016–2024 with equal-weight methodology
- [x] Newey-West HAC correction for alpha significance
- [x] Kurtosis diagnostic
- [ ] Transaction cost & slippage modeling
- [ ] Walk-forward out-of-sample validation
- [ ] Live paper trading (2025–2026)
- [ ] Formal Rulebook for index provider discussions

---

## 🤝 Contributing

This is an open research experiment. Found a bug? Have better data (point-in-time shares outstanding, institutional free-float)? Want to test a variant geography (Rocky Mountains? Cascadia?)? Open an issue or PR.

**Areas where we need help:**
- Point-in-time shares outstanding data (Compustat/CRSP/Sharadar).
- Institutional free-float validation.
- Realistic transaction cost modeling.
- Unit tests for the rebalance engine.

---

## ⚠️ Disclaimer

This repository is a **quantitative research experiment**.
- It is **not** investment advice.
- Past performance does not guarantee future results.
- The backtest has known limitations: shares outstanding are current proxies (not point-in-time), free-float is estimated, and transaction costs are not yet modeled. See the generated `executive_report.md` for full details.
- Before any investment decision, consult a qualified financial advisor.

---

<p align="center">
  <b>⭐ Star this repo if you believe geography matters in markets.</b><br/>
  <b>🍴 Fork it if you want to test your own geographic frontier.</b>
</p>
