# End-to-End Flight Weather Delay Prediction Pipeline

**Author:** Ali Yıldızhan *(Add your teammates here)*
**Date:** June 2026

## 📌 Project Overview
This project investigates the predictive relationship between localized atmospheric weather events and commercial airline delays. Using raw flight records and high-frequency ASOS weather sensor data, this pipeline processes hundreds of thousands of rows to engineer custom aviation features (like Airport Congestion and Wind Severity) to predict FAA-standard delays (>15 minutes).

### The Core Hypothesis: "Weather is a Trigger, Not a Timer"
Through rigorous modeling, this project scientifically proves that while weather reliably initiates network disruptions (solvable via Classification), the exact duration of a delay is dictated by unrecorded airline operational logistics (rendering pure weather-based Regression models ineffective). 

## 📊 Key Results
* **XGBoost Classifier (Stage 1):** Achieved **73.64% accuracy** in predicting whether a flight will be delayed by more than 15 minutes, utilizing a custom 2-hour rolling airport congestion feature.
* **XGBoost Regressor (Stage 2 - Hub Isolated):** Proved the mathematical limits of public aviation data. Even when isolating severely delayed flights at a major hub (ORD) and tracking physical aircraft previous-leg delays, the regression $R^2$ remained negative. 
* **Vanilla Baseline Sanity Check:** Verified that advanced feature engineering did not break the model; an un-engineered baseline produced a 6.5% $R^2$ merely by guessing "0 minutes" for clear-weather days, validating the pivot to classification.

## 🗄️ Data Sources
1. **Flight Data:** Bureau of Transportation Statistics (BTS) On-Time Reporting records.
2. **Weather Data:** ASOS (Automated Surface Observing System) 5-minute interval sensor data, fetched dynamically via the Iowa Environmental Mesonet (IEM) API.

## ⚙️ Pipeline Architecture
This repository contains a unified Jupyter Notebook that executes the entire data science lifecycle:

* **Phase 1: Preprocessing & Balancing:** Cleans raw BTS CSVs, reconstructs naive local times, removes cancellations, and balances the target classes to prevent predictive bias.
* **Phase 2: Multithreaded API Fetching:** Maps IATA to ICAO codes and dynamically downloads high-resolution ASOS weather data for all valid airports using concurrent threading.
* **Phase 3: 4-Way Time Merge & Imputation:** Maps exact airport coordinates to timezones, converts flight times to UTC, and uses a memory-efficient `merge_asof` technique to lock weather readings to Scheduled/Actual Departure and Arrival times. Forward-fills and logically imputes missing sensor data.
* **Phase 3.8: Feature Engineering:** Calculates Visibility Drops, Wind Severity multipliers, and prunes redundant atmospheric noise.
* **Phase 4: Modeling & Analytics:** Engineers live `Airport_Congestion_2hr` and `Previous_Leg_Delay` features, followed by comprehensive XGBoost Classification and multialgorithm Regression bake-offs.

## 🚀 How to Run
1. Ensure the raw flight data CSVs are located in a folder named `raw-data-reduced-min-2024` in the root directory.
2. Install required packages: `pip install pandas numpy xgboost scikit-learn matplotlib seaborn timezonefinder requests`
3. Run the Jupyter Notebook sequentially from top to bottom. 
    * *Note: Intermediate checkpoints (e.g., `05_modeling_ready_data.csv`) are automatically saved to your hard drive to prevent data loss and allow skipping lengthy API fetch phases on subsequent runs.*