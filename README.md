# SVR Regression · Airbnb Santiago Pricing

**Support Vector Regression applied to short-term rental price prediction — Santiago, Chile (Inside Airbnb, Dec 2024)**

---

## Overview

End-to-end machine learning pipeline that predicts Airbnb listing prices in Santiago using **SVR with RBF kernel**, covering data preprocessing, comparative feature selection across six methods, hyperparameter tuning, and operational deployment on unseen data.

**Test performance:** RMSE = 0.3544 (log1p) · R² = 0.706 · MAE ≈ CLP 16,161/night

---

## Pipeline

```
Raw Data → Price Filter → EDA → Encoding → Train/Test Split
→ Clean Algorithm → MinMax Normalization → Comparative Feature Selection → SVR Tuning → Evaluation
```

| Phase | Steps |
|---|---|
| Data & EDA | Load · Inventory · Categorization · Price filter · OHE |
| Preprocessing | Clean Algorithm (constant + correlated features) · MinMax normalization (Train-only) |
| Feature Selection | 6 selectors · shared k\* · CV-5 on Train only |
| Modeling | SVR RBF · Grid Search · Cross-validation |
| Evaluation | Train/Test metrics · Residual analysis · Back-transform to CLP |
| Operation | Synthetic new data · Segmentation · Intervention framework |

---

## Feature Selection — Comparative Strategy

Six selectors compete under identical conditions: same **k\* = 23** (consensus across Fisher J, Elastic Net, Random Forest natural k), same SVR evaluator (C=5, γ=0.05, ε=0.1), CV-5 exclusively on Train. Test is sealed until final evaluation.

| Selector | Type | Ψ Score |
|---|---|---|
| **S4 — XGBoost (Gain)** ✔ | Embedded · nonlinear | **0.8496** |
| S5 — Random Forest | Embedded · permutation | 0.8446 |
| S6 — SFS-SVR | Wrapper · greedy | 0.8416 |
| S3 — Branch & Bound | Filter · multivariate | 0.8376 |
| S2 — Elastic Net | Embedded · L1+L2 | 0.8331 |
| S1 — Fisher J | Filter · univariate | 0.8301 |

**Winner selection criterion:**
$$\Psi_s = \frac{1}{2}\left(1 - \frac{\text{RMSE}_s^{CV} - \min\text{RMSE}^{CV}}{\max\text{RMSE}^{CV} - \min\text{RMSE}^{CV}}\right) + \frac{1}{2} \cdot R^{2,CV}_s$$

---

## Key Results

| Metric | Train | Test |
|---|---|---|
| RMSE (log1p) | 0.3251 | 0.3544 |
| MAE (log1p) | — | 0.2413 |
| R² | 0.7492 | 0.7058 |
| RMSE (CLP) | — | ~34,741 |
| MAE (CLP) | — | ~16,161 |

Train–Test gap: ΔR² = 0.044 · No relevant overfitting detected.

---

## Geographic Insight

The model identified **5 communes** with statistically significant price discrimination power. The map below shows their Fisher J scores over the official RM cartography.

![Georreferenciación comunas seleccionadas — Región Metropolitana](georef_comunas_RM.png)

> Communes retained by XGBoost (Gain): **Las Condes · Lo Barnechea · Providencia · Vitacura · Santiago**. Exclusions are data-driven, not editorial — communes with homogeneous price distributions relative to the regional baseline add no predictive value under SVR-CV5 evaluation.

---

## Stack

| Layer | Tools |
|---|---|
| Language | R 4.x |
| Modeling | `e1071` · `caret` |
| Feature Selection | `glmnet` · `randomForest` · `xgboost` · custom Fisher J / B&B / SFS |
| Visualization | `ggplot2` · `chilemapas` · `sf` · `igraph` |
| Report | Quarto (`.qmd`) → self-contained HTML |

---

## Repository Structure

```
├── SVM_REGRESION.qmd          # Full reproducible report (Quarto)
├── listings.csv.gz            # Raw data — Inside Airbnb Santiago (Dec 2024)
├── georef_comunas_RM.png      # Geographic visualization — selected communes
└── README.md
```

> **To render:** update the `listings.csv.gz` path in chunk `load-raw`, then run `quarto render SVM_REGRESION.qmd`.

---

## Data Source

[Inside Airbnb — Santiago de Chile](https://insideairbnb.com/get-the-data/) · December 2024

---

*Alejandro Figueroa Rojas · Data & Business Intelligence*
