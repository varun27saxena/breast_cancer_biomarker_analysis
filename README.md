# 🔬 Breast Cancer Biomarker Discovery & Explainable AI

> Identifying the cell-nucleus measurements that best distinguish malignant from benign tumours — using classical ML, feature selection, and SHAP-based model explainability.

---

## Overview

This project goes beyond simply training a classifier. The central question is:

**Which measurable properties of a cell nucleus actually drive a malignancy prediction — and by how much?**

To answer it, three independent feature-importance methods are applied and cross-validated against each other: logistic regression coefficients, Random Forest Gini importance, and SHAP (SHapley Additive exPlanations). The consistency across methods provides high confidence in the identified biomarkers.

---

## Dataset

**Breast Cancer Wisconsin (Diagnostic)**  
Source: [UCI Machine Learning Repository](https://archive.ics.uci.edu/ml/datasets/Breast+Cancer+Wisconsin+(Diagnostic))

| Property | Value |
|----------|-------|
| Samples | 569 patients |
| Features | 30 numeric (10 measurements × 3 statistics: mean, SE, worst) |
| Target | Binary — Malignant (M) / Benign (B) |
| Class split | 63% Benign, 37% Malignant |
| Missing values | None |

Each feature describes a geometric property of cell nuclei extracted from digitised fine-needle aspirate (FNA) images: radius, texture, perimeter, area, smoothness, compactness, concavity, concave points, symmetry, and fractal dimension.

---

## Project Structure

```
breast-cancer-biomarker/
│
├── breast_cancer_biomarker_analysis.ipynb   # Main notebook
├── data.csv                                 # Dataset (Wisconsin Diagnostic)
└── README.md
```

---

## Methodology

### Phase 1 — Exploratory Data Analysis
- Shape, dtypes, and column inventory
- Class distribution check (stratification decision)
- Missing value and duplicate scan
- Correlation heatmap → multicollinearity identified among size features
- Boxplots confirming class separability for key features

### Phase 2 — Preprocessing
- Label encoding: `B → 0`, `M → 1`
- Stratified 80/20 train-test split
- `StandardScaler` fit on training data only (no data leakage)

### Phase 3 — Baseline: Logistic Regression (all 30 features)
- Trained on scaled features
- Evaluated with accuracy, precision, recall, F1, and confusion matrix
- Feature ranking via absolute coefficient magnitude

### Phase 4 — Ensemble: Random Forest (all 30 features)
- 200 estimators, no scaling required
- Feature ranking via mean decrease in Gini impurity

### Phase 5 — Feature Selection
- Top-10 features from Random Forest importance retained
- Logistic Regression retrained on reduced feature set
- Accuracy compared against full-feature baseline → nearly identical

### Phase 6 — SHAP Explainability
- `TreeExplainer` applied to the Random Forest
- Beeswarm summary plot: per-patient, per-feature attribution
- Direction of contribution (positive → malignant, negative → benign) confirmed for each top feature

### Phase 7 — Cross-Method Comparison & Conclusions

---

## Results

### Model Performance

| Model | Features | Accuracy |
|-------|----------|----------|
| Logistic Regression | All 30 | ~96% |
| Random Forest | All 30 | ~97% |
| Logistic Regression | Top 10 only | ~96% |

Reducing from 30 to 10 features caused no meaningful accuracy loss — confirming heavy redundancy from correlated size measurements.

### Biomarker Consensus

| Rank | Logistic Regression | Random Forest | SHAP |
|------|---------------------|---------------|------|
| 1 | texture_worst | perimeter_worst | perimeter_worst |
| 2 | radius_se | area_worst | area_worst |
| 3 | concavity_worst | radius_worst | concave_points_worst |
| 4 | concave_points_worst | concave_points_worst | radius_worst |
| 5 | area_worst | concavity_mean | concavity_worst |

**Key finding:** tumour size (`radius`, `perimeter`, `area`) and boundary irregularity (`concavity`, `concave points`) — specifically their *worst-case* measurements — are the dominant malignancy predictors across all three methods. This is consistent with established oncological knowledge.

---

## Techniques Covered

- Exploratory Data Analysis (EDA)
- Label Encoding & Feature Scaling
- Logistic Regression
- Random Forest
- Gini Feature Importance
- Coefficient-based Feature Ranking
- Feature Selection (top-k)
- SHAP (SHapley Additive exPlanations)
- Beeswarm Summary Plots
- Confusion Matrix Visualisation
- Cross-method Explainability Comparison

---

## Tech Stack

![Python](https://img.shields.io/badge/Python-3.10-blue?logo=python&logoColor=white)
![Scikit-Learn](https://img.shields.io/badge/Scikit--Learn-1.x-orange?logo=scikit-learn&logoColor=white)
![SHAP](https://img.shields.io/badge/SHAP-0.4x-blueviolet)
![Pandas](https://img.shields.io/badge/Pandas-2.x-150458?logo=pandas&logoColor=white)
![Seaborn](https://img.shields.io/badge/Seaborn-0.13-teal)

```
scikit-learn
shap
pandas
numpy
matplotlib
seaborn
```

---

## Key Takeaways

1. **SHAP and Random Forest converge** on the same top features, giving high confidence in the identified biomarkers without relying on any single method.
2. **Logistic Regression partially disagrees** — `texture_worst` ranks first under its linear lens, while ensemble methods deprioritise it. This illustrates why model choice affects interpretability.
3. **Dimensionality can be cut by 67%** (30 → 10 features) with no meaningful accuracy loss, thanks to the multicollinearity among size measurements.
4. **SHAP adds directionality** that plain importance scores lack: high perimeter and area values *increase* malignancy probability — a clinically actionable finding.


