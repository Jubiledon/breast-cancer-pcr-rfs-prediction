# Breast Cancer PCR & RFS Prediction

## Overview

This project evaluates a range of ML models across two clinical prediction tasks:

- **PCR Classification** — predicting whether a tumour is eliminated before surgery
- **RFS Regression** — estimating the likelihood of cancer returning after treatment

The dataset contains 118 features (11 clinical, 107 radiomic) across 400 patients, sourced from the [ACRIN 6698/I-SPY 2 Trial](https://www.cancerimagingarchive.net/collection/acrin-6698/).

---

## Results

| Task | Best Model | Performance |
|------|-----------|-------------|
| PCR Classification | Logistic Regression (wrapper features) | Accuracy: 0.8225 ± 0.0496 |
| RFS Regression | RBF-kernel SVR | R² ≈ 0.02, MSE ≈ 714.55 |

PCR prediction proved more tractable, with logistic regression achieving strong generalisation once irrelevant features were removed. RFS prediction was substantially harder — the low R² across all models suggests the available features do not fully capture the biological factors driving relapse risk.

---

## Pipeline

### 1. Preprocessing

- Missing values (encoded as `999`) replaced with explicit `NaN` markers
- **Clinical features**: mode imputation or stratified random imputation (for `Gene` feature due to class imbalance)
- **PCR task**: KNN imputation for context-aware estimates
- **Outlier handling**: IQR-based Winsorisation on all continuous/radiomic features — values clipped to Q1 – 1.5×IQR and Q3 + 1.5×IQR thresholds

### 2. Feature Selection

Dimensionality reduced from 118 to ~20–25 features using a multi-method framework:

- **Filter methods**: ANOVA F-tests, f-regression, Mutual Information
- **Embedded methods**: LASSO, ElasticNet, Random Forest and ExtraTrees importance
- **Wrapper methods**: RFE with Random Forest, Sequential Forward Selection
- **Model-based**: Permutation importance on held-out data

Rankings were normalised and aggregated via a weighted voting scheme, with correlation filtering applied to remove collinear radiomic variables. `ER`, `HER2`, and `Gene` features were retained throughout due to established clinical relevance.

### 3. Models Evaluated

**Classification (PCR):**
- Logistic Regression
- Support Vector Classifier (SVC)
- Decision Tree
- MLP Classifier

**Regression (RFS):**
- Linear Regression
- Decision Tree Regressor
- Support Vector Regressor (RBF kernel, tuned via GridSearchCV)
- MLP Regressor

All models used a `StandardScaler → model` pipeline with 5-fold stratified cross-validation.

---

## Getting Started

### Requirements

```bash
pip install -r requirements.txt
```

Key dependencies:
- Python 3.8+
- scikit-learn
- pandas
- numpy
- matplotlib / seaborn

### Dataset

Download the I-SPY 2 dataset from the [Cancer Imaging Archive](https://www.cancerimagingarchive.net/collection/acrin-6698/) and place it in the `data/` directory.

### Running the Pipelines

```bash
# PCR Classification
python pcr_classification.py

# RFS Regression
python rfs_regression.py

---

## References

- I-SPY 2 Dataset: [ACRIN 6698 — The Cancer Imaging Archive](https://www.cancerimagingarchive.net/collection/acrin-6698/)
- Pyradiomics: [pyradiomics.readthedocs.io](https://pyradiomics.readthedocs.io/en/latest/)