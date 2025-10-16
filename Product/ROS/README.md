# **ROS2-Based Integrated Network Design**

---

## 1. Overview

To ensure reliable integration of various **sensors** and **control modules**, our project adopted **ROS2** as the primary middleware framework.

While **ROS1** (approaching its End of Life, *EOL*) suffered from key structural limitations—most notably, system-wide dependency on a central **master node (roscore)**—**ROS2** overcomes this by using a **DDS (Data Distribution Service)**-based distributed architecture. This enables high system stability, modular scalability, and flexible message routing.

Additionally, ROS2’s **QoS (Quality of Service)** profiles allow fine-grained control of **data reliability** and **real-time responsiveness**, tailored to each topic’s purpose (e.g., control feedback vs. sensor data).

Among available distributions, we selected **ROS2 Humble Hawksbill** instead of the newer **Jazzy Jalisco**. Although Jazzy introduced new features, it was still in an early release phase with compatibility limitations, whereas Humble provided extensive documentation, community support, and a stable toolchain. This decision enabled a **robust, stable integrated network** across all modules.

<img src="../../images/ROS1.gif" alt="ROS2 Network Overview" width="800"/>

---

## 2. QoS Optimization Strategy

To achieve both **real-time control** and **loss-free communication**, different QoS policies were configured per topic:

| **Topic**            | **Purpose**                 | **QoS Policy**                                  | **Reasoning**                                                         |
|----------------------|-----------------------------|-------------------------------------------------|-----------------------------------------------------------------------|
| `/current_location`  | Vehicle position feedback   | **Keep Last (1)** + **Best Effort**            | Always deliver the newest pose to the control loop with minimal lag.  |
| `/waypoints`         | Path planning & control     | **Reliable** + **Keep All** + **Transient Local** | Prevent waypoint loss; keep history available even after late joins.  |

This achieved **low-latency pose feedback** and **loss-free path delivery** simultaneously. In real driving tests, the ROS2 framework remained stable even under complex network conditions.

---

## 3. Simulink Integration and Helper Node

<img src="../../images/Ros2.gif" alt="Simulink and Helper Node Integration" width="800"/>

To simplify the ROS2–Simulink interface (and avoid custom msg overhead), we used **`geometry_msgs/Point`** to transmit commands as:

```
(Velocity, Steering Angle, 0)
```

### Helper Node — Core Functions
- Sequential **transmission of waypoints**
- Sending **drive mode** based on path context
- Delivering **PID gains** to the motor controller
- Sending **stop signals** at destination

This structure ensured **timely, reliable, context-aware** data flow and improved overall control stability.

---

## 4. Physical Deployment

For on-vehicle deployment, we used **MATLAB Code Generation** to build **standalone executables** for the target controller (different chipset/OS from the laptop). The **ROS2-based control** verified in simulation was reproduced identically on the physical platform, confirming **consistency and reliability** across the pipeline.

---

## 5. Summary

- **Framework:** ROS2 Humble Hawksbill  
- **Architecture:** DDS-based distributed system with topic-wise QoS  
- **Integration:** Simulink controller + `geometry_msgs/Point`  
- **Helper Node:** waypoints, drive mode, PID gains, stop signals  
- **Deployment:** MATLAB Coder standalone on RC vehicle  
- **Outcome:** Unified, reliable, real-time communication for the entire system
