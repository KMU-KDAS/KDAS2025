# AEB Simulation Setup & Environment

We verified the AEB algorithm in a Simulink + Unreal Engine 3–based simulation.  
The project was developed on **MATLAB R2024b** with the **default variable-step** configuration and **ode45** solver.

Simulink supports mixed **continuous** (vehicle dynamics) and **discrete** (sensors/controllers) execution in one model.  
Each block is given a sample time; the solver integrates continuous states while discrete blocks are updated coherently through ZOH / Rate Transition.  
Linking with Unreal enables **visual and intuitive** validation in addition to purely mathematical analysis.

---

## Setup Script

Key model parameters are defined centrally in a **Setup Script** so we can sweep scenarios consistently and avoid configuration mistakes.

- `maxSteer`, `minSteer`, `min_ac`, `max_ac`, `max_dc`  
  → Actuation constraints (steering limits, accel/decel bounds) from MATLAB examples and typical passenger-car limits.

- `PB1_decel`, `PB2_decel`, `FB_decel`  
  → Stage-wise deceleration for AEB (Partial Brake 1/2, Full Brake) depending on relative distance and ego speed.

> Vehicle-dynamics parameters should ideally come from measurement. In this study, we used values from MATLAB examples and course notes due to measurement constraints.

We employ both **Tire (Magic Tyre)** and **Bicycle** models; key parameters are:

| Variable | Definition |
|---|---|
| `m` | Vehicle mass |
| `Iz` | Yaw inertia |
| `lf, lr` | CG to front/rear axle distances |
| `Cf, Cr` | Cornering stiffness (front/rear) |
| `mu_B / C / D / E` | Magic Tyre model coefficients |
| `h` | CG height above axle plane |
| `J_wheel` | Wheel inertia |
| `r_wheel` | Tire radius |
| `v_0` | Initial speed |

---

## Dynamics Calculations

### Brake model
<br/>
<img src="../../../images/aebs1.png" width="820" alt="Brake model diagram"/>

The brake model above assumes four identical wheels and a **front:rear torque split of 3:2**, reflecting typical forward load transfer under braking.

<br/>
<img src="../../../images/aebs2.png" width="820" alt="Wheel dynamics diagram"/>

The wheel dynamics implement the following relationship between torques and longitudinal tire force. The tire model supplies the **effective radius** and the **road force** used in the wheel balance.

### Tire (Magic Tyre) model
<br/>
<img src="../../../images/aebs3.png" width="820" alt="Magic Tyre model block"/>

We use the Bakker–Pacejka **Magic Tyre Model**, expressed as:
$$
F_{\mathrm{road}}(s)=
D \,\sin\!\left(
  C \,\arctan\!\left( B s \;-\; E\big(B s - \arctan(B s)\big) \right)
\right).
$$

This is implemented in a Function block. Input is slip \(s\); camber adjustment is neglected, slip bias \(S_h\) is set to \(0\); coefficients \((B,C,D,E)\) are parameters.

The **slip ratio** \(s\) is
$$
s \;=\; \frac{h_{\mathrm{eff}}\omega - v}{\max(h_{\mathrm{eff}}\omega,\,v)}.
$$

<br/>
<img src="../../../images/aebs4.png" width="820" alt="Slip computation and protection"/>

Division-by-zero is prevented with a Switch that substitutes **0.01** when needed.

### Bicycle model integration
Front/rear longitudinal forces from brake and tire models feed a **Bicycle Model** to compute the vehicle pose.  
Because we only evaluate **stopping scenarios**, the steering wheel angle is **fixed at 0 rad**.

We use the **Automated Driving Toolbox** Bicycle Model block rather than a custom block to match the 3-D simulation stack and minimize integration errors.

For environment consistency we convert **radians → degrees** (where required) and transform coordinates between **NED → NWU**.

#### UE actor bus packing
In Unreal-linked simulation, ego-vehicle data is passed as a struct over the **Actor bus**. The *Pack Ego Actor* MATLAB Function assembles:
Struct(
ActorID,
Position [x, y],
Velocity [x, y],
Roll, Pitch, Yaw,
AngularVelocity [roll, pitch, yaw]
)

**Key symbols and context**

| Symbol | Description | Context |
|---|---|---|
| \(J_\omega\) | Wheel rotational inertia | Wheel dynamics |
| \(\dot{\omega}\) | Angular acceleration | Wheel dynamics |
| \(T_{\mathrm{shaft}}\) | Shaft torque | Wheel dynamics |
| \(T_{\mathrm{brake}}\) | Brake torque | Wheel dynamics |
| \(h_{\mathrm{eff}}\) | Effective wheel radius | Wheel dynamics, Slip ratio |
| \(F_{\mathrm{road}}\) | Longitudinal tire force | Wheel dynamics, Tyre model |
| \(D,C,B,E\) | Magic Tyre coefficients | Tyre model |
| \(S_h\) | Slip bias | Tyre model |
| \(s\) | Slip ratio | Slip computation |
| \(v\) | Longitudinal velocity | Slip ratio |
| \(\omega\) | Wheel angular velocity | Slip ratio |

---

## Unreal Engine Simulation
<br/>
<img src="../../../images/aebs5.png" width="820" alt="Unreal environment overview"/>

We used Automated Driving Toolbox to couple Simulink with Unreal Engine for **3-D visualization** and **intuitive diagnostics** beyond numeric plots.

<br/>
<img src="../../../images/aebs6.png" width="820" alt="Scenario assets and sensor generation"/>

### Driving scenario design
Two vehicles (Ego, Lead) run **straight-line** paths to isolate AEB performance. We prepare **low/mid/high-speed** cases and evaluate **TTC-based braking**.

Since no physical sensors exist in the virtual world, we **generate** sensor data in simulation:
- **Cuboid to 3-D Simulation** converts objects to 3-D entities,
- **Simulation 3-D Vehicle with Ground Following** defines/updates the vehicles,
- **Detection Concatenation** + **Multi-Object Tracker** perform sensor fusion and tracking.

> Our goal is AEB control evaluation; therefore we use built-in MATLAB blocks for perception rather than custom recognition networks.

---

## Lead Car Detection

We determine if a **lead vehicle** exists by comparing tracker outputs with ego pose:

- If **\(x>0\)** → object is ahead of ego.
- If **\(|y| \le 1.8\,\mathrm{m}\)** → object lies within ±1.8 m lateral band (≈ lane width / 2) assuming ego tracks lane center.

When both hold, the system treats the object as a **lead vehicle**, then computes **relative distance** and **relative velocity**, from which **TTC** is derived for deceleration control.

---

## AEB Controller

We adopt a **TTC**-based strategy because it is simple, intuitive, and suitable for real-time execution.

<br/>
<img src="../../../images/aebs7.png" width="820" alt="AEB top-level controller"/>

The controller has:
- **Brake Controller**,  
- **Normal Driving Controller** (EKF + NMPC; retained from MATLAB example but not central for straight runs),
- **Mode Selector**.

### Braking system structure
<br/>
<img src="../../../images/aebs8.png" width="820" alt="Braking system sections"/>

Three sections run in sequence:

1) **TTC computation**  
2) **Stopping-time computation**  
3) **AEB operation logic** (Stateflow)

<br/>
<img src="../../../images/aebs9.png" width="820" alt="TTC computation block"/>

**TTC** is computed as
$$
\mathrm{TTC} \;=\; \mathrm{sign}\!\left(v_{\mathrm{rel}}\right)\;
\frac{d_{\mathrm{rel}} - h_0}{\lvert v_{\mathrm{rel}}\rvert},
$$
where  
\(d_{\mathrm{rel}}\): relative distance,  
\(v_{\mathrm{rel}}\): relative velocity,  
\(h_0\): headway offset (safety distance).

A **Saturation** block prevents division by zero; we explicitly restore the **sign** to preserve approach (negative TTC) vs separation (positive TTC).

<br/>
<img src="../../../images/aebs10.png" width="820" alt="Stopping time block"/>

**Stopping time** under a given deceleration \(a\) is
$$
t_{\mathrm{stop}} \;=\; \frac{v}{a},
$$
with an added **time margin** for safety.

<br/>
<img src="../../../images/aebs11.png" width="820" alt="AEB Stateflow logic"/>

The **Stateflow** machine defines the states:
- Normal Driving
- Alert
- Partial Braking
- Full Braking

**Transition rule:** when \(\mathrm{TTC} < t_{\mathrm{stop}}\), the state advances according to the risk level and outputs the corresponding **deceleration command**.

### Mode selection
<br/>
<img src="../../../images/aebs12.png" width="820" alt="Controller mode selector"/>

The **Controller Mode Selector** takes the AEB status from Stateflow and outputs the final brake command.  
Latching prevents spurious release due to transient signal glitches.  
While braking is active, **steering is held** (no steering commands) to avoid lateral load transfer and to maximize braking force.

---

## Results and Discussion

We implemented the AEB system with Simulink + Stateflow and validated it in Unreal.

**Plots**
<br/>
<img src="../../../images/aebs13.png" width="820" alt="Relative distance"/>
<br/>
<img src="../../../images/aebs14.png" width="820" alt="TTC"/>
<br/>
<img src="../../../images/aebs15.png" width="820" alt="Deceleration"/>
<br/>
<img src="../../../images/aebs16.png" width="820" alt="Relative velocity"/>
<br/>
<img src="../../../images/aebs17.png" width="820" alt="Ego velocity"/>

**Detailed signal analysis**

In a scenario where the lead vehicle runs slower than ego:

- **Relative distance** reaches a minimum near **3 s**, then increases as braking engages and ego decelerates.
- **TTC** drops sharply around **3.5 s** and becomes **negative** with large magnitude as \(|v_{\mathrm{rel}}|\) approaches zero (denominator shrinks). After ego deceleration, TTC stabilizes.
- **Deceleration** increases in **stages** (PB1 → PB2 → FB) as risk escalates, confirming the stage-wise logic.
- **Relative velocity** is negative (approaching) before braking, then moves toward zero after intervention.
- **Ego speed** decreases steadily after ~**4 s**, avoiding collision and restoring a safe gap.

The behavior confirms that the **TTC-based decision** and **stage-wise braking** function as intended.

---

## Takeaways and Future Work

Linking with Unreal provided **visual validation**—we could inspect vehicle motion, collision distance, and interactions that are not obvious from plots alone.

Currently braking uses **three discrete stages** (PB1, PB2, FB). A future improvement is a **continuous deceleration profile** parameterized by risk or by \(\dot{v}_{\mathrm{rel}}\), to achieve smoother stops.

TTC works well but has **limitations** on curves, with acceleration changes, and with multiple objects, because TTC assumes approximately constant relative speed and a straight bearing.

**Potential enhancements**

| Name | Concept | Pros | Cons |
|---|---|---|---|
| Enhanced TTC | TTC + acceleration | More stable at high/rapid accel | Sensitive to accel noise |
| Decel Rate to Avoid Collision | Minimum decel to avoid impact | Physically intuitive | May misjudge static obstacles |
| Stopping Distance Index | Hazard from stopping-distance ratio | Directly reflects brake capability | Needs lead-car decel estimate |
| Risk Function | Weighted sum of distance/speed/accel risks | Continuous risk; flexible | Weight design required |
| Collision Possibility | Probabilistic risk | Handles uncertainty/multi-object | Heavier compute / tuning |
| AI Prediction | Learned collision probability | Handles nonlinear, complex scenes | Data + training required |

For **curved driving**, add steering/yaw-rate terms (curvilinear TTC) or switch to alternative hazard metrics.

Finally, simulation is controlled and idealized; **real-world transfer** requires calibration due to lighting, surface, sensor noise, and latency.  
Even so, simulation remains essential for safe early-stage validation and structured debugging. We therefore treat it as a **crucial transition step** toward on-vehicle deployment.

---
