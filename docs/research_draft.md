Below is a clean way to tackle this project end to end, with the analysis logic, modeling strategy, and a Jupyter notebook structure you can implement directly.

One important note before we start: the Excel file you uploaded appears to have **207 rows**, not 208, across **2013–2024**, **3 classes**, and **22 agents**. I would state that explicitly in the notebook so no one thinks the model lost a record in preprocessing.

## 1) What the assignment is really asking

The project objective is to compare **parametric** and **machine learning** approaches for **sales volume prediction**, then discuss their trade-offs for **prediction, inference, and estimation**, and finally use the selected model to estimate **ROI / elasticity / drivers of sales** and generate **actionable recommendations**. That is the heart of the assignment brief.

The lectures also make the framing very clear:

* In MMM, the job is not just prediction; it is also to explain how marketing inputs relate to outcomes and support decisions.
* The recommended modeling process is: **ask the right question → understand the business → represent it mathematically → estimate and benchmark → interpret and generate actionable insights**.
* You are expected to check for **nonlinearities**, **interactions**, and use a **meaningful train/test split**, especially since this is time-indexed panel-style data.

## 2) Clear problem statement

**Problem statement**

Build and benchmark parametric and machine learning models to predict prescription drug sales from marketing mix variables—**Detailing** and **DTCA**—while accounting for heterogeneity across **drug classes**, **brands/agents**, and **time**. Then select the best final model based on a balance of out-of-sample accuracy, interpretability, and usefulness for estimating business response measures such as elasticity and ROI.

## 3) The ask

Your notebook should answer five business questions:

1. **Can we predict sales well from Detailing, DTCA, Class, Agent, and Year?**
2. **Are the effects of Detailing and DTCA linear, or do they show diminishing returns / interactions?**
3. **Which model family performs better: parametric or ML?**
4. **Which variables appear to drive sales, and do effects differ by class/agent?**
5. **What should managers do with budget allocation across Detailing and DTCA?**

## 4) Objectives

### Predictive objective

Minimize out-of-sample prediction error for sales.

### Inferential objective

Understand the direction, size, and heterogeneity of Detailing and DTCA effects.

### Managerial objective

Estimate response measures that support budget allocation:

* elasticity
* marginal effect
* rough ROI / marginal revenue per extra marketing dollar

## 5) Recommended approach

Because the dataset is very small, this is not the moment for a deep-learning grand opera. This is a “sharp knife, not bigger hammer” problem.

### Data structure

This is a **small panel-like dataset**:

* cross-sectional units: Agent nested in Class
* time dimension: Year
* response: Actual Sales
* decision variables: Detailing, DTCA
* structural heterogeneity: Class, Agent, Year

### Modeling philosophy

Use two parallel tracks:

#### Parametric track

Good for:

* coefficient interpretation
* elasticity estimation
* agent/class effects
* direct inference

#### ML track

Good for:

* flexible nonlinearities
* interactions without manual specification
* stronger predictive benchmarking

Then compare the champions from each track, exactly as the assignment asks.

---

## 6) What to test in EDA

The lectures emphasize that residual plots and variable-response plots are how you diagnose nonlinearity and model misspecification.

In EDA, check:

* distribution of sales
* distribution of Detailing and DTCA
* zero values in DTCA
* trends over Year
* average sales by Class and Agent
* scatterplots:

    * Sales vs Detailing
    * Sales vs DTCA
* log-transformed relationships:

    * log(Sales) vs log(Detailing + 1)
    * log(Sales) vs log(DTCA + 1)
* correlation matrix
* whether some classes or agents have systematically different baselines

### Expected patterns

From MMM theory and response modeling notes, marketing response often shows:

* diminishing returns
* thresholds
* saturation
* possible S-shaped behavior in some settings

So it is reasonable to try:

* log transforms
* squared terms
* interactions with class/agent

---

## 7) Best strategy to split the data

With only ~207 observations, random row splitting is a bad idea here because it leaks future years into training.

### Recommended split

Use a **time-based split by year**.

#### Final holdout

* **Train:** 2013–2020
* **Validation:** 2021–2022
* **Test:** 2023–2024

Why this works:

* preserves time order
* gives enough training rows
* still leaves a true final holdout
* avoids “seeing the future”

### For hyperparameter tuning

Inside the train period, use **rolling year-based validation**:

Example folds:

* train 2013–2016, validate 2017
* train 2013–2017, validate 2018
* train 2013–2018, validate 2019
* train 2013–2019, validate 2020

This is much better than standard KFold for this use case.

### Why not random split?

Because the assignment explicitly asks for a meaningful split, “train on the first, validate on the second,” and your data is temporal.

---

## 8) Recommended modeling experimentation

## Parametric family: 3 models

### Parametric Model 1: Pooled OLS

Baseline linear model with one-hot controls.

Example:
[
Sales = \beta_0 + \beta_1 Detailing + \beta_2 DTCA + \beta_3 Year + Class + Agent + \epsilon
]

Use this as a sanity-check model.

### Parametric Model 2: Log-linear / FE-style OLS

Use transformed variables and interactions to capture diminishing returns.

Recommended:
[
\log(1+Sales) = \beta_0 + \beta_1 \log(1+Detailing) + \beta_2 \log(1+DTCA)

* Class + Agent + Year + \text{interactions} + \epsilon
  ]

Possible interactions:

* log_detailing × Class
* log_dtca × Class

This is probably your best parametric candidate because:

* log-log or semi-log forms align with MMM thinking on diminishing returns
* coefficients become easier to interpret as elasticities or semi-elasticities
* fixed effects via categorical dummies help control heterogeneity

### Parametric Model 3: Mixed Effects Model

Random intercept for Agent, fixed effects for Class and Year.

Example:
[
\log(1+Sales_{it}) = \beta_0 + \beta_1 \log(1+Detailing_{it}) + \beta_2 \log(1+DTCA_{it})

* Class_i + Year_t + u_{Agent(i)} + \epsilon_{it}
  ]

Why this matters:

* the assignment and workflow hint toward panel-aware / mixed-effects thinking
* brands likely have different baseline demand
* mixed models are often a strong compromise between interpretability and heterogeneity control

---

## ML family: 3 models

Since the data is small, keep the ML family practical.

### ML Model 1: Random Forest Regressor

Good nonlinear benchmark, robust on small datasets.

### ML Model 2: Gradient Boosting Regressor

Strong tabular baseline.

### ML Model 3: HistGradientBoostingRegressor

Modern boosting option available in sklearn; good for small/medium tabular data.

You could also use XGBoost/LightGBM if installed, but for a Jupyter assignment, sklearn is safer and more portable.

### Feature handling for ML

Use:

* one-hot encoding for Class and Agent
* numeric passthrough for Year, Detailing, DTCA
* optionally add log_detailing and log_dtca as engineered features

---

## 9) Experiment combinations to run

Use a small, disciplined model grid.

### Parametric experiments

1. OLS raw-scale main effects
2. OLS log-transformed main effects
3. OLS log-transformed + class interactions
4. FE-style OLS with `C(Agent) + C(Year)`
5. MixedLM with random intercept by Agent

### ML experiments

1. RF with raw features
2. RF with engineered logs
3. GBR with raw + logs
4. HGBR with raw + logs

### Champion duel

Compare:

* best parametric model
* best ML model

On:

* validation
* final test
* interpretability
* elasticity / ROI usability

---

## 10) Hyperparameter tuning strategy

Because the dataset is small, do **narrow, sensible grids**, not a fishing expedition.

### Random Forest

Tune:

* `n_estimators`: [100, 300, 500]
* `max_depth`: [3, 5, 8, None]
* `min_samples_leaf`: [1, 2, 4, 6]
* `max_features`: ['sqrt', 0.7, 1.0]

### Gradient Boosting

Tune:

* `n_estimators`: [100, 200, 300]
* `learning_rate`: [0.03, 0.05, 0.1]
* `max_depth`: [2, 3, 4]
* `min_samples_leaf`: [1, 2, 4]
* `subsample`: [0.7, 0.9, 1.0]

### HistGradientBoosting

Tune:

* `learning_rate`: [0.03, 0.05, 0.1]
* `max_depth`: [3, 5, None]
* `max_leaf_nodes`: [15, 31, 63]
* `min_samples_leaf`: [5, 10, 20]
* `l2_regularization`: [0.0, 0.1, 1.0]

### Parametric tuning

This is not grid search in the same way. Instead, compare specifications:

* raw vs log
* with vs without year fixed effects
* with vs without class interactions
* FE-style vs mixed-effects

---

## 11) Evaluation metrics

The assignment suggests AIC, BIC, R², MSE, MAPD.

Use this structure:

### For parametric models

* AIC
* BIC
* adjusted R²
* RMSE
* MAE
* MAPE or MAPD on validation/test

### For ML models

* RMSE
* MAE
* MAPE/MAPD
* test R²

### My recommendation

Use **RMSE as the primary model-selection metric** and **MAPE/MAPD as business-readable backup**.

Why:

* Sales is continuous
* RMSE penalizes large misses
* MAPE is useful for executives, but can misbehave near small denominators

Also show:

* actual vs predicted plots
* residual plots for parametric models
* feature importance / permutation importance for ML

---

## 12) How to handle inference, elasticity, and ROI

## Elasticity

### If final model is log-log parametric

Elasticity is straightforward:

* coefficient on `log_detailing` = detailing elasticity
* coefficient on `log_dtca` = dtca elasticity

If you include interactions by class, class-specific elasticity becomes:

* base coefficient + interaction coefficient

### If final model is not log-log

Use local numerical elasticity:
[
Elasticity_X = \frac{\partial \hat{Y}}{\partial X} \cdot \frac{X}{\hat{Y}}
]

Approximate numerically by bumping X by a small percentage, say 1% or 5%.

## ROI / marginal return

Since sales is in dollars and inputs are marketing spend, estimate:
[
Marginal\ ROI = \frac{\Delta \hat{Sales}}{\Delta Spend}
]

Important caveat:
One source says both Detailing and DTCA are in millions, while the lecture transcript snippet suggests DTCA may be in different units. I would add one line in your notebook saying:

> “For ROI conversion, I assume the assignment sheet is authoritative and treat Detailing and DTCA as dollar-denominated spend fields; verify workbook units before final submission.”

That is the sort of sentence that saves marks and arguments.

---

## 13) Recommended final modeling decision

My recommendation for this assignment:

### Best parametric candidate

**Log-linear FE-style OLS** or **Mixed Effects log model**

Why:

* appropriate for tiny dataset
* directly interpretable
* supports elasticity
* handles baseline differences across agents/classes
* aligned with MMM purpose of inference, not just prediction

### Best ML candidate

**Gradient Boosting** or **Random Forest**

Why:

* robust on small structured data
* easy benchmarking
* can reveal nonlinearities

### Likely final winner

For an academic MMM assignment, I would **expect the final chosen model to be the best parametric model unless ML is materially better on holdout accuracy**.

That is because the assignment explicitly values:

* inference
* estimation
* ROI / elasticity
* actionable managerial insights

And parametric models are far better at those.

---

# Jupyter notebook implementation

Now the practical part. Below is a clean notebook structure by cells.

---

## CELL 1 — Title and setup

```python
# Rx Drug MMIX Analysis
# Comparing Parametric and Machine Learning Models
# for Sales Volume Prediction, Inference, and Estimation

import warnings
warnings.filterwarnings("ignore")

import numpy as np
import pandas as pd

import matplotlib.pyplot as plt
import seaborn as sns

from sklearn.model_selection import ParameterGrid
from sklearn.metrics import mean_squared_error, mean_absolute_error, r2_score

import statsmodels.api as sm
import statsmodels.formula.api as smf

from sklearn.compose import ColumnTransformer
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import OneHotEncoder, StandardScaler
from sklearn.impute import SimpleImputer

from sklearn.ensemble import RandomForestRegressor, GradientBoostingRegressor, HistGradientBoostingRegressor
from sklearn.inspection import permutation_importance

sns.set_theme(style="whitegrid")
pd.set_option("display.max_columns", 100)
pd.set_option("display.float_format", lambda x: f"{x:,.4f}")
```

---

## CELL 2 — Load data

```python
file_path = "MMM_Drug Data.xlsx"
df = pd.read_excel(file_path)

df.head()
```

---

## CELL 3 — Basic data check

```python
print("Shape:", df.shape)
print("\nColumns:")
print(df.columns.tolist())

print("\nDtypes:")
print(df.dtypes)

print("\nMissing values:")
print(df.isna().sum())

print("\nYears:", df["Year"].min(), "to", df["Year"].max())
print("Unique classes:", df["Class"].nunique(), df["Class"].unique())
print("Unique agents:", df["Agent"].nunique())
```

---

## CELL 4 — Rename columns for easier modeling

```python
df = df.rename(columns={
    "Direct-to-Consumer Advertising (DTCA)": "DTCA",
    "Actual Sales": "Sales"
})

df.head()
```

---

## CELL 5 — Feature engineering

```python
df["log_sales"] = np.log1p(df["Sales"])
df["log_detailing"] = np.log1p(df["Detailing"])
df["log_dtca"] = np.log1p(df["DTCA"])

df["detailing_sq"] = df["Detailing"] ** 2
df["dtca_sq"] = df["DTCA"] ** 2

df["detailing_x_dtca"] = df["Detailing"] * df["DTCA"]
df["log_detailing_x_log_dtca"] = df["log_detailing"] * df["log_dtca"]

df.head()
```

---

## CELL 6 — EDA summary tables

```python
display(df.describe(include="all"))

display(
    df.groupby("Class")[["Sales", "Detailing", "DTCA"]].mean().sort_values("Sales", ascending=False)
)

display(
    df.groupby("Agent")[["Sales", "Detailing", "DTCA"]].mean().sort_values("Sales", ascending=False).head(10)
)
```

---

## CELL 7 — EDA plots: distributions

```python
fig, axes = plt.subplots(1, 3, figsize=(18, 5))

sns.histplot(df["Sales"], kde=True, ax=axes[0])
axes[0].set_title("Distribution of Sales")

sns.histplot(df["Detailing"], kde=True, ax=axes[1])
axes[1].set_title("Distribution of Detailing")

sns.histplot(df["DTCA"], kde=True, ax=axes[2])
axes[2].set_title("Distribution of DTCA")

plt.tight_layout()
plt.show()
```

---

## CELL 8 — EDA plots: response curves

```python
fig, axes = plt.subplots(2, 2, figsize=(14, 10))

sns.scatterplot(data=df, x="Detailing", y="Sales", hue="Class", ax=axes[0,0])
axes[0,0].set_title("Sales vs Detailing")

sns.scatterplot(data=df, x="DTCA", y="Sales", hue="Class", ax=axes[0,1])
axes[0,1].set_title("Sales vs DTCA")

sns.scatterplot(data=df, x="log_detailing", y="log_sales", hue="Class", ax=axes[1,0])
axes[1,0].set_title("log(Sales) vs log(Detailing)")

sns.scatterplot(data=df, x="log_dtca", y="log_sales", hue="Class", ax=axes[1,1])
axes[1,1].set_title("log(Sales) vs log(DTCA)")

plt.tight_layout()
plt.show()
```

---

## CELL 9 — Sales over time

```python
yearly = df.groupby("Year")[["Sales", "Detailing", "DTCA"]].mean().reset_index()

fig, axes = plt.subplots(3, 1, figsize=(12, 12), sharex=True)

sns.lineplot(data=yearly, x="Year", y="Sales", marker="o", ax=axes[0])
axes[0].set_title("Average Sales by Year")

sns.lineplot(data=yearly, x="Year", y="Detailing", marker="o", ax=axes[1])
axes[1].set_title("Average Detailing by Year")

sns.lineplot(data=yearly, x="Year", y="DTCA", marker="o", ax=axes[2])
axes[2].set_title("Average DTCA by Year")

plt.tight_layout()
plt.show()
```

---

## CELL 10 — Correlation matrix

```python
corr_cols = ["Sales", "Detailing", "DTCA", "log_sales", "log_detailing", "log_dtca"]
corr = df[corr_cols].corr()

plt.figure(figsize=(8, 6))
sns.heatmap(corr, annot=True, cmap="Blues", fmt=".2f")
plt.title("Correlation Matrix")
plt.show()
```

---

## CELL 11 — Time-based split

```python
train_df = df[df["Year"] <= 2020].copy()
valid_df = df[(df["Year"] >= 2021) & (df["Year"] <= 2022)].copy()
test_df  = df[df["Year"] >= 2023].copy()

print("Train shape:", train_df.shape)
print("Validation shape:", valid_df.shape)
print("Test shape:", test_df.shape)

print("\nYears in train:", sorted(train_df["Year"].unique()))
print("Years in validation:", sorted(valid_df["Year"].unique()))
print("Years in test:", sorted(test_df["Year"].unique()))
```

---

## CELL 12 — Utility functions for evaluation

```python
def regression_metrics(y_true, y_pred, model_name="model", dataset_name="dataset"):
    rmse = np.sqrt(mean_squared_error(y_true, y_pred))
    mae = mean_absolute_error(y_true, y_pred)
    r2 = r2_score(y_true, y_pred)
    mape = np.mean(np.abs((y_true - y_pred) / np.clip(np.abs(y_true), 1e-8, None))) * 100
    
    return pd.DataFrame({
        "model": [model_name],
        "dataset": [dataset_name],
        "RMSE": [rmse],
        "MAE": [mae],
        "R2": [r2],
        "MAPE_pct": [mape]
    })

def plot_actual_vs_pred(y_true, y_pred, title):
    plt.figure(figsize=(7, 6))
    plt.scatter(y_true, y_pred)
    lims = [min(y_true.min(), y_pred.min()), max(y_true.max(), y_pred.max())]
    plt.plot(lims, lims, linestyle="--")
    plt.xlabel("Actual")
    plt.ylabel("Predicted")
    plt.title(title)
    plt.show()
```

---

# PARAMETRIC MODELS

## CELL 13 — Parametric Model 1: pooled OLS

```python
formula_ols_1 = "Sales ~ Detailing + DTCA + C(Class) + C(Agent) + Year"
ols_1 = smf.ols(formula_ols_1, data=train_df).fit()

print(ols_1.summary())
```

---

## CELL 14 — Evaluate Parametric Model 1

```python
valid_pred_ols_1 = ols_1.predict(valid_df)
test_pred_ols_1 = ols_1.predict(test_df)

results_parametric = []
results_parametric.append(regression_metrics(valid_df["Sales"], valid_pred_ols_1, "OLS_raw", "validation"))
results_parametric.append(regression_metrics(test_df["Sales"], test_pred_ols_1, "OLS_raw", "test"))

display(pd.concat(results_parametric, ignore_index=True))

print("AIC:", ols_1.aic)
print("BIC:", ols_1.bic)

plot_actual_vs_pred(valid_df["Sales"], valid_pred_ols_1, "OLS Raw: Validation Actual vs Predicted")
```

---

## CELL 15 — Parametric Model 2: log-linear FE-style model with interactions

```python
formula_ols_2 = """
log_sales ~ log_detailing + log_dtca
          + C(Class) + C(Agent) + C(Year)
          + log_detailing:C(Class)
          + log_dtca:C(Class)
"""

ols_2 = smf.ols(formula_ols_2, data=train_df).fit()
print(ols_2.summary())
```

---

## CELL 16 — Evaluate Parametric Model 2

```python
valid_pred_log_ols_2 = np.expm1(ols_2.predict(valid_df))
test_pred_log_ols_2 = np.expm1(ols_2.predict(test_df))

results_parametric.append(regression_metrics(valid_df["Sales"], valid_pred_log_ols_2, "OLS_log_FE_interactions", "validation"))
results_parametric.append(regression_metrics(test_df["Sales"], test_pred_log_ols_2, "OLS_log_FE_interactions", "test"))

display(pd.concat(results_parametric, ignore_index=True))

print("AIC:", ols_2.aic)
print("BIC:", ols_2.bic)

plot_actual_vs_pred(valid_df["Sales"], valid_pred_log_ols_2, "OLS Log FE Interactions: Validation Actual vs Predicted")
```

---

## CELL 17 — Parametric Model 3: Mixed Effects Model

```python
formula_mixed = "log_sales ~ log_detailing + log_dtca + C(Class) + C(Year)"
mixed_1 = smf.mixedlm(formula_mixed, data=train_df, groups=train_df["Agent"]).fit(reml=False)

print(mixed_1.summary())
```

---

## CELL 18 — Evaluate Parametric Model 3

```python
valid_pred_mixed = np.expm1(mixed_1.predict(valid_df))
test_pred_mixed = np.expm1(mixed_1.predict(test_df))

results_parametric.append(regression_metrics(valid_df["Sales"], valid_pred_mixed, "MixedLM_log", "validation"))
results_parametric.append(regression_metrics(test_df["Sales"], test_pred_mixed, "MixedLM_log", "test"))

parametric_results_df = pd.concat(results_parametric, ignore_index=True)
display(parametric_results_df)

print("AIC:", mixed_1.aic)
print("BIC:", mixed_1.bic)

plot_actual_vs_pred(valid_df["Sales"], valid_pred_mixed, "MixedLM Log: Validation Actual vs Predicted")
```

---

## CELL 19 — Residual diagnostics for best parametric model

```python
# Choose your best parametric model manually after reviewing results.
best_parametric_model = ols_2
best_parametric_name = "OLS_log_FE_interactions"

train_fitted = np.expm1(best_parametric_model.predict(train_df))
train_resid = train_df["Sales"] - train_fitted

fig, axes = plt.subplots(1, 2, figsize=(14, 5))

sns.scatterplot(x=train_fitted, y=train_resid, ax=axes[0])
axes[0].axhline(0, linestyle="--")
axes[0].set_title("Residuals vs Fitted")

sm.qqplot(train_resid, line="45", ax=axes[1])
axes[1].set_title("Q-Q Plot of Residuals")

plt.tight_layout()
plt.show()
```

---

# MACHINE LEARNING MODELS

## CELL 20 — ML preprocessing

```python
feature_cols = ["Class", "Agent", "Year", "Detailing", "DTCA", "log_detailing", "log_dtca"]
target_col = "Sales"

X_train = train_df[feature_cols].copy()
y_train = train_df[target_col].copy()

X_valid = valid_df[feature_cols].copy()
y_valid = valid_df[target_col].copy()

X_test = test_df[feature_cols].copy()
y_test = test_df[target_col].copy()

categorical_cols = ["Class", "Agent"]
numeric_cols = ["Year", "Detailing", "DTCA", "log_detailing", "log_dtca"]

preprocessor = ColumnTransformer(
    transformers=[
        ("cat", Pipeline([
            ("imputer", SimpleImputer(strategy="most_frequent")),
            ("onehot", OneHotEncoder(handle_unknown="ignore"))
        ]), categorical_cols),
        ("num", Pipeline([
            ("imputer", SimpleImputer(strategy="median"))
        ]), numeric_cols)
    ]
)
```

---

## CELL 21 — Rolling validation helper for hyperparameter tuning

```python
def get_year_folds(df, min_train_end=2016, max_train_end=2019):
    folds = []
    for train_end in range(min_train_end, max_train_end + 1):
        tr = df[df["Year"] <= train_end].copy()
        va = df[df["Year"] == train_end + 1].copy()
        if len(tr) > 0 and len(va) > 0:
            folds.append((tr.index, va.index))
    return folds

cv_folds = get_year_folds(train_df)
cv_folds
```

---

## CELL 22 — Generic ML tuning function

```python
def evaluate_model_pipeline(model, X_tr, y_tr, X_va, y_va, name):
    pipe = Pipeline([
        ("prep", preprocessor),
        ("model", model)
    ])
    pipe.fit(X_tr, y_tr)
    pred_va = pipe.predict(X_va)
    metrics = regression_metrics(y_va, pred_va, name, "validation")
    return pipe, metrics

def rolling_year_cv_score(model_class, param_grid, df_train, feature_cols, target_col, model_name):
    rows = []
    for params in ParameterGrid(param_grid):
        fold_scores = []
        for tr_idx, va_idx in cv_folds:
            tr = df_train.loc[tr_idx]
            va = df_train.loc[va_idx]
            
            X_tr = tr[feature_cols]
            y_tr = tr[target_col]
            X_va = va[feature_cols]
            y_va = va[target_col]
            
            model = model_class(**params)
            pipe = Pipeline([
                ("prep", preprocessor),
                ("model", model)
            ])
            pipe.fit(X_tr, y_tr)
            pred = pipe.predict(X_va)
            rmse = np.sqrt(mean_squared_error(y_va, pred))
            fold_scores.append(rmse)
        
        rows.append({
            "model": model_name,
            "params": params,
            "cv_rmse_mean": np.mean(fold_scores),
            "cv_rmse_std": np.std(fold_scores)
        })
    
    return pd.DataFrame(rows).sort_values("cv_rmse_mean")
```

---

## CELL 23 — Tune Random Forest

```python
rf_grid = {
    "n_estimators": [100, 300, 500],
    "max_depth": [3, 5, 8, None],
    "min_samples_leaf": [1, 2, 4],
    "max_features": ["sqrt", 0.7, 1.0],
    "random_state": [42]
}

rf_cv = rolling_year_cv_score(RandomForestRegressor, rf_grid, train_df, feature_cols, target_col, "RandomForest")
rf_cv.head(10)
```

---

## CELL 24 — Fit best Random Forest

```python
best_rf_params = rf_cv.iloc[0]["params"]
best_rf = RandomForestRegressor(**best_rf_params)

rf_pipe = Pipeline([
    ("prep", preprocessor),
    ("model", best_rf)
])

rf_pipe.fit(X_train, y_train)

valid_pred_rf = rf_pipe.predict(X_valid)
test_pred_rf = rf_pipe.predict(X_test)

results_ml = []
results_ml.append(regression_metrics(y_valid, valid_pred_rf, "RandomForest", "validation"))
results_ml.append(regression_metrics(y_test, test_pred_rf, "RandomForest", "test"))

display(pd.concat(results_ml, ignore_index=True))
plot_actual_vs_pred(y_valid, valid_pred_rf, "Random Forest: Validation Actual vs Predicted")
```

---

## CELL 25 — Tune Gradient Boosting

```python
gbr_grid = {
    "n_estimators": [100, 200, 300],
    "learning_rate": [0.03, 0.05, 0.1],
    "max_depth": [2, 3, 4],
    "min_samples_leaf": [1, 2, 4],
    "subsample": [0.7, 0.9, 1.0],
    "random_state": [42]
}

gbr_cv = rolling_year_cv_score(GradientBoostingRegressor, gbr_grid, train_df, feature_cols, target_col, "GradientBoosting")
gbr_cv.head(10)
```

---

## CELL 26 — Fit best Gradient Boosting

```python
best_gbr_params = gbr_cv.iloc[0]["params"]
best_gbr = GradientBoostingRegressor(**best_gbr_params)

gbr_pipe = Pipeline([
    ("prep", preprocessor),
    ("model", best_gbr)
])

gbr_pipe.fit(X_train, y_train)

valid_pred_gbr = gbr_pipe.predict(X_valid)
test_pred_gbr = gbr_pipe.predict(X_test)

results_ml.append(regression_metrics(y_valid, valid_pred_gbr, "GradientBoosting", "validation"))
results_ml.append(regression_metrics(y_test, test_pred_gbr, "GradientBoosting", "test"))

ml_results_df = pd.concat(results_ml, ignore_index=True)
display(ml_results_df)
plot_actual_vs_pred(y_valid, valid_pred_gbr, "Gradient Boosting: Validation Actual vs Predicted")
```

---

## CELL 27 — Tune HistGradientBoosting

```python
hgb_grid = {
    "learning_rate": [0.03, 0.05, 0.1],
    "max_depth": [3, 5, None],
    "max_leaf_nodes": [15, 31, 63],
    "min_samples_leaf": [5, 10, 20],
    "l2_regularization": [0.0, 0.1, 1.0],
    "random_state": [42]
}

hgb_cv = rolling_year_cv_score(HistGradientBoostingRegressor, hgb_grid, train_df, feature_cols, target_col, "HistGradientBoosting")
hgb_cv.head(10)
```

---

## CELL 28 — Fit best HistGradientBoosting

```python
best_hgb_params = hgb_cv.iloc[0]["params"]
best_hgb = HistGradientBoostingRegressor(**best_hgb_params)

hgb_pipe = Pipeline([
    ("prep", preprocessor),
    ("model", best_hgb)
])

hgb_pipe.fit(X_train, y_train)

valid_pred_hgb = hgb_pipe.predict(X_valid)
test_pred_hgb = hgb_pipe.predict(X_test)

results_ml.append(regression_metrics(y_valid, valid_pred_hgb, "HistGradientBoosting", "validation"))
results_ml.append(regression_metrics(y_test, test_pred_hgb, "HistGradientBoosting", "test"))

ml_results_df = pd.concat(results_ml, ignore_index=True)
display(ml_results_df)
plot_actual_vs_pred(y_valid, valid_pred_hgb, "HistGradientBoosting: Validation Actual vs Predicted")
```

---

## CELL 29 — Compare ML models

```python
ml_results_df = pd.concat(results_ml, ignore_index=True)
display(ml_results_df.sort_values(["dataset", "RMSE"]))
```

---

## CELL 30 — Pick best ML and feature importance

```python
best_ml_pipe = gbr_pipe   # change this after reviewing results
best_ml_name = "GradientBoosting"

perm = permutation_importance(best_ml_pipe, X_valid, y_valid, n_repeats=20, random_state=42)

feature_names = best_ml_pipe.named_steps["prep"].get_feature_names_out()
importance_df = pd.DataFrame({
    "feature": feature_names,
    "importance_mean": perm.importances_mean,
    "importance_std": perm.importances_std
}).sort_values("importance_mean", ascending=False)

importance_df.head(20)
```

---

## CELL 31 — Plot top feature importances

```python
top_imp = importance_df.head(15).sort_values("importance_mean")

plt.figure(figsize=(10, 7))
plt.barh(top_imp["feature"], top_imp["importance_mean"])
plt.title(f"Top Permutation Importances - {best_ml_name}")
plt.xlabel("Mean Importance")
plt.show()
```

---

# CHAMPION MODEL DUEL

## CELL 32 — Compare best parametric vs best ML on validation and test

```python
champion_results = pd.concat([
    regression_metrics(y_valid, valid_pred_gbr, "Best_ML", "validation"),
    regression_metrics(y_test, test_pred_gbr, "Best_ML", "test"),
    regression_metrics(valid_df["Sales"], valid_pred_log_ols_2, "Best_Parametric", "validation"),
    regression_metrics(test_df["Sales"], test_pred_log_ols_2, "Best_Parametric", "test")
], ignore_index=True)

display(champion_results.sort_values(["dataset", "RMSE"]))
```

---

## CELL 33 — Retrain final selected model on train + validation

```python
train_valid_df = df[df["Year"] <= 2022].copy()

X_train_valid = train_valid_df[feature_cols]
y_train_valid = train_valid_df["Sales"]

X_test = test_df[feature_cols]
y_test = test_df["Sales"]
```

### Option A: final selected model = parametric

```python
final_parametric = smf.ols(formula_ols_2, data=train_valid_df).fit()

final_test_pred_parametric = np.expm1(final_parametric.predict(test_df))
final_parametric_test_metrics = regression_metrics(y_test, final_test_pred_parametric, "Final_Parametric", "test")
display(final_parametric_test_metrics)
```

### Option B: final selected model = ML

```python
final_ml = Pipeline([
    ("prep", preprocessor),
    ("model", GradientBoostingRegressor(**best_gbr_params))
])

final_ml.fit(X_train_valid, y_train_valid)
final_test_pred_ml = final_ml.predict(X_test)

final_ml_test_metrics = regression_metrics(y_test, final_test_pred_ml, "Final_ML", "test")
display(final_ml_test_metrics)
```

---

# ELASTICITY, ROI, AND ACTIONABLE INSIGHTS

## CELL 34 — Elasticity from parametric model

Use this if your final model is the log model.

```python
print(final_parametric.summary())
```

Then interpret:

* coefficient on `log_detailing`
* coefficient on `log_dtca`
* plus class interaction adjustments

If you want a table:

```python
coef_table = pd.DataFrame({
    "term": final_parametric.params.index,
    "coef": final_parametric.params.values,
    "pvalue": final_parametric.pvalues.values
})

coef_table.sort_values("pvalue").head(20)
```

---

## CELL 35 — Local elasticity calculator for any model

```python
def local_elasticity(model, data, feature, base_pred_col=None, bump_pct=0.01, is_parametric=False):
    data_up = data.copy()
    data_up[feature] = data_up[feature] * (1 + bump_pct)
    
    if is_parametric:
        pred_base = np.expm1(model.predict(data))
        pred_up = np.expm1(model.predict(data_up))
    else:
        pred_base = model.predict(data[feature_cols])
        pred_up = model.predict(data_up[feature_cols])
    
    elasticity = ((pred_up - pred_base) / np.clip(pred_base, 1e-8, None)) / bump_pct
    return elasticity

# Example for parametric:
elasticity_detailing = local_elasticity(final_parametric, test_df, "Detailing", is_parametric=True)
elasticity_dtca = local_elasticity(final_parametric, test_df, "DTCA", is_parametric=True)

print("Average Detailing Elasticity:", np.mean(elasticity_detailing))
print("Average DTCA Elasticity:", np.mean(elasticity_dtca))
```

---

## CELL 36 — Elasticity by class

```python
test_df_eval = test_df.copy()
test_df_eval["elasticity_detailing"] = elasticity_detailing
test_df_eval["elasticity_dtca"] = elasticity_dtca

display(
    test_df_eval.groupby("Class")[["elasticity_detailing", "elasticity_dtca"]]
    .mean()
    .sort_values("elasticity_detailing", ascending=False)
)

display(
    test_df_eval.groupby("Agent")[["elasticity_detailing", "elasticity_dtca"]]
    .mean()
    .sort_values("elasticity_detailing", ascending=False)
    .head(10)
)
```

---

## CELL 37 — Marginal ROI approximation

```python
def marginal_roi(model, data, spend_feature, spend_unit_dollars=1_000_000, bump_units=1.0, is_parametric=False):
    data_up = data.copy()
    data_up[spend_feature] = data_up[spend_feature] + bump_units
    
    if is_parametric:
        pred_base = np.expm1(model.predict(data))
        pred_up = np.expm1(model.predict(data_up))
    else:
        pred_base = model.predict(data[feature_cols])
        pred_up = model.predict(data_up[feature_cols])
    
    # sales assumed in billions of dollars
    incremental_sales_dollars = (pred_up - pred_base) * 1_000_000_000
    incremental_spend_dollars = bump_units * spend_unit_dollars
    
    roi = incremental_sales_dollars / incremental_spend_dollars
    return roi

roi_detailing = marginal_roi(final_parametric, test_df, "Detailing", spend_unit_dollars=1_000_000, bump_units=1.0, is_parametric=True)
roi_dtca = marginal_roi(final_parametric, test_df, "DTCA", spend_unit_dollars=1_000_000, bump_units=1.0, is_parametric=True)

print("Average marginal revenue per +1 unit Detailing:", np.mean(roi_detailing))
print("Average marginal revenue per +1 unit DTCA:", np.mean(roi_dtca))
```

---

## CELL 38 — Business insight tables

```python
insight_df = test_df.copy()
insight_df["pred_sales"] = np.expm1(final_parametric.predict(test_df))
insight_df["elasticity_detailing"] = elasticity_detailing
insight_df["elasticity_dtca"] = elasticity_dtca
insight_df["roi_detailing"] = roi_detailing
insight_df["roi_dtca"] = roi_dtca

class_summary = insight_df.groupby("Class")[[
    "pred_sales", "elasticity_detailing", "elasticity_dtca", "roi_detailing", "roi_dtca"
]].mean().sort_values("pred_sales", ascending=False)

agent_summary = insight_df.groupby("Agent")[[
    "pred_sales", "elasticity_detailing", "elasticity_dtca", "roi_detailing", "roi_dtca"
]].mean().sort_values("pred_sales", ascending=False)

display(class_summary)
display(agent_summary.head(15))
```

---

## CELL 39 — Final write-up bullets generated from results

```python
print("Suggested write-up structure:")
print("""
1. Best parametric model:
   - state model form
   - explain why log transforms / interactions / FE or mixed effects were used

2. Best ML model:
   - state algorithm
   - summarize tuning result and holdout performance

3. Champion comparison:
   - compare RMSE, MAE, MAPE, R2
   - discuss prediction vs interpretation trade-off

4. Final model:
   - justify why selected
   - summarize key coefficients / elasticities / ROI patterns

5. Actionable insights:
   - where detailing appears stronger
   - where DTCA appears stronger
   - which classes or agents show higher marginal response
   - note diminishing returns if supported
""")
```

---

# How to write the discussion section

Use this structure in your report.

## A. Why the final split was chosen

Because the data is panel-like and time indexed, a time-based split better reflects real forecasting conditions and avoids future leakage. This is aligned with the assignment’s instruction to train on earlier data and validate on later data.

## B. Why the parametric models were chosen

Because MMM is often used for **inference and decision support**, not just black-box prediction. Log or nonlinear response forms are also consistent with diminishing-return theory in response modeling.

## C. Why the ML models were chosen

Because they flexibly capture nonlinearities and interactions that may not be obvious in a small dataset.

## D. What trade-off to discuss

This is the money paragraph:

* **ML advantage:** often better predictive fit and automatic interaction discovery
* **Parametric advantage:** easier inference, clearer elasticity, more defensible managerial recommendations
* **Final choice:** depends on whether holdout gain from ML is big enough to justify giving up interpretability

That trade-off is exactly what the assignment wants.

---

# My recommendation for your final submission

Use this as your likely final storyline unless the data strongly contradicts it:

1. EDA suggests heterogeneity across class/agent and likely diminishing returns.
2. Benchmark 3 parametric models and 3 ML models.
3. Best parametric model is the **log-transformed FE-style OLS** or **MixedLM**.
4. Best ML model is **Gradient Boosting**.
5. Compare both on validation and test.
6. If ML only slightly beats parametric, choose **parametric** as final model because it is much better for elasticity, ROI, and explanation.
7. Present actionable insights by class and agent.

That is solid, defensible, and very much in the spirit of the course materials.

I can also turn this into a polished `.ipynb` notebook file for you.
