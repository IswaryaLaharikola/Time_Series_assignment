# Appliance Energy Forecasting – Assignment Results

## Project Overview

This project investigates short-term household appliance energy forecasting using the **Appliances Energy Prediction** dataset. The analysis compares simple benchmark methods with SARIMAX, a feature-based Random Forest model, and the Chronos foundation model.

The original dataset is sampled every 10 minutes and was aggregated to hourly observations for this forecasting analysis. The forecasting objective is to predict the **next 24 hours** of appliance energy consumption. The original project brief identifies benchmark models, SARIMAX, feature-based models and foundation models as the main modelling approaches. 

## Dataset and Preprocessing

The dataset contains appliance energy consumption together with indoor temperature, indoor humidity and outdoor weather measurements.

The target variable is:

```text
Appliances
```

The processed analysis used:

- 19,735 original observations
- 28 original variables
- 10-minute original sampling frequency
- 3,290 hourly observations after aggregation
- No missing values identified
- `rv1` and `rv2` removed from the modelling data

The data were converted to datetime format, sorted chronologically and aggregated to hourly observations.

## Forecasting Design

The forecasting horizon is **24 hours**.

| Dataset | Start | End | Observations |
|---|---|---|---:|
| Training | 2016-01-11 17:00 | 2016-05-13 18:00 | 2,954 h |
| Test | 2016-05-13 19:00 | 2016-05-14 18:00 | 24 h |
| Subsequent observations | 2016-05-14 19:00 | 2016-05-27 18:00 | 312 h |

A chronological split was used so that future observations were not used during model training.

## Exploratory Time-Series Analysis

Stationarity was assessed using the Augmented Dickey-Fuller test. The original series produced an ADF statistic of **-8.949 with p < 0.001**, supporting stationarity.

ACF and PACF analysis identified important dependence around **lag 24**, corresponding to one day in the hourly dataset. Seasonal decomposition also indicated a clear daily seasonal pattern. These findings supported the use of a 24-hour seasonal component in SARIMAX.

## Models

### Benchmark Models

The following benchmark models were evaluated:

- Mean
- Naive
- Daily Seasonal Naive
- Weekly Seasonal Naive
- Drift

For hourly data, Daily Seasonal Naive uses lag 24 and Weekly Seasonal Naive uses lag 168.

### SARIMAX

An AIC-based search was performed across:

```text
p = 0 to 6
d = 0 to 2
q = 0 to 6
```

This resulted in **147 parameter combinations**.

The selected model was:

```text
SARIMAX(4,1,6) × (1,0,1,24)
```

A 24-hour forecast with 95% confidence intervals was produced.

### Random Forest

The feature-based Random Forest model used 62 features, including:

- Hour of day
- Day of week
- Cyclic time variables
- Appliance-energy lags
- Rolling statistics
- Sensor variables
- Weather variables

Lagged and rolling features were created using previous observations to reduce data leakage.

### Chronos

Chronos was evaluated as a time-series foundation model for the same 24-hour forecasting task.

## Benchmark Results

| Model | RMSE |
|---|---:|
| Mean | 50.01 |
| Naive | 247.60 |
| Daily Seasonal Naive | 118.04 |
| **Weekly Seasonal Naive** | **48.81** |
| Drift | 248.76 |

Weekly Seasonal Naive was the strongest benchmark, showing that weekly household patterns provide useful information for forecasting appliance energy use.

## Overall Results

| Rank | Model | RMSE | MAE | MAPE | MASE |
|---:|---|---:|---:|---:|---:|
| 1 | **SARIMAX** | **32.37** | **24.15** | 26.68 | **0.45** |
| 2 | Chronos | 47.69 | 33.36 | 24.73 | 0.62 |
| 3 | Weekly Seasonal Naive | 48.81 | 30.00 | **22.44** | 0.56 |
| 4 | Mean | 50.01 | 39.38 | 40.46 | 0.74 |
| 5 | Random Forest | 73.64 | 56.93 | 51.78 | 1.07 |
| 6 | Daily Seasonal Naive | 118.04 | 80.28 | 70.61 | 1.50 |
| 7 | Naive | 247.60 | 242.64 | 298.67 | 4.54 |
| 8 | Drift | 248.76 | 243.88 | 299.91 | 4.57 |

## Key Findings

- **SARIMAX was the best overall model**, with an RMSE of 32.37.
- **Weekly Seasonal Naive was the strongest benchmark**, with an RMSE of 48.81.
- Random Forest achieved an RMSE of 73.64 and therefore did not improve forecasting accuracy.
- Chronos achieved an RMSE of 47.69 but did not outperform SARIMAX.
- The results indicate that temporal dependence and daily seasonality are highly useful for this forecasting task.
- More complex models did not automatically produce better forecasts.

## Residual Diagnostics

The SARIMAX residuals had:

- Skewness: **2.03**
- Kurtosis: **11.60**
- Jarque-Bera p-value: **< 0.001**

This indicates a right-skewed and heavy-tailed residual distribution. Sudden appliance-energy spikes therefore remain difficult to model.

## Random Forest Feature Importance

The most important features included:

| Feature | Importance |
|---|---:|
| Hour | 0.27 |
| Appliances lag 168 | 0.10 |
| Appliances lag 24 | 0.09 |
| Hour sin | 0.08 |
| Appliances rolling mean 24 | 0.08 |
| Hour cos | 0.06 |

Time-of-day and historical appliance consumption were more informative than many additional sensor and weather variables.

## Forecast-Origin Considerations

Time variables and historical appliance measurements are available at the forecast origin. Actual future temperature, humidity, weather and sensor measurements would not normally be available.

Therefore, using realised future environmental measurements from the test set would produce a **conditional forecast**, rather than a fully realistic operational forecast.

## Comparison with Subsequent Observations

The subsequent **312 hourly observations** were retained after the formal 24-hour test period. They showed continued daily variation and occasional high-consumption spikes. These observations provide additional evidence that regular seasonal behaviour can be identified, while sudden energy changes remain harder to predict.

## Limitations

- Formal model evaluation was based on one 24-hour test period.
- Sudden appliance-energy spikes were difficult to predict.
- SARIMAX residuals were skewed and heavy-tailed.
- Future environmental variables may not be available at forecast time.
- Random Forest and Chronos did not outperform SARIMAX.

## Future Improvements

Future work could include:

- Rolling-origin evaluation over multiple forecast periods.
- Testing XGBoost or LightGBM.
- Using genuine weather forecasts instead of realised future weather.
- Improving modelling of extreme energy-consumption spikes.
- Testing additional SARIMAX specifications.
- Evaluating foundation models over multiple forecast origins.

## Figures to Include

The report should include:

1. Hourly appliance energy consumption.
2. ACF and PACF plots.
3. Seasonal decomposition.
4. SARIMAX forecast with 95% confidence intervals.
5. SARIMAX residual diagnostics.
6. Random Forest feature importance.
7. Overall model forecast comparison.
8. Comparison with subsequent observations.
9. Individual forecasts for all models in the appendix.

## Recommended Model

**SARIMAX is recommended** for this forecasting task because it achieved the lowest RMSE and MAE while providing an interpretable statistical framework and confidence intervals. Its performance indicates that modelling temporal dependence and daily seasonality is particularly effective for this appliance energy dataset.

## Report Structure

The accompanying report is structured as:

1. Introduction
2. Dataset and Preprocessing
3. Exploratory Time-Series Analysis
4. Forecasting Methodology
5. Benchmark Models
6. SARIMAX Modelling
7. Feature-Based Random Forest
8. Chronos Foundation Model
9. Overall Model Comparison
10. Critical Discussion
11. Comparison with Subsequent Observations
12. Limitations and Future Improvements
13. Conclusion
14. References

## Repository Structure

```text
appliance-energy-forecasting/
├── README.md
├── requirements.txt
├── data/
├── notebooks/
├── src/
├── scripts/
├── outputs/
│   ├── figures/
│   ├── forecasts/
│   └── metrics/
├── reports/
└── tests/
```

## Installation

```bash
python -m venv .venv
```

Windows:

```bash
.venv\Scripts\activate
```

Install dependencies:

```bash
pip install -r requirements.txt
```

Core packages include:

```text
numpy
pandas
matplotlib
scikit-learn
statsmodels
```

Additional packages may be required for Chronos or other foundation models.

## Running the Project

Run the main pipeline with:

```bash
python scripts/run_pipeline.py
```

The pipeline should load and preprocess the data, create features, train the models, generate forecasts, calculate evaluation metrics and save figures.

## Output Files

Recommended output locations:

```text
outputs/figures/
outputs/forecasts/
outputs/metrics/
```

The forecast output should contain actual values and predictions from the benchmark, SARIMAX, Random Forest and Chronos models.

## Reproducibility

All models should use the same chronological test period for fair comparison. Random seeds should be fixed where appropriate, and future target information should not be used when constructing forecasting features.
