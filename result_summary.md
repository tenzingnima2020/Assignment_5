# Assignment 5 — Result Summary

---

## Problem Statement

- Predict whether a company will go **bankrupt** (`Bankrupt? = 1`) based on financial ratios.
- Binary classification with a rare positive class — accuracy alone is misleading.

---

## Dataset Summary

- Target column: `Bankrupt?`
- All 95 feature columns are numeric (no categorical features).
- Missing values handled by median imputation

---

## Class Imbalance Statement

- Positive class rate: **3.23%** (bankrupt companies).
- ~29.99 non-bankrupt companies for every 1 bankrupt company.
- Strongly imbalanced 
- Addressed in Experiment 3 via `scale_pos_weight = 29.99`; addressed indirectly in Experiment 4 via tuning.

---

## Discrimination Metric Explanation

- **Primary metric: PR-AUC** (Area Under the Precision-Recall Curve).
- PR-AUC is preferred over ROC-AUC when the positive class is rare 
- Secondary thresholded metrics (precision, recall, **F2-score**) are also tracked.

---

## Calibration Metric Explanation

- **Primary calibration metric: Brier score** 
- Lower is better; a perfect model scores 0
- A **calibration curve** (reliability diagram) is also plotted for the winning model 
- Brier score answers a different question than PR-AUC: not just whether the model *ranks* cases correctly

---

## Feature Selection Explanation

- **Feature Set A:** All 95 usable numeric features after basic cleaning (no selection).
- **Feature Set B:** Top 25 features ranked by a **training-only XGBoost feature importance** model.
  - Importance extracted from `feature_importances_` on the training split only — no leakage from validation or test.


---

## 5-Row Experiment Table

| # | Model | Features | Main Settings | Train PR-AUC | Val PR-AUC | Overfit Gap | Val ROC-AUC | Val Brier | Threshold | Val Precision | Val Recall | Val F2 | Winner? |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| 1 | Logistic Regression | A | median impute + scale + logistic regression | 0.5308 | 0.2587 | 0.2721 | 0.8766 | 0.0316 | 0.50 | 0.3333 | 0.1818 | 0.2000 | No |
| 2 | XGBoost Baseline | A | n_estimators=250, lr=0.05, depth=3 | 0.9933 | 0.5808 | 0.4125 | 0.9627 | 0.0198 | 0.50 | 0.6923 | 0.2727 | 0.3103 | No |
| 3 | XGBoost Imbalance | A | scale_pos_weight=29.99, depth=3, lr=0.05 | 0.9947 | 0.4524 | 0.5423 | 0.9574 | 0.0334 | 0.50 | 0.4000 | 0.7273 | 0.6250 | No |
| 4 | XGBoost Tuned | A | n_estimators=300, lr=0.08, depth=2 (best of 5-candidate search) | 1.0000 | **0.6012** | 0.3988 | 0.9650 | **0.0201** | 0.50 | 0.7692 | 0.3030 | 0.3448 | **Yes** |
| 5 | XGBoost Selected Features | B | Top 25 features + tuned XGBoost | 0.9990 | 0.4916 | 0.5074 | 0.9510 | 0.0216 | 0.50 | 0.6667 | 0.3030 | 0.3401 | No |

---

## Winning Model and Why It Won

- **Winner: Experiment 4 — XGBoost Tuned (Feature Set A)**
- Selection rule: highest validation PR-AUC, then lowest Brier score, then smallest overfit gap.
- Val PR-AUC of **0.6012** was the highest, Val Brier score of **0.0201** was the second-lowest.
- Best hyperparameters: `n_estimators=300`, `learning_rate=0.08`, `max_depth=2`.

---

## Final Test Results

> Test set evaluated **once**, after winner and threshold were both fixed on the validation set.

| Metric | Value |
|---|---|
| Test PR-AUC | 0.4977 |
| Test ROC-AUC | 0.9493 |
| Test Brier Score | 0.0232 |
| Test Precision | 0.3878 |
| Test Recall | 0.5758 |
| Test F2-Score | 0.5249 |
| Decision Threshold | 0.05 |

---

## Confusion Matrix

> At threshold = 0.05 on the test set.

|  | Predicted Negative | Predicted Positive |
|---|---|---|
| **Actual Negative** | TN = 960 | FP = 30 |
| **Actual Positive** | FN = 14 | TP = 19 |

- Out of 33 actual bankruptcies, the model caught **19** (recall ≈ 57.6%).
- 30 false alarms out of 990 true negatives (false positive rate ≈ 3.0%).
- Low threshold (0.05) was chosen deliberately to prioritise recall over precision.

---

## Calibration Result

- Test Brier score: **0.0232** (low; close to the validation score of 0.0201).
- The calibration curve is **fairly close to the diagonal overall**, indicating the predicted probabilities are broadly reliable.

---

## Feature Importance Result

- Most important feature: **Persistent EPS in the Last Four Seasons**.
- Feature importance derived from XGBoost's `feature_importances_` (gain-based) on the final model trained on Feature Set A.
- **Limitation:** XGBoost feature importance measures split gain, not causal effect; correlated features can split importance arbitrarily between them.

---

## Overfitting Discussion

- Final model (Exp 4) train PR-AUC: **1.0000**
- Final model validation PR-AUC: **0.6012**
- Overfit gap: **0.3988** which is **large**
- Mitigations that could reduce the gap: stronger regularisation (`reg_alpha`, `reg_lambda`), lower `n_estimators` with early stopping on a held-out set, or cross-validation-based tuning.

---

## AI Usage

- AI tool used: **Codex in VS Code**.
- It helped structure the notebook, build a reusable `evaluate_model` function, and organise the five experiments into a consistent format.

