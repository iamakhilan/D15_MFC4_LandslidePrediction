# Hybrid Grey-Fourier Landslide Forecasting

Done by:
- Dhakshin N — CB.SC.U4AIE24313
- Rohith Ravi — CB.SC.U4AIE24350
- Sanjay Kumar S — CB.SC.U4AIE24354
- Akhilan S — CB.SC.U4AIE24362

---

## What is this project about?

We are trying to predict landslide movement before it actually happens. To do that, we use sensor data from a real slope in Switzerland called Preonzo. The sensors measure how fast the ground is moving, in millimeters per day.

The problem is, this sensor data is very noisy and messy. Normal prediction models don't work well with noisy data. So we use a special model called GM(1,1) which was made for exactly this kind of situation — small, uncertain data.

But GM(1,1) alone still makes some errors. So on top of that, we use FFT (a math technique) to find the pattern in those errors and correct them. This combination is what we call the Hybrid Grey-Fourier model.

We also built a risk index that tells us when a landslide is about to happen, using the prediction results.

---

## Dataset

We used a real dataset from the Preonzo landslide site in Switzerland. It has daily extensometer (ground movement) readings starting from May 2002.

- File: `Preonzo Landslide dataset.xlsx`
- Sheet: `Extensometer data`
- Total rows: around 38,906
- Columns we used: E1, E2, E3 (displacement velocity in mm/day)
- The actual landslide happened around row 36,302 to 37,105 in the E1 sensor data

Note: E3 has some missing values in the beginning. We remove those before doing anything.

---

## How does our model work? (simple explanation)

**Step 1 — Smooth the noisy data**

Raw sensor data jumps up and down randomly. We add up all the values one by one (called AGO — Accumulated Generating Operation). This turns a noisy signal into a smooth curve that is easier to work with.

**Step 2 — Fit the Grey Model**

We fit GM(1,1) on this smooth curve. This model finds two numbers, `a` and `b`, that describe how the ground is moving. `a` tells us how fast the movement is changing and `b` is the constant driving force (like continuous rainfall pushing the soil).

We use Ridge Regression to find `a` and `b`. This just means we add a small penalty to stop the model from overfitting (memorizing noise instead of learning the real pattern).

**Step 3 — Correct the errors using FFT**

Even after GM(1,1), there are still some leftover errors. These errors are not random — they repeat in a pattern because landslides are affected by seasons, rainfall cycles and so on. We use FFT to find the top 3 repeating patterns in these errors and correct the prediction.

**Step 4 — Two versions of the model**

We built two versions and compared them:

- Classical GM(1,1) + Fourier: fits the model once using all past data
- Metabolic GM(1,1) + Fourier: uses only the last 5 data points and updates itself at every step. This is more accurate because it adjusts to recent changes in the ground behavior.

**Step 5 — Risk index**

We calculate a risk number at each time step. If this number goes above 1.75, we raise an Advanced Warning. Otherwise it shows Safe. We do this for all 3 sensors and average them to get a global risk value.

---

## The math (just the key parts)

AGO — smoothing the data:

x1(k) = x0(1) + x0(2) + ... + x0(k)

Grey model prediction:

x1_hat(k) = (x0(1) - b/a) × e^(-a(k-1)) + b/a

We get back the original scale by taking the difference between consecutive predicted values:

x0_hat(k) = x1_hat(k) - x1_hat(k-1)

Finding a and b using Ridge Regression:

theta = (B'B + lambda_r × I) \ (B'Y)   where lambda_r = 0.1

Risk index formula:

R(k) = (|actual - predicted| / actual × 100) + (|a(k)| × 5)

If R(k) > 1.75 → Advanced Warning, else → Safe

Global risk (average of all 3 sensors):

risk_global = (risk_E1 + risk_E2 + risk_E3) / 3

---

## Why did we pick these values?

- Window size L = 5: L=5 means the model only looks at the last 5 readings at each step. This is small enough to react quickly to changes in the slope but large enough to still have enough points to estimate a and b reliably.
- lambda = 0.6: The Preonzo data shows moderate acceleration. Setting lambda above 0.5 puts more weight on the current state, which fits this behavior better.
- lambda_r = 0.1: Small enough to not over-restrict the model, but enough to keep the estimation stable.
- K = 3 (FFT components): For the Preonzo sensor data, there are three known physical cycles — yearly seasonal rainfall, half-yearly, and shorter atmospheric cycles. So we keep the top 3 frequency components from FFT to capture all of them.
- Risk threshold RT = 1.75: We looked at where the risk values spike during the known failure event and chose 1.75 as the cutoff that catches the warning without too many false alarms.

---

## What each graph shows

**Graph 1 — AGO plot (first 50 points)**

Shows x0 (original noisy data) and x1 (the accumulated smooth version) together. You can see how messy the raw data is and how AGO makes it smooth. If x1 is not going steadily upward, something is wrong with the data.

**Graph 2 — Error before and after FFT**

Shows the leftover error from GM(1,1) before FFT correction (the messy line) and the FFT-reconstructed version (the periodic line). If FFT is working correctly, the reconstructed line should match the repeating pattern of the original error — meaning it picked up the seasonal cycles hiding in the residuals.

**Graph 3 — Original vs Predicted (full data)**

Shows the actual sensor reading and our model prediction on the same graph. They should follow each other closely across all 38,000+ points.

**Graph 4 — Displacement and Risk plot (full dataset)**

Left axis: actual and predicted displacement. Right axis: blue bars showing the risk value at each point. The orange dashed line is the threshold at 1.75. The bars should mostly stay below 1.75 and only spike near the actual failure event.

**Graph 5 — Zoomed plot near the failure event**

Same as Graph 4 but zoomed into the time range where the landslide actually happened. This is the most important graph to check. If the risk bars clearly cross 1.75 during this window, our early warning is working correctly.

**Graph 6 — Global risk monitoring (two subplots)**

Top subplot: E1 displacement with the global risk line (average of all 3 sensors). Bottom subplot: all three individual sensor risks plus the global average. The global risk is more stable than any single sensor because averaging reduces noise.

**Graph 7 — Zoomed global risk at the failure event**

Same zoomed window as Graph 5 but using global risk. This confirms that all three sensors agree on the warning, not just one sensor acting up.

**Graph 8 — Soil strain experiment (Updated Report)**

A separate test using soil strain data from a different location. Three lines: actual (black solid), GM+Fourier prediction (red dashed), Metabolic GM prediction (blue dashed). The X axis goes up to about 38,000 time steps and the Y axis shows soil strain going from 0 to nearly 900. For most of the range all three lines follow each other very closely, which means both models are tracking well. Near the end (around index 35,000 onwards) there is a sharp upward spike — this is where the failure happens. The lines start to separate slightly here, and this is exactly where the Metabolic model performs better because it is updating itself using recent data while the Classical GM is still relying on old patterns. The RMSE difference (19.47 vs 3.01) comes mainly from this end region.


![Landslide Prediction using Soil Strain](d:\mfc-4 update\Result.png)
.
---

## How we measure accuracy

We use three numbers to check how good the predictions are:

RMSE: penalizes big errors more heavily. Lower is better.

MAE: average of all errors without any extra penalty. Lower is better.

MAPE: error shown as a percentage. Useful when comparing across different sensors.

### Our results (from Updated Report — soil strain experiment)

| Model | RMSE | MAE |
|-------|------|-----|
| Classical GM(1,1) + Fourier | 19.4761 | 8.5648 |
| Metabolic GM(1,1) + Fourier | 3.0129 | 1.6074 |

The Metabolic model is clearly better here. Its RMSE dropped from 19.47 to 3.01 and MAE dropped from 8.56 to 1.61. This shows that updating the model using only recent data (sliding window of 5) works much better than fitting once on all the data, especially when the ground behavior is changing over time.





---

## How to run the code

You have to run the files in this order. The global file needs results from E2 and E3 files first.

Step 1 — Run `Real_data_Coding_E2_sensor.mlx`
Processes E2 sensor. Saves a variable called `risk_values_2` in the MATLAB workspace.

Step 2 — Run `Real_data_coding_E3_sensor.mlx`
Processes E3 sensor. Saves `risk_values_3`.

Step 3 — Run `Global_risk_monitoring_report.mlx`
Processes E1, then picks up `risk_values_2` and `risk_values_3` from the workspace, combines all three, and generates the final global risk plots.

`Updated_Report.mlx` can be run on its own at any time. It does not depend on the other files.

---

## How long it takes

| File | Time |
|------|------|
| E2 sensor file | 2.32 seconds |
| E3 sensor file | 3.24 seconds |
| Global monitoring file | 3.53 seconds |
| Total | 9.09 seconds |

Measured using tic/toc in MATLAB. Time can vary a little depending on the computer.

---

## Tools used

- MATLAB R2024a
- FFT (built into MATLAB, no extra toolbox needed)
- Grey System Theory — GM(1,1) model
- Ridge Regression for finding model parameters

Tested on Windows 11, Intel Core i7, 16GB RAM. No GPU needed.

---

## What can be improved later

Right now we only use one sensor variable at a time. It would be better to feed in multiple inputs like rainfall and temperature alongside displacement using GM(1,N).

We also fixed K=3 for FFT. It would be smarter to let the code automatically pick the right number of frequency components based on how the data looks.

The lambda value is fixed at 0.6 for all time steps. In reality, the ground sometimes speeds up and sometimes slows down, so lambda should ideally change based on what the recent data is doing.

---

## Reference

Liu, S., Yang, Y., & Forrest, J. (2017). Grey Data Analysis: Methods, Models and Applications. Springer.
