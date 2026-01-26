# Precision Landslide Displacement Prediction using a Hybrid Metabolic Grey Model and Fourier Series Error Correction

## 1. Our Team
We are developing a dynamic, small-dataset early warning system for landslide hazards.

*   **Rohith Ravi** - CB.SC.U4AIE24350
*   **Dhakshin N** - CB.SC.U4AIE24313
*   **Sanjay Kumar S** - CB.SC.U4AUE24354
*   **Akhilan S** - CB.SC.U4AIE2436

## 2. Base & Reference Papers
Our research is primarily grounded in the following core literature:

*   **Primary Reference:** Ma, K., et al. (2025). "A Dynamic Landslide Warning Model Based on Grey System Theory." IEEE Access.
*   **Theoretical Foundation:** Liu Sifeng's Grey System Theory, specifically the optimization of background values and 1-AGO operations.

## 3. Project Outline
In this project, we address the critical need for landslide prediction using limited monitoring data. We have built a hybrid engine that integrates **Grey System Theory (GM(1,1))** with a **Metabolic "Sliding Window"** approach to ensure the model prioritizes real-time sensor updates over outdated historical data. To handle non-linear fluctuations caused by environmental factors like rainfall, we apply **Fourier Series Error Correction**. Our model is designed to provide a **three-level warning system** (Normal, Advanced, and Highest) to ensure public safety and minimize property loss.

## 4. Current Updates
*   **Metabolic Sliding Window:** We implemented a window-based update system that replaces the oldest data point with the most recent monitoring value once the window reaches capacity (e.g., `n=5` or `n=10`).
*   **Matrix Parameter Estimation:** We utilize the Least Squares Method to estimate the development coefficient (`a`) and grey action quantity (`b`) through a parameter vector:

    ```
    a_hat = (B^T B)^-1 B^T Y
    ```

*   **Fourier Spectral Smoothing:** We extract residuals (`e = x^(0) - x_hat^(0)`) and process them through a Discrete Fourier Transform (DFT) to isolate and retain the most significant low-frequency periodic structures while filtering high-frequency noise.
*   **Math Refinement:** Based on our analysis of established Grey System theory, we corrected our background value equation to use the addition/mean formula:

    ```
    z^(1)(k) = (x^(1)(k) + x^(1)(k-1)) / 2
    ```

## 5. Challenges & Issues Faced
*   **Physical Reality vs. Complex Numbers:** A major technical challenge was ensuring our Fourier correction remained real-valued. We implemented **Conjugate Symmetry** in our filtered error array (`E_filt`) so that imaginary components cancel out during the Inverse FFT, resulting in a physical correction in millimeters.
*   **Sensitivity to Noise:** Small dataset models are highly sensitive to sensor inaccuracies. We introduced a **regularization term** (`λ_r`) in our cost function to prevent the model from overfitting to random jitter.
*   **Threshold Selection:** Manually determining the warning threshold (`R_T`) and error calculation threshold (`R_C`) requires careful calibration to avoid false alarms while ensuring no real threats are missed.

## 6. Future Plans: Moving to Global Real-World Datasets
Our current validation uses simulated scenarios, but our primary objective is to transition to the large-scale datasets used in our base paper.

### Our Path Forward:
*   **Multi-Location Validation:** We plan to test our model against the **Preonzo slope (Switzerland)**, **Wolongsi landslide (1971)**, and **Longjingcun landslide (2019)** datasets. These real-world cases represent different geological terrains and movement patterns (stable phase vs. pre-landslide acceleration).
*   **Automated Thresholding:** We aim to integrate big data technologies to automatically estimate `R_T` and `R_C` based on rainfall intensity and duration, reducing the potential for human error.
*   **Robustness & Multi-Sensor Integration:** Future versions will move beyond single-dimension displacement by incorporating multiple sensors and machine learning filters to generate accurate data replacements in case of sensor failure.
