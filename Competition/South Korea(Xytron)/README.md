# Domestic Competition  (Hosted by Xytron)

---

## 1. Purpose & Significance

<img src="../../../images/xyt1.png" width="620" alt="Competition Overview"/>

The competition, organized by **Xytron Co., Ltd.**, aimed to promote the educational dissemination and industrial application of autonomous driving technology.  
It was held at **Kookmin University’s Autonomous Driving Studio**, where participants were evaluated using both **the Xytron Simulator** and **a real 1/10-scale autonomous vehicle**.  
The main goal of the competition was to provide participants with **hands-on, field-oriented autonomous driving development experience**.

---

## 2. Competition Overview

- **Organizer:** Xytron Co., Ltd.  
- **Venue:** Kookmin University Autonomous Driving Studio (Final Stage)  
- **Eligibility:** Undergraduate students from all domestic universities (graduates and postgraduates excluded)  
- **Participants:** 126 teams from 104 departments across 53 universities (~538 students)  
- **Competition Structure:**
  - **Stage 1 (Preliminary):** Virtual driving using the *Xytron Simulator*  
  - **Stage 2 (Final):** On-track evaluation using real 1/10-scale autonomous vehicles  
- **Finalists:** Top 25 teams qualified for the final stage  
- **Vehicle Requirement:** Teams could choose between Xytron’s provided vehicle or their own  
- **Our Team Platform:** Custom-built vehicle based on **Traxxas 4TEC 2.0**

---

## 3. Assigned Tasks

### Preliminary (Simulation Stage)

Teams were required to complete a full lap on the virtual track within the Xytron simulator while performing the following tasks:

<br/>

<img src="../../../images/xyt2.png" width="700" alt="Traffic Light and Lane Detection"/>

- **Traffic light recognition**  
- **Stable lane-keeping based on lane detection**

<br/>

<img src="../../../images/xyt3.png" width="700" alt="S-Cone Navigation"/>

- **Collision-free navigation through an S-shaped cone section**

<br/>

<img src="../../../images/xyt4.png" width="700" alt="Vehicle Overtaking"/>

- **Safe overtaking of two slower vehicles on a straight road**

### Final (Physical Stage)

<br/>

<img src="../../../images/xyt5.png" width="720" alt="Physical Stage Track Layout"/>

Teams drove a **1/10-scale vehicle** for **three laps** on the real test track.  
Performance was evaluated on **lane detection accuracy**, **cone collision avoidance**, and **vehicle overtaking capability**.

---

## 4. Our Approach

<br/>

<img src="../../../images/xyt6.png" width="740" alt="Simulation System Diagram"/>

- The Xytron simulator provided only a **front-facing 180° LiDAR** and **RGB camera**, disallowing the use of **SLAM** or **Waypoint-based planning**.  
- We implemented a **lightweight driving algorithm** combining **SCNN-based lane detection** with a **PID controller**.

<br/>

<img src="../../../images/xyt7.gif" width="740" alt="Traffic Cone Avoidance Simulation"/>

**Traffic Cone Avoidance:**  
Using LiDAR-based distance estimation, the vehicle calculated the midpoint between cones and adjusted its steering via PID control to pass safely without collision.

<br/>

<img src="../../../images/xyt8.gif" width="740" alt="Vehicle Avoidance Simulation"/>

**Vehicle Avoidance:**  
YOLO-based object detection identified slower vehicles ahead.  
When detected, the steering angle was gradually changed to execute a lane change maneuver.  
After passing the obstacle, the vehicle returned smoothly to its original lane.

- To operate the **ROS1-based simulator** in a **ROS2 environment**, we constructed a custom **ROS1–ROS2 bridge**.  
- All nodes were fully integrated within **ROS2 Humble**, ensuring system compatibility.  
- Despite limited sensor inputs, the system successfully completed the course without collisions and advanced to the **Top 25 (Final Round)** among 126 teams.

Detailed technical implementation and simulation results are documented under  
📁 **`Simulation_Result/Xytron_Simulation`**

---

### 🎥 Simulation Submission Video

<br/>

<img src="../../../images/xyt9.gif" width="760" alt="Simulation Result Demo"/>

**Content:** SCNN-based lane detection and PID-controlled driving demonstration  
**🔗 YouTube:** [https://youtu.be/zG0h3WG9YYc](https://youtu.be/zG0h3WG9YYc)

---

## 5. Participation Notes

- For the final round, our team used a **Traxxas 4TEC 2.0-based** autonomous vehicle.  
  Initially, teams could select between Xytron’s vehicle and their own, provided that it met the following criteria:  
  - 1/10-scale RC chassis  
  - 4-wheel **Ackermann steering** mechanism  

- However, during the **final stage**, Xytron officially designated its **in-house truck-type vehicle** (*Traxxas Slash 4X4 VXL*) as the **standard model** and released new vehicle requirements:

  | Parameter | Standard Spec | Our Vehicle (4TEC 2.0) | Difference |
  |------------|---------------|------------------------|-------------|
  | Wheelbase | 300 mm | 256 mm | –44 mm (≈15% shorter) |
  | Length | 500 mm | 379 mm | –121 mm (≈24% shorter) |
  | Width | 250 mm | 200 mm | –50 mm (≈20% narrower) |
  | Type | Truck (Off-road) | Sedan (On-road) | Structural mismatch |

<br/>

<img src="../../../images/xyt10.png" width="360" alt="Traxxas 4TEC 2.0"/>
<img src="../../../images/xyt11.png" width="360" alt="Traxxas Slash 4X4 VXL"/>

*(Left: Traxxas 4TEC 2.0 / Right: Traxxas Slash 4X4 VXL)*  

The chassis and body structures were fundamentally different.  
Hence, it was **structurally impossible** to adjust our sedan-type 4TEC 2.0 to meet the truck-type dimensions (wheelbase 300 mm · length 500 mm · width 250 mm) required for the final stage.

- Additionally, the **final preparation period** overlapped with  
  - the **ACC 2025 competition trip** (2–3 weeks abroad), and  
  - subsequent **school funding and administrative processing** tasks,  
  making physical track testing impossible.

Due to these operational changes and scheduling constraints, we **did not participate in the final round**.  
However, the developed system architecture and algorithms were later utilized as a technical reference in our ongoing research.

---

## 6. Results & Reflection

- During the preliminary stage, we achieved **complete task fulfillment** (traffic light detection, cone navigation, vehicle avoidance), securing **Top 25 / 126 teams**.  
- Despite limited sensors and simulation resources, we successfully implemented a **cost-efficient, lane-based autonomous driving framework**, proving its practicality under constrained conditions.  
- This project validated the importance of **lightweight design, sensor efficiency**, and **control stability** using a simplified system structure.

