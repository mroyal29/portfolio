# EV Charge Time Prediction
Summer 2025

## Overview
This project predicts electric vehicle (EV) battery charging time remaining using machine learning on real-world fleet data. Unlike gas-powered vehicles that refuel quickly and predictably, EV charging is highly variable and time-consuming, creating challenges for travel planning and fleet management.

Traditional charging time formulas assume constant charging speed, known usable battery range, no battery degradation, and no temperature effects. The standard baseline formula is:

**Charge Time (hrs) = (Battery Capacity × (Target SOC - Current SOC)) / Power (kW)**

This formula achieved a 57-minute Mean Absolute Error (MAE) on our test set—an unacceptable margin given that the average charging session is only 100 minutes. We aimed to dramatically improve upon this using machine learning.

We evaluated eight different model architectures—linear regression, random forest, XGBoost, feedforward neural networks, and time-series models (LSTM, GRU)—on data from 460 chargers and 760 vehicles spanning 15 different EV models. Our best model, a GRU (Gated Recurrent Unit), achieved a Mean Absolute Error of 12.8 minutes, while maintaining a compact 63K parameter size suitable for production deployment.

## Project Background & My Role

This project was conducted as part of my work at **Camber**, a fleet charging management company, in partnership with UC Berkeley's Machine Learning course. I led a 4-person student team on this Camber-sponsored project, leveraging my role as Camber's data platform engineer and my domain expertise in EV fleet operations.

As project lead, I was responsible for the entire data infrastructure and pipeline. I had built Camber's data platform in Databricks (our data lake) and spent considerable time ensuring data quality of charger asset data and building out the vehicle dataset that provided make, model, year, and battery capacity information. This gave me direct access to production data and deep understanding of data quality issues inherent in real-world EV charging operations.

The goal was to support **Camber Core**, Camber's software platform that enables real-time fleet monitoring and management, with a new charge time prediction feature.

## Data Platform & Acquisition

Built and managed the complete data infrastructure:

* **Data Platform**: Architected and built Camber's Databricks-based data lake that ingested charging data from 500+ chargers across the US
* **Charger Asset Data**: Ensured data quality of charger specifications including power ratings and cable current limitations
* **Vehicle Dataset**: Built comprehensive vehicle dataset from scratch, collecting and validating make, model, year, battery capacity, and in-service dates for 760 vehicles across 15 different models (transit buses, school buses, vans, and cars)
* **OCPP Protocol Data**: Extracted metrics recorded every minute during charging sessions based on Open Charge Point Protocol (OCPP): power, energy, current, voltage, state of charge (SOC), power offered, and vehicle IDs
* **Pre-processing Pipeline**: Implemented downsampling at SOC transition points to reduce data volume from raw minute-by-minute readings while preserving critical charging behavior patterns


## Data Cleaning & Preprocessing

Developed comprehensive data quality procedures leveraging my deep understanding of charging infrastructure issues:

* **Filtered anomalies and equipment malfunctions**: Removed sessions longer than 10 hours, sessions ending below 85% SOC, sessions with sudden SOC drops or changes >5%, and physically impossible cases (increasing SOC with zero power input)
* **Validated battery capacity consistency**: Created methodology to verify capacity by plotting SOC change vs. energy charged, identifying vehicles with inaccurate capacity measurements and questionable sessions starting at SOC = 0%
* **Handled data collection bugs**: Removed rows with missing values from unknown vehicle models and sessions with invalid zipcodes that prevented temperature data integration
* **Engineered features**: Derived days in service (for battery degradation modeling), energy/time deltas between timesteps, and the target variable (time remaining until final SOC)
* **Scaled and encoded data**: Applied z-score normalization and one-hot encoding for categorical variables (vehicle make, model, type)

## Model Standardization & Training

Spearheaded efforts to keep all models comparable and prevent data leakage:

* **Standardized data processing** across regression and sequential model approaches to ensure apples-to-apples comparisons
* **Implemented temporal train/validation/test splits** (70/15/15) within each vehicle group to ensure models couldn't "see" future data and all vehicle types were properly represented
* **Final splits**: 791,560 training examples, 176,116 validation, 175,174 test examples

## Models Trained

I built and trained six different architectures:

1. **Linear Regression** - Keras TensorFlow baseline without hidden layers (40 min MAE)
2. **Random Forest** - sklearn RandomForestRegressor with overfitting detection (21.91 min MAE)
3. **XGBoost** - Gradient boosting model, best for regression-only approaches (15.76 min MAE)
4. **Feedforward Neural Network** - 4-layer architecture with dropout and regularization (16.13 min MAE)
5. **LSTM** - Recurrent neural network with 5-point sliding window sequences (13.29 min MAE, 2.1M parameters)
6. **GRU** - **Best overall model**: Recurrent neural network with 5-point sliding window sequences (12.80 min MAE, 63K parameters)

For sequential models, I created time-series sequences using 5-point sliding windows (last 4 SOC transitions + current point) with features including SOC, power, voltage, current, temperature, and time deltas between measurements. Zero-padding was used for cold-start predictions when fewer than 5 points were available.

## Results

The GRU model achieved dramatic improvement over the baseline formula:

* **12.80 minutes MAE** (78% improvement over 57-minute baseline)
* **26.03 minutes RMSE**
* **15.00% MAPE**
* **Only 63,000 parameters** (3% of LSTM's size, suitable for production)

XGBoost performed best for cold predictions (first point of session: 10.05 min MAE vs GRU's 15.50 min MAE), so this model could be deployed as well for the cases where there is no historical data showing how quickly the vehicle has been charging. 

## Analysis & Insights

* **Subgroup analysis** identified model weaknesses: low starting SOC (0-30%), underrepresented vehicle models (RAM Promaster: 739 datapoints vs TBB school bus: 318,669 datapoints), and extreme temperatures (>95°F with only 3,319 datapoints)
* **Feature importance analysis** validated that models learned underlying physics principles, with battery capacity, current SOC, target SOC, and power as top features across most models
* **Discovered real-world battery degradation**: Fleet data revealed ~2.5% capacity loss per year for high-volume vehicles, successfully captured by the "days in service" feature

## Future Work

* **Mixed physics-ML approach**: Incorporate the physics formula directly into the loss function or augment training data with physics-based charge curves across a continuous spread of power values and battery capacities to improve generalization to unseen vehicle/charger combinations
* **Address underrepresented subgroups**: Implement data augmentation, oversampling, or duplicate data for rare vehicle models to improve performance on edge cases
* **Train on full-resolution data**: Use non-downsampled minute-by-minute data with additional computing resources to capture finer-grained temporal patterns

## Technologies Used

* **Data Platform**: Databricks
* **Machine Learning**: scikit-learn (Random Forest), XGBoost
* **Deep Learning**: TensorFlow/Keras
* **Data Processing**: SQL, PySpark, pandas, numpy
* **Hyperparameter Tuning**: Keras Tuner, sklearn RandomizedSearchCV
* **External APIs**: Visual Crossing Weather API

## Key Takeaways

* **Sequential models outperform regression models** for time-series charging data in cases other then the cold start (GRU: 12.8 min vs XGBoost: 15.76 min MAE)
* **Simpler architectures can outperform complex ones**: GRU with 63K parameters beat LSTM with 2.1M parameters
* **Data quality infrastructure is foundational**: Building reliable charger asset and vehicle datasets enabled high-quality model training
* **Physics-informed features are critical**: Models that properly weighted battery capacity, SOC, and power (the physics formula components) generalized better
* **Temporal data splitting prevents leakage**: Critical for time-series problems to avoid models "seeing the future"
* **Underrepresented subgroups need special attention**: Rare vehicle types struggled, suggesting need for data augmentation strategies
