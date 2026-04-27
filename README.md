# 🛒 Retail Sales Forecasting
 
**End-to-end time series forecasting project predicting retail sales using classical and modern forecasting models.**
 
---
 
## 📌 Project Overview
 
This project builds a complete forecasting pipeline on real retail sales data spanning **January 2011 to July 2013**. The goal was to accurately forecast future sales by analyzing seasonality, trends, and anomalies — then comparing multiple models to find the best performer.
 
---
 
## 📁 Dataset
 
| Property | Details |
|---|---|
| Records | 150,150 rows |
| Scope | Multiple stores & SKUs |
| Period | Jan 2011 – Jul 2013 |
| Key Features | `base_price`, `is_featured_sku`, `is_display_sku`, `units_sold` |
 
---
 
## 🔍 Key Findings
 
- 📈 Weekly sales volatility: **26%**
- 🏷️ Strongest sales drivers: `is_featured_sku` (+0.34 correlation) and `is_display_sku` (+0.36)
- 📅 Clear **annual seasonality** with constant amplitude → Additive decomposition applied
- ⚠️ Anomaly detected in **April 2012** — flagged and handled before modeling
---
 
## 🤖 Models & Results
 
| Model | MAPE | RMSE | Notes |
|---|---|---|---|
| **ARIMA(1,0,0)** ✅ | **8.53%** | **6,236** | Best overall accuracy |
| Prophet | 9.33% | 7,413 | Strong on seasonality |
| Holt-Winters | 12.78% | 9,297 | Weaker on this dataset |
 
✅ **ARIMA** selected as the final model with the lowest MAPE of **8.53%**
 
---
 
## 🛠️ Tech Stack
 
![Python](https://img.shields.io/badge/Python-3776AB?style=flat&logo=python&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-150458?style=flat&logo=pandas&logoColor=white)
![Statsmodels](https://img.shields.io/badge/Statsmodels-blue?style=flat)
![Prophet](https://img.shields.io/badge/Prophet-0693E3?style=flat)
![Scikit--learn](https://img.shields.io/badge/Scikit--learn-F7931E?style=flat&logo=scikit-learn&logoColor=white)
![Matplotlib](https://img.shields.io/badge/Matplotlib-blue?style=flat)
 
---
 
## 📂 Project Structure
 
```
retail-sales-forecasting/
│
├── retail_forecasting.ipynb   # Main notebook
├── data/
│   └── retail_sales.csv       # Raw dataset
├── screenshots/
│   └── dashboard.png          # Visualizations
└── README.md
```
 
---
 
## ▶️ How to Run
 
```bash
# Install dependencies
pip install pandas numpy matplotlib statsmodels prophet scikit-learn
 
# Launch notebook
jupyter notebook retail_forecasting.ipynb
```
 
---
 
## 📊 Workflow
 
```
Data Loading → EDA → Stationarity Testing (ADF) → Decomposition
→ ACF/PACF Analysis → Model Training → Evaluation → Forecasting
```
