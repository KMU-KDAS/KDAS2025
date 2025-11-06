# **Kalman Filter-Based Localization Refinement**

---

##  Overview

This document presents the **Kalman Filter (KF)** design and implementation to improve SLAM-based localization for autonomous driving.  
The filter fuses the **MATLAB Simulink dynamic model** with **ROS2 sensor data (LiDAR, IMU)** and operates in two core stages — **Prediction** and **Update**.

---

##  1. Background and Problem Definition

In the SLAM-based localization process, minor coordinate jitter (2nd–3rd decimal place) was observed even when the vehicle was stationary.  
Although small, such drift accumulates in narrow and curved track environments, degrading **trajectory tracking accuracy** and **driving stability**.

To address this, we applied a **Kalman Filter** to refine SLAM outputs by explicitly modeling sensor noise and process uncertainty, resulting in more stable and reliable localization.

---

##  2. Methodology

### **Sensor Configuration**
- Localization from **LiDAR** and **IMU** (no GPS due to indoor environment).  
- SLAM estimates position `(x, y, heading)`.

### **State Estimation Refinement**
- SLAM outputs are **post-processed** through a Kalman Filter to smooth short-term fluctuations.  

### **Log Data Analysis and Parameter Tuning**
1. SLAM log preprocessing (duplicate timestamp removal, chronological sorting).  
2. Compute mean/variance of position `(x, y)` to estimate initial **measurement noise covariance R**.  
3. Iteratively tune Q and R:
   - If fluctuations are high → **↓ R** (trust sensors more)  
   - If filter responds slowly → **↑ Q** (trust model more)
4. After convergence, remove all temporary ROS publishers and observation logs for data integrity.

---

### **Code Example – SLAM Log Preprocessing**

~~~python
import pandas as pd
import numpy as np

# Load SLAM log data
df = pd.read_csv("slam_log.csv")

# Remove duplicate timestamps and sort
df = df.drop_duplicates(subset='timestamp').sort_values('timestamp')

# Calculate mean and variance for position coordinates
x_mean, y_mean = df['x'].mean(), df['y'].mean()
x_var, y_var = df['x'].var(), df['y'].var()

print(f"Mean Position: ({x_mean:.4f}, {y_mean:.4f})")
print(f"Variance (x, y): ({x_var:.6f}, {y_var:.6f})")

# Initial R matrix (measurement noise covariance)
R = np.diag([x_var, y_var])
print("Initial Measurement Noise Covariance R:\n", R)
~~~
> This script preprocesses SLAM logs, extracts noise characteristics, and generates an initial covariance matrix (R) for the Kalman Filter.

---

##  3. Kalman Filter Design Overview

 **Insert Image Placeholder:** `Kalman_Filter_Concept.png`  
*(Insert your conceptual diagram of the Prediction–Update flow here.)*

The Kalman Filter operates in two primary stages: **Prediction** and **Update**, as shown below.

$$
\textbf{Prediction:}
\begin{cases}
x_{k|k-1} = F x_{k-1|k-1} + B u_k \\
P_{k|k-1} = F P_{k-1|k-1} F^{T} + Q
\end{cases}
$$

$$
\textbf{Update:}
\begin{cases}
K_k = P_{k|k-1} H^{T} (H P_{k|k-1} H^{T} + R)^{-1} \\
x_{k|k} = x_{k|k-1} + K_k (z_k - H x_{k|k-1}) \\
P_{k|k} = (I - K_k H) P_{k|k-1}
\end{cases}
$$

---

### **Parameter Definitions**

| Symbol | Description |
|:-------:|-------------|
| **x** | State vector `[x, y, heading, v]` |
| **F** | State transition matrix (from Simulink vehicle model) |
| **u** | Control input (acceleration, steering) |
| **P** | Covariance matrix (uncertainty) |
| **Q, R** | Process and measurement noise covariances |
| **H** | Measurement matrix (maps SLAM outputs to state space) |
---

##  4. System Integration Overview

| **Role** | **Platform / System** | **Description** |
|-----------|-----------------------|-----------------|
| **SLAM** | ROS | Estimates vehicle position `(x, y, heading)` from LiDAR + IMU |
| **Vehicle Dynamics (F, u)** | MATLAB Simulink | Publishes state transition model and control inputs to ROS2 |
| **Kalman Filter (Correction)** | Python ROS2 Node | Fuses SLAM position with Simulink-published model for refinement |
| **Output** | ROS Topic `/kf_output` | Publishes corrected position to MATLAB, RViz, or control modules |

---

##  5. Results and Analysis

 **Insert Image Placeholder:** `KF_Results_Graph.png`  
*(Include a figure comparing raw SLAM vs. KF-corrected trajectories.)*

### **Key Observations**
- The Kalman Filter successfully reduced high-frequency oscillations in the SLAM position data.  
- Position estimates became smoother and more stable across curved track segments.  
- However, full convergence was not always achieved — sudden deviations occurred when **model uncertainty (Q)** or **sensor noise (R)** were inaccurately set.  
- In some cases, the filter **overreacted to transient sensor noise**, temporarily destabilizing the estimates.

### **Insights**
- Localization stability is highly sensitive to the balance between **model trust (Q)** and **sensor trust (R)**.  
- Proper parameter tuning and accurate vehicle dynamic modeling are essential to maintain reliable correction.  
- The experiment verified that **filter accuracy directly correlates with the realism of the dynamic model (F, B, u)**.

---

##  6. Limitations and Future Improvements

| **Limitation** | **Description** |
|----------------|----------------|
| **Noise Modeling** | Difficult to mathematically define Q and R precisely with limited data, leading to empirical tuning. |
| **Dynamic Model Fidelity** | Simplified bicycle model could not fully capture real vehicle motion during prediction. |
| **Response Sensitivity** | KF occasionally produced delayed or oscillatory corrections when SLAM uncertainty increased. |

### **Future Work**
- Implement **Adaptive Kalman Filter (AKF)** to update Q and R dynamically based on real-time sensor variance.  
- Improve timestamp synchronization between SLAM, IMU, and Simulink publishers.  
- Extend to **nonlinear variants (EKF, UKF)** for better modeling of vehicle kinematics.  
- Conduct statistical evaluation on covariance convergence and residual innovation consistency.

---

##  7. Summary 

- The proposed Kalman Filter effectively mitigated **SLAM position jitter** (2nd–3rd decimal place), enhancing tracking stability.  
- Real-time fusion between **Simulink vehicle dynamics (F, u)** and **ROS2 sensor data** provided smooth localization on narrow, curved tracks.  
- This study highlights the critical role of noise tuning and dynamic modeling in real-world autonomous vehicle localization pipelines.  
- The outcome forms the foundation for **Adaptive and Nonlinear KF frameworks** in future research.

---
