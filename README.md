# KDAS2025

# Kookmin Autonomous Driving Project (KMU-KDAS)

<img src="image/kdasmain.gif" alt="KDAS" width="800"/>

This is the main project repository aggregating **all deliverables under `Product/`** for the Quanser Autonomous Car Competition (ACC).  
Each module lives in its own folder (no “Simulation” mega-folder; **Control** and **ROS2** remain independent).  
This README provides quick previews (GIFs) and deep links to every component.

---

## Whole Project (GIF Previews)

Click each caption to jump to the corresponding module folder under `Product/`.

<table>
  <tr>
    <td align="center">
      <img src="SCNN1.gif" width="300"><br>
      <b><a href="Product/Decision/SCNN/">SCNN – Lane</a></b>
    </td>
    <td align="center">
      <img src="SCNN11.gif" width="300"><br>
      <b><a href="Product/Decision/SCNN/">SCNN – Lane (Alt)</a></b>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="yolo9.gif" width="300"><br>
      <b><a href="Product/Decision/YOLO/">YOLO – Detection #1</a></b>
    </td>
    <td align="center">
      <img src="yolo10.gif" width="300"><br>
      <b><a href="Product/Decision/YOLO/">YOLO – Detection #2</a></b>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="yolo11.gif" width="300"><br>
      <b><a href="Product/Decision/YOLO/">YOLO – Detection #3</a></b>
    </td>
    <td align="center">
      <img src="control18.gif" width="300"><br>
      <b><a href="Product/Control/">Control – Pure Pursuit #1</a></b>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="control19.gif" width="300"><br>
      <b><a href="Product/Control/">Control – Pure Pursuit #2</a></b>
    </td>
    <td align="center">
      <img src="abdq8.gif" width="300"><br>
      <b><a href="Product/Decision/path%20planning/">RRT/DQN – Planning #1</a></b>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="abdq9.gif" width="300"><br>
      <b><a href="Product/Decision/path%20planning/">RRT/DQN – Planning #2</a></b>
    </td>
    <td align="center">
      <img src="ROS1.gif" width="300"><br>
      <b><a href="Product/ROS/">ROS2 – System Integration #1</a></b>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="Ros1.gif" width="300"><br>
      <b><a href="Product/ROS/">ROS2 – System Integration #2</a></b>
    </td>
    <td align="center">
      <img src="abdq8.gif" width="300" style="visibility:hidden"><br>
      <b style="visibility:hidden">placeholder</b>
    </td>
  </tr>
</table>

> Note: AEB still images (e.g., `aebd*.jpg/.jpeg/.png`) are available in the AEB folders; no AEB GIF is published yet.

---

## Project Modules Overview (Deep Links to Small Components)

All modules are organized **inside `Product/`**.  
Below are direct links to **small components** (algorithms, models, docs).

### Perception (Sensor Suite)
- LiDAR (RPLIDAR A2M12): [Product/Sensors/](Product/Sensors/)
- RGB-D (Intel RealSense D435i): [Product/Sensors/](Product/Sensors/)
- Encoder & Tachometer: [Product/Sensors/](Product/Sensors/)
- Sensor-fusion rationale & ROS2 topics: [Product/Sensors/](Product/Sensors/)

### Localization
- Cartographer SLAM (mapping & localization): [Product/Localization/](Product/Localization/)
- AMCL comparison notes: [Product/Localization/](Product/Localization/)

### Planning & Decision
- **SCNN (lane)**: [Product/Decision/SCNN/](Product/Decision/SCNN/)
- **YOLO (objects/signs/lights)**: [Product/Decision/YOLO/](Product/Decision/YOLO/)
- **LSTM (lateral error / curvature prediction)**: [Product/Decision/Deep%20Learning/CAN_DATA(LSTM)/](Product/Decision/Deep%20Learning/CAN_DATA(LSTM)/)
- **RRT**: [Product/Decision/path%20planning/RRT/](Product/Decision/path%20planning/RRT/)
- **DQN (waypoint ordering)**: [Product/Decision/Reinforcement%20Learning/DQN/](Product/Decision/Reinforcement%20Learning/DQN/)
- **PH curve / TG (avoidance trajectory)**: [Product/Decision/path%20planning/](Product/Decision/path%20planning/)

### Control (separate from ROS2)
- Pure Pursuit (final controller): [Product/Control/Steering%20Control%20&%20Breaking/](Product/Control/Steering%20Control%20&%20Breaking/)
- MPC: [Product/Control/MPC/](Product/Control/MPC/)
- Kalman Filter (EKF etc.): [Product/Control/Kalman_Filter/](Product/Control/Kalman_Filter/)
- Avoidance Control: [Product/Control/Avoidance%20Control/](Product/Control/Avoidance%20Control/)
- Motor/ESC control: [Product/Control/Motor%20Control/](Product/Control/Motor%20Control/)

### Dynamics
- AEB dynamics model: [Product/Dynamics/AEB_Dynamics/](Product/Dynamics/AEB_Dynamics/)
- QCar dynamics model: [Product/Dynamics/QCAR_Dynamics/](Product/Dynamics/QCAR_Dynamics/)
- Docs: [AEB dynamics doc](Product/Dynamics/github_AEBdynamics.docx), [QCar dynamics doc](Product/Dynamics/github_QCarDynamics.docx)

### ROS2 (system integration only; not merged with Control)
- Launch files, node graph, topic schema: [Product/ROS/](Product/ROS/)

### Product (hardware/circuits live here too)
- Circuit & DC-DC: [Product/Circuit/](Product/Circuit/)
- VESC / ESC integration: [Product/VESC/](Product/VESC/)
- All other build artifacts also under `Product/`

---

## Notes

- There is **no unified “Simulation” folder** in this repository.  
  Simulation-related materials are distributed in their corresponding modules under `Product/` (e.g., AEB in Dynamics/Control, planning in Decision, etc.).
- **Control** and **ROS2** are **kept separate** by design. Control contains controllers and logic; ROS2 contains integration and launch.

---

## Directory Glimpse

