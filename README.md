# HPC Job Resource Prediction

Author: hariThorve  
Repository: hariThorve/hpcProject

## Project Overview

This repository contains an exploratory analysis and a machine learning pipeline for predicting resource usage of jobs submitted to an HPC cluster. The aim is to predict job resource requests and/or actual usage (e.g., requested time, used memory) to improve scheduling and cluster performance. The work is based on the Standard Workload Format (SWF) dataset and reimplements ideas from the research paper:
"Improving HPC System Performance by Predicting Job Resources via Supervised Machine Learning".

Two Jupyter notebooks are included:

- Data exploration + preprocessing: hpcProject.ipynb  
  Notebook: https://github.com/hariThorve/hpcProject/blob/84812057ecf4a9f5d151afd7a9e1884f37a9e492/hpcProject.ipynb

- Machine learning model (XGBoost + preprocessing + training/evaluation): hpc_ml_model.ipynb  
  Notebook: https://github.com/hariThorve/hpcProject/blob/84812057ecf4a9f5d151afd7a9e1884f37a9e492/hpc_ml_model.ipynb

## Goals

- Predict resource utilization for HPC jobs (primary targets in current notebooks):
  - UsedMemory (memory actually used by jobs)
  - RequestedTime (requested walltime) — dataset contains this column and can be targeted similarly
- Provide preprocessing and feature engineering for SWF-like logs
- Train and evaluate supervised ML models (XGBoost used in the provided notebook) to predict the above targets

## Dataset

The notebooks use SWF-like logs (Standard Workload Format). Example file referenced in the notebooks:
- `HPC2N-2002-2.2-cln.csv` (a cleaned SWF dataset). The initial exploratory notebook (`hpcProject.ipynb`) reads this file.

Note: The ML notebook (`hpc_ml_model.ipynb`) expects a preprocessed CSV `filtered_hpc_dataset.csv`. See "Preprocessing" below for how to generate it.

Columns commonly used in this repository:
- JobID, SubmitTime, RequestedTime, RunTime, AllocatedProcessors,
  RequestedProcessors, UsedMemory, RequestedMemory

Missing values in SWF logs are encoded as `-1` (per SWF convention); the preprocessing notebook replaces `-1` with NaN and drops or filters rows as appropriate.

## Preprocessing (high level)

The `hpcProject.ipynb` notebook performs:
- Loading the SWF CSV
- Column selection to focus on features relevant for resource prediction: JobID, SubmitTime, RequestedTime, RunTime, AllocatedProcessors, RequestedProcessors, UsedMemory, RequestedMemory
- Replace `-1` values with `NaN`
- Drop rows with missing important fields (e.g., UsedMemory / RequestedMemory) for the ML experiments
- Create derived features such as `requestedMemory_per_cpu = RequestedMemory / RequestedProcessors`
- Export a filtered CSV (named `filtered_hpc_dataset.csv`) that is consumed by the ML notebook

Run `hpcProject.ipynb` top-to-bottom to produce the filtered dataset used in the ML notebook.

## Machine learning pipeline (high level)

The `hpc_ml_model.ipynb` notebook includes:
- Loading `filtered_hpc_dataset.csv`
- Basic exploratory analysis and correlation plotting (matplotlib / seaborn)
- Preparing data for predicting `UsedMemory`:
  - Drop `RunTime` when predicting UsedMemory in the current notebook
  - Split features/target, train/test split
  - Standard scaling of features
- Train an XGBoost regressor (XGBoost package installed in the notebook)
- Evaluate model performance on test split (check the notebook for metric outputs and plots)

The notebook is organized so it can be modified to predict other targets (e.g., RequestedTime) by changing the `y` variable and adjusting features.

## Requirements

Recommended Python packages (used by the notebooks):

- Python 3.8+ (notebooks use features compatible with modern Python 3)
- pandas
- numpy
- scikit-learn
- xgboost
- matplotlib
- seaborn
- jupyter / jupyterlab (or Google Colab)

Install with pip:
pip install pandas numpy scikit-learn xgboost matplotlib seaborn jupyter

Note: The ML notebook includes a `!pip install xgboost` cell for convenience when running in Colab.

## How to run

Option A — Local (Jupyter / JupyterLab)
1. Clone the repository:
   git clone https://github.com/hariThorve/hpcProject.git
2. Install requirements (see above).
3. Place your SWF CSV (e.g., `HPC2N-2002-2.2-cln.csv`) in the repository root (or update paths in the notebooks).
4. Open the notebooks:
   jupyter lab
   - Run `hpcProject.ipynb` first to preprocess and produce `filtered_hpc_dataset.csv`.
   - Then run `hpc_ml_model.ipynb` to train models and evaluate.

Option B — Google Colab
- Open `hpc_ml_model.ipynb` in Colab (or upload both notebooks and dataset).
- The notebook already contains a `!pip install xgboost` step.

## File structure (not exhaustive)

- hpcProject.ipynb           — Data exploration + preprocessing (generates filtered dataset)
- hpc_ml_model.ipynb         — ML experiments and model training (XGBoost)
- (dataset files are expected, not included): `HPC2N-2002-2.2-cln.csv`, `filtered_hpc_dataset.csv`

## Reproducibility notes

- The preprocessing notebook drops rows with missing values for certain columns to create a clean dataset for ML. Depending on your goals you may prefer imputation instead of dropping rows.
- SWF datasets often represent times (e.g., SubmitTime, RunTime, RequestedTime) in seconds since some epoch. Convert or normalize time features if you want calendar-aware features (hour of day, weekday, etc.).
- Models trained on one cluster's workload may not generalize to other clusters without domain adaptation.

## Suggestions & Future Work

- Add experiments to predict `RequestedTime` in addition to `UsedMemory`.
- Try classification-style models to predict coarse-grained resource classes (e.g., low/medium/high memory) to help conservative scheduling.
- Improve feature engineering:
  - Extract temporal features from SubmitTime (hour/day/week)
  - User-level historical features (average usage per user)
  - Queue/partition-level features if available
- Implement cross-validation and hyperparameter tuning (GridSearchCV / Optuna).
- Provide model persistence (joblib) and a small inference script to plug predictions into a scheduler simulator.


