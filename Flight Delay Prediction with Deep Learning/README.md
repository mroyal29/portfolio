# Flight Delay Prediction at Scale

## Overview
This project predicts U.S. flight departure delays using five years of historical flight and weather data (~30 million records) at 2 hours prior to depture time. Rather than framing the problem as binary classification (delayed vs. not delayed), we modeled delay as a **regression task**, enabling more informative and actionable predictions.

A core challenge of this project was **scale**: the dataset was too large to fit in memory, requiring distributed data processing and careful model training design. The work was completed as part of a *Machine Learning at Scale* course and executed on a Databricks cluster with 6-12 workers.

---

## My Contributions
I focused on **data preparation, feature engineering, and large-scale neural network training**, specifically:

### Spark-Based Data Preparation (Databricks)
- Implemented Spark-based joins between large flight and weather datasets.
- Handled missing weather values using null imputation strategies, including look at other nearby weather stations, forward fill, and recent averages. 
- Engineered **leakage-safe temporal features**, including lagged values and rolling window statistics, for flight delays grouped by different metrics like origin and depature airport, tail number and flight number.
- All data preparation and feature engineering was performed in **Spark within Databricks** to handle dataset scale efficiently.

### Neural Network Modeling with Learned Embeddings
- Trained a **PyTorch MLP** model with **learned embeddings for high-cardinality categorical features** (e.g., airports, carriers, tail numbers).
- We used a cross validation strategy for tuning with a sliding window of 2 years for train and 0.5 years for validation. 
- We applied a log transformation on our highly skewed target variable and experimented with a weighted loss functions and oversampling/undersampling to handle data imbalance. 
- Compared performance against a linear regression baseline, achieving a **23% improvement in MAE**.
- Logged experiment parameters and metrics using **MLflow** for experiment tracking and model comparison.

### Training at Scale: Ray Tune and PyArrow
- Parallelized hyperparameter tuning using **Ray Tune**, running multiple trials concurrently across the cluster (up to 80 trials at once).
- Developed custom PyTorch IterableDataset using **PyArrow to stream mini-batches from Parquet files** enabling out-of-core training on datasets 3x larger than available memory with parallel I/O workers. 

---

## Notebooks Included
This repository contains **only the notebooks I directly worked on**, including:
- Spark-based data preparation and feature engineering
- PyTorch MLP training and hyperparameter tuning

---

## Key Takeaways
- Learned embeddings can significantly improve performance on high-cardinality categorical features.
- Streaming data from columnar storage (Parquet) enables neural network training on datasets that exceed memory limits.
- Distributed hyperparameter tuning is essential for iterating efficiently at scale and Ray Tune is the best product in the market for this. It has great observability and work well with autoscaling clusters. 
- Distributing training (rather than tuning) with the torch distributor does not work well with auto scaling clusters.
- XGBoost works much better at scale and produces better results for this type of regression task than a deep neural network. This is something we anticipated coming in, but we were required to build an MLP. 
- A zero inflated target variable can benefit from a 2-stage process. One for classifying the zero and the next for the regression task. (I attempted an 2-headed/2-output MLP with one classification head and one regression head as well, but the results were really unstable and we ran out of time.)
  
---

## Technologies Used
- **Databricks / Apache Spark (PySpark)**
- **PyTorch**
- **Ray Tune**
- **PyArrow** (for mini-batch streaming)
- **MLflow**
- **Spark MLlib** (features scaling, encoding, vecotrization, and baseline linear model)
- **DBFS mount in AWS S3** (parquet files)
- **Python**

## Project Contributors
- Maagie Royal: mroyal7@berkeley.edu
- Lyn Wang: fulingw@berkeley.edu
- Michael White: mwhite@berkeley.edu
- Emily Zhao: emily_zhao@berkeley.edu
- Aidan Connerly: aidenconnerly@berkeley.edu
