# 🏔️ Optimized Hybrid Grey-Fourier Modeling for Precision Landslide Displacement Forecasting

> A real-time, self-adaptive landslide early warning system using Metabolic GM(1,1) with FFT residual correction and a multi-sensor ensemble Risk Index — applied to the Preonzo Landslide dataset.

---

## 👥 Team Members

| Name | Roll Number |
|------|-------------|
| Dhakshin N | CB.SC.U4AIE24313 |
| Rohith Ravi | CB.SC.U4AIE24350 |
| Sanjay Kumar S | CB.SC.U4AIE24354 |
| Akhilan | CB.SC.U4AIE24362 |

---

## 📌 Table of Contents

- [Project Overview](#-project-overview)
- [Dataset](#-dataset)
- [Project Architecture](#-project-architecture)
- [Methodology](#-methodology)
- [Model Parameters](#-model-parameters)
- [Risk Index](#-risk-index)
- [File Structure](#-file-structure)
- [How to Run](#-how-to-run)
- [Execution Time & Performance](#-execution-time--performance)
- [Results](#-results)
- [Key Design Decisions](#-key-design-decisions)

---

## 📖 Project Overview

This project implements a **hybrid predictive monitoring system** for landslide early warning. The core challenge is that landslide displacement data is noisy, non-stationary, and contains both long-term exponential trends and short-term periodic cycles driven by seasonal and atmospheric factors.

Our solution combines:
- **Metabolic GM(1,1)** — a sliding-window Grey Model that continuously re-estimates displacement dynamics from the 5 most recent observations, adapting in real time to evolving slope behaviour.
- **FFT Residual Correction** — a Fourier-based post-processor that identifies and recovers structured periodic patterns in the Grey Model's prediction errors, producing a final hybrid prediction that tracks both trend and periodicity.
- **Ensemble Risk Index** — a scalar alert metric computed independently for each sensor and averaged across all three, designed to trigger an "Advanced Warning" only when both prediction error and slope acceleration are simultaneously elevated.

The system is validated on the **Preonzo landslide (Switzerland)**, a well-documented slope failure event, using three independent extensometer sensors (E1, E2, E3).

---

## 📂 Dataset

- **Name:** Preonzo Landslide Dataset
- **Source:** Real sensor measurements from the Preonzo slope, Switzerland
- **Format:** `.xlsx` — columns `E1`, `E2`, `E3` represent displacement velocity readings (mm) from three independent extensometer sensors
- **Size:** ~37,000–38,500 data points per sensor
- **Known Event:** The actual Preonzo slope failure is recorded near time index **36,302–37,105** in the dataset

> **Note:** Update the dataset path in each `.mlx` file before running:
> ```matlab
> data = readtable("YOUR_PATH\Preonzo Landslide dataset.xlsx");
> ```

---

## 🏗️ Project Architecture

```
Preonzo Dataset (.xlsx)
        │
        ├──► E2 Sensor MLX ──► risk_values_2  ─┐
        ├──► E3 Sensor MLX ──► risk_values_3  ─┤──► Global Risk Monitoring MLX
        └──► E1 Sensor MLX ──► risk_values_1  ─┘
                  │                                       │
                  └── (also runs full pipeline) ──► Ensemble Average Risk
                                                          │
                                              ┌───────────┴───────────┐
                                         R(k) ≤ 1.75              R(k) > 1.75
                                           "Safe"             "Advanced Warning"
```

---

## 🔬 Methodology

### Step 1 — Data Preparation
- Raw sensor readings are stripped of NaN values and converted to absolute values (Grey Model requires non-negative input).
- **1-AGO (Accumulated Generating Operation)** is applied: `x⁽¹⁾ = cumsum(x⁽⁰⁾)` — transforms the noisy raw signal into a smooth monotonically increasing curve suitable for exponential fitting.

### Step 2 — Metabolic GM(1,1) with Ridge Regression
- A **sliding window of L = 5** points moves forward one step at a time.
- At every step `t`, the Grey Model is freshly re-fitted using only the 5 most recent observations — this is the **Metabolic (online) formulation**, ensuring the model continuously adapts to current slope dynamics.
- The **background value sequence** `z⁽¹⁾` is computed with weight `λ = 0.6` to reflect the Preonzo slope's moderately accelerating displacement pattern.
- Parameters `[a, b]` are estimated using **Ridge Regression** with `λ_r = 0.1`:
  ```
  θ = (BᵀB + λ_r · I)⁻¹ · BᵀY
  ```
- `a(t)` is the **development coefficient** — the instantaneous exponential growth rate. Strongly negative `a` is a direct pre-failure indicator.
- The one-step prediction is recovered via **IAGO (Inverse AGO)**:
  ```
  x̂⁽⁰⁾(t) = x̂⁽¹⁾(t) − x̂⁽¹⁾(t−1)
  ```
- **Statistical validation** (residual variance, covariance matrix, t-values for `a` and `b`) is computed at every window to confirm parameter significance.

### Step 3 — FFT Residual Correction
- Residuals `res = x⁽⁰⁾ − x̂⁽⁰⁾_meta` are computed after the full metabolic loop.
- **FFT** is applied to the full residual sequence to decompose it into frequency components.
- The **top K = 3 dominant frequencies** are retained (physically corresponding to the annual seasonal cycle, sub-annual harmonic, and short-term rainfall cycle of the Preonzo slope). All other frequencies are zeroed out.
- **Inverse FFT** reconstructs a smooth periodic correction signal, which is added back to the Grey Model prediction:
  ```
  x̂⁽⁰⁾_final = x̂⁽⁰⁾_meta + res_hat
  ```

### Step 4 — Risk Index & Warning Decision
- A scalar **Risk Index R(k)** is computed at every time step:
  ```
  R(k) = (|x⁽⁰⁾(k) − x̂⁽⁰⁾_final(k)| / x⁽⁰⁾(k)) × 100  +  |a(k)| × 5
  ```
- If `R(k) > RT = 1.75` → **"Advanced Warning"**; otherwise → **"Safe"**
- Risk values are capped at 20 for visualization stability only.
- The **Ensemble Global Risk** is the average across all three sensors:
  ```
  R_global(k) = (R_E1(k) + R_E2(k) + R_E3(k)) / 3
  ```

---

## ⚙️ Model Parameters

| Parameter | Value | Reason |
|-----------|-------|--------|
| `L` | 5 | Minimum window for GM(1,1); keeps model responsive to rapid changes |
| `λ` | 0.6 | Weighted toward current observation; suitable for moderately accelerating slope |
| `λ_r` | 0.1 | Minimum ridge penalty to stabilize the 2×2 system from L=5 without shrinking `a` |
| $\lambda_r$ | 0.1 | Minimum ridge penalty to stabilize the 2×2 system from L=5 without shrinking `a` |
| `K` | 3 | Maps to 3 physically real periodic drivers of Preonzo displacement |
| `ω₁` | 100 | Amplifies normalized error so 2% deviation crosses the warning threshold |
| `ω₂` | 5 | Scales acceleration as secondary contributor; prevents false warnings from normal seasonal acceleration alone |
| `RT` | 1.75 | Warning threshold; requires combined error + acceleration to contribute meaningfully above zero |

---

## 📊 Risk Index

The Risk Index equation:

```
R(k) = ( |x⁽⁰⁾(k) − x̂⁽⁰⁾_final(k)| / x⁽⁰⁾(k) ) × 100  +  |a(k)| × 5
```

| Term | Meaning |
|------|---------|
| `x⁽⁰⁾(k)` | Actual sensor displacement at time k |
| `x̂⁽⁰⁾_final(k)` | Hybrid Grey + FFT model prediction |
| `a(k)` | Development coefficient (acceleration proxy) from current window |
| `ω₁ = 100` | Error sensitivity weight |
| `ω₂ = 5` | Acceleration sensitivity weight |
| `RT = 1.75` | Warning threshold |

**Decision Logic:**
```
IF  R(k) > 1.75  →  "Advanced Warning"
ELSE             →  "Safe"
```

---

## 📁 File Structure

```
├── Real_data_coding_E1_sensor.mlx      # E1 sensor pipeline (run FIRST or standalone)
├── Real_data_Coding_E2_sensor.mlx      # E2 sensor pipeline (MUST run before Global)
├── Real_data_coding_E3_sensor.mlx      # E3 sensor pipeline (MUST run before Global)
├── Global_risk_monitoring_report.mlx   # Global ensemble risk + final combined plots
└── README.md
```

---

## ▶️ How to Run

> ⚠️ **Critical:** The Global Risk Monitoring file depends on workspace variables (`risk_values_2`, `risk_values_3`) that are only created by running the E2 and E3 sensor files first. Running the Global file without them will throw an undefined variable error.

### Correct Execution Order

**Step 1 — Run E2 Sensor file**
```
Open Real_data_Coding_E2_sensor.mlx → Run All
```
This creates `risk_values_2` in the MATLAB workspace.

**Step 2 — Run E3 Sensor file**
```
Open Real_data_coding_E3_sensor.mlx → Run All
```
This creates `risk_values_3` in the MATLAB workspace.

**Step 3 — Run Global Risk Monitoring file**
```
Open Global_risk_monitoring_report.mlx → Run All
```
This file:
- Runs the full E1 pipeline internally (creates `risk_values_1`)
- Reads `risk_values_2` and `risk_values_3` from the shared workspace
- Computes the ensemble average: `risk_values = (risk_values_1 + risk_values_2 + risk_values_3) / 3`
- Generates all final plots

> The E1 standalone file (`Real_data_coding_E1_sensor.mlx`) can be run independently at any time for E1-only analysis. It does not affect the Global pipeline.

### Update Dataset Path
Before running any file, update line 1 in each `.mlx`:
```matlab
data = readtable("YOUR_FULL_PATH\Preonzo Landslide dataset.xlsx");
```

---

## ⏱️ Execution Time & Performance

Execution time is measured using MATLAB's built-in `tic` / `toc` commands, placed at the start and end of each `.mlx` file:

```matlab
tic          % placed at the very start of the script
% ... all code ...
toc          % placed at the very end — prints elapsed time automatically
```

`toc` outputs the elapsed wall-clock time in seconds to the MATLAB Command Window, for example:
```
Elapsed time is 2.323202 seconds.
```

### Measured Execution Times

| File | Sensor | Recorded Time |
|------|--------|---------------|
| `Real_data_Coding_E2_sensor.mlx` | E2 | **2.32 seconds** |
| `Real_data_coding_E3_sensor.mlx` | E3 | **3.24 seconds** |
| `Global_risk_monitoring_report.mlx` | E1 with Global | *3.24 seconds* |
| `Total Time Taken` | E2+E3+E1 with Global | *9.09 seconds* |

### Platform & Hardware

| Specification | Details |
|---------------|---------|
| **Platform** | Personal Laptop |
| **Processor** | CPU *Intel Core i7* |
| **RAM** | *16GB* |
| **Operating System** | Windows 11 |
| **Software** | MATLAB *R2024a* |

> **Note on execution time variability:** The dominant cost is the metabolic loop (~37,000 iterations of ridge regression per sensor). Execution time scales linearly with dataset length. On the same hardware, re-runs may vary by ±0.3s depending on background CPU load. GPU is not utilized — all operations are CPU-bound matrix algebra.


## 📈 Results

- The hybrid Metabolic GM(1,1) + FFT model successfully tracks both the long-term exponential displacement trend and short-term periodic cycles across all three sensors.
- The Global Risk Index remains below the threshold (RT = 1.75) during the entire stable creep phase (~37,000 time steps).
- The system triggers **"Advanced Warning"** at time index ~36,302–37,105, which corresponds exactly to the documented Preonzo slope failure event.
- All three sensors simultaneously breach the threshold at the failure event — confirming the warning is a true positive and not a single-sensor false alarm.

---

## 💡 Key Design Decisions

- **Why Metabolic (online) and not classical GM(1,1)?** Classical GM(1,1) fits once on all historical data — it cannot adapt to the changing dynamics of an accelerating slope. The metabolic formulation re-fits at every step, capturing regime changes in real time.
- **Why Ridge Regression?** The 5-point window produces a near-singular 2×2 system during stable (flat displacement) periods. Ridge regression (`λ_r = 0.1`) guarantees invertibility without meaningfully shrinking the development coefficient `a`.
- **Why K=3 in FFT?** The Preonzo slope has three physically documented periodic displacement drivers: annual seasonal cycle, sub-annual harmonic, and short-term rainfall cycle. K=3 captures all three without absorbing noise.
- **Why ensemble average across sensors?** A single sensor can produce false spikes from instrument noise or local micro-movements. Averaging across E1, E2, and E3 requires the entire slope to show elevated risk before a warning is issued — dramatically reducing false alarm probability.
- **Why RT = 1.75?** The risk formula has no floor constant (unlike the +1 baseline in earlier versions). The threshold was set to 1.75 to require a meaningful combined contribution from both error and acceleration before triggering — avoiding warnings from either signal alone.
