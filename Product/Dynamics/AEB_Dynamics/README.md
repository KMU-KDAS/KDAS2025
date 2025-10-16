# QCar Vehicle Dynamics Model (RC Implementation)

> Motor-driven, **brake-less** 1/10-scale RC vehicle dynamics modeled from first principles to match **QCar** hardware (BLDC + ESC).  
> All equations are implemented explicitly (no “Vehicle Dynamics Bicycle Model” block) and are compatible with **QLabs** and **ROS2** I/O rates/types.

---

## 1. Overview

Unlike a gasoline car, **QCar** uses a BLDC motor and an **ESC** without a hydraulic brake. The model therefore:
- computes **longitudinal** and **lateral** dynamics directly from torque, slip, and simple tire laws,
- maps **ESC duty** to motor torque,
- omits explicit brake torque and uses **reverse/coast torque** from the ESC for deceleration,
- remains lightweight for **real-time control** (MPC, AEB, path tracking) and simulation parity.

---

## 2. Architecture Differences (Why a custom model?)

| Item | Gasoline Car | QCar (RC) |
|---|---|---|
| Powertrain | Engine → Gearbox → Wheel | ESC → **BLDC Motor** → Wheel |
| Braking | Hydraulic brake torque | **No physical brake** (ESC reverse/coast) |
| Control Inputs | Throttle, Brake pressure | **ESC duty** (and decel command `Pmc`) |
| Key Outputs | Engine RPM, Vehicle speed | **Motor speed**, Vehicle speed |
| Brake Torque | Explicit term in equations | **Absent** (replaced by friction/ESC decel) |

> We reconstruct a compact **3-DOF** structure (longitudinal + planar lateral/yaw) instead of shrinking a passenger-car bicycle block.

---

## 3. Symbols

| Symbol | Name | Unit | Note |
|---|---:|---:|---|
| $J_w$ | Wheel equivalent inertia | kg·m² | Wheel + motor reflected inertia |
| $\omega_w$ | Wheel angular speed | rad/s | From encoder |
| $\dot{\omega}_w$ | Wheel angular accel. | rad/s² | – |
| $T_m$ | Motor torque | N·m | From ESC duty via $k_t$ |
| $T_{reverse}$ | Reverse / Coast torque | N·m | ESC-induced decel when commanded |
| $F_t$ | Tire tractive force | N | Nonlinear vs slip $s$ |
| $r_w$ | Wheel radius | m | Measured on RC tire |
| $v_x$ | Longitudinal vehicle speed | m/s | $v_x = r_w \omega_w (1-s)$ |
| $s$ | Slip ratio | – | $s = \dfrac{r_w \omega_w - v_x}{\max(v_x,\varepsilon)}$ |
| $\mu$ | Friction coefficient | – | Slip-dependent |
| $F_z$ | Normal load | N | From mass distribution |

---

## 4. Longitudinal Dynamics

### 4.1 Wheel–Motor Dynamics

$$
J_w \dot{\omega}_w = T_m - T_{reverse} - F_t r_w
$$

Slip, tire force, and friction:

$$
s = \frac{r_w \omega_w - v_x}{\max(v_x,\varepsilon)}, \qquad
F_t = \mu(s) F_z
$$

<p align="center">
  <img src="../../../images/abdq1.png" alt="Wheel/motor longitudinal block" width="760"/>
</p>

---

### 4.2 Motor Torque (ESC Duty → Torque)

$$
T_m = k_t I_m = k_t \frac{V_{duty} - k_e \omega_m}{R}
$$

where $k_t$ is the torque constant, $k_e$ the back-EMF constant, and $R$ the internal resistance.  
Deceleration is realized by duty reduction or reverse command (no brake torque term).

<p align="center">
  <img src="../../../images/abdq2.png" alt="ESC duty to motor torque" width="760"/>
</p>

---

## 5. Tire Model (Longitudinal)

We use a low-cost **Pacejka-style** (Magic Formula) curve to connect linear to saturation regions:

$$
F_t(s) = D \sin \!\Big( C \arctan \!\big( B s - E (B s - \arctan(B s)) \big) \Big)
$$

The model reproduces both linear and saturation regions up to peak slip $s \approx 0.15 \sim 0.2$.

<p align="center">
  <img src="../../../images/abdq3.png" alt="Magic Formula block" width="500"/>
</p>

> Aerodynamic drag is negligible at RC speeds; a small residual term is retained for MPC stability.

---

## 6. Vehicle Longitudinal Equation

$$
m \dot{v}_x = \sum F_t - F_{roll} - F_{aero} - m g \sin\theta
$$

<p align="center">
  <img src="../../../images/abdq4.png" alt="Longitudinal sum of forces" width="500"/>
</p>

---

## 7. Lateral Dynamics

### 7.1 Inputs / Outputs / States

- **Inputs:** steering angle $\delta$, longitudinal speed $v_x$  
- **States:** $v_y$ (lateral speed), $r$ (yaw rate)  
- **Kinematics:** global pose $(x, y, \psi)$

> For real-time control, we keep a compact state set $(v_x, v_y, r)$.  
> A learned **LSTM** side-model provides auxiliary predictions for steering/lateral error.

---

### 7.2 Tire Slip Angles (Front / Rear)

$$
\alpha_f = \delta - \frac{v_y + l_f r}{v_x}, \qquad
\alpha_r = -\frac{v_y - l_r r}{v_x}
$$

Corresponding lateral tire forces:

$$
F_{yf} = -C_f \tan^{-1}(\alpha_f), \qquad
F_{yr} = -C_r \tan^{-1}(\alpha_r)
$$

<p align="center">
  <img src="../../../images/abdq5.png" alt="Lateral tire forces with arctan saturation" width="500"/>
</p>

---

### 7.3 Planar Lateral / Yaw Equations

$$
m(\dot{v}_y + v_x r) = F_{yf} + F_{yr}, \qquad
I_z \dot{r} = l_f F_{yf} - l_r F_{yr}
$$

Integrate to get $v_y(t)$ and $r(t)$, then update the global pose.  
A 2-track model is unnecessary at QCar speeds but can be added for higher fidelity.

<p align="center">
  <img src="../../../images/abdq6.png" alt="Planar lateral/yaw dynamics" width="500"/>
</p>

---

## 8. Motor–Wheel–Vehicle Speed Mapping

With no gearbox, motor speed maps almost directly to vehicle speed:

$$
v_x = r_w \omega_{motor} (1 - s)
$$

<p align="center">
  <img src="../../../images/abdq7.png" alt="Motor speed to vehicle speed" width="500"/>
</p>

This mapping feeds **TTC**, **MPC**, and simulator I/O; on-vehicle we compute $v_x$ from measured $\omega_{motor}$, while in simulation $\omega_{motor}$ can be commanded and mirrored.

---

## 9. TTC-Based Deceleration Logic (AEB / Avoidance)

Relative-distance / velocity TTC:

$$
\text{TTC} = \frac{D_{rel}}{v_x - v_{obj}}
$$

If the TTC-based **Avoidance cost** falls below a threshold, a deceleration command $Pmc$ is issued and the ESC applies reverse/coast torque.  
This reproduces **AEB** behavior without a physical brake system.

---

## 10. Vision-Triggered Stop Logic (Parallel to TTC)

In parallel with TTC, **YOLO** detections (traffic light, stop sign, crosswalk) are fed to **Stateflow**:
- Stateflow combines **State** (current mode) and **Event** (vision) → issues a **stop** command ($v_{motor}=0$).  
- Operates **in parallel** with AEB; either mechanism can trigger stop control.

---

## 11. Validation & Conclusion

Track tests confirmed that the simulated responses closely matched measured QCar behavior in both longitudinal and lateral axes.  
The model runs stably with **MPC / AEB / path-tracking** modules.

<table>
  <tr>
    <td align="center" valign="top">
      <img src="../../../images/control18.gif" alt="Improved Tracking Results" width="400"/><br/>
      <b>Improved Tracking Results</b>
    </td>
    <td align="center" valign="top">
      <img src="../../../images/control19.gif" alt="Final Simulation Results" width="400"/><br/>
      <b>Final Simulation Results</b>
    </td>
  </tr>
</table>

*(Optional) Xytron avoidance photos can be inserted here.*

---
