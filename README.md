# Retail Sales Forecasting

**Forecasting weekly retail demand from 150K store-SKU records: ARIMA vs Holt-Winters vs Prophet**

![Python](https://img.shields.io/badge/Python-3.x-3776AB?logo=python&logoColor=white)
![pandas](https://img.shields.io/badge/pandas-150150_rows-150458?logo=pandas&logoColor=white)
![statsmodels](https://img.shields.io/badge/statsmodels-ARIMA_%7C_Holt--Winters-4B8BBE)
![Prophet](https://img.shields.io/badge/Prophet-best_usable_model-0A66C2)
![Status](https://img.shields.io/badge/Status-Complete-2EA44F)

![Model comparison](images/model_comparison.png)

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [Business Problem](#business-problem)
3. [Dataset](#dataset)
4. [Approach](#approach)
5. [Exploratory Data Analysis](#exploratory-data-analysis)
6. [Time Series Analysis](#time-series-analysis)
7. [Forecasting and Model Selection](#forecasting-and-model-selection)
8. [Business Takeaways](#business-takeaways)
9. [Limitations and Next Steps](#limitations-and-next-steps)
10. [How to Run](#how-to-run)
11. [Repository Structure](#repository-structure)
12. [About the Author](#about-the-author)

---

## Executive Summary

| | |
|---|---|
| **Data** | 150,150 store-SKU weekly records, Jan 2011 to Jul 2013, rolled up to 130 weekly totals of units sold |
| **Level of demand** | Roughly 58K to 61K units per week, with no strong growth. The trend drifts down about 5% over the period |
| **Volatility** | Week-over-week change has a standard deviation of 26.1%, so the series is noisy |
| **What moves sales** | Display placement (+0.36) and featured placement (+0.34) correlate with units sold more than price does (-0.24) |
| **Final model** | **Prophet: 9.33% MAPE, 7,413 RMSE** on a 26-week holdout. That is 27% lower MAPE than Holt-Winters (12.78%) |
| **Model rejected despite a better score** | ARIMA scored 8.53% MAPE but forecasts a flat line, so it gives a planner nothing to act on |

---

## Business Problem

Replenishment and inventory planning depend on knowing how many units will sell next week. This project forecasts **total weekly units sold across all stores and SKUs**, compares three standard time series approaches, and checks whether the best number on paper is also the model you would use in practice.

---

## Dataset

- **Size:** 150,150 rows, 9 columns
- **Period:** January 2011 to July 2013 (weekly)
- **Grain:** one row per store, SKU and week

| Column | Description |
|---|---|
| `record_ID` | Sequential row ID |
| `week` | Week date (`dd/mm/yy`) |
| `store_id` | Store identifier |
| `sku_id` | Product identifier |
| `total_price` | Selling price |
| `base_price` | Base (list) price |
| `is_featured_sku` | 1 if the SKU was featured that week |
| `is_display_sku` | 1 if the SKU was on display that week |
| `units_sold` | Units sold (target) |

**Data quality:** the only missing value in the whole dataset is one `total_price`, filled with the column median. No other column has nulls.

---

## Approach

```
Load and clean  ->  EDA  ->  Weekly aggregation  ->  Time series diagnostics  ->  Model comparison  ->  Model selection
```

1. **Cleaning:** parsed `week` to datetime, filled the one missing price.
2. **EDA:** distributions, outliers, correlations with `units_sold`.
3. **Aggregation:** summed `units_sold` per week to get one time series.
4. **Diagnostics:** decomposition, moving averages, volatility, autocorrelation.
5. **Modeling:** chronological 80/20 split (104 training weeks, 26 test weeks), three models, MAPE and RMSE.
6. **Selection:** judged on error metrics **and** on whether the forecast follows the real shape of demand.

---

## Exploratory Data Analysis

### Correlation with units sold

![Correlation heatmap](images/correlation_heatmap.png)

| Feature | Correlation with `units_sold` |
|---|---|
| `is_display_sku` | **+0.36** |
| `is_featured_sku` | **+0.34** |
| `total_price` | -0.24 |
| `sku_id` | -0.18 |
| `base_price` | -0.14 |

### What stood out

- **Promotions carry the signal.** Display and featured flags correlate more strongly with sales than price does. Only about 9% of rows are featured and about 13% are on display, so promoted items are a small slice of the data.
- **Sales are heavily right-skewed.** Most rows sell under 100 units, with a few reaching around 3,000.
- **The two price columns are redundant.** `total_price` and `base_price` correlate at 0.96.
- **A small set of SKUs dominates.** One SKU cluster (IDs roughly 200K to 230K) accounts for 110K+ of the 150K records.
- **Store activity is uneven.** Some stores contribute 5 to 7 times more records than others.

> These are correlations, not causal effects. Featured and displayed items may also be the ones stores expect to sell well.

---

## Time Series Analysis

### Weekly units sold

![Weekly sales](images/weekly_sales.png)

Weekly totals range from **25,220** (week of 20 March 2012) to **109,207** (week of 14 February 2011).

| Event | Week | Units | Note |
|---|---|---|---|
| Highest week | 14 Feb 2011 | 109,207 | About double the weeks around it |
| Second spike | Dec 2011 | ~100K | Read from the plot |
| Third spike | 6 Mar 2012 | 88,928 | Two weeks before the trough |
| **Lowest week** | **20 Mar 2012** | **25,220** | Neighbouring weeks: 62,541 and 56,932 |

The 20 March 2012 drop is the clearest anomaly. It looks like missing data or an operational issue rather than a real demand collapse, but the dataset cannot confirm which. It was left in the data for modeling.

### Trend and seasonality

![Decomposition](images/decomposition.png)

- **Trend:** a 52-week centered average moves from about 61K down to about 58K (roughly -5%). Demand is stable, not growing.
- **Seasonality:** an additive decomposition with a 52-week period was fitted. With only about 2.5 years of data, the seasonal component rests on very few full cycles, so it should be read as indicative, not as proof of a strong annual pattern. Its deepest dip lines up with the March 2012 anomaly.
- **Residuals:** flat at zero for part of 2012. This is a side effect of fitting a 52-week period on a series this short, not a sign of missing data.

### Moving averages

![Moving averages](images/moving_averages.png)

The 50-week moving average stays close to 60K units per week for the whole period. The 4-week average follows the short-term swings, which are real demand movements and not just noise.

### Volatility and autocorrelation

![Autocorrelation](images/autocorrelation.png)

- **Volatility:** the standard deviation of week-over-week % change is **26.1%**.
- **Autocorrelation:** weak. Lag 1 is about 0.2, and there is a small bump near lag 52 to 53 of about 0.2. Both sit only just outside the 95% band. Past sales alone are a weak predictor, which suggests pure time series models will have limited accuracy here.

---

## Forecasting and Model Selection

**Setup:** chronological split, trained on Jan 2011 to Jan 2013 (104 weeks), tested on Jan 2013 to Jul 2013 (26 weeks).

| Model | Configuration | MAPE | RMSE (units/week) |
|---|---|---|---|
| ARIMA | order (1, 0, 0) | **8.53%** | **6,236** |
| Holt-Winters | additive seasonality, period 52 | 12.78% | 9,297 |
| **Prophet** | yearly seasonality, no external regressors | 9.33% | 7,413 |

![Holt-Winters forecast](images/holt_winters_forecast.png)

### Why Prophet, and not the lowest-error model

- **ARIMA wins on the metrics but produces a flat line.** Its forecast sits at the series mean for all 26 weeks. It has the best average error because it never takes a risk, and it cannot tell a planner anything about which weeks will be high or low.
- **Holt-Winters follows the shape but overshoots and undershoots.** It misses some weeks by a wide margin, including a predicted dip to around 37K in spring 2013.
- **Prophet follows the actual ups and downs** and stays within 9.33% MAPE. Against Holt-Winters it cuts MAPE by 27% and RMSE by 20%.

A forecast that scores well and says nothing is not a useful forecast, so Prophet is the model I would ship.

It is also worth noticing that a flat mean forecast beats both seasonal models on average error. That fits the autocorrelation result: most of the week-to-week variation is not explained by the series' own history.

---

## Business Takeaways

1. **Use the Prophet forecast as the baseline for weekly aggregate planning,** and expect an average miss of about 9%. Misses will be larger in promotion-driven spike weeks.
2. **Feed the promotion calendar into the next version.** Display and featured flags correlate with sales more strongly than price or past sales do, so they are the most promising extra inputs.
3. **Check the 20 March 2012 week with the data owner** before relying on it for training. One bad week at this size can distort seasonality estimates.
4. **Track MAPE week by week in production.** With 26% weekly volatility, a single bad week says little, but a rising rolling error would.

---

## Limitations and Next Steps

**Limitations**

- Single holdout split of 26 weeks. One window is a thin basis for ranking models.
- No naive or seasonal-naive baseline for comparison.
- All three models are univariate and use total weekly units only. Price, featured and display flags were not used as inputs.
- Forecasts are at total level. Nothing is forecast by store or SKU.
- About 2.5 years of history limits how well annual seasonality can be estimated.
- The March 2012 anomaly was not treated.

**Next steps**

- Add promotion share and average price as regressors (Prophet or SARIMAX).
- Rolling-origin cross-validation instead of a single split.
- Add a seasonal-naive baseline.
- Treat or flag the March 2012 anomaly and compare results.
- Forecast at SKU or store level for the dominant SKU group.
- Apply a log transform to the skewed target for any row-level model.

---

## How to Run

```bash
git clone https://github.com/zeyad902/Retail-Sales-Forecasting-project.git
cd Retail-Sales-Forecasting-project
pip install pandas numpy matplotlib seaborn statsmodels scikit-learn prophet jupyter
jupyter notebook e-commerce.ipynb
```

Place the dataset at `data/data.csv` before running.

---

## Repository Structure

```
Retail-Sales-Forecasting-project/
├── data/
│   └── data.csv
├── images/                 # Plots used in this README
├── e-commerce.ipynb        # Full analysis: cleaning, EDA, time series, forecasting
└── README.md
```

---

## About the Author

**Zeyad Mohamed Goda**: Business Information Systems student at Port Said University (expected graduation 2027), building a portfolio of end-to-end data projects toward a Data Analyst role.

- GitHub: [zeyad902](https://github.com/zeyad902)
- LinkedIn: [zeyad-mohamed-goda](https://www.linkedin.com/in/zeyad-mohamed-goda)
- Email: zeyadmohamedgoda@gmail.com
