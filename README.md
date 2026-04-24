# Assignment 5 — Comparative Binary Classification with XGBoost

**Task:** Bankruptcy Prediction  
**Author:** Tenzing Nima Tamang  
**Random Seed:** 42

---

## Prerequisites

- Python 3.9+
- Jupyter Notebook or JupyterLab

---

## Installation

```bash
pip install numpy pandas scikit-learn xgboost matplotlib seaborn ipython
```

Or install from a `requirements.txt` if provided:

```bash
pip install -r requirements.txt
```

---

## Dataset Setup

Place the dataset file at one of the following paths (the notebook searches them in order):

```
C:\Users\A S U S\aidi-1204-project\ml-workshop\ml-workshop\training\data.csv
ml-workshop/ml-workshop/training/data.csv
ml-workshop/training/data.csv
training/data.csv
data.csv
```

The simplest option is to place `data.csv` in the same directory as the notebook.

---

## Running the Notebook

```bash
jupyter notebook assignment_5.ipynb
```

Then run all cells in order via **Kernel → Restart & Run All**.

---

## Notebook Structure

| Section | Description |
|---|---|
| 1 | Problem Definition |
| 2 | Quick EDA |
| 3–4 | Class Imbalance Check & Train/Val/Test Split |
| 5 | Feature Sets (A = all 95 features, B = top 25) |
| 6 | Helper functions (evaluate_model, find_best_threshold) |
| 7–11 | Experiments 1–5 |
| 12 | 5-Row Experiment Table & Winner Selection |
| 13 | Threshold Selection (validation set only) |
| 14 | Final Test Evaluation |
| 15–16 | Calibration Check & Feature Importance |
| 17–19 | Metric/Feature Set Discussion & Overfitting Check |
| 20 | AI Usage |

---

## Key Constants (top of notebook)

| Constant | Value | Purpose |
|---|---|---|
| `RANDOM_STATE` | 42 | Reproducibility seed |
| `DEFAULT_THRESHOLD` | 0.50 | Default decision threshold during experiments |
| `FINAL_BETA` | 2 | Beta for F-score (weights recall more than precision) |
| `TOP_N_FEATURES` | 25 | Number of features in Feature Set B |

---

## Important Notes

- **The test set is never touched until Section 14.** All model selection and threshold tuning uses the validation set only.
- Feature Set B is derived from a training-only XGBoost importance model — no leakage from validation or test.
- All five experiments are evaluated with a fixed default threshold of 0.50; the final threshold is chosen separately on the validation set after winner selection.
