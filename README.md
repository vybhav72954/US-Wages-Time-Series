# Forecasting Real Wages in the United States

A state-of-the-art multi-model time series analysis of U.S. real hourly wages, built as a Statistical & Predictive Analytics project for the **Post Graduate Diploma in Business Analytics (PGDBA)** at the **Indian Statistical Institute, Kolkata**.

The study extends well beyond the course-taught SARIMA framework to benchmark classical, additive-decomposition, machine-learning, and deep-learning forecasters against each other on the same series.

---

## Authors

- **Vybhav Chaturvedi** (25BM6JP60)
- **Pranav Taneja** (25BM6JP37)

Under the guidance of **Dr. Samarjit Bose**, Professor, Indian Statistical Institute, Kolkata.

December 2025.

---

## Problem

Real wages — nominal earnings adjusted for inflation — measure the actual purchasing power of labour. They drive conversations about living standards, labour-market health, monetary policy, and the "wage stagnation" narrative. They are also non-trivial to forecast: structural shifts (1970s stagflation, 2008 crisis, COVID-19), seasonality, business-cycle autocorrelation, and shifting industry mix all contribute noise.

This project asks: **given monthly U.S. nominal earnings and CPI, what forecasting approach gives the most stable and accurate projection of real wages over short, medium, and long horizons?**

## Data

| Field | Value |
|---|---|
| Frequency | Monthly |
| Period | 1 Jan 1964 – 1 Aug 2025 (~740 observations) |
| Nominal earnings | `AHETPI` — Average Hourly Earnings of Production and Nonsupervisory Employees (U.S. BLS, via FRED) |
| Price index | `CPIAUCSL` — CPI for All Urban Consumers, All Items (U.S. BLS, via FRED) |

Real wages are constructed as `Real Wage_t = Nominal Wage_t / CPI_t`. Modelling is performed on the year-over-year transformation `YoY_t = (Real_t - Real_{t-12}) / Real_{t-12} × 100`, which removes the long-run inflationary drift and deterministic seasonality, leaving a series that passes stationarity tests directly.

> Raw CSVs are excluded from version control (see `.gitignore`). Pull fresh copies from FRED:
> - https://fred.stlouisfed.org/series/AHETPI
> - https://fred.stlouisfed.org/series/CPIAUCSL
>
> Place them in `data/AHETPI.csv` and `data/CPIAUCSL.csv` before running the notebook.

## Methodology

1. **Construction & EDA** — build real-wage series, plot nominal/CPI/real/YoY views, summary statistics.
2. **Decomposition** — STL decomposition on real wages and on YoY change to isolate trend, seasonal, and residual components.
3. **Stationarity testing** — ADF and KPSS on the raw real-wage series and on the YoY-transformed series.
4. **Autocorrelation diagnostics** — ACF / PACF on raw, first-differenced, and YoY series to guide order selection.
5. **SARIMA identification** — grid search over `(p,d,q)(P,D,Q)_12` candidates, ranked by AIC / BIC, RMSE, Ljung–Box residual whiteness, and parameter significance.
6. **Rolling-origin evaluation** — RMSE computed across 3-, 6-, 12-, and 36-month horizons over multiple origins covering the 2001 recession, 2008 crisis, and 2020 pandemic.
7. **Benchmarking** — SARIMA compared against:
   - **Prophet** (additive trend + Fourier seasonality)
   - **LSTM** recurrent neural network
   - **LightGBM**, **Theta**, **Gradient Boosting** baselines
   - **Ensemble** approaches: simple average, weighted average, and a meta-learner
8. **Final forecast** — 12-month and extended 36-month projections with 95% confidence intervals.

## Key Findings

- The preferred model is **SARIMA(1,1,2)(2,0,1)\_12**, with **SARIMA(3,1,2)(1,0,1)\_12** as a close runner-up.
- SARIMA models produce the most stable and accurate forecasts across all tested horizons. Forecast accuracy declines as horizon length grows, as expected.
- **Prophet** captures long-run macro trend cleanly but underperforms on short-term dynamics; reduced YoY seasonality erodes its main advantage on this series.
- **LSTM** is hamstrung by sample size (~740 monthly points vs. the thousands LSTMs typically need); residuals retain autocorrelation, indicating incomplete learning.
- **Ensembles** reduce variance and are robust across regimes but do **not** beat the top SARIMA — the ensemble is dominated by its SARIMA weight.
- **Forecast outlook:** modest, positive YoY real-wage growth over the next 12 months, with confidence intervals widening monotonically over the 36-month horizon.

## Repository Layout

```
.
├── US-Hourly-Wages Analysis.ipynb     # Main analysis notebook (EDA → SARIMA → Prophet/LSTM/Ensemble → forecast)
├── requirements.txt                   # Python dependencies
├── Group_3 - US Wages Time Forecasting.pdf   # Final report (~30 pages)
├── data/                              # (gitignored) AHETPI.csv, CPIAUCSL.csv from FRED
├── Final/                             # (gitignored) generated figures
├── Sample/                            # (gitignored) reference reports
└── LICENSE                            # MIT
```

## Reproducing the Analysis

```bash
# 1. Clone
git clone https://github.com/vybhav72954/US-Wages-Time-Series.git
cd US-Wages-Time-Series

# 2. Create a virtual environment (Python 3.10+ recommended)
python -m venv timeseries
# Windows:
timeseries\Scripts\activate
# macOS / Linux:
source timeseries/bin/activate

# 3. Install dependencies
pip install -r requirements.txt

# 4. Download the FRED CSVs into ./data/ (see Data section above)

# 5. Launch the notebook
jupyter notebook "US-Hourly-Wages Analysis.ipynb"
```

The notebook runs end-to-end top-to-bottom. Prophet, LSTM, and ensemble sections are independent of each other once the SARIMA block has run, so they can be re-executed in isolation.

## Stack

`numpy` · `pandas` · `statsmodels` · `pymannkendall` · `scikit-learn` · `prophet` · `tensorflow` (LSTM) · `lightgbm` · `arch` · `ydata-profiling` · `matplotlib` · `seaborn` · `tqdm` · `openpyxl`

See `requirements.txt` for the full list.

## Report

The complete report — full diagnostic plots, model-selection tables, residual analysis, and the 36-month forecast — is in **`Group_3 - US Wages Time Forecasting.pdf`**.

## License

[MIT](LICENSE).
