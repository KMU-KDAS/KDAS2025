# Setup & Simulation Environment

To verify the performance of the AEB (Autonomous Emergency Braking) algorithm, we constructed a simulation environment in **Simulink** integrated with **Unreal Engine 3**.  
The system was developed on **MATLAB R2024b**, using the default **variable-step solver (ode45)**.

Simulink enables hybrid simulation combining **continuous** (e.g., vehicle dynamics) and **discrete** (e.g., sensors, controllers) subsystems.  
Each block’s sample time is explicitly defined so that the solver integrates continuous states while discrete subsystems are updated coherently via **ZOH** and **Rate Transition** blocks.  

By linking to Unreal Engine, the system provides **visual and intuitive validation**, allowing evaluation not only through mathematical signals but also within a realistic 3D driving environment.

---

## Setup Script

Simulation parameters were batch-defined through a **Setup Script** to ensure efficient configuration and repeatability.  
This approach allowed fast parameter tuning, reduced human error, and enabled consistent multi-scenario testing.

Main parameters include:

- **`maxSteer, minSteer, min_ac, max_ac, max_dc`**  
  Define steering and acceleration constraints (actuation limits).  
  Values were based on MATLAB references and physical limits of standard passenger vehicles.

- **`PB1_decel, PB2_decel, FB_decel`**  
  Define deceleration stages for AEB activation.  
  PB1 = mildest braking, FB = full braking.  
  The stage depends on relative distance and vehicle speed.

The most critical parameters are those governing **vehicle dynamics**.  
Although these are ideally measured experimentally, limited resources required estimation from MATLAB examples and course materials.

Both **Tyre Model** and **Bicycle Model** were used; their main parameters are summarized below.

| Variable | Definition |
|-----------|-------------|
| `m` | Vehicle mass |
| `Iz` | Yaw inertia |
| `lf`, `lr` | Distances from CG to front/rear axle |
| `Cf`, `Cr` | Cornering stiffness (front/rear) |
| `mu_B / C / D / E` | Magic Tyre model coefficients |
| `h` | CG height above axle plane |
| `I_wheel` | Wheel inertia |
| `h_wheel` | Tire radius |
| `v_0` | Initial speed |

---

## Dynamics Calculations

### Brake Model

<br/>
<img src="../../../images/aebs1.png" width="820" alt="Brake model diagram"/>

The braking system assumes four identical wheels with a **front/rear torque ratio of 3:2**.  
This configuration reflects real-world design: due to forward load transfer during braking, front wheels bear greater force.

---

### Wheel Dynamics

<br/>
<img src="../../../images/aebs2.png" width="820" alt="Wheel dynamics block"/>

The wheel dynamics block implements the following torque balance equation:

$$
J\dot{\omega} = T_{\mathrm{shaft}} - T_{\mathrm{brake}} - h_{\mathrm{eff}}F_{\mathrm{road}}
$$

The **effective wheel radius** and **longitudinal force** \( F_{\mathrm{road}} \) are computed through the tire model.

---

### Magic Tyre Model (Bakker–Pacejka)

<br/>
<img src="../../../images/aebs3.png" width="820" alt="Magic Tyre model and slip computation"/>

We used the **Bakker–Pacejka “Magic Tyre Model”**, expressed as:

$$
F_{\mathrm{road}} = D \sin\!\big[C \arctan\!\big\{B(S + S_h) - E\big(B(S + S_h) - \arctan(B(S + S_h))\big)\big\}\big]
$$

The function block takes slip \( s \) as input, assumes no camber adjustment, sets slip bias \( S_h = 0 \), and uses the coefficients \( B, C, D, E \).

Slip ratio \( s \) is defined as:

$$
s = \frac{h_{\mathrm{eff}}\omega - v}{\max(h_{\mathrm{eff}}\omega, v)}
$$

Division by zero is avoided using a **switch block** that substitutes 0.01 when needed.

---

### Vehicle Pose via Bicycle Model

<br/>
<img src="../../../images/aebs4.png" width="780" alt="Bicycle model integration"/>

Front and rear longitudinal forces computed above feed into a **Bicycle Model**, which outputs vehicle **pose (x, y, ψ)**.  
Since this study focuses on **stopping scenarios**, the **steering angle** was fixed at **0 rad**.

We used MATLAB’s **Automated Driving Toolbox Bicycle Model block** rather than a custom one, ensuring compatibility with 3D simulation while minimizing errors.  
For environmental consistency, all angular units were converted **(radians → degrees)**, and coordinate systems were transformed **(NED → NWU)**.

---

### UE Actor Bus Packing

In Unreal-integrated simulation, ego-vehicle data is transferred as a structured message over the **Actor Bus**.  
The **Pack Ego Actor** MATLAB Function assembles the data as follows:


**Key variables and their roles:**

| Symbol | Description | Context |
|---|---|---|
| \(J_{\omega}\) | Wheel rotational inertia | Wheel dynamics |
| \(\dot{\omega}\) | Angular acceleration | Wheel dynamics |
| \(T_{\mathrm{shaft}}\) | Shaft torque | Wheel dynamics |
| \(T_{\mathrm{brake}}\) | Brake torque | Wheel dynamics |
| \(h_{\mathrm{eff}}\) | Effective wheel radius | Wheel dynamics, Slip ratio |
| \(F_{\mathrm{road}}\) | Longitudinal tire force | Wheel dynamics, Tyre model |
| \(D, C, B, E\) | Magic Tyre model coefficients | Tyre model |
| \(S_h\) | Slip bias | Tyre model |
| \(s\) | Slip ratio | Slip ratio computation |
| \(v\) | Longitudinal vehicle velocity | Slip ratio |
| \(\omega\) | Wheel angular velocity | Slip ratio |

---

## Unreal Engine Simulation

<br/>
<img src="../../../images/aebs5.png" width="820" alt="Unreal Engine simulation environment 1"/>
<br/>
<img src="../../../images/aebs6.png" width="820" alt="Unreal Engine simulation environment 2"/>

We utilized MATLAB’s **Automated Driving Toolbox** to link Simulink with **Unreal Engine**, building a real-time 3D simulation environment.  
This setup enables **visual validation** of braking dynamics beyond abstract signals.

---

### Driving Scenario Design

The simulation involves two vehicles — **Ego** and **Lead** — both following straight-line paths to focus exclusively on AEB performance.  
Three speed cases (low, medium, high) were tested to evaluate **TTC-based braking** under varying conditions.

Since no physical sensors exist in the virtual world, **synthetic sensor data** was generated via simulation blocks:
- **Cuboid to 3D Simulation** converts object data to 3D form.
- **Simulation 3D Vehicle with Ground Following** defines ego geometry and motion.

The generated data was fused and tracked using **Detection Concatenation** and **Multi-Object Tracker** blocks, forming the perception layer for AEB evaluation.

---

## Lead Vehicle Detection

For reliable AEB triggering, it is crucial to detect whether a **lead vehicle** is present ahead.

From tracker outputs:
- \(x > 0\): the object is ahead of ego  
- \(|y| \le 1.8\,m\): the object lies within the same lane

Given typical lane width ≈ 3.6 m, this region defines the **collision risk zone**.  
If both conditions are satisfied, the object is labeled as a lead vehicle.  
The system computes **relative distance**, **relative velocity**, and **TTC**, which drives the **braking logic**.

---

## AEB Controller

<br/>
<img src="../../../images/aebs7.png" width="820" alt="AEB controller block"/>

The AEB controller consists of:
1. **Brake Controller**
2. **Normal Driving Controller** (EKF + NMPC)
3. **Mode Selector**

For straight-line tests, the normal driving controller’s function is minimal; it is retained for consistency with MATLAB’s reference design.

<br/>
<img src="../../../images/aebs8.png" width="820" alt="Brake system sections"/>

Braking logic is divided into:
1. **TTC (Time-to-Collision) computation**
2. **Stopping time estimation**
3. **AEB operation logic (Stateflow)**

Each follows the flow: *collision risk detection → braking decision → command execution*.

---

### TTC Calculation

<br/>
<img src="../../../images/aebs9.png" width="820" alt="TTC calculation block"/>

TTC is computed as:

$$
\mathrm{TTC} = \operatorname{sign}(v_{\mathrm{rel}})\frac{d_{\mathrm{rel}} - h_0}{|v_{\mathrm{rel}}|}
$$

where  
- \(d_{\mathrm{rel}}\): relative distance  
- \(v_{\mathrm{rel}}\): relative velocity  
- \(h_0\): headway offset (safety distance)

A **Saturation block** prevents division by zero, and sign restoration ensures correct directionality.  
The TTC sign indicates approach (−) or separation (+).

---

### Stopping Time Estimation

<br/>
<img src="../../../images/aebs10.png" width="820" alt="Stopping time computation"/>

Stopping time is derived from:

$$
t_{\mathrm{stop}} = \frac{v}{a}
$$

where \(v\) is current velocity and \(a\) is applied deceleration.  
A safety margin is added to accommodate delays and ensure comfortable stops.

---

### AEB Operation Logic

<br/>
<img src="../../../images/aebs11.png" width="820" alt="AEB Stateflow logic"/>

Implemented in **Stateflow**, the logic manages discrete braking states:

- **Normal Driving**
- **Alert**
- **Partial Braking**
- **Full Braking**

Transition condition:  
When \( \mathrm{TTC} < t_{\mathrm{stop}} \), the system shifts to a braking state corresponding to the risk level and outputs the associated **deceleration command**.

---

### Mode Selection

<br/>
<img src="../../../images/aebs12.png" width="820" alt="Controller mode selector"/>

The **Mode Selector** receives AEB status and outputs the corresponding brake command.

- Normal: 0 (no braking)  
- Partial: increasing brake torque  
- Latching logic prevents signal loss  
- During braking, **steering** is held constant to avoid lateral load transfer and maintain maximum braking force.

---

## Results & Analysis

<br/>
<img src="../../../images/aebs13.png" width="760" alt="Relative distance graph"/>
<br/>
<img src="../../../images/aebs14.png" width="760" alt="TTC graph"/>
<br/>
<img src="../../../images/aebs15.png" width="760" alt="Deceleration graph"/>
<br/>
<img src="../../../images/aebs16.png" width="760" alt="Relative velocity graph"/>
<br/>
<img src="../../../images/aebs17.png" width="760" alt="Ego velocity graph"/>

When the lead vehicle moved slower than ego, the relative distance minimized around **3 s**, then increased as braking activated.  
Deceleration increased stepwise, corresponding to AEB stages.  
TTC dropped sharply (~3.5 s) due to diminishing denominator \(v_{\mathrm{rel}}\), then stabilized as braking progressed.  
Relative velocity converged to zero, confirming safe separation.  
Ego speed decreased smoothly after 4 s, validating the correct behavior of the TTC-based control logic.

By integrating Unreal Engine visualization, real-time **vehicle interaction**, **stopping distance**, and **collision risk** could be observed dynamically.

---

## Discussion & Future Work

Current AEB logic uses three discrete braking levels (**PB1**, **PB2**, **FB**).  
Future implementations may adopt **continuous braking profiles** proportional to risk or velocity change rate for smoother deceleration.

### TTC Limitations
TTC assumes constant relative velocity, which fails in:
- Curved roads (misaligned sensor axis)
- Rapid acceleration/deceleration
- Multi-object scenarios (misdetections)

### Improvement Strategies
- Adaptive TTC thresholding using real-time acceleration  
- Prediction-based correction using historical velocity trends  
- Weighted fusion of multiple risk estimators

### Comparative Collision Metrics

| Method | Concept | Pros | Cons |
|---|---|---|---|
| Enhanced TTC | TTC + acceleration | Robust at high accel rates | Sensitive to noise |
| DRAC | Required deceleration to avoid collision | Intuitive physical meaning | Misinterprets static obstacles |
| Stopping Distance Index | Braking distance ratio | Reflects braking capability | Hard to estimate lead decel |
| Risk Function | Weighted sum of distance, speed, accel | Continuous risk estimation | Requires weight design |
| Collision Possibility | Probabilistic risk | Multi-object handling | Lower real-time response |
| AI Prediction | Learned collision probability | Handles nonlinear complex cases | Requires large dataset |

### Curved Road Scenarios
Curves introduce lateral load, steering, and yaw effects.  
Possible solutions:
- Curvilinear TTC (incorporating steering/yaw rate)
- Early braking thresholds on curves
- Alternative fusion-based collision detection

---

### Simulation vs Reality

The academic MATLAB–UE setup suffices for basic validation but lacks real-world complexity.  
Future validation could use **CARLA**, **PreScan**, or **IPG CarMaker** with high-fidelity vehicle models, sensor noise, and road geometry.

Simulations, though idealized, are invaluable for:
- Algorithm verification
- Safe debugging
- Logical validation before field testing

However, real-world deployment requires parameter **recalibration** to handle environmental variations (lighting, road surface, communication latency).  
Therefore, simulation serves as a **bridge stage** — not a substitute — toward full-scale autonomous driving validation.

---
