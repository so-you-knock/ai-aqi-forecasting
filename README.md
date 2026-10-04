# 🌫️ AI-Based Air Quality Forecasting System

**Forecasts PM2.5 and AQI for Indian cities — and fixes a data-leakage bug the original version had.**

![Python](https://img.shields.io/badge/Python-3776AB?style=flat&logo=python&logoColor=white)
![scikit-learn](https://img.shields.io/badge/scikit--learn-F7931E?style=flat&logo=scikitlearn&logoColor=white)
![Streamlit](https://img.shields.io/badge/Streamlit-FF4B4B?style=flat&logo=streamlit&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green.svg)

---

## What it does

Two Random Forest Regressors forecast PM2.5 and AQI for Indian cities using lag/rolling pollution features, an Isolation Forest flags anomalous pollution spikes, and an interactive Streamlit dashboard exposes real-time prediction, trend analysis, and city comparison — including a 7-step autoregressive outlook.

## A real bug, found and fixed

The original version of this project computed lag/rolling features (PM2.5 yesterday, 2 days ago, etc.) by shifting the **entire** dataframe — ignoring city boundaries. Because rows from different cities were interleaved, a "lag" feature for a Chennai row could actually be pulling a completely unrelated city's PM2.5 value from the row above it: a data-leakage bug that inflates apparent accuracy while producing features that don't mean what they claim to. **Fixed** by computing lag/rolling features with `groupby("City")`, so a city's lag features only ever look at that city's own history.

## Architecture

```
Raw Excel dataset (10,000 rows, 12 Indian cities)
        │
        ▼
Feature engineering (PER CITY via groupby): PM25_lag1/2/3, PM25_roll3,
                                              aqi_lag1/2/3, aqi_roll3
        │
        ├──► Isolation Forest ──► anomaly flag
        │
        ├──► RandomForestRegressor #1 (16 features) ──► predicts PM2.5
        │
        └──► RandomForestRegressor #2 (17 features, includes PM2.5) ──► predicts AQI
        │
        ▼
Streamlit dashboard: Prediction · Pollution Analysis · City Comparison
```

## Key engineering decisions

- **Fixed the cross-city data leakage** described above — the single highest-value change made to the original pipeline.
- **Two-stage forecast, not a redundant input.** The original UI asked the user to type in PM2.5 *and* separately showed a predicted PM2.5 — two numbers that could disagree. Redesigned as a genuine pipeline: predict PM2.5 first, then feed that prediction into the AQI model as an input feature.
- **Lag context pulled from history automatically.** A fresh manual entry has no history of its own — the dashboard pulls the selected city's most recent record to supply realistic lag features, the way a live sensor feed would.

## Results

| Model | Features | RMSE | R² |
|---|---|---|---|
| PM2.5 (Random Forest, 200 trees, max_depth 12) | 16 | 7.94 | 0.987 |
| AQI (Random Forest, 200 trees, max_depth 12) | 17 | 15.34 | 0.986 |

Isolation Forest flagged ≈1.01% of readings as anomalous pollution spikes.

## Tech stack

Python 3.10 · scikit-learn (RandomForestRegressor, IsolationForest) · pandas · NumPy · Streamlit · Plotly · openpyxl

## Project structure

```
train_models.py             Cleans data, engineers features per-city, trains both models
app.py                       Streamlit dashboard (3 tabs + 7-step outlook)
pm25_model.pkl / aqi_model.pkl
processed_pollution_data.csv
requirements.txt
```

## Run it locally

```bash
pip install -r requirements.txt
python train_models.py      # regenerates the models + processed_pollution_data.csv
streamlit run app.py
```

## License

MIT
