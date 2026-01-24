# Optimized Hybrid Grey-Fourier Modeling for Precision Landslide Displacement Forecasting
## A Comparative Study of Classical and Metabolic GM(1,1)

This project implements and compares two approaches for forecasting landslide displacement: **Classical GM(1,1) + Fourier** and **Metabolic GM(1,1) + Fourier**. By integrating Grey System Theory with Fast Fourier Transform (FFT) residual error correction, the models are designed to handle the "sparse" and "noisy" nature of geological sensor data.

---

## Table of Contents
1. [What Are We Actually Doing?](#what-are-we-actually-doing-in-our-project)
2. [Why GM(1,1)?](#why-do-we-use-gm11-model-for-our-project)
3. [Classical GM(1,1) + Fourier vs Metabolic GM + Fourier](#classical-gm11--fourier-vs-metabolic-gm--fourier)
4. [Part A: Classical GM(1,1) + Fourier](#part-a-classical-gm11--fourier)
    - [Derivation of Parameter Estimation](#1-derivation-of-parameter-estimation)
    - [Whitenization Equation](#2-whitenization-equation)
    - [Fast Fourier Transform (FFT)](#fast-fourier-transform-fft)
5. [Part B: Metabolic GM(1,1) + Fourier](#part-b-metabolic-gm11--fourier)
6. [Part C: Accuracy Comparison](#part-c-accuracy-comparison)
7. [Part D: Plotting](#part-d-plotting)
8. [Part E: Future Updates](#part-e-future-updates)

---

## What Are We Actually Doing in Our Project?

### 1. Training on "Random" Data
**The Problem:** Normal models (like Deep Learning or standard Regression) usually require thousands of data points to understand a trend and often fail when data is "noisy" or "meagre".

**The Grey Solution:** By using the **1-AGO (1-order Accumulated Generating Operation)**, we transform that random, "jagged" sensor data into a smooth exponential curve `x^(1)`. This allows the model to "see" the underlying physics of the landslide even if the sensors are fluctuating.

### 2. The Training and Correction Phase
- **Parameter Estimation:** We use the historical data points to find the coefficients `a` (velocity) and `b` (driving force).
- **Error Learning:** We don't just ignore the mistakes the Grey Model makes; we use **Fourier analysis** to "learn" the pattern of those errors (the residuals).
- **Hybrid Prediction:** We add that learned error back to the Grey prediction, making the model "ready" and highly tuned to the specific behavior of that slope.

### 3. Predicting the "Unseen"
- **The Goal:** Once the model is "ready," we can feed it a time step that has not happened yet (e.g., Day `n+1`).
- **The Verdict:** The model will output a predicted displacement. If that predicted value exceeds a safety threshold (like 10mm of movement in an hour), we can definitively say, "A landslide is likely occurring or imminent".

---

## Why Do We Use GM(1,1) Model for Our Project?

- **The first "1":** This signifies that the model uses a **first-order differential equation** to describe the displacement trend. It means the model assumes the rate of change in landslide movement can be captured by a single derivative.
- **The second "1":** This indicates that the model is **univariate**, meaning it focuses on a single variable—in this case, the surface displacement (mm) over time.

---

## Classical GM(1,1) + Fourier vs Metabolic GM + Fourier

Here is the setup for the data generation used to test the models:

```matlab
clc; clear; close all;
rng(1);

% -------- 1. Generate Data --------
n = 10;
k = 1:n;
```

We simulate the original data using the equation:

```
x^(0)(k) = x0 + Trend + Seasonal + Noise
```

Code implementation:
```matlab
% x0 = 5 + 0.35*k + 1.2*sin(2*pi*k/8) + 0.3*randn(1,n);
x0 = 5 + 0.35*k + 1.2*sin(2*pi*k/8) + 0.3*randn(1,n);

% Positivity requirement (absolute values if negative found)
x0 = abs(x0);
```

- **The Baseline (5):** Initial state or steady-state elevation.
- **The Growth Trend (0.35k):** Creep deformation/gravity.
- **The Seasonal/Periodic Component (1.2sin(...)):** Seasonal rainfall or snowmelt cycles.
- **The Stochastic Noise (0.3randn):** Sensor errors and environmental jitter.

---

## Part A: Classical GM(1,1) + Fourier

### The Accumulated Generating Operation (AGO)
To find the value of `x^(1)`:

```
x^(1)(k) = Σ x^(0)(i)   (sum from i=1 to k)
```

Where `x^(1)` is the **1-order Accumulated Generating Operation Sequence**.

```matlab
x1 = cumsum(x0);
```

### Background Value Sequence
```
z^(1)(k) = λ * x^(1)(k) + (1-λ) * x^(1)(k-1)
```

Where `λ` (lambda) is usually 0.5, but can be adjusted. `λ` controls how the background value balances current and previous accumulated states.

- **`λ ≈ 0.5`:** Standard/Linear Growth.
- **`λ < 0.5`:** High-growth (rapidly accelerating).
- **`λ > 0.5`:** Decelerating or approaching a plateau.

```matlab
% Background value
lambda = 0.6;
z1 = lambda*x1(2:end) + (1-lambda)*x1(1:end-1);
```

### 1. Derivation of Parameter Estimation
We use **Regularized Least Squares (Ridge Regression)** to find parameters `a` and `b` for the discrete Grey equation:

```
x^(0)(k) + a * z^(1)(k) = b
```

In matrix form `Y = Bθ`:
- `B = [-z^(1), 1]` (Data Matrix)
- `Y = [x^(0)(2), ..., x^(0)(n)]^T` (Constant Vector)

```matlab
B = [-z1', ones(n-1,1)];
Y = x0(2:end)';
```

**Step 1:** Minimize the Cost Function `J(θ)` with respect to `θ`, including a regularization term `λ_r` to prevent overfitting:

```
J(θ) = (Y - Bθ)^T (Y - Bθ) + λ_r * θ^T θ
```

**Why minimize w.r.t `θ`?** To find optimal parameters that make predicted movement close to actual measurements while keeping the model simple.

**The role of `a` and `b`:**
- **`a` (Development Coefficient):** Determines growth/decay trend. Small negative value = steady creep; change in `a` = potential acceleration.
- **`b` (Grey Driving Coefficient):** Represents external influence/driving force (e.g., rainfall).

**Step 2:** Expand the norms (using matrix properties).

**Step 3:** Differentiate with respect to `θ`:

```
∂J/∂θ = -2B^T Y + 2B^T Bθ + 2λ_r θ = 0
```

**Step 4:** Solve for `θ`:

```
(B^T B + λ_r I)θ = B^T Y
θ = (B^T B + λ_r I)^(-1) B^T Y
```

```matlab
% Parameter estimation
lambda_r = 0.1;
theta = (B'*B + lambda_r*eye(2)) \ (B'*Y);
a = theta(1);
b = theta(2);
```

### 2. Whitenization Equation
The whitenization equation converts the discrete model into a continuous ODE:

```
dx^(1)/dt + a * x^(1) = b
```

**Solution (Time Response Function):**

```
x^(1)_hat(k) = (x^(0)(1) - b/a) * e^(-a(k-1)) + b/a
```

```matlab
% Time response
x1_hat = (x1(1) - b/a)*exp(-a*(k-1)) + b/a;
```

**Restore `x^(0)_hat` (Inverse AGO):**

```
x^(0)_hat(k) = x^(1)_hat(k) - x^(1)_hat(k-1)
```

```matlab
% Restore x^(0)
x0_hat_gm = zeros(1,n);
x0_hat_gm(1) = x0(1); % Initial condition
for i = 2:n
    x0_hat_gm(i) = x1_hat(i) - x1_hat(i-1);
end
```

**Residuals:**
```
ε(k) = x^(0)(k) - x^(0)_hat(k)
```

```matlab
res_gm = x0 - x0_hat_gm
```

### Fast Fourier Transform (FFT)
We use FFT to identify and model structured periodic patterns in the residual errors that GM(1,1) cannot capture.

**Why FFT instead of DFT?**
The `fft(x)` function returns the exact same values as the DFT formula but is much faster (`O(N log N)` vs `O(N^2)`).

**The Equations of Discrete Fourier Transform:**

```
E(k) = Σ e(n) * e^(-i * (2π/N) * kn)   (sum from n=0 to N-1)
```

**Implementation:**
1.  **Compute FFT of residuals:**
    ```matlab
    E = fft(res_gm)
    magE = abs(E) % Magnitude Spectrum
    ```
2.  **Identify Dominant Frequencies:** Truncate to keep top `K` frequencies.
    ```matlab
    K = 3;
    [~, idx] = sort(magE(2:floor(n/2)), 'descend');
    idx = idx(1:K) + 1
    ```
3.  **Filter Spectrum:** Keep only dominant frequencies and their conjugates.
    ```matlab
    E_filt = zeros(size(E));
    E_filt(1) = E(1); % DC Component
    E_filt(idx) = E(idx);
    E_filt(n-idx+2) = E(n-idx+2) % Conjugate Symmetry
    ```
4.  **Reconstruct Residuals (IDFT):**
    ```matlab
    res_hat_gm = real(ifft(E_filt))
    ```
5.  **Final Prediction:**
    ```matlab
    x0_hat_gm_final = x0_hat_gm + res_hat_gm;
    err_gm = x0 - x0_hat_gm_final
    ```

---

## Part B: Metabolic GM(1,1) + Fourier

### What Does the "Sliding Window" Mean?
The sliding window `L` represents the most recent `L` observations used to build the model at time `t`. By discarding older data, the metabolic model adapts to time-varying system behavior.

```matlab
L = 5;   % sliding window size
```

### Window Prediction
For each step, we use data from `t-L` to `t-1` to predict `t`.

```matlab
x0_hat_meta = zeros(1,n);
x0_hat_meta(1:L) = x0(1:L); % Use actual data for initial window

for t = L+1:n
    x0_win = x0(t-L:t-1);
    x1_win = cumsum(x0_win);

    z1 = lambda*x1_win(2:end) + (1-lambda)*x1_win(1:end-1);

    B = [-z1', ones(L-1,1)];
    Y = x0_win(2:end)';

    theta = (B'*B + lambda_r*eye(2)) \ (B'*Y);
    a = theta(1);
    b = theta(2);

    % Predict next point (time L in the local window, which is global time t)
    x1_hat = (x1_win(1) - b/a)*exp(-a*(0:L-1)) + b/a;
    x0_hat_meta(t) = x1_hat(end) - x1_hat(end-1);
end
```

**Fourier Correction for Metabolic Model:**
Similar to Part A, we compute residuals for the metabolic predictions and apply FFT correction.

```matlab
x0 = x0(:)';
x0_hat_meta = x0_hat_meta(:)';
res_meta = x0 - x0_hat_meta;

% FFT ... (same process as above)
% ...
x0_hat_meta_final = x0_hat_meta + res_hat_meta;
err_meta = x0 - x0_hat_meta_final;
```

---

## Part C: Accuracy Comparison

We use three metrics to compare the models:

1.  **RMSE (Root Mean Square Error):** Sensitive to large errors.
    ```matlab
    RMSE_gm   = sqrt(mean(err_gm.^2));
    RMSE_meta = sqrt(mean(err_meta.^2));
    ```
2.  **MAE (Mean Absolute Error):** Reflects stable performance.
    ```matlab
    MAE_gm   = mean(abs(err_gm));
    MAE_meta = mean(abs(err_meta));
    ```
3.  **MAPE (Mean Absolute Percentage Error):** Scale-independent.
    ```matlab
    MAPE_gm   = mean(abs(err_gm ./ x0)) * 100;
    MAPE_meta = mean(abs(err_meta ./ x0)) * 100;
    ```

**Why might Classical be better at MAPE?**
Classical model focuses on the global trend. If displacement is large at the end, a fixed error is a smaller percentage. Metabolic model might accumulate small percentage errors at every step.

---

## Part D: Plotting

The project includes visualization of:
1.  **Prediction Comparison:** Actual vs GM+Fourier vs Metabolic GM+Fourier.
2.  **Prediction Errors:** Residuals over time.
3.  **Accuracy Metrics:** Bar chart of RMSE, MAE, MAPE.

---

## Part E: Future Updates

1.  **Multi-Variable Integration (GM(1,N)):**
    -   *Current:* Univariate (displacement only).
    -   *Proposed:* Input rainfall, groundwater pressure.
    -   *Benefit:* Understands the "cause" (e.g., rain) not just the movement.

2.  **Adaptive Fourier Truncation:**
    -   *Current:* Fixed `K=3` frequencies.
    -   *Proposed:* Energy-based selection (e.g., 95% variance).
    -   *Benefit:* Adapts to new noise patterns (e.g., soil cracking vibrations).

3.  **Dynamic Optimization of Horizontal Control Parameter λ:**
    -   *Current:* Fixed `λ = 0.6`.
    -   *Proposed:* Genetic Algorithm or PSO to find optimal `λ` per window.
    -   *Benefit:* Adapts "curvature" during rapid acceleration phases.
