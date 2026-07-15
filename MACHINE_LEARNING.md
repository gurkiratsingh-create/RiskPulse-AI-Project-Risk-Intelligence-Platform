# Machine Learning Model Specification — RiskPulse

This document details the Machine Learning pipeline, model configuration, dataset characteristics, feature engineering equations, and explainability algorithms implemented in the RiskPulse prediction system.

---

## 📊 The Dataset (`project_data.csv`)

The model is trained on a synthetically generated but highly realistic dataset containing **1,000 historical project entries** with 12 features and a binary target label.

### Target Label
*   `delayed` *(Binary)*:
    *   `1`: Indicates the project experienced significant delay.
    *   `0`: Indicates the project was completed on track.

### Dataset Features
The source dataset contains 12 metrics representing project progress, budgeting, and team composition, which correspond to the input features described below.

---

## ⚙️ Feature Engineering Pipeline

When a user submits project variables via the client interface (`progress`, `deadline`, `budget_percent`, `team_size`), the backend processes them through a custom feature engineering pipeline. This scales the raw inputs and computes secondary metrics to match the model's 12-dimensional training input space.

Below are the mathematical formulas and default configurations used for feature engineering:

| Feature Name | Origin / Equation | Description |
| :--- | :--- | :--- |
| `total_tasks` | `DEFAULT_CONFIG["TOTAL_TASKS"]` (Default: `100`) | Baseline total scope of tasks. |
| `completed_tasks`| `progress` | Derived directly from progress (0–100%). |
| `avg_task_delay` | $\frac{\text{deadline}}{10}$ | Proxy metric scaling days remaining to average delay pressure. |
| `budget_allocated`| `DEFAULT_CONFIG["BUDGET_ALLOCATED"]` (Default: `200000`) | Baseline capital allocation. |
| `budget_spent` | $\frac{\text{budget\_percent}}{100} \times \text{budget\_allocated}$ | Total spending in currency units. |
| `team_size` | `team` (If `0`, forced to `1`) | Number of personnel on the project. |
| `team_experience` | $\min(10, \text{team\_size} \times 1.5)$ | Estimates team seniority based on size (capped at 10). |
| `sprint_velocity` | $\text{team\_size} \times 10$ | Proxy sprint output capacity. |
| `past_delay_rate` | `DEFAULT_CONFIG["PAST_DELAY_RATE"]` (Default: `0.3`) | Organizational historical delay rate. |
| `completion_rate` | $\frac{\text{completed\_tasks}}{\text{total\_tasks}}$ | Percentage of tasks finished (0.0 to 1.0). |
| `budget_burn_rate`| $\frac{\text{budget\_spent}}{\text{budget\_allocated}}$ | Financial burn rate ratio (0.0 to 1.0). |
| `productivity_score`| $\frac{\text{sprint\_velocity}}{\text{team\_size}}$ | Team efficiency constant (defaults to 10.0). |

*Deterministic feature array ordering passed to the classifier:*
`[total_tasks, completed_tasks, avg_task_delay, budget_allocated, budget_spent, team_size, team_experience, sprint_velocity, past_delay_rate, completion_rate, budget_burn_rate, productivity_score]`

---

## 🤖 Model Specifications

RiskPulse utilizes a **Random Forest Classifier** from `scikit-learn` to perform predictions. Random Forests are highly suited for this task because they handle non-linear combinations of features, require minimal scaling, and provide built-in feature importance ratings.

### Model Hyperparameters
*   `n_estimators` = `100` (Number of decision trees in the forest)
*   `random_state` = `42` (Ensures prediction determinism)
*   `criterion` = `"gini"` (Split quality measurement)

### Training Process
1.  **Split**: The dataset `project_data.csv` is split into an 80/20 train/test split.
    ```python
    X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
    ```
2.  **Fitting**: The forest is fitted to the training features.
3.  **Serialization**: The fitted model pipeline is dumped to a binary pickle file (`risk_model.pkl`) using `joblib`.

---

## 🧠 Explainability & Reason Codes

To avoid a "black box" prediction, the system implements two explainability mechanisms:

### 1. Dynamic Feature Importance (Top Factors)
When predicting a project's risk, the system extracts the model's global tree-based node importances:
```python
importances = model.feature_importances_
indices = np.argsort(importances)[::-1][:3] # Get top 3 features
```
The model ranks which of the 12 features contributed most heavily to the decision tree splits for the prediction, returning them to the client with their percentage weights (e.g. `completion_rate: 24.12%`).

### 2. Business Logic Reason Codes
To provide immediate action items for project managers, the system reviews the engineered inputs against warning thresholds to append diagnostic reasons:
*   **Low progress**: Triggered if `completion_rate < 0.4` (40% completion).
*   **High budget usage**: Triggered if `budget_burn_rate > 0.8` (80% burn rate).
*   **High delay pressure**: Triggered if `avg_task_delay > 5` (derived from a tight deadline).

---

## 🔄 Retraining Cycle

If the underlying dataset is updated (e.g., more projects are completed) or if the host Python environment shifts (which causes `InconsistentVersionWarning` warnings during unpickling), the model can be retrained.

### Step-by-Step Retraining Execution
Run the retraining script from the repository root:
```bash
python retrain_model.py
```

This script:
1.  Detects and logs the active environment's `sklearn` package version.
2.  Loads `project_data.csv`.
3.  Re-runs the 80/20 split and trains the Random Forest model.
4.  Evaluates test metrics (Accuracy, Precision, Recall, F1-Score).
5.  Overwrites `risk_model.pkl` with the newly compiled version.
