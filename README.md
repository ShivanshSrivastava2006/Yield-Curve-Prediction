# Yield Curve Prediction using CIR Methods

## Overview

This project implements a **yield-curve reconstruction pipeline** using the **Cox-Ingersoll-Ross (CIR) framework**, a classical non-negative short-rate model for interest-rate modeling. The central challenge is to reconstruct short-end yields (6M, 9M, 1Y, 2Y) from a single observed 3-month (3M) yield.

**By:** 
> Finance Club, IIT Roorkee

## Project Motivation

The CIR model is attractive for interest-rate modeling because:
- Its **square-root volatility term** naturally prevents unrealistic negative rates under realistic conditions
- It provides a theoretically sound framework for term-structure modeling
- It balances tractability with realistic mean-reversion dynamics

However, this project addresses a practical challenge: **reconstructing a full yield curve from a single observed 3M yield**. The core insight is that a parameter set fitting the 3M dynamics may not accurately reconstruct longer-maturity yields. This project compares multiple calibration strategies to find the best approach.

## Problem Statement

### Input Data
- **Training Dataset**: 1,976 observations containing zero-coupon yields at maturities: 3M, 6M, 9M, 1Y, 2Y, 5Y, 10Y, 20Y, 30Y (from 2016-05-19 onwards)
- **Test Dataset**: 495 observations with ground truth yields at maturities: 3M, 6M, 9M, 1Y, 2Y (2024-04-29 onwards)

### Output Prediction
Given **only the 3M yield** at test time, predict:
- 6M yield
- 9M yield
- 1Y yield
- 2Y yield

### Strict Test-Time Constraint
**Only the test 3M yields may be used during prediction**. Ground-truth test curves are used exclusively after predictions are generated for model comparison.

## Datasets

### Raw Data Mapping
The raw data files use zero-coupon yield columns with leading spaces in headers. The mapping to standardized maturities is:

| Raw Column | Maturity |
|---|---|
| `ZC025YR` | 3M |
| `ZC050YR` | 6M |
| `ZC075YR` | 9M |
| `ZC100YR` | 1Y |
| `ZC200YR` | 2Y |
| `ZC500YR` | 5Y |
| `ZC1000YR` | 10Y |
| `ZC2000YR` | 20Y |
| `ZC3000YR` | 30Y |

### Training Data
- **Size**: 1,976 × 10 (Date + 9 maturities)
- **Period**: 2016-05-19 onwards
- **Missing Values**: 0% (complete data)
- **Contains**: Full yield curves for model calibration

### Test Data
- **Size**: 495 × 6 (Date + 5 maturities: 3M to 2Y)
- **Period**: 2024-04-29 onwards
- **Missing Values**: 0% (complete data)
- **Note**: Ground truth only up to 2Y; predictions must target 6M, 9M, 1Y, 2Y

### Test 3M Only
- **Size**: 495 × 2 (Date + 3M yield)
- **What's available at test time**: Only the 3M yield
- **Purpose**: Serves as the single input for yield curve reconstruction

## Methodology

The project compares **three calibration strategies**:

### 1. **Baseline Time-Series CIR (MLE)**
- **Approach**: Estimate CIR parameters using Maximum Likelihood Estimation (MLE) from the training 3M time-series dynamics
- **Assumption**: Parameters that fit observed 3M dynamics will generalize to the full curve
- **Limitation**: Optimizes for short-rate volatility, not necessarily for yield reconstruction accuracy

### 2. **Cross-Sectional Affine CIR**
- **Approach**: Calibrate parameters directly to minimize training yield-curve reconstruction error
- **Assumption**: Parameters that fit the training cross-section best will generalize
- **Advantage**: Explicitly targets curve reconstruction accuracy

### 3. **Simple CIR++ Mean Shift**
- **Approach**: Deterministic maturity-wise mean residual correction learned only from training data
- **Assumption**: Add a constant correction to each maturity based on training errors
- **Advantage**: Simple, interpretable, captures systematic biases

## Model Selection

The final model is selected by **out-of-sample aggregate R²** computed after all predictions are generated without using test ground truth. The R² threshold benchmark is **0.85**.

### Evaluation Metrics
- **R² (Coefficient of Determination)**: Measures goodness of fit
- **Per-maturity R²**: Evaluate performance at each maturity (6M, 9M, 1Y, 2Y)
- **Aggregate R²**: Overall model performance across all predictions

## CIR Model Framework

### Dynamics
The short rate $r_t$ follows:
$$dr_t = \kappa(\theta - r_t) dt + \sigma\sqrt{r_t} dW_t$$

Where:
- $\kappa$: Mean reversion speed
- $\theta$: Long-term mean
- $\sigma$: Volatility coefficient
- $W_t$: Wiener process

### Affine Term Structure
Bond prices in the CIR model have closed-form solutions:
$$P(t, T) = A(t, T) \exp(-B(t, T) r_t)$$

Where $A$ and $B$ are affine functions derived from the model parameters.

### Advantages
- Non-negative rates by construction (for reasonable parameters)
- Closed-form bond pricing formulas
- Empirically validated for interest-rate modeling
- Tractable for calibration

## Key Findings

1. **Challenge**: A single maturity (3M) contains insufficient information to perfectly reconstruct the entire curve
2. **Trade-off**: Time-series fit vs. cross-sectional fit may be in tension
3. **Solution**: Hybrid approaches combining parametric models with learning-based corrections perform best
4. **Practical Insight**: Mean-shift corrections capture important systematic patterns in yield-curve shape

## Repository Structure
