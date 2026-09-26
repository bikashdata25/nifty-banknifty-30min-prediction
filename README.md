# NIFTY & BANK NIFTY 30-Minute Direction Prediction System

A Python-based market intelligence and machine learning system designed to analyze **NIFTY 50** and **BANK NIFTY** market behavior and estimate the probability of their short-term direction during the first 30 minutes of the trading session.

The project combines historical market data, pre-market/pre-open behavior, opening gaps, intraday price action, technical features, machine learning, and real-time market data through the **Dhan API**.

> **Disclaimer:** This project is for educational, research, and paper-trading purposes. Market predictions are probabilistic and do not guarantee future returns. This system should not be treated as financial advice.

---

## Project Objective

The primary objective is to answer:

> **Given the previous day's market behavior, pre-market/pre-open information, opening gap, and early-session price action, can we estimate whether NIFTY or BANK NIFTY is likely to move UP, DOWN, or remain NEUTRAL over the next 30 minutes?**

Example:

```text
Previous Day Close     : 24,500
Opening Price          : 24,650
Gap                    : +0.61%

First 5-Min Return     : +0.12%
First 10-Min Return    : +0.21%
VWAP Position          : Above VWAP
RSI                    : 63.4

Model Prediction       : UP
Probability            : 73.2%
```

The system focuses on **probabilistic prediction**, not guaranteed market forecasting.

---

# Architecture

```text
                         ┌───────────────────┐
                         │     Dhan API      │
                         └─────────┬─────────┘
                                   │
                    ┌──────────────┴──────────────┐
                    │                             │
              Historical Data               Live WebSocket
                    │                             │
                    └──────────────┬──────────────┘
                                   │
                                   ▼
                          Python Data Layer
                                   │
                                   ▼
                           Raw Market Data
                                   │
                                   ▼
                         Data Processing Layer
                                   │
                            Tick → 1-Minute
                                   │
                                   ▼
                         Feature Engineering
                                   │
                                   ▼
                         ML Feature Dataset
                                   │
                    ┌──────────────┴──────────────┐
                    │                             │
                    ▼                             ▼
               Model Training                Backtesting
                    │                             │
                    ▼                             │
              Trained Model                      │
                    │                             │
                    └──────────────┬──────────────┘
                                   ▼
                          Live Prediction Engine
                                   │
                                   ▼
                         Streamlit Dashboard
```

---

# Key Features

## Market Data

The system is designed to work with:

* NIFTY 50 data
* BANK NIFTY data
* Historical OHLCV data
* 1-minute market data
* Live market data
* Pre-market/pre-open information where available
* Opening price
* Previous trading session information

Live market data can be obtained through the Dhan API.

---

# Gap Analysis

One of the core components of the project is analyzing **gap-up and gap-down behavior**.

For example:

```text
Previous Close = 24,500
Opening Price  = 24,650

Gap Points = 24,650 - 24,500
           = +150

Gap % = 150 / 24,500 × 100
      = +0.61%
```

The system can classify sessions into:

```text
GAP UP
GAP DOWN
NO SIGNIFICANT GAP
```

It can then investigate what historically happened after similar gaps.

---

# Feature Engineering

The feature engineering layer will create features from multiple categories.

## Gap Features

```text
gap_points
gap_percentage
gap_direction
gap_size
gap_size_bucket
```

## Previous-Day Features

```text
previous_day_return
previous_day_range
previous_day_high
previous_day_low
previous_day_close
previous_day_volatility
previous_day_close_position
```

## Opening-Session Features

```text
first_1min_return
first_5min_return
first_10min_return
first_15min_return

opening_range
opening_range_percentage

first_5min_high
first_5min_low
first_15min_high
first_15min_low
```

## Momentum Features

```text
RSI
EMA_9
EMA_20
VWAP
ROC
```

## Volatility Features

```text
ATR
rolling_std
range_percentage
```

## Volume Features

```text
volume
relative_volume
volume_change
volume_ratio
```

Additional features will be added only after evaluating whether they provide useful out-of-sample information.

---

# Target Variable

The initial target is the market direction over a future 30-minute window.

The system will use three classes:

```text
UP
DOWN
NEUTRAL
```

Conceptually:

```text
Future Return > Positive Threshold
        ↓
       UP

Future Return < Negative Threshold
        ↓
      DOWN

Otherwise
        ↓
     NEUTRAL
```

The thresholds will be determined through historical analysis and validated through out-of-sample testing.

---

# Important: Avoiding Data Leakage

Financial time-series prediction requires strict control of future information.

For example, if the model makes a prediction at:

```text
09:20 AM
```

it must only use information available up to:

```text
09:20 AM
```

It must not use:

```text
09:25 price
09:30 price
day high
day low
day close
future volume
future indicators
```

The project therefore separates:

```text
FEATURES
Information available at prediction time

TARGET
Future market behavior
```

This is one of the most important principles of the project.

---

# Machine Learning

The project will use progressively more sophisticated models.

### Baseline

```text
Logistic Regression
```

### Tree-Based Models

```text
Random Forest
XGBoost
LightGBM
```

### Future Research

Potential future experiments may include:

```text
LSTM
Temporal Neural Networks
Transformer-based Time-Series Models
```

Complex models will only be introduced if simpler models establish a meaningful baseline.

---

# Backtesting

Random train/test splitting is avoided because market data is time-dependent.

Instead, the project will use:

```text
Walk-Forward Validation
```

Example:

```text
Train: 2019–2022
Test : 2023

Train: 2019–2023
Test : 2024

Train: 2019–2024
Test : 2025

Train: 2019–2025
Test : 2026
```

This better reflects how a model would encounter future market data.

---

# Evaluation Metrics

The project will evaluate both machine-learning performance and market-related performance.

### Classification Metrics

```text
Accuracy
Precision
Recall
F1 Score
ROC-AUC
Confusion Matrix
```

### Probability Quality

```text
Prediction Calibration
Confidence Distribution
Probability vs Actual Outcome
```

### Research / Backtesting Metrics

```text
Average Return
Win Rate
Maximum Drawdown
Sharpe Ratio
Profit Factor
Transaction Costs
Slippage
```

A high classification accuracy alone does not necessarily imply that a strategy has economic value.

---

# Project Structure

```text
nifty-banknifty-30min-prediction/
│
├── README.md
├── requirements.txt
├── .env.example
├── .gitignore
├── main.py
│
├── config/
│   ├── settings.py
│   └── instruments.py
│
├── dhan/
│   ├── auth.py
│   ├── rest_client.py
│   ├── websocket_client.py
│   └── instrument_mapper.py
│
├── ingestion/
│   ├── historical_data.py
│   ├── live_data.py
│   ├── premarket_data.py
│   └── data_validator.py
│
├── processing/
│   ├── tick_to_candle.py
│   ├── candle_processor.py
│   └── market_session.py
│
├── features/
│   ├── gap_features.py
│   ├── price_features.py
│   ├── momentum_features.py
│   ├── volatility_features.py
│   ├── volume_features.py
│   ├── technical_features.py
│   ├── market_structure.py
│   └── feature_pipeline.py
│
├── targets/
│   └── target_builder.py
│
├── models/
│   ├── baseline.py
│   ├── train.py
│   ├── predict.py
│   ├── evaluate.py
│   ├── model_registry.py
│   └── saved/
│
├── backtesting/
│   ├── walk_forward.py
│   ├── backtest.py
│   ├── metrics.py
│   └── reports.py
│
├── live_prediction/
│   ├── prediction_engine.py
│   ├── scheduler.py
│   ├── signal_generator.py
│   └── prediction_logger.py
│
├── dashboard/
│   ├── app.py
│   ├── charts.py
│   └── components.py
│
├── notebooks/
│   ├── 01_data_exploration.ipynb
│   ├── 02_gap_analysis.ipynb
│   ├── 03_feature_analysis.ipynb
│   ├── 04_model_experimentation.ipynb
│   └── 05_backtesting.ipynb
│
├── tests/
│   ├── test_dhan.py
│   ├── test_candles.py
│   ├── test_features.py
│   ├── test_targets.py
│   └── test_model.py
│
├── data/
│   ├── raw/
│   ├── processed/
│   └── features/
│
└── logs/
```

---

# Data Flow

The complete pipeline will follow:

```text
Dhan API
   ↓
Data Ingestion
   ↓
Raw Data
   ↓
Data Validation
   ↓
Tick Processing
   ↓
1-Minute OHLCV
   ↓
Feature Engineering
   ↓
Feature Dataset
   ↓
Target Creation
   ↓
Training Dataset
   ↓
ML Model
   ↓
Backtesting
   ↓
Model Evaluation
   ↓
Live Prediction
   ↓
Dashboard
```

---

# Live Prediction Workflow

During a trading session:

```text
09:15
  ↓
Market Opens
  ↓
Start Live Data Collection
  ↓
Build 1-Minute Candles
  ↓
09:20
  ↓
Generate Features
  ↓
Run NIFTY Model
  ↓
Run BANK NIFTY Model
  ↓
Generate Probabilities
  ↓
Log Prediction
```

The system can then update predictions at predefined intervals.

Example:

```text
09:20 → UP 64%
09:25 → UP 71%
09:30 → UP 74%
09:35 → UP 77%
```

The actual result can later be compared against each prediction.

---

# Dashboard

The planned Streamlit dashboard will display:

```text
NIFTY

Previous Close     24,500
Opening Price      24,640
Gap                +0.57%

5M Return          +0.12%
10M Return         +0.19%
15M Return         +0.26%

VWAP Position      Above
RSI                62.1

Prediction         UP
Probability        73.4%
```

A corresponding BANK NIFTY section will display the same information.

Historical prediction performance will also be available.

---

# Technology Stack

## Programming

```text
Python
```

## Data Processing

```text
Pandas
NumPy
PyArrow
Parquet
```

## Machine Learning

```text
Scikit-learn
XGBoost
LightGBM
```

## Market Data

```text
Dhan API
Dhan WebSocket
```

## Visualization

```text
Matplotlib
Plotly
Streamlit
```

## Testing

```text
Pytest
```

---

# Installation

Clone the repository:

```bash
git clone https://github.com/<YOUR_USERNAME>/nifty-banknifty-30min-prediction.git

cd nifty-banknifty-30min-prediction
```

Create a virtual environment:

```bash
python -m venv .venv
```

Activate it on Windows:

```bash
.venv\Scripts\activate
```

Install dependencies:

```bash
pip install -r requirements.txt
```

---

# Environment Variables

Create a `.env` file:

```env
DHAN_CLIENT_ID=your_client_id
DHAN_ACCESS_TOKEN=your_access_token
```

Never commit the `.env` file.

The repository should contain:

```text
.env
```

inside `.gitignore`.

---

# Development Roadmap

## Phase 1 — Data Foundation

* [ ] Configure Dhan API
* [ ] Identify NIFTY instrument
* [ ] Identify BANK NIFTY instrument
* [ ] Download historical data
* [ ] Build standardized OHLCV schema
* [ ] Store data as Parquet
* [ ] Implement data validation

## Phase 2 — Market Analysis

* [ ] Calculate gap-up/gap-down statistics
* [ ] Analyze gap continuation
* [ ] Analyze gap filling
* [ ] Analyze opening range
* [ ] Analyze first 5/10/15 minutes
* [ ] Analyze NIFTY vs BANK NIFTY relationship

## Phase 3 — Feature Engineering

* [ ] Gap features
* [ ] Previous-day features
* [ ] Opening-range features
* [ ] Momentum features
* [ ] Volatility features
* [ ] Volume features
* [ ] Market-structure features

## Phase 4 — Machine Learning

* [ ] Build baseline
* [ ] Train Logistic Regression
* [ ] Train Random Forest
* [ ] Train XGBoost
* [ ] Train LightGBM
* [ ] Compare models
* [ ] Feature importance analysis
* [ ] Probability calibration

## Phase 5 — Backtesting

* [ ] Walk-forward validation
* [ ] Out-of-sample testing
* [ ] Transaction-cost assumptions
* [ ] Slippage analysis
* [ ] Drawdown analysis
* [ ] Performance report

## Phase 6 — Live System

* [ ] Dhan WebSocket
* [ ] Real-time tick ingestion
* [ ] Tick-to-candle engine
* [ ] Real-time feature generation
* [ ] Live prediction engine
* [ ] Prediction logging

## Phase 7 — Dashboard

* [ ] Streamlit application
* [ ] Live NIFTY prediction
* [ ] Live BANK NIFTY prediction
* [ ] Probability visualization
* [ ] Historical predictions
* [ ] Model performance dashboard

---

# Future Architecture

Once the Python-only system is stable, it can be extended into a cloud data-engineering architecture:

```text
Dhan API
    ↓
Azure Data Factory / Python
    ↓
ADLS Gen2
    ↓
Databricks
    ↓
Delta Lake
    ↓
PySpark
    ↓
Feature Engineering
    ↓
MLflow
    ↓
Model Registry
    ↓
Real-Time Prediction API
    ↓
Power BI / Streamlit
```

This will transform the project from a Python ML experiment into a complete **Data Engineering + Machine Learning + MLOps platform**.

---

# Research Principles

This project follows several principles:

1. No future information in features.
2. Time-aware train/test splitting.
3. Walk-forward validation.
4. Out-of-sample evaluation.
5. Baseline models before complex models.
6. Probability calibration rather than binary predictions alone.
7. Transaction costs and slippage should be considered.
8. Live predictions should initially be treated as paper/research signals.
9. Model performance should be continuously monitored.
10. Historical performance does not guarantee future performance.

---

# Disclaimer

This repository is intended for **educational and research purposes only**.

Financial markets are uncertain and highly dynamic. Machine-learning predictions can fail, including during regime changes, extreme volatility, unexpected news events, liquidity changes, and data-quality issues.

Nothing in this project constitutes financial, investment, or trading advice.

---

# License

This project is intended for educational and research use.

Add an appropriate open-source license before publicly distributing the repository.
