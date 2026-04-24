# ⚡ Grid Demand Forecasting: Multi-Year Hourly Prediction Engine

This repository contains a high-precision forecasting pipeline designed to predict hourly electrical grid demand. Utilizing **LightGBM** and **Optuna**, the model achieves a **2.006% Test MAPE** on unseen 2024–2025 data by integrating historical demand, meteorological variables, and macro-economic indicators.

## 🚀 Performance Overview
* **Final Test MAPE:** 2.006%
* **Final Test RMSE:** 347.01 MW
* **Validation MAPE (2023):** 2.072%
* **Training History:** 75,088 hours (2015–2023)

---

## 🛠️ The Pipeline Architecture

### 1. Data Cleaning & Contextual Imputation
Raw grid data often contains gaps that simple linear interpolation would destroy by smoothing out daily cycles.
* **Seasonal Imputation:** Missing values were filled using **Hourly Seasonal Imputation**—calculating the mean of that specific hour and month (e.g., average solar at 1 PM in July). This preserves the "heartbeat" of the grid.
* **Contextual Outlier Neutralization:** Implemented a rolling 48-hour Z-score filter. Instead of deleting spikes, we neutralized them to the local mean, ensuring the model learned from realistic fluctuations rather than sensor errors.


### 2. Feature Engineering (Temporal Intelligence)
Decision trees lack inherent temporal sequence awareness. To solve this, I physically encoded the arrow of time into the feature matrix:
* **Cyclical Encodings:** Transformed `hour`, `month`, and `dayofweek` into **Sine and Cosine pairs**. This ensures the model mathematically understands that 11:00 PM ($t$) and Midnight ($t+1$) are adjacent.

* **Autoregressive Lags:** Engineered lags (1h, 2h, 3h, 24h, etc.) to capture grid inertia and weekly seasonality.
* **Rolling Statistics:** 12h and 24h moving averages and standard deviations provide a sense of recent trends and volatility.

### 3. Objective Feature Selection (The Noise Baseline)
To prevent "Feature Dilution" (where useless features confuse the model), I implemented a **Random Noise Baseline** test:
1.  Injected a column of pure random numbers into the training set.
2.  Ranked all 57 features using a Random Forest inspector.
3.  **The Cut:** Ruthlessly dropped any feature that scored lower than literal random noise.
4.  **Result:** Refined the model to **41 validated signals**, improving training speed and generalization.

### 4. Bayesian Hyperparameter Optimization
Used **Optuna** with a **Strict Chronological Split**:
* **Training Window:** 2015–2022
* **Validation Window:** 2023 (The final year before the test set)
* **Objective:** Minimize MAPE via Tree-structured Parzen Estimator (TPE) to find the most robust architecture.

---

## 📊 Key Insights: What Drives the Grid?
Analysis of the final model's feature importance revealed a clear hierarchy of drivers:

* **Inertia (97.2%):** The current hour's demand is the strongest predictor of the next, representing the high autocorrelation inherent in energy systems.
* **Social Cycles (1.5%):** The cyclical hour features were the secondary drivers, proving that human social schedules drive the grid more than individual weather events.
* **Weather Sensitivity:** Temperature and Solar rankings confirm they are the primary drivers of "non-routine" demand spikes (cooling/heating loads).


---

## 📁 Repository Structure
```bash
├── Analysis.ipynb         # End-to-end pipeline (EDA to Evaluation)
├── PGCB_demand_data.xlsx  # Historical grid demand
├── weather_data.xlsx      # Hourly meteorological features
├── economic_data.csv      # Macro-indicators (GDP, Population)
└── README.md              # Project documentation
```

## 🔧 Setup & Requirements
```bash
pip install pandas numpy matplotlib seaborn lightgbm optuna openpyxl scikit-learn
```

---
**Author:** Md Adnan Khalid  
