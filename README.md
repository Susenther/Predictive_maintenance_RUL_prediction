# Predictive Maintenance for Industrial Equipment Using Time Series Sensor Data

This project uses machine learning to predict the **Remaining Useful Life (RUL)** of industrial equipment from sensor data.

The project uses the **NASA C-MAPSS FD001 turbofan engine dataset** and builds a complete workflow from data preprocessing and feature engineering to model training and final evaluation.

## What this project is trying to solve

Equipment does not usually fail without showing changes in its sensor readings first. If those changes can be learned from historical data, we can estimate how many operating cycles an engine has left before failure.

In this project, the model learns those degradation patterns from previous engine runs and predicts RUL for engines it has not seen before.

## Dataset

The project uses the **NASA C-MAPSS FD001** subset.

Required files:

```text
CMAPSSData/
├── train_FD001.txt
├── test_FD001.txt
└── RUL_FD001.txt
```

The notebook also contains validation checks for the official test set and its corresponding RUL file.

> The dataset is not redistributed in this repository. Obtain the NASA C-MAPSS dataset from its official/public distribution and place the required files under `CMAPSSData/`.

## How the pipeline works

The notebook follows this flow:

```text
NASA C-MAPSS FD001
        │
        ▼
Data Ingestion
        │
        ▼
Data Cleaning & Inspection
        │
        ▼
RUL Calculation
        │
        ▼
RUL Capping at 125 Cycles
        │
        ▼
Temporal / Statistical Feature Engineering
        │
        ├── Rolling Mean Features
        ├── Rolling Standard Deviation
        └── Sensor Rate-of-Change
        │
        ▼
Engine-Level Development / Holdout Split
        │
        ▼
Expanding Engine-Level Validation
        │
        ├── Random Forest
        ├── Gradient Boosting
        └── XGBoost
        │
        ▼
Hyperparameter Tuning
        │
        ▼
Model Comparison & Selection
        │
        ▼
Final Random Forest
        │
        ▼
Official NASA FD001 Test Evaluation
        │
        ├── Primary capped-RUL metrics
        └── True last-cycle benchmark (100 engines)
```

## Feature Engineering

The raw sensor values contain useful information, but looking at one reading at a time doesn't tell the whole story. Engine degradation is a process, so the notebook creates features that capture how sensor behavior changes over time.

### Rolling Mean

Rolling averages help smooth short-term noise and make local trends easier for the models to learn.

### Rolling Standard Deviation

Rolling variability gives the model information about how stable or unstable a sensor has been recently.

### Sensor Rate of Change

The change between consecutive sensor readings helps capture whether a signal is moving up or down and how quickly it is changing.

Together, these features give the models more context than the raw sensor readings alone.

## Validation Strategy

One of the biggest risks with this dataset is accidentally letting information from the same engine appear in both training and validation data.

To avoid that, the training engines are split at the **engine level** into development and holdout groups. The development set is then used for expanding engine-level validation and model selection.

The official NASA FD001 test set stays completely separate from hyperparameter tuning. This gives us a cleaner estimate of how the final model performs on engines it has never seen before.

## Models Compared

I compared three tree-based regression approaches:

| Model | Purpose |
|---|---|
| Random Forest | Strong nonlinear baseline and final selected model |
| Gradient Boosting | Sequential boosting baseline |
| XGBoost | Gradient-boosted tree model with hyperparameter tuning |

Randomized hyperparameter search is used for model optimization.

Based on the validation results, **Random Forest** was selected as the final model and retrained using all available labeled training engines before the official test evaluation.

## Results

The notebook reports three main regression metrics:

- **MAE** — average absolute RUL prediction error in cycles
- **RMSE** — penalizes larger prediction errors more strongly
- **R²** — proportion of variance explained by the regression model

### Final Holdout Performance

On the internal holdout set, the final Random Forest achieved:

| Metric | Holdout |
|---|---:|
| MAE | **10.22 cycles** |
| RMSE | **14.52 cycles** |
| R² | **0.8789** |

The holdout contains **4,070 predictions**.

### Official NASA FD001 Evaluation

The model was then evaluated on the official NASA FD001 test set using the capped RUL target:

| Metric | Official FD001 Test |
|---|---:|
| MAE | **10.40 cycles** |
| RMSE | **15.28 cycles** |
| R² | **0.6930** |

The notebook also includes a **true last-cycle benchmark**. It produces one prediction for the final observed cycle of each of the 100 FD001 test engines and compares those predictions directly with `RUL_FD001.txt`.

This is kept separate from the supplementary per-row reconstructed-RUL analysis so the main benchmark remains easy to interpret.

## Model Selection Results

Model selection is performed using the development-engine validation folds rather than the official test set.

The final tuned fold-wise evaluation in the notebook reports:

| Model | Mean MAE | Mean RMSE | Mean R² |
|---|---:|---:|---:|
| Random Forest | 14.35 | 18.89 | 0.7836 |
| Gradient Boosting | 14.48 | 19.48 | 0.7712 |
| XGBoost | 14.44 | 19.85 | 0.7602 |

The Random Forest provides the strongest overall validation profile among the evaluated tuned models and is therefore used for final training.

## Interpretability

The notebook calculates feature importance for the final Random Forest model.

This provides a direct view of which engineered and sensor-derived variables contribute most strongly to the model's RUL predictions.

The notebook also includes:

- Holdout prediction diagnostics
- Official test prediction diagnostics
- RUL-range error analysis
- Feature-importance analysis

## Repository Structure

```text
Predictive_maintenance_RUL_prediction/
│
├── notebooks/
│   └── Predictive_Maintenance_RUL.ipynb
│
├── .gitignore
│
└── README.md
```

## Requirements

Recommended environment:

- Python 3.x
- Jupyter Notebook or Google Colab

Python libraries used by the notebook include:

```text
numpy
pandas
matplotlib
scikit-learn
xgboost
```

Install them with:

```bash
pip install numpy pandas matplotlib scikit-learn xgboost
```

## Running the Project

### Option 1 — Google Colab

1. Open the notebook in Google Colab.
2. Upload/provide the NASA C-MAPSS FD001 dataset.
3. Ensure the files are available under:

```text
CMAPSSData/
```

4. Run the notebook from top to bottom.

The notebook includes the complete pipeline from data ingestion through official test evaluation.

### Option 2 — Local Jupyter

Clone the repository:

```bash
git clone https://github.com/Susenther/Predictive_maintenance_RUL_prediction.git
cd Predictive_maintenance_RUL_prediction
```

Install dependencies:

```bash
pip install numpy pandas matplotlib scikit-learn xgboost
```

Place the FD001 dataset files under:

```text
CMAPSSData/
```

Launch Jupyter:

```bash
jupyter notebook
```

Open:

```text
notebooks/Predictive_Maintenance_RUL.ipynb
```

and run the notebook sequentially.

## Links

**Google Colab:**  
https://colab.research.google.com/drive/1uiVJC6g27gC8pG9Ndv_HqqMlN_4N3YDS?usp=sharing


