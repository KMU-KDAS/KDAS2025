# KDAS2025

# Kookmin Autonomous Driving Project (KMU-KDAS)

<img src="images/kdasmain.gif" alt="KDAS" width="800"/>

The **Kookmin Autonomous Driving System (KMU-KDAS)** is a full-stack autonomous RC-car platform  
developed for the **Quanser Autonomous Car Competition (ACC)**.  
This repository documents our journey from basic vehicle dynamics simulation to a fully integrated  
ROS2-based AI driving system running on the QCar2 hardware.

---

## 1. Overall Results (GIF Summary)

Below are the final results showing each module operating in real-time.
Click on each caption to open the corresponding folder in `Product/`.

<table>
  <tr>
    <td align="center">
      <img src="images/SCNN1.gif" width="300"><br>
      <b><a href="Product/Decision/SCNN/">Lane Detection (SCNN)</a></b>
    </td>
    <td align="center">
      <img src="images/yolo10.gif" width="300"><br>
      <b><a href="Product/Decision/YOLO/">Object Detection (YOLO)</a></b>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="images/abdq9.gif" width="300"><br>
      <b><a href="Product/Decision/path%20planning/">Path Planning (RRT + DQN)</a></b>
    </td>
    <td align="center">
      <img src="images/control19.gif" width="300"><br>
      <b><a href="Product/Control/">Control (Pure Pursuit)</a></b>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="images/ROS1.gif" width="300"><br>
      <b><a href="Product/ROS/">ROS2 System Integration</a></b>
    </td>
    <td align="center">
      <img src="images/SCNN11.gif" width="300"><br>
      <b><a href="Product/Decision/SCNN/">Lane Tracking Visualization</a></b>
    </td>
  </tr>
</table>

---

## 2. Development Story

### **Step 1. Modeling and Simulation**
Our work began by modeling vehicle dynamics in MATLAB/Simulink.  
Using [AEB Dynamics](Product/Dynamics/AEB_Dynamics/) and [QCar Dynamics](Product/Dynamics/QCAR_Dynamics/),  
we replicated longitudinal and lateral motion to design braking and steering control strategies.  

Manual PID tuning proved unstable under varying loads, leading to the development of  
an automatic tuning pipeline and an [MPC controller](Product/Control/MPC/) for time-to-collision (TTC)-based braking.

> **Result:** Stable braking under dynamic conditions using MPC.

---

### **Step 2. Real-Vehicle Implementation**
We transitioned from simulation to hardware control.  
Using STM32 and VESC, the [Pure Pursuit controller](Product/Control/Steering%20Control%20&%20Breaking/)  
was validated as the final steering algorithm due to its robust performance at low speeds.

<img src="images/control18.gif" width="400"/>

Early tests with Stanley control led to oscillations and understeer;  
Pure Pursuit resolved this by aligning steering with dynamic lookahead geometry.

> **Result:** Smooth and stable real-vehicle steering control.

---

### **Step 3. Perception (Sensor Integration)**
To perceive the environment, we integrated:
- LiDAR (RPLIDAR A2M12) for 360° mapping,
- RGB-D (RealSense D435i) for semantic perception, and
- Encoders for precise odometry.

[Sensor setup details](Product/Sensors/) explain the ROS2 topic design and fusion rationale.

> **Problem:** Single-sensor reliability was limited under reflections and shadows.  
> **Solution:** Multi-sensor fusion ensured consistent performance under varying conditions.

---

### **Step 4. Localization**
Accurate localization was vital for stable planning.  
We evaluated **AMCL** and **Cartographer** in [Product/Localization/](Product/Localization/).

AMCL showed drift and inconsistency, while Cartographer provided  
real-time loop closure and map correction using direct LiDAR scan matching.

<img src="images/local3.png" width="400"/>

> **Result:** Cartographer adopted as the default SLAM engine with ±2 cm average pose error.

---

### **Step 5. Deep-Learning-Based Perception**
To extract semantics from images, three deep models were integrated:

- **[SCNN](Product/Decision/SCNN/)** — Lane detection and centerline generation for tracking control.  
  <img src="images/SCNN1.gif" width="400"/>

- **[YOLOv8s](Product/Decision/YOLO/)** — Real-time detection of traffic lights, signs, and obstacles.  
  <img src="images/yolo11.gif" width="400"/>

- **[LSTM](Product/Decision/Deep%20Learning/CAN_DATA(LSTM)/)** — Future-state prediction for lateral deviation and curvature.

> **Outcome:** Integrated perception pipeline achieved 45 FPS and enabled proactive control decisions.

---

### **Step 6. Path Planning and Decision Making**
Our decision layer combined classical and reinforcement learning planners.

- **[RRT](Product/Decision/path%20planning/RRT/)** — Generates safe corridors between predefined waypoints.  
- **[DQN](Product/Decision/Reinforcement%20Learning/DQN/)** — Learns waypoint order to minimize travel distance and time.  
- **[PH Curve / TG](Product/Decision/path%20planning/)** — Smooths the jagged RRT paths into continuous, drivable curves.

<img src="images/abdq8.gif" width="400"/>

> **Result:** PH-based smoothing reduced lateral jerk and produced collision-free avoidance paths.

---

### **Step 7. ROS2 Integration**
All components—SLAM, perception, planning, and control—were unified through ROS2.  
Each node communicates through a structured topic tree with real-time QoS settings.  
The launch configuration in [Product/ROS/](Product/ROS/) coordinates all systems.

<img src="images/ROS1.gif" width="400"/>

> **Outcome:** End-to-end autonomous driving achieved in real time on QCar2.

---

## 3. Summary

The KDAS2025 project evolved from simulation-based control experiments  
into a fully integrated AI-driven autonomous vehicle system.  
Each step built upon the last—solving control instability, improving localization,  
enhancing perception through deep learning, and refining planning through DQN and PH-curve smoothing—  
culminating in robust, real-time autonomous driving under ROS2.

All code, data, and detailed documentation are contained under the respective folders in `Product/`.
