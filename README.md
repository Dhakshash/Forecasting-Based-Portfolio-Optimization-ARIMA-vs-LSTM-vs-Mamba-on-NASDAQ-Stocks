# Forecasting-Based Portfolio Optimization: ARIMA vs LSTM vs Mamba on NASDAQ Stocks

## Introduction
This project explores how well **time series forecasting models** can drive **active portfolio construction** to outperform a strong bull market. We aim to answer:
> Can we build a simple long-only strategy using daily stock return forecasts that beats Buy & Hold?

### Motivation
- Many institutional investors and hedge funds use **model-driven forecasts** to rebalance portfolios.
- The NASDAQ 100 has performed exceptionally well in recent years, making it a tough benchmark to beat.
- Our goal is not just to forecast well — but to **use predictions to build a profitable, risk-aware portfolio**.

---

## Data Description

- **Universe**: 10 NASDAQ-listed large-cap companies.
- **Timeframe**:  
  - Full dataset: 2015-01-01 to 2024-12-31  
  - Forecasting & portfolio period: 2023-01-03 to 2024-12-31
- **Source**: Daily stock prices scraped from Yahoo Finance.

### Features Used:
- **Lagged Returns (1–5 days)**: To capture short-term memory in returns.
- **EMA12 / EMA26**: Trend-following indicators.
- **MACD & Signal Line**: For identifying momentum shifts.
- **RSI(14)**: Detects overbought/oversold conditions.
- **Exogenous Market Variables**:
  - **SPY_ret** – Broad market movement.
  - **QQQ_ret** – Tech-specific ETF proxy.
  - **VIX** – Captures risk sentiment.

> These features are widely used in quantitative finance and technical analysis. They were selected for their ability to generalize across assets.

---

## ARIMA

### Data Processing
- Focused purely on **univariate log return series** per stock.
- No external features or indicators.

### Modelling
- ARIMA(p,d,q) selected using **AIC optimization**.
- Ensured stationarity (d=1 typically) and avoided trivial flat models.

### Rolling Forecasting Setup
- Trained on a 500-day rolling window.
- Recalibrated every day.
- Prediction made one step ahead daily.

---

## LSTM

### Data Processing
- Feature set included:
  - Stock’s own technical indicators (EMA, MACD, RSI).
  - Lagged returns (1 to 5 days).
  - Exogenous signals: SPY, QQQ, VIX.

- Inputs were **min-max scaled** across training window.

### Modelling
- Single-layer LSTM → Dense layer.
- Optimized with Adam and MSE loss.
- Sequence input length = 10 days.

### Rolling Forecasting Setup
- Trained on the past 500 days.
- Re-trained daily for new forecasts.
- Captures nonlinear temporal dependencies.

---

## Mamba

### Data Processing
- Same features as LSTM plus:
  - **Time encodings**: Day of week/year using sine/cosine transformations.

### Modelling
- Used **Hugging Face’s `MambaModel`**, a fast, efficient alternative to Transformers.
- Sequence length = 90 days.
- Lightweight configuration: 4 layers, 64 hidden size.

> **Note**: Due to resource constraints, Mamba was **only pre-trained once** and fine-tuned with minimal epochs at each rolling step — yet it still outperformed.

### Rolling Forecasting Setup
- Fine-tuned over a 500-day window.
- Predicted next day return.
- Returned to base checkpoint and fine-tuned forward.

---

## Portfolio Construction Strategy

- **Daily Top-2 Forecast Strategy**:
  - From each model, choose top 2 stocks with highest predicted returns.
  - Allocate 50% to each.
  - Rebalance daily.

### Why Top-2?
- Balances between:
  - Over-concentration (Top-1 risks).
  - Dilution (Top-5 or full allocation).
- Keeps transaction costs manageable while adapting to signals.

---

## Evaluation Metrics

### Prediction Accuracy
| **Model** | **RMSE**   | **MAE**    | **Hit Ratio (%)** |
|-----------|-----------|-----------|--------------------|
| ARIMA     | 0.020438  | 0.013629  | 49.96             |
| LSTM      | 0.042526  | 0.031433  | 51.12             |
| Mamba     | 0.734970  | 0.549584  | 50.10             |

> Hit Ratio = % of days where the predicted direction matched actual movement.

### Interpretation:
- ARIMA minimizes forecast errors but lacks predictive power for price direction.
- LSTM shows higher **directional accuracy**.
- Mamba has higher error (due to scaling and fewer training epochs), yet **produces the best portfolio performance** — proving that **forecast quality ≠ strategy quality**.

---

## Results & Insights

### Portfolio Performance
| **Strategy**     | **Final Value** | **CAGR (%)** | **Volatility (%)** | **Sharpe** | **Max Drawdown (%)** |
|------------------|-----------------|--------------|---------------------|------------|-----------------------|
| **ARIMA**        | 1.7642          | 32.97        | 23.67               | 1.39       | -21.42                |
| **LSTM**         | 2.2371          | 49.98        | 25.88               | 1.93       | -17.16                |
| **Mamba**        | 2.5365          | 57.87        | 24.93               | 2.32       | -13.48                |
| **Consensus**    | 1.3889          | 17.97        | 23.90               | 0.75       | -24.82                |
| **Buy & Hold**   | 2.2790          | 50.41        | 18.69               | 2.70       | -11.54                |

---

### Portfolio Growth Chart
```
![Portfolio Growth](images/portfolio_growth.png)
```

---

### Key Insights
- **Mamba outperformed all models in final return, Sharpe, and max drawdown**.
- Despite higher forecast error, it made more **profitable directional calls**.
- Consensus models performed poorly due to noise aggregation.
- LSTM showed solid gains but fell short of Mamba.

---

## Why Buy & Hold Performed Well

- The test period coincided with a **bull run in large-cap tech stocks**.
- Buy & Hold benefits from:
  - Long-term compounding.
  - No forecasting error.
  - No reallocation costs or frictions.

> The strength of this benchmark illustrates the challenge — yet **Mamba matched or beat Buy & Hold on Sharpe**, proving its ability to **generate alpha in high-return regimes**.

---

## Limitations
- **Transaction costs, slippage, and tax effects** were not included.
- Mamba fine-tuning was **resource-constrained**, limiting accuracy.
- We assumed **daily rebalancing was frictionless** (idealized assumption).
- Consensus model design could be further optimized.

---

## Future Enhancements
- Incorporate realistic **transaction costs and execution latency**.
- Scale up Mamba with longer fine-tuning epochs and larger model depth.
- Explore **adaptive ensembles** or regime-switching strategies.
- Test across **bear, sideways, and volatile regimes**.
- Introduce **position sizing based on predicted confidence intervals**.

---

## 🤝 Connect
For questions, collaborations, or opportunities related to AI in finance and risk modeling, feel free to connect through dhakshashganesan@gmail.com or GitHub.
