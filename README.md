# 🌋 Hybrid Grey–Fourier Landslide Forecasting

A MATLAB-based project that predicts landslide displacement using **Grey System Theory** and **Fourier error correction**, designed for accurate forecasting from small and noisy sensor data.

---

## ✨ Overview
Landslide prediction is challenging because real-world sensor data is **noisy**, **short**, and often **nonlinear**.  
This project combines:

- **Grey System Theory (GM(1,1))** → strong with limited data  
- **Fourier Transform (FFT)** → corrects periodic residual errors  

✅ Result: **better accuracy + early warning capability**

---

## 📂 Dataset

- **Name:** Preonzo Landslide Dataset  
- **Source:** Real sensor measurements from the Preonzo slope, Switzerland  
- **Format:** `.xlsx`  
- **Columns:**  
  - **E1, E2, E3** → displacement velocity (mm)  
- **Size:** ~37,000–38,500 data points per sensor  
- **Known Event:** landslide failure around **index 36,302–37,105**

---

## 🧠 Key Concepts (Beginner‑Friendly)

### GM(1,1)
A Grey Model for forecasting when data is limited or noisy.

$$ \frac{dx^{(1)}}{dt} + ax^{(1)} = b $$

### AGO (Accumulated Generating Operation)
Smooths noisy data.

$$ x^{(1)}(k) = \sum_{i=1}^{k} x^{(0)}(i) $$

### Error Metrics
Used to compare model accuracy.

$$ RMSE = \sqrt{\frac{1}{n} \sum (y_i - \hat{y}_i)^2} $$  
$$ MAE = \frac{1}{n} \sum |y_i - \hat{y}_i| $$  
$$ MAPE = \frac{100}{n} \sum \left|\frac{y_i - \hat{y}_i}{y_i}\right| $$

---

## 🛠️ Methodology

1. Read sensor data  
2. Apply GM(1,1)  
3. Compute residuals  
4. Apply FFT correction  
5. Generate improved predictions  
6. Forecast future displacement  

---

## 🔬 Model Variants

- **Classical GM(1,1) + Fourier**  
- **Metabolic GM(1,1) + Fourier (sliding window)**  

**Metabolic GM(1,1)** adapts over time and gives higher accuracy.

---

## 🚨 Risk Index (Early Warning)

$$
R(k) = \left( \frac{\left|x^{(0)}(k) - \hat{x}^{(0)}_{\text{final}}(k)\right|}{x^{(0)}(k)} \right)\times 100 + |a(k)| \times 5
$$

### 🔎 Meaning of Terms

| Symbol | Meaning |
|--------|---------|
| $x^{(0)}(k)$ | actual displacement |
| $\hat{x}^{(0)}_{\text{final}}(k)$ | predicted value |
| $a(k)$ | development coefficient |
| $\omega_1 = 100$ | error weight |
| $\omega_2 = 5$ | acceleration weight |
| $RT = 1.75$ | threshold |

### ✅ Decision Logic

- **IF** $R(k) > 1.75$ → **Advanced Warning**  
- **ELSE** → **Safe**

---

## ✅ Additional Experiment: Soil Strain Forecasting

A separate experiment was added using **soil strain data from a different location**.  
The full MATLAB code is documented inside **Updated Report.mlx**.

### 📊 Result Plot

<img>

### 📊  Graph shows

 plotted:

```
plot(x0,'k')              % Black → Actual
plot(x0_hat_gm_final,'r--')   % Red → GM + Fourier
plot(x0_hat_meta_final,'b--') % Blue → Metabolic GM
```
<img width="465" height="292" alt="Diff place result" src="https://github.com/user-attachments/assets/5e29a27c-a739-4b0d-97d2-813771a2dce0" />

### 🧠 Meaning of each line

⚫ **Black line — Actual data**

👉 This is  real sensor data (Soil Strain)

- What actually happened in the ground
- Real deformation over time

🔴 **Red line — GM + Fourier**

👉 This is  model prediction (classical GM)

- GM captures trend
- Fourier corrects error

👉 So this is  improved prediction

🔵 **Blue line — Metabolic GM**

👉 This is adaptive prediction

- Uses only recent data (window = 5)
- Updates continuously

### 🔍 How to read the graph

✔ **If lines overlap**

👉 GOOD model

Black ≈ Red ≈ Blue

Means:

- prediction is accurate
- model understands data

❌ **If lines are far apart**

👉 BAD model

Means:

- prediction is wrong
- model assumptions failed

### 📈 What  graph shows

From  result:

👉 All lines are very close

👉 Graph looks almost flat around ~1

---

## 📏 Evaluation Metrics
- **RMSE** → penalizes large errors  
- **MAE** → average absolute error  
- **MAPE** → percentage error  

---

## 📈 Results

- Hybrid model improves accuracy  
- Captures both long‑term trend + periodic behavior  
- Provides early warning before failure  

---

## 🧰 Technologies Used

- **MATLAB**  
- **FFT**  
- **Grey System Theory**

---

## ▶️ How to Run (IMPORTANT)

⚠️ **The Global Risk Monitoring file depends on workspace variables.**

### ✅ Execution Order

**Step 1**  
- Run: `Real_data_Coding_E2_sensor.mlx`  
- Creates: `risk_values_2`

**Step 2**  
- Run: `Real_data_coding_E3_sensor.mlx`  
- Creates: `risk_values_3`

**Step 3**  
- Run: `Global_risk_monitoring_report.mlx`  
- This file:
  - Generates `risk_values_1` internally  
  - Uses `risk_values_2` and `risk_values_3`  
  - Computes:
    ```
    risk_values = (risk_values_1 + risk_values_2 + risk_values_3) / 3
    ```
  - Generates final plots  

✅ **Note:** E1 standalone file can run independently.

---

## ⏱️ Execution Time &amp; Performance

Uses MATLAB `tic/toc`.

| File | Sensor | Time |
|------|--------|------|
| E2 file | E2 | 2.32s |
| E3 file | E3 | 3.24s |
| Global file | E1 + Global | 3.24s |
| **Total** | All | **9.09s** |

---

## 🖥️ Platform &amp; Hardware

- **CPU:** Intel Core i7  
- **RAM:** 16GB  
- **OS:** Windows 11  
- **MATLAB:** R2024a  

**Notes:**
- Execution time varies ±0.3s  
- CPU-only (no GPU)  
- Linear scaling with dataset size  

---

## 🔮 Future Improvements

- Extend to **GM(1,N)**  
- Adaptive Fourier selection  
- Multi-sensor fusion models  

---

## 👥 Authors

- Dhakshin N — CB.SC.U4AIE24313  
- Rohith Ravi — CB.SC.U4AIE24350  
- Sanjay Kumar S — CB.SC.U4AIE24354  
- Akhilan S — CB.SC.U4AIE24362  

---

## 📚 References

- *Grey Data Analysis: Methods, Models and Applications* (Computational Risk Management)
