# Multivariate Analysis of Air Quality in the Monterrey Metropolitan Area (Obispado Station)

A multivariate statistics project that studies the behavior of fine particulate matter (PM2.5) and its relationship with other pollutants and meteorological variables, using regression, hierarchical clustering, and Principal Component Analysis (PCA) on monthly air quality data from the Obispado monitoring station.

---

## Overview

This project applies a full multivariate workflow to ten years of monthly air quality measurements (2006–2015) from the Obispado station in the Monterrey Metropolitan Area. The goal is to understand how PM2.5 relates to other pollutants (NO, NO2, NOX, O3, PM10, CO) and meteorological factors (pressure, relative humidity, solar radiation, temperature), and to identify the underlying structure of the data.

The analysis combines four complementary techniques: missing-value imputation, multiple linear regression with formal diagnostics, agglomerative hierarchical clustering of variables, and PCA (computed both with scikit-learn and "by hand" via eigendecomposition). Together they describe which variables drive most of the variability, how the variables group by similarity, and how well a linear model can explain PM2.5.

---

## Problem

Fine particulate matter (PM2.5) is one of the most health-relevant air pollutants, and the Monterrey Metropolitan Area regularly records elevated concentrations. However, PM2.5 does not behave in isolation: it interacts with other pollutants emitted from common sources and with meteorological conditions that modulate dispersion and photochemistry.

The practical problem is twofold:

1. The PM2.5 series contains missing months that must be reconstructed before any multivariate technique can be applied.
2. The eleven variables are strongly intercorrelated and measured on very different scales (e.g., atmospheric pressure around 710 hPa vs. CO around 1 ppm), which makes it hard to identify the true drivers of variability and the natural groupings among variables without careful preprocessing.

---

## Objectives

- Reconstruct the missing PM2.5 values so the dataset is complete and usable.
- Quantify the linear relationship between PM2.5 and the remaining variables, and evaluate whether a linear model is adequate.
- Detect and diagnose multicollinearity among predictors.
- Discover the natural grouping of variables based on their correlation structure.
- Reduce dimensionality with PCA and determine which variables contribute most to the main axes of variation.
- Produce reproducible, internally consistent results where every conclusion is backed by the corresponding code.

---

## Methods / Models

| Stage | Technique |
|---|---|
| Imputation | Holt–Winters Exponential Smoothing (additive trend + seasonality, period = 12) |
| Exploratory analysis | Correlation heatmap, pair plots, time-series plots |
| Regression | Multiple Linear Regression |
| Regression diagnostics | R², MSE, residual plots, Q–Q plots, Breusch–Pagan test, VIF |
| Variable clustering | Agglomerative hierarchical clustering (correlation distance, average linkage) + dendrogram |
| Observation clustering | K-Means with elbow method and silhouette score |
| Dimensionality reduction | PCA, standardized — both `sklearn.decomposition.PCA` and a manual eigendecomposition of the correlation matrix |

---

## Key Technical Features

- **Standardized PCA (correlation-based).** All variables are scaled to mean 0 / standard deviation 1 before PCA, so components reflect the correlation structure rather than the raw measurement scale. This prevents high-variance variables (such as PM10) from dominating the components purely because of their units.
- **Two PCA implementations.** PCA is computed both with scikit-learn and manually via eigendecomposition of the correlation matrix, with a trace-vs-eigenvalue-sum check confirming the calculation is correct.
- **Robust eigenvalue ordering.** Eigenvalues and eigenvectors are sorted explicitly with `np.argsort`, so the principal components are always ordered by variance regardless of the order returned by the solver.
- **Formal regression diagnostics.** The model is not just fitted but validated: residual and Q–Q plots, the Breusch–Pagan test for heteroscedasticity, and Variance Inflation Factors (VIF) to expose multicollinearity.
- **Two clustering perspectives.** Hierarchical clustering groups the *variables* (by correlation distance), while K-Means groups the *observations* (months); the notebook keeps these two perspectives clearly separated.
- **Time-series imputation.** Missing PM2.5 values are filled with seasonal exponential smoothing rather than a constant or mean, preserving the seasonal pattern of the series.

---

## Results / Key Findings

- **Linear model is limited.** The multiple linear regression for PM2.5 yields R² ≈ 0.24 and MSE ≈ 14.25, indicating the relationship is not well captured by a purely linear model and that additional factors are likely involved.
- **No heteroscedasticity.** The Breusch–Pagan test (p ≈ 0.54) shows no evidence of heteroscedasticity in the residuals.
- **Severe multicollinearity.** VIF values for NO, NO2, and NOX are extremely high (≈880, ≈1500, ≈3475), which is expected since NOX is essentially the sum of NO and NO2; this destabilizes the regression coefficients.
- **Variable grouping (dendrogram).** Two main branches emerge: one with O3, SR, and TOUT (radiation and temperature drive ozone), and another with the combustion pollutants NO, NO2, NOX, PM10, and PM2.5, to which CO attaches; RH and PRS are peripheral and weakly correlated with the rest.
- **PCA (standardized).** The first component explains ≈38.7% of the variance, the second ≈17.5% (≈56% combined), and the first three ≈69.5%. The largest contributors to the first component are TOUT, NOX, NO2, and SR — not PM10, confirming that PM10's apparent dominance disappears once the data are properly standardized.

---


## Future Work

- **Address multicollinearity.** Drop or combine redundant predictors (e.g., keep NOX instead of NO + NO2, or use the principal components as regression inputs) to stabilize the model.
- **Non-linear models.** Try regression trees, random forests, or gradient boosting to capture the non-linear behavior the linear model misses.
- **Lag and seasonal features.** Add lagged pollutant values, month/season indicators, and weather interactions to better model PM2.5 dynamics.
- **Higher temporal resolution.** Repeat the analysis with daily or hourly data to capture short-term episodes hidden by monthly averaging.
- **Multiple stations.** Extend the study to other monitoring stations in the metropolitan area to assess spatial variability.
- **Validation.** Add cross-validation and a held-out test set to report out-of-sample performance rather than in-sample fit.
