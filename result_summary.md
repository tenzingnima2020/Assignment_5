# Assignment 5 — Result Summary

---

## Problem Statement

- Predict whether a company will go **bankrupt** (`Bankrupt? = 1`) based on financial ratios.
- A risk team can use elevated-risk flags to act before losses occur.
- Binary classification with a rare positive class — accuracy alone is misleading.
- Primary discrimination metric chosen: **PR-AUC**; calibration assessed via **Brier score**.

---

## Dataset Summary

- Source file: `data.csv`
- Shape: **6,819 rows × 96 columns**
- Target column: `Bankrupt?`
- All 95 feature columns are numeric (no categorical features).
- Missing values handled by median imputation inside a sklearn `Pipeline`.

---

## Class Imbalance Statement

- Positive class rate: **3.23%** (bankrupt companies).
- ~29.99 non-bankrupt companies for every 1 bankrupt company.
- Strongly imbalanced — accuracy is not a reliable metric; a model that always predicts 0 gets >96% accuracy but catches nothing.
- Addressed in Experiment 3 via `scale_pos_weight = 29.99`; addressed indirectly in Experiment 4 via tuning.

---

## Discrimination Metric Explanation

- **Primary metric: PR-AUC** (Area Under the Precision-Recall Curve).
- PR-AUC is preferred over ROC-AUC when the positive class is rare because it focuses on how well the model ranks and retrieves the minority class.
- A model that always predicts the majority class collapses PR-AUC to near the base rate (≈0.03), making poor models easy to identify.
- Secondary thresholded metrics (precision, recall, **F2-score**) are also tracked; F2 weights recall twice as heavily as precision because missing a bankrupt company (false negative) is more costly than a false alarm.

---

## Calibration Metric Explanation

- **Primary calibration metric: Brier score** — measures the mean squared error between predicted probabilities and true binary outcomes.
- Lower is better; a perfect model scores 0, a naïve model that always predicts the base rate scores ≈ 0.031.
- A **calibration curve** (reliability diagram) is also plotted for the winning model to check whether predicted probabilities match observed bankruptcy rates in each probability bucket.
- Brier score answers a different question than PR-AUC: not just whether the model *ranks* cases correctly, but whether the *probability values* are numerically credible.

---

## Feature Selection Explanation

- **Feature Set A:** All 95 usable numeric features after basic cleaning (no selection).
- **Feature Set B:** Top 25 features ranked by a **training-only XGBoost feature importance** model.
  - Importance extracted from `feature_importances_` on the training split only — no leakage from validation or test.
  - Reduces dimensionality, improves explainability, and tests whether a smaller set stays competitive.
- Constant or fully-missing columns were excluded before either set was formed.

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
- Val PR-AUC of **0.6012** was the highest across all five experiments.
- Val Brier score of **0.0201** was the second-lowest (very close to Exp 2's 0.0198) and well-calibrated.
- Tuning approach: 5-candidate manual search on train/val splits only — no test set involvement.
- Best hyperparameters: `n_estimators=300`, `learning_rate=0.08`, `max_depth=2`.
- Shallower trees (`depth=2`) reduced overfitting compared to Experiments 2 and 3, and the tuned learning rate improved ranking quality.

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
- Low threshold (0.05) was chosen deliberately to prioritise recall over precision — catching more bankruptcies matters more than avoiding false alarms in this context.

---

## Calibration Result

- Test Brier score: **0.0232** (low; close to the validation score of 0.0201).
- The calibration curve is **fairly close to the diagonal overall**, indicating the predicted probabilities are broadly reliable.
- At threshold 0.05, the model already flags a meaningful share of the positive class, consistent with the low base rate.
- No post-hoc calibration (e.g., Platt scaling) was applied; the XGBoost probabilities were used directly.

---

## Feature Importance Result

- Most important feature: **Persistent EPS in the Last Four Seasons**.
- Feature importance derived from XGBoost's `feature_importances_` (gain-based) on the final model trained on Feature Set A.
- Top features broadly relate to profitability ratios, earnings stability, and financial leverage — consistent with established bankruptcy-prediction theory (Altman Z-score factors).
- **Limitation:** XGBoost feature importance measures split gain, not causal effect; correlated features can split importance arbitrarily between them.

---

## Overfitting Discussion

- Final model (Exp 4) train PR-AUC: **1.0000**
- Final model validation PR-AUC: **0.6012**
- Overfit gap: **0.3988**
- The overfit gap is **large**, flagging potential overfitting despite the shallow tree depth (`max_depth=2`).
- Likely causes: XGBoost with 300 estimators memorises the small positive class (only ~220 bankrupt cases in training); the imbalanced data amplifies this.
- Mitigations that could reduce the gap: stronger regularisation (`reg_alpha`, `reg_lambda`), lower `n_estimators` with early stopping on a held-out set, or cross-validation-based tuning.
- Despite the gap, test performance (PR-AUC 0.4977, F2 0.5249) remains substantially above the logistic regression baseline, suggesting the model has learned real signal.

---

## AI Usage

- AI tool used: **Codex in VS Code**.
- It helped structure the notebook, build a reusable `evaluate_model` function, and organise the five experiments into a consistent format.
- Verified independently: the test set was kept untouched until the final evaluation section (Section 14).
- Fixed independently: *(replace this line with a real example from your own work — placeholder left in original notebook)*.
