# **Pure Pursuit & Stanley Controllers for Autonomous Driving**

---

## 1. Introduction and Background

In the early development stages (AEB System simulation, Xytron simulation),  
autonomous driving was implemented using a **PID controller**.  
However, both the **Xytron competition** and **QLabs environment** required high-speed driving, tight curves, and obstacle sections simultaneously.  
Under such conditions, the PID controller could not ensure stable performance through gain tuning alone,  
and unstable output spikes frequently occurred at high speeds.

---

## 2. PID Control Analysis

To analyze control stability, performance comparisons were made across **PID** at both **high** and **low speeds**.

### PID stability table

<table>
  <tr>
    <td align="center">
      <img src="../../../images/control1.png" alt="High-speed PID Δx" width="400"/><br/>
      <b>High-speed PID Δx</b>
    </td>
    <td align="center">
      <img src="../../../images/control2.png" alt="High-speed PID ΔSteering Angle" width="400"/><br/>
      <b>High-speed PID ΔSteering Angle</b>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="../../../images/control3.png" alt="Low-speed PID Δx" width="400"/><br/>
      <b>Low-speed PID Δx</b>
    </td>
    <td align="center">
      <img src="../../../images/control4.png" alt="Low-speed PID ΔSteering Angle" width="400"/><br/>
      <b>Low-speed PID ΔSteering Angle</b>
    </td>
  </tr>
</table>

In low-speed environments (see [Xytron Simulator](#)),  
the PID controller showed relatively stable behavior.  
However, in the **high-speed** conditions required for the competition, the output fluctuated sharply,  
and overall driving stability degraded significantly.  
This demonstrated that manual gain tuning was **inefficient and unreliable**, motivating the adoption of new controllers.  
We therefore investigated the **Stanley** and **Pure Pursuit** controllers—both known for their stability, efficiency, and low computational cost.

---

## 3. Stanley Controller

### 3.1 Control Law
The Stanley controller computes steering angle δ as:

$$
\delta = \phi(t) + \tan^{-1}\left(\frac{k e(t)}{v_f(t)}\right)
$$

(Reference: [Robotics StackExchange](https://robotics.stackexchange.com/questions/22989/what-is-wrong-with-my-stanley-controller-for-car-steering-control))

<img src="../../../images/control5.png" alt="Stanley Control Law" width="800"/>

The Stanley method uses vehicle kinematics, geometric path information, and heading angle to follow the target trajectory.  
It maintains lane centering effectively even on curved sections by compensating both **heading error** and **lateral error** simultaneously.  
However, at very low speeds, steering may become overly sensitive, and parameter **k** tuning can be challenging.

---

### 3.2 Integration with SCNN and Depth-based Coordinates

The Stanley controller used the **SCNN-based lane centerline** and **depth-estimated coordinates** as input.  
It calculates:
- **Heading error:** between the vehicle’s forward direction and the tangent of the path.  
- **Cross-track error:** lateral offset from the centerline.

By combining both errors, the controller generates the steering command δ, enabling stable trajectory tracking.

<img src="../../../images/control6.png" alt="Stanley Integration with SCNN and Depth" width="800"/>

---

### 3.3 Stateflow-based Stop Modeling (Quanser Specification)

To comply with competition specifications, **Stateflow** was used to model stop events.  
Two conditions trigger the stop logic:

1. **Signal/Sign-based Stop (YOLO integrated)**  
   - YOLO detects a traffic light or stop sign → sends a ROS flag (`stopsign == 1`).  
   - Stateflow transitions to **Stopping**, sets `output_speed = 0`, waits `after(t_sec)`, then resumes normal driving.

2. **Coordinate-based Stop (CoorStop)**  
   - MATLAB function `GetStopTarget()` calculates Euclidean distance between current and target positions.  
   - If `(stopIndex <= numStops)` and `(distance_to_target < threshold)`, transition to **CoorStop** → stop vehicle → resume after delay and increment `stopIndex`.

<img src="../../../images/control7.png" alt="Stateflow Stop Logic Diagram" width="800"/>

---

## 4. Simulation and Verification (MATLAB/Simulink)

A sine-wave path was generated in MATLAB to verify Stanley control performance —  
to confirm correct waypoint tracking, and the proper handling of stop/resume transitions.

### 4.1 Path Generation

<img src="../../../images/control20.png" alt="Sine Path Generation" width="400"/><br/>
<b>Sine Path Generation</b>
      
<img src="../../../images/control8.png" alt="Path Generation model" width="400"/><br/>
<b>Path Generation model</b>




---

### 4.2 Lateral Error
The controller outputs lateral error as a **vector**,  
since multiple predicted coordinates and error terms are computed across time.  
Thus, several error curves appear in the scope window.  
Overall, errors remained within acceptable bounds,  
with slightly lower accuracy observed near the end of the trajectory.

<img src="../../../images/control9.png" alt="Lateral Error" width="800"/>
<p align="center"><b>Lateral Error Trace</b></p>

---

### 4.3 Steering Angle
The steering signal was discretized using a **zero-order hold** block, producing stepwise commands.  
This prevents overly sharp changes that could stress the servo motor and destabilize control.  
The discrete output thus reflects **hardware-aware design**.

<img src="../../../images/control10.png" alt="Steering Angle Output" width="800"/>
<p align="center"><b>Discrete Steering Angle Output</b></p>

---

### 4.4 Vehicle Speed
Vehicle speed adjusted naturally according to path curvature —  
slower in sharp curves, faster in straight segments — confirming that control output properly reflects curvature sensitivity.

<img src="../../../images/control11.png" alt="Vehicle Speed" width="800"/>
<p align="center"><b>Vehicle Speed Profile</b></p>

---

### 4.5 Stateflow Speed Output
At **t = 20s**, a pulse stop signal was injected.  
The controller correctly stopped and resumed tracking afterward,  
demonstrating that it handled **YOLO-based** and **coordinate-based** stop logic properly.

<img src="../../../images/control12.png" alt="Stateflow Speed Output" width="800"/>
<p align="center"><b>Stop and Resume Event</b></p>

---

### 4.6 Zero-Order Hold and Discretization Verification
All control signals were processed via **zero-order hold** to output discrete command values.  
This design minimizes excessive motor stress and ensures safe actuator operation.  
Through Simulink scopes, all control variables were verified to behave normally.

<img src="../../../images/control13.png" alt="Zero Order Hold Verification" width="400"/>

---

## 5. From Simulation to Real System

In the **Xytron simulator**, stable driving performance was achieved overall.  
However, in **QLabs**, the organizers reported a **Depth Camera malfunction**,  
preventing the intended **SCNN+Depth** coordinate system from being deployed.  
As a result, the final system used the **Pure Pursuit** controller for the competition.

---

## 6. Pure Pursuit Controller

### 6.1 Control Law
The Pure Pursuit steering angle is given by:

$$
\delta = \tan^{-1}\left(\frac{2L \sin(\alpha)}{l_d}\right)
$$

(Reference: [ShuffleAI Blog](https://www.shuffleai.blog/blog/Three_Methods_of_Vehicle_Lateral_Control.html))

<img src="../../../images/control14.png" alt="Pure Pursuit Control Law" width="400"/>

The controller computes the curvature toward the **look-ahead point**,  
producing smooth trajectories and robust high-speed tracking.  
It is simple and computationally efficient, though **look-ahead distance** tuning strongly affects performance.

---

### 6.2 Simulink Implementation
The Pure Pursuit block received:
- **Pose (x, y, θ)** from Cartographer SLAM  
- **Waypoints** from the global planner  

and output **linear velocity** and **steering angle** commands.

<img src="../../../images/control15.png" alt="Pure Pursuit Simulink Block" width="800"/>

---

## 7. Performance Improvement and Results

To enhance performance, two main strategies were applied:

### 7.1 Adaptive Look-Ahead Distance
- Straight segments: **0.1 m** → minimizes unnecessary steering, maintains stability at high speed.  
- Curved segments: **3 m** → improves curve tracking.

<img src="../../../images/control16.png" alt="Adaptive Look-Ahead Distance" width="800"/>

---

### 7.2 Steering Saturation
- Straight segments: **±0.13 rad**  
- Curved segments: **±0.35 rad**  
→ prevents over-aggressive steering and ensures actuator stability.

<img src="../../../images/control17.png" alt="Steering Saturation Settings" width="800"/>

---

### 7.3 Results
After implementing these improvements,  
the system exhibited reduced output spikes and maintained smooth motion.


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


Both **simulation and real-world experiments** confirmed that Pure Pursuit achieved stable performance across low- and high-speed, curved, and straight sections.  
Adaptive look-ahead tuning and saturation settings significantly reduced steering jitter and improved driving stability.

---

## 8. Conclusion

This study compared and implemented **PID**, **Stanley**, and **Pure Pursuit** controllers.  
While PID was sufficient for basic low-speed tasks,  
Stanley and Pure Pursuit provided superior performance in **high-speed** and **tight-curve** scenarios.  

Stanley’s geometric correction and Stateflow-based event logic offered strong path adherence,  
whereas Pure Pursuit achieved smooth, continuous steering with lower computational cost.  

Ultimately, due to hardware constraints in QLabs, **Pure Pursuit** was selected for final deployment,  
demonstrating stable performance and precise tracking in both simulation and real driving tests.

Future work will involve reintegrating **SCNN+Depth-based** lane input,  
and exploring **MPC** for adaptive, constraint-aware trajectory control once full sensor synchronization is available.
