# Predictive Maintenance --- Remaining Useful Life (RUL) Prediction

A supervised Machine Learning regression project that estimates the
**Remaining Useful Life (RUL)** of turbofan engines from operating
conditions and sensor measurements. The notebook uses an engine-level
validation strategy to reduce information leakage, compares regression
models, and evaluates predictions in the official NASA C-MAPSS FD001
test scenario.

## Project Objectives

-   Estimate how many operating cycles remain before an engine reaches
    failure.
-   Build RUL targets from run-to-failure training trajectories.
-   Remove constant or uninformative sensor channels.
-   Avoid row-level leakage by splitting data by engine.
-   Compare regression models using validation RMSE.
-   Evaluate the selected model on the official test scenario.
-   Analyze prediction errors and feature importance.

## Dataset

-   **Dataset:** NASA C-MAPSS Turbofan Engine Degradation Simulation
    Data
-   **Subset:** FD001
-   **Source:** [NASA Turbofan Engine Degradation Simulation Data ---
    Kaggle](https://www.kaggle.com/datasets/behrad3d/nasa-cmaps)
-   **Target:** Remaining Useful Life (`RUL`), measured in cycles

FD001 contains run-to-failure trajectories for training engines and
truncated trajectories for test engines. The provided RUL file supplies
the true RUL at the last observed cycle of each test engine.

## Workflow

1.  Download and load the FD001 data using `kagglehub`.
2.  Inspect the training and test trajectories.
3.  Build training RUL labels from each engine's final observed cycle.
4.  Explore sensor behavior across engine life.
5.  Remove zero-variance sensor features and retain useful
    operational/time features.
6.  Split training engines into separate training and validation groups.
7.  Build a pseudo-test validation scenario by using a cutoff before the
    end of each validation engine's trajectory.
8.  Compare:
    -   Random Forest Regressor
    -   HistGradientBoosting Regressor
9.  Select the model using validation RMSE.
10. Retrain the selected model on all available training engines.
11. Predict RUL at the final observed cycle for each official test
    engine.
12. Evaluate official test performance and inspect errors and feature
    importance.

## Validation Strategy

Rows from the same engine are related over time, so randomly splitting
individual rows can leak engine-specific information between training
and validation. This notebook splits by engine and creates a pseudo-test
cutoff for validation engines to better approximate the real scenario in
which an engine's future trajectory is not available.

The validation score is used for model selection. The official FD001
test set is evaluated separately after model selection.

## Models and Metrics

**Models compared** - Random Forest Regressor - HistGradientBoosting
Regressor

**Primary metric:** RMSE\
**Additional metrics:** MAE and R²

RMSE is the primary selection metric because it penalizes larger errors
more strongly and is used in the referenced predictive-maintenance
competition setup.

## Results

**Selected model:** Random Forest Regressor

### Validation Performance

  Metric     Score
  -------- -------
  MAE         6.25
  RMSE        8.73
  R²         0.000

### Official FD001 Test Performance

  Metric     Score
  -------- -------
  MAE        19.83
  RMSE       26.41
  R²         0.596

The official test results are materially different from the validation
results, which highlights why the official test scenario should be
reported separately. These values are specific to this notebook run and
setup.

## Visual Results

### 1. Official Test --- Actual vs. Predicted RUL

The scatter plot compares true RUL values with model predictions for the
official test engines. The dashed diagonal represents perfect
predictions; points farther from the line indicate larger errors.

![Official test actual versus predicted
RUL](assets/official_test_actual_vs_predicted_rul.png)

### 2. Final Model Summary

The summary reports the selected model, validation RMSE, official test
MAE/RMSE/R², and the top feature by model importance.

![Predictive maintenance final model
summary](assets/final_model_summary.png)

### 3. Official Test Error Distribution

The histogram shows the distribution of `Actual RUL - Predicted RUL`.
Positive errors indicate underprediction of RUL, while negative errors
indicate overprediction.

![Official test RUL error
distribution](assets/official_test_rul_error_distribution.png)

## Feature Importance

In the saved notebook output, **`time_cycles`** is the top feature by
Random Forest feature importance. Feature importance reflects the fitted
model's reliance on variables; it does not establish that a feature
causes engine degradation.

## Requirements

Main libraries: - Python - pandas - NumPy - Matplotlib - Seaborn -
scikit-learn - kagglehub

Install the main dependencies with:

``` bash
pip install pandas numpy matplotlib seaborn scikit-learn kagglehub
```

## How to Run

1.  Open `predictive-maintenance.ipynb` in Jupyter Notebook, JupyterLab,
    or Google Colab.
2.  Install the required packages.
3.  Configure Kaggle access if required by `kagglehub`.
4.  Run the notebook cells from top to bottom. The data loader searches
    for the FD001 files after downloading the dataset.

## Repository Structure

``` text
.
├── predictive-maintenance.ipynb
├── README.md
└── assets/
    ├── official_test_actual_vs_predicted_rul.png
    ├── final_model_summary.png
    └── official_test_rul_error_distribution.png
```

Keep the `assets/` directory alongside `README.md` so the images render
correctly on GitHub.

## Limitations

-   This portfolio project focuses on FD001, which represents one
    operating condition and one main fault mode.
-   RUL estimates have uncertainty and should not be treated as exact
    failure dates.
-   The difference between validation and official test performance
    indicates that generalization is challenging.
-   Real-world deployment would require additional validation,
    monitoring, and integration with maintenance workflows.
