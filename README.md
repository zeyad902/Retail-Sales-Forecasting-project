# Retail Sales Forecasting

## Overview
End-to-end time series forecasting project on retail sales data
spanning January 2011 to July 2013.

## Dataset
- 150,150 records across multiple stores and SKUs
- Features: `base_price`, `is_featured_sku`, `is_display_sku`, `units_sold`

## Key Findings
- Weekly sales volatility: **26%**
- Strongest sales drivers: `is_featured_sku` (+0.34) and `is_display_sku` (+0.36)
- Clear annual seasonality with constant amplitude → Additive decomposition
- Anomaly detected in **April 2012**

## Models & Results
| Model | MAPE | RMSE |
|---|---|---|
| ARIMA(1,0,0) | 8.53% | 6,236 |
| Holt-Winters | 12.78% | 9,297 |
| Prophet ✅ | 9.33% | 7,413 |

## Tech Stack
Python · Pandas · Statsmodels · Prophet · Matplotlib · Scikit-learn

## How to Run
```bash
pip install pandas numpy matplotlib statsmodels prophet scikit-learn
jupyter notebook retail_forecasting.ipynb
```
