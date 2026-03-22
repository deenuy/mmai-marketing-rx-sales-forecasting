# Rx MMIX Sales Modeling

Comparing parametric and machine learning models for Rx drug sales prediction, inference, and estimation using marketing mix data.

---

## Project Overview

This project analyzes prescription drug sales as a function of marketing investments, with a focus on understanding **how marketing translates into revenue** and how different modeling approaches support business decision-making.

The analysis is grounded in a pharmaceutical marketing context, where brands compete within therapeutic classes using two primary levers:

- **Detailing**: physician-targeted promotion
- **DTCA**: direct-to-consumer advertising

The commercial mechanism is:

> Marketing investment → influences physicians and patients → drives prescriptions → generates sales

---

## Business Objective

The goal of this project is to evaluate how marketing investments influence sales outcomes across brands and therapeutic classes, and to identify modeling approaches that best support:

- Sales prediction
- Marketing effectiveness estimation
- Elasticity and ROI interpretation
- Strategic budget allocation

---

## Core Business Questions

This analysis is structured around eight key strategic questions:

1. Which brands dominate portfolio revenue, and how concentrated is the portfolio?
2. How has portfolio performance evolved over time?
3. How are detailing and DTCA budgets allocated across brands and classes?
4. Which channel appears more efficient before formal modeling?
5. Is there evidence of diminishing returns to marketing spend?
6. Do classes and brands differ in baseline sales and marketing sensitivity?
7. Are there structurally different marketing regimes, especially around zero DTCA?
8. What does the panel structure imply for modeling and validation?

---

## Scope of Analysis

This project uses an 11-year panel dataset with the following characteristics:

- **Observation Period:** 2013–2024
- **Therapeutic Classes:**
    - PPI (Proton Pump Inhibitors)
    - SSRI (Antidepressants)
    - Statin (Cholesterol drugs)
- **Portfolio Size:** 22 brands

### Key Variables

- `actual_sales` → Sales ($ Billions)
- `detailing` → Physician detailing spend ($ Millions)
- `direct_to_consumer_ad` → DTCA spend ($ Millions)

---

## Analytical Approach

The project follows a structured workflow:

1. **Exploratory Data Analysis (EDA)**
    - Understand portfolio structure, growth, and marketing allocation
    - Identify nonlinearities and diminishing returns
    - Evaluate heterogeneity across classes and brands
    - Prepare transformed variables for modeling

2. **Parametric Modeling**
    - Log-log regression models
    - Fixed effects (class, agent, year)
    - Elasticity estimation

3. **Machine Learning Modeling**
    - Tree-based models (Random Forest, Gradient Boosting)
    - Nonlinear prediction comparison

4. **Model Evaluation**
    - Out-of-sample performance (RMSE, MAE, R², MAPE)
    - Stability and interpretability comparison

5. **Business Interpretation**
    - Elasticity estimates
    - Marketing ROI insights
    - Channel effectiveness comparison

---

## Repository Structure

```text
rx-mmix-sales-modeling/
│
├── data/
│   └── MMM_Drug_Data.xlsx
│
├── notebooks/
│   ├── 01_eda.ipynb
│   ├── 02_parametric_modeling.ipynb
│   ├── 03_ml_modeling.ipynb
│   └── 04_evaluation.ipynb
│
├── src/
│   ├── data_prep.py
│   ├── eda.py
│   ├── parametric_models.py
│   ├── ml_models.py
│   ├── evaluation.py
│   └── elasticity_roi.py
│
├── reports/
│   └── figures/
│
├── requirements.txt
├── pyproject.toml
└── README.md