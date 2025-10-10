# **Pure Pursuit & Stanley Controllers for Autonomous Driving**

---

## 1. Introduction and Motivation

In early stages (AEB simulation, Xytron simulation), we implemented autonomous driving with a **PID controller**.  
However, the **Xytron competition** and **QLabs** demanded **high-speed driving**, **tight curves**, and **obstacle sections** simultaneously.  
Under these conditions, PID tuning alone could not guarantee stability; at higher speeds the output often spiked, degrading control performance.

To address this, we studied and implemented **Stanley** and **Pure Pursuit** controllers — both computationally efficient and robust for high-speed path tracking.

<img src="../../../images/Control1.png" alt="PID vs Pure Pursuit vs Stanley Concept" width="800"/>

---

## 2. Comparative Analysis: PID vs. Pure Pursuit vs. Stanley

- **PID**: simple and responsive at **low speed**, but prone to oscillation and gain sensitivity at **high speed**, especially in sharp curves or noisy signals.  
- **Stanley**: uses **heading error** and **lateral (cross-track) error** for geometric path tracking; strong lane-centering even on curves.  
- **Pure Pursuit**: computes curvature to the **look-ahead point**; simple, smooth paths at high speed; performance sensitive to look-ahead choice.

<img src="../../../images/Control2.png" alt="Controller Comparison Overview" width="800"/>

---

## 3. Stanley Controller

**Principle.** Stanley corrects both **heading error** and **lateral error** relative to the reference centerline.  
We used **SCNN-based lane centerline** (depth-projected coordinates) as input when available, calculating steering to reduce both errors simultaneously.

**Pros**
- Robust lane-centering on curves  
- Uses geometric information and vehicle heading

**Cons**
- Can be overly aggressive at very low speed  
- Parameter **k** tuning is non-trivial

<img src="../../../images/Control3.png" alt="Stanley Controller Geometry" width="800"/>

---

## 4. Event Handling with Stateflow (Quanser Prelim Spec)

We modeled stopping events in **Stateflow** to satisfy competition rules:

1) **Sign/Signal-based stop** (YOLO integration)  
- YOLO detects traffic light / stop sign → publishes a ROS flag  
- Stateflow transitions to **Stopping** when `stopsign == 1`, sets `output_speed = 0`, waits for `after(t_sec)`, then resumes

2) **Coordinate-based stop** (CoorStop)  
- MATLAB function computes Euclidean distance to next stop target  
- If condition `(stopIndex <= numStops) && (distance_to_target < threshold)` holds → transition to **CoorStop**, set `output_speed = 0`, then resume and increment `stopIndex`

<img src="../../../images/Control4.png" alt="Stateflow Stop Logic" width="800"/>

---

## 5. Simulation: Sine-Path Tracking (MATLAB/Simulink)

We generated a **sine-wave path** (waypoints) and validated Stanley tracking performance:
- Confirmed waypoint tracking stability
- Verified correct behavior under **stop** and **resume** events

<img src="../../../images/Control5.png" alt="Sine Path Generation and Tracking" width="800"/>

### 5.1 Lateral Error (Cross-Track Error)

Multiple error traces appear due to vectorized computations across time and segments;  
errors stayed within bounds, with slightly larger deviations near the end of the path.

<img src="../../../images/Control6.png" alt="Lateral Error Traces" width="800"/>

### 5.2 Steering Angle Output

We used **zero-order hold** and discrete command intervals to avoid overly sharp steering spikes that could stress the steering actuator.  
The discretized output is intentional for **hardware safety**.

<img src="../../../images/Control7.png" alt="Discrete Steering Output" width="800"/>

### 5.3 Vehicle Speed Profile

Speed adapts naturally to path curvature — slower on sharp curves and faster on straights — indicating appropriate response to path characteristics.

<img src="../../../images/Control8.png" alt="Speed Profile Along Sine Path" width="800"/>

### 5.4 Stateflow Speed Output (Pulse Stop Event)

At `t = 20s`, a pulse stop command was injected; the controller stopped and then resumed tracking correctly, demonstrating integration with YOLO/coordinate-based stop logic.

<img src="../../../images/Control9.png" alt="Stateflow Speed Output with Stop/Resume" width="800"/>

---

## 6. From Simulation to Competition Systems

- In **Xytron Simulator**, overall driving stability was achieved.  
- In **QLabs**, the organizer reported a **Depth camera issue** in simulation, meaning our planned **SCNN+Depth coordinate** input could not be deployed on-site.  
- Therefore, for the final system, we adopted **Pure Pursuit** (with SCNN centerline used when feasible in other environments).

<img src="../../../images/Control10.png" alt="System Integration across Environments" width="800"/>

---

## 7. Pure Pursuit Controller

**Principle.** Pure Pursuit steers the vehicle toward the **look-ahead point** on the path, computing curvature for smooth tracking.

- **Pros**: simple, effective at high speed, smooth trajectories  
- **Cons**: error can increase on sharp curves; sensitive to **look-ahead distance**

**System Modeling.** We built a controller centered on the **Pure Pursuit block**:  
Inputs were vehicle pose `(x, y, θ)` from **Cartographer SLAM** and **waypoints** from the planner; outputs were **linear velocity** and **steering angle**.

<img src="../../../images/Control11.png" alt="Pure Pursuit Model and Look-Ahead Logic" width="800"/>

---

## 8. Performance Improvements and Results

**Look-Ahead Distance (adaptive by segment)**
- Straight: **0.1 m** → reduce unnecessary steering, maintain stable high-speed straight driving  
- Curve: **3 m** → strengthen curve tracking

**Steering Saturation**
- Straight: **±0.13 rad**  
- Curve: **±0.35 rad**  
→ Prevent excessive spikes, ensure robust actuation

**Outcome.** In both simulation and real driving, **Pure Pursuit** provided stable performance at **low and high speeds** and on **curved segments**.  
Adaptive look-ahead and saturation notably reduced sharp output spikes.

<img src="../../../images/Control12.png" alt="Improved Results: Adaptive Look-Ahead and Saturation" width="800"/>

---

## 9. Conclusion

- We transitioned from **PID** to **Stanley** and **Pure Pursuit** to meet high-speed, tight-curve requirements under noise.  
- **Stateflow** handled rule-based stops; MATLAB/Simulink validated tracking on sine paths.  
- Due to QLabs depth issues, the **final competition system** used **Pure Pursuit**, which achieved stable, spike-free control with adaptive parameters.  
- Future work: restore **SCNN+Depth** input where available, and explore **MPC** once sensor synchronization and compute latency are fully addressed.

