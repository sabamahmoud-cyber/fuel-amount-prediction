# Fuel Consumption Modeling

## Overview

This notebook builds regression models to predict transit fuel consumption using **Gasoline Gallon Equivalent (GGE)** as the main target.

The cleaned dataset contains **2,942 rows**, and no additional cleaning or row removal is performed during the modeling stage.

The workflow includes:

- Feature selection
- Agency-based train/test split
- Preprocessing
- Random Forest modeling
- Log-target experiment
- Hyperparameter tuning
- Decision Tree
- Linear Regression
- Final model comparison

---

## Features and Target

### Numerical Features
- `primary_uza_population`
- `agency_voms`
- `mode_voms`

### Categorical Features
- `state`
- `organization_type`
- `mode_name`
- `typeofservicecd`
- `Fuel Type`

### Target
The main target is:

`GGE`

GGE is used because it puts different fuel and energy types on one common scale.

The notebook also initially compares GGE with `Amount_Original`.

---

## Data Split and Preprocessing

The data is split using `GroupShuffleSplit` based on `agency`.

This prevents the same agency from appearing in both training and testing sets.

| Dataset | Rows | Agencies |
|---|---:|---:|
| Training | 2,348 | 416 |
| Testing | 594 | 104 |

Agency overlap between train and test:

`0`

For preprocessing:

- Numerical features use median imputation.
- Categorical features use most-frequent imputation and One-Hot Encoding.
- Linear Regression additionally uses `StandardScaler`.
- All preprocessing is handled inside Scikit-Learn pipelines.

---

## Models Tested

The notebook evaluates:

### Random Forest

Experiments include:

- Random Forest with original fuel units
- Random Forest with GGE
- Random Forest with log-transformed GGE
- Tuned Random Forest

The log transformation improved MAE and Median Absolute Error, but reduced R² and increased RMSE.

Random Forest hyperparameter tuning was performed using:

- `RandomizedSearchCV`
- `GroupKFold`
- RMSE as the optimization metric

### Decision Tree

Both baseline and tuned Decision Tree models were tested.

The tuned version performed worse on the test set, so the baseline Decision Tree was used in the final comparison.

### Linear Regression

Linear Regression was used as a simpler baseline model and produced the lowest overall predictive performance.

---

## Final Model Results

| Model | R² | MAE | RMSE | Median AE |
|---|---:|---:|---:|---:|
| **Tuned Random Forest** | **0.5643** | **378,274** | **1,859,433** | 15,503 |
| Decision Tree | 0.5025 | 406,718 | 1,986,847 | **10,883** |
| Linear Regression | 0.4068 | 711,627 | 2,169,554 | 268,918 |

<img width="872" height="490" alt="image" src="https://github.com/user-attachments/assets/d0f13295-5a1a-4c51-9554-1401cd4aebd0" />


---

## Final Model

The **Tuned Random Forest Regressor** was selected as the final model.

Final test performance:

- **R²:** 0.564
- **MAE:** 378,274 GGE
- **RMSE:** 1.86 million GGE
- **Median Absolute Error:** 15,503 GGE

It achieved the highest R² and the lowest MAE and RMSE among the final models.
