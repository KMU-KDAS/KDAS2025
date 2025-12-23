<img src="images/kdasmain.gif" alt="KDAS" width="800"/>

# KDAS2025: Kookmin Autonomous Driving System

*Kookmin University Autonomous Driving Research Team – Alpha Project 2024–2025*

The **Kookmin Autonomous Driving System (KMU-KDAS)** is a full-stack autonomous RC-car platform developed for the **Quanser Autonomous Car Competition (ACC)**. This repository documents our journey from basic vehicle dynamics simulation to a fully integrated ROS2-based AI driving system running on the QCar2 hardware.

---

## 1. Overall Results 

Below are the final results showing each module operating in real time. Click on each caption to open the corresponding folder under `Product/`.

<table>
  <tr>
    <td align="center">
      <img src="images/SCNN1.gif" width="300"><br>
      <b><a href="Product/Perception/SCNN/">Lane Detection (SCNN)</a></b>
    </td>
    <td align="center">
      <img src="images/yolo10.gif" width="300"><br>
      <b><a href="Product/Perception/YOLO/">Object Detection (YOLO)</a></b>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="images/rrt1.png" width="300"><br>
      <b><a href="Product/Decision/Path%20Planning/">Path Planning (RRT + DQN)</a></b>
    </td>
    <td align="center">
      <img src="images/SCNN11.gif" width="300"><br>
      <b><a href="Product/Perception/SCNN/">Lane Tracking Visualization</a></b>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="images/control19.gif" width="300"><br>
      <b><a href="Product/Control/Steering Control & Breaking Control/">Control QCar</a></b>
    </td>
    <td align="center">
      <img src="images/control18.gif" width="300"><br>
      <b><a href="Product/Control/Steering Control & Breaking Control/">Control simulation</a></b>
    </td>
  </tr>
</table>

---

## 2. Development Story and Repository Overview

This section summarizes the final system modules, representative figures, and links them to the directory structure under `Product/`. It complements the detailed narrative in the later sections.

### 2.1 Overall Results 

**Repository:** `Product/`

The final system integrates perception, planning, control, and ROS2 infrastructure. Each main module runs in real time on QCar2.

**Module entries:**

- Lane Detection (SCNN): `Product/Decision/SCNN/`  
- Object Detection (YOLO): `Product/Decision/YOLO/`  
- Path Planning (RRT + DQN): `Product/Decision/path_planning/`  
- Control (Pure Pursuit): `Product/Control/`  
- ROS2 System Integration: `Product/ROS/`  

---

### 2.2 Step 1 – Modeling and Simulation

<img src="images/control18.gif" width="450" alt="Modeling & Simulation (AEB/QCar Dynamics)"/>

**Repository:**  
- `Product/Dynamics/`  
- `Product/Control/MPC/`

We began by modeling vehicle dynamics in MATLAB/Simulink:

- **AEB Dynamics:** `Product/Dynamics/AEB_Dynamics/`  
- **QCar Dynamics:** `Product/Dynamics/QCAR_Dynamics/`

These models allowed us to replicate longitudinal and lateral vehicle motion, design braking strategies, and explore control methods.

Manual PID tuning proved unstable under varying loads. This led to the development of:

- an automatic tuning pipeline, and  
- an MPC controller at `Product/Control/MPC/` for TTC-based braking.

**Result:** stable braking under dynamic conditions using MPC.

---

### 2.3 Step 2 – Real-Vehicle Implementation

<img src="images/control19.gif" width="450" alt="Real Vehicle Control (Pure Pursuit on QCar2)"/>

**Repository:**  
- `Product/Control/Steering_Control_&_Braking/`

We transitioned from simulation to real hardware control using **STM32** and **VESC**.

The **Pure Pursuit controller** in `Product/Control/Steering_Control_&_Braking/` was validated as our final steering algorithm due to its robust low-speed performance and smooth path tracking.

Initial tests with Stanley revealed oscillations and understeering issues at higher speeds, whereas Pure Pursuit aligned steering with lookahead geometry and provided stable behavior.

**Result:** smooth and stable real-vehicle steering control.

---

### 2.4 Step 3 – Perception (Sensor Integration)

<img src="images/SCNN10.gif" width="450" alt="Perception"/>

**Repository:**  
- `Product/Sensors/`

To perceive the environment, we integrated:

- **RPLiDAR A2M12** for 360° mapping,  
- **RealSense D435i** for RGB-D semantics, and  
- **encoders** for odometry.

The sensor setup is described in `Product/Sensors/`, where we detail ROS2 topics and fusion rationale.

- **Problem:** reliability of single sensors degraded under reflections and shadows.  
- **Solution:** multi-sensor fusion improved robustness across varying conditions.

---

### 2.5 Step 4 – Localization

<img src="images/local2.jpg" width="450" alt="Localization – Cartographer Map and Trajectory"/>

**Repository:**  
- `Product/Localization/`

Accurate localization was essential for stable planning. We evaluated:

- **AMCL**, and  
- **Cartographer**

under `Product/Localization/`.

AMCL suffered from drift and map dependency, while Cartographer offered:

- real-time loop closure,  
- map correction, and  
- robust LiDAR-based scan matching.

**Result:** Cartographer was selected as the default SLAM backend, achieving pose errors on the order of centimeters.

---

### 2.6 Step 5 – Deep Learning-Based Perception (SCNN / YOLO / LSTM)

<table>
  <tr>
    <td align="center">
      <img src="images/SCNN10.gif" width="260"><br>
      <b><a href="Product/Perception/SCNN/">SCNN – Lane Detection</a></b>
    </td>
    <td align="center">
      <img src="images/SCNN11.gif" width="260"><br>
      <b><a href="Product/Perception/SCNN/">SCNN – Lane Tracking</a></b>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="images/yolo9.gif" width="260"><br>
      <b><a href="Product/Perception/YOLO/">YOLOv8s – Object Detection</a></b>
    </td>
    <td align="center">
      <img src="images/yolo11.gif" width="260"><br>
      <b><a href="Product/Perception/YOLO/">YOLOv8s – Traffic Light & Sign</a></b>
    </td>
  </tr>
</table>

**Repository:**

- SCNN: `Product/Decision/SCNN/`  
- YOLOv8s: `Product/Decision/YOLO/`  
- LSTM: `Product/Decision/Deep_Learning/CAN_DATA(LSTM)/`

We integrated three deep models:

- **SCNN** for lane detection and centerline generation,  
- **YOLOv8s** for real-time traffic light, sign, and obstacle detection, and  
- **LSTM** for future-state prediction (e.g., lateral deviation and curvature).

This pipeline enabled proactive control decisions and operated at real-time frame rates.

---

### 2.7 Step 6 – Path Planning and Decision Making

<table>
  <tr>
    <td align="center">
      <img src="images/rrt1.png" width="260"><br>
      <b><a href="Product/Decision/Path%20Planning/RRT/">RRT </a></b>
    </td>
    <td align="center">
      <img src="images/rrt10.png" width="260"><br>
      <b><a href="Product/Decision/Path%20Planning/">RRT </a></b>
    </td>
  </tr>
</table>

**Repository:**  
- `Product/Decision/path_planning/`

Our decision layer combined:

- **RRT:** `Product/Decision/path_planning/RRT/` for safe corridor generation between waypoints,  
- **DQN:** `Product/Decision/Reinforcement_Learning/DQN/` for learning waypoint ordering to minimize travel cost, and  
- **PH Curve / Trajectory Generation:** `Product/Decision/path_planning/` for smoothing jagged RRT paths into continuous curves.

**Result:** PH-based smoothing reduced lateral jerk and produced collision-free avoidance paths suitable for real-time control.

---

### 2.8 Step 7 – ROS2 Integration

<img src="images/Ros1.gif" width="450" alt="ROS2 Integration – Full Stack on QCar2"/>

**Repository:**  
- `Product/ROS/`

All components—SLAM, perception, planning, and control—were unified via ROS2. Nodes communicate through a structured topic tree with carefully tuned QoS settings.

The launch configuration in `Product/ROS/` orchestrates the full system, enabling end-to-end autonomous driving on QCar2 in real time.

**Outcome:** robust, real-time autonomous driving achieved on QCar2 with the full stack under ROS2.

---

## 3. Project Overview and Motivation


> “Can we implement the things we learned in class as industry-level technology?”

This question was the starting point of the Alpha Project.

As students majoring in automotive engineering, we felt that we should at least be able to build a small platform and implement autonomous driving on it. Simply taking courses and receiving grades was not enough; we wanted to see control, circuits, artificial intelligence, and modeling—concepts learned in lectures—embedded in a real, working vehicle.

This project was an experiment to answer the question:

> “Is it possible to build an industry-level system using only what we have learned?”

We decided to turn the limitations of an undergraduate “student project” into the very conditions of our challenge. Instead of staying inside the safe frame of a domestic student competition, we held ourselves to the same standards as research-level autonomous driving projects carried out by university teams worldwide. A key principle of the team was to evaluate our results from a **global perspective**.

Our final goal was to implement an autonomous vehicle and achieve measurable results in actual competitions.

At first, the task seemed overwhelming. We knew abstract concepts like **Perception**, **Path Planning**, and **Control**, but had no clear idea how these subsystems interact and cooperate inside a real vehicle. Moreover, even after deciding to implement them, we were unsure which technology stack and algorithms to choose among so many possibilities.

To resolve this uncertainty, we decided to **continuously test theory in real environments**, observing its limitations and potential firsthand. We did not merely apply the concepts learned in class; we supplemented our theoretical foundation by analyzing research papers and technical documents, and then chose methods that fit our constraints and resources.

Some technologies we encountered were difficult to handle directly at the undergraduate level. However, we did not simply abandon them. Instead, we simplified their structure within an accessible range or looked for functionally equivalent alternatives. At the same time, we made a constant effort to understand **why** such approaches were necessary and **what principles** they relied on, organizing that theoretical rationale and reflecting it in practical implementation.

Whenever issues emerged during implementation, we analyzed the cause and iterated through a cycle of testing and modification to find solutions. Through this process, we gradually evolved from “people who understand theory” into “engineers who can implement it.”

The core of this project was the belief that we could build real technology with what we had learned—and the determination to prove that belief through a complete autonomous driving system.



---

## [4. First Challenge – Motor Control (PID Autotuning)](Product/Control/Motor%20Control/)

<img src="images/control18.gif" width="450" alt="Motor Control – PID Autotuning in Simulink"/>


> “What does it actually mean to ‘move’ a vehicle?”

Our first concern was how to make the vehicle move. For the wheels to rotate, the motor must be controlled; therefore, the starting point of driving was **precise motor control**. We realized that the first step toward implementing autonomous driving had to begin with **motor control**.

The most realistic control strategy available to us at that time was **PID control**, which we had learned in the “Automotive Integration Experiment” course. PID has a simple structure yet offers fast response, making it suitable for stable control in limited computational environments.

However, even though the theory appears simple, in a real system small changes in the PID gains can drastically alter the response characteristics. Fine-tuning was essential.

To apply PID to a realistic motor model, we referred to the **Control Tutorials for MATLAB and Simulink** provided by the University of Michigan. Based on the Motor Control section, we used **Simscape Electrical** to build a virtual model of our DC motor.

Modeling alone did not allow us to directly apply PID, because:

- **Simulink** uses a *signal-flow* computational model, while  
- **Simscape** uses a *physical energy-flow* model.

Their computational structures and time scales are fundamentally different.

After recognizing that the two environments operate on different computation paradigms, we explored ways to connect them. Through MathWorks interface documentation, we identified the **PS Converter** block as the critical bridge between signal-based and physical domains. By integrating this block, we established a unified framework in which both environments could exchange data on a shared time base.

Only after this integration was complete could we apply our designed PID controller to the virtual motor model.

However, the tuning process revealed new issues. Because PID responses change significantly with even small gain adjustments, we had to repeat the cycle of “adjust gain → observe response” many times. Even then, it was difficult to be confident that the chosen gains were truly optimal.

<table>
  <tr>
    <td align="center">
      <img src="images/PID3.png" alt="Motor Voltage Response" width="400"/><br/>
      <b>Voltage Graph</b>
    </td>
    <td align="center">
      <img src="images/PID4.png" alt="Motor Speed Response" width="400"/><br/>
      <b>Motor Speed</b>
    </td>
  </tr>
</table>

To overcome this inefficiency and uncertainty, we searched for a more systematic method to derive gains and adopted **PID Autotuner**. PID Autotuner computes gains based on the actual system response, providing more stable and consistent control performance compared to manual tuning.

---

## [5. AEB System Design and Simulation – Matlab_AEB_Unreal](Simulations/Matlab%20-%20Unreal%20Simulation/)

<img src="images/AEB.gif" alt="AEB Simulation" width="800"/>

> “If we can move the vehicle, how do we make it stop safely?”

Once we had implemented forward motion via motor control, the next task was **stopping in dangerous situations**. This is where **AEB (Autonomous Emergency Braking)** comes in—a system that allows the vehicle to detect hazards and perform deceleration or full braking at the right moment.

However, testing a control algorithm that assumes potential collisions in a real vehicle poses obvious risks: vehicle damage, sensor failure, and high cost. We therefore needed a **virtual environment** capable of replacing real-world crash scenarios.

To this end, we integrated **MATLAB/Simulink** with **Unreal Engine 3**, building a 3D simulation environment where the AEB controller could be tested and validated. In this environment, heterogeneous elements such as:

- discrete-time sensors,
- fixed-step controllers, and
- continuous-time vehicle dynamics

could be aligned on a single time axis, closely approximating real driving conditions.

<img src="images/Overview1.png" width="900" alt="AEB System"/>

The essence of AEB design lies not in “braking itself,” but in determining **when** to start braking—that is, in **prediction**.

We used the relative distance and velocity between the ego vehicle and the object ahead to compute **TTC (Time-To-Collision)**. We then designed a multi-stage braking logic that increased braking force stepwise as TTC approached critical thresholds:

- Large TTC → sufficient time until collision → overly aggressive braking would destabilize vehicle motion.  
- Small TTC → imminent collision → strong braking must be applied immediately.

Based on this relationship, we divided braking into multiple stages and increased braking strength gradually as risk increased. This simple, structured approach was highly suitable for real-time control.

The braking logic was implemented in **Simulink Stateflow**. We defined states such as **Partial Brake (Stage 1 & 2)** and **Full Brake**, and visualized transition conditions, allowing us to clearly observe and validate the initiation and release of braking.

However, MATLAB-based 3D environments have limitations in modeling complex scenarios such as slopes, curved roads, and varying friction. In future work, more specialized simulators like **CARMAKER** or **CARLA** could be used to further develop and refine our AEB system.

Through these experiments, we came to understand that AEB is not a simple braking rule, but a complex system combining:

- mathematical modeling for TTC computation,
- organization of diverse input signals,
- controller design,
- unified time-base integration of vehicle and sensor models, and
- linking to 3D simulation environments.

Most importantly, we implemented the entire pipeline—from modeling to visualization—directly in MATLAB, which we consider one of the core achievements of this project.

---

## 6. Dynamics Modeling – [AEB_Dynamics](Product/Dynamics/AEB%20Dynamics/) & [QCar_Dynamics](Product/Dynamics/Qcar%20Dynamics/)

<img src="images/dyn1.png" width="900" alt="Vehicle Dynamics – Longitudinal/Lateral Modeling"/>

As we experimented with more and more control structures, one question kept coming back:  
sensor outputs alone were not enough to fully control the vehicle’s motion.

Yet every autonomous driving algorithm fundamentally assumes:

> “Given the steering and throttle I apply now,  
> what kind of motion will this create in the future?”

In other words, on top of the **current measurements** from sensors,  
we needed a **predictive understanding of how the vehicle can actually move** in physical terms.  
This is where our dynamics modeling began.

---
<img src="images/aebd2.jpeg" width="600" alt="Modeling & Simulation (AEB/QCar Dynamics)"/>

### 6.1 First Model – Understanding “How a Car Moves” with the Bicycle Model

In the early phase, we used MATLAB’s built-in **Bicycle Vehicle Model**.  
This model is based on the behavior of a typical gasoline vehicle and provides a good reference structure for understanding:

- tire–road interaction, and  
- basic longitudinal and lateral vehicle dynamics.

Our reasons for using this model were straightforward:

- it is already a **validated physical model**,  
- it allowed us to quickly understand vehicle dynamics block structures, and  
- it was well suited for learning **“by what principles does a vehicle actually move?”**

Because our test track was a **flat surface**,  
we ignored the z-axis (vertical motion) from the original 3DOF structure  
and used a simplified **2DOF (x–y plane)** configuration.

However, this model is fundamentally built for a **full-size gasoline vehicle**.  
So while it was ideal for learning general “vehicle physics,”  
it did not accurately reflect the actual behavior of **QCar, the platform we had to control in practice**.

That realization became the starting point for building a **QCar-specific dynamics model (`QCar_Dynamics`)**.

---

### 6.2 Why We Rebuilt QCar Dynamics – When the Platform Changes, the Physics Change Too

Quanser’s **QCar** is structurally very different from a conventional car:

- BLDC motor + ESC-based drive
- no mechanical brake system
- a 1/10-scale platform with low mass and low speed
- rolling resistance and motor response are much more prominent

Because of these structural differences,  
the standard Bicycle Model alone could not properly explain QCar’s:

- speed changes,  
- deceleration patterns, or  
- slip behavior.

We therefore revisited the underlying physics and selected only those relationships that were:

> “strictly necessary to determine how QCar actually moves.”

Using these, we built a separate **`QCar_Dynamics`** block.

---

### 6.3 Criteria for Selecting QCar Physics – To Build “Predictable Behavior”

We examined many physical equations to construct QCar Dynamics,  
but we were not trying to model “every physical phenomenon.”  
What we really needed were only the relationships that are:

> essential to understanding the next motion of the vehicle.

Our key criteria were:

#### ① ESC Duty → Voltage → Current → Motor Torque → Wheel Rotation → Vehicle Speed

The most critical characteristic of QCar is that  
**changes in vehicle speed are almost directly tied to changes in ESC duty**.

So this entire transfer chain had to be part of the model.  
Without it, we would have no way to interpret *why* speed changes occur.

#### ② Deceleration Comes from “Resistive Forces”, Not Brakes

QCar has no dedicated brake system.  
Deceleration is produced by a combination of:

- motor back-torque,  
- rolling resistance, and  
- aerodynamic drag.

We had to include these terms so that modules like SLAM, evasive maneuvers, and controllers could judge whether a given speed value is

> **“physically feasible in this situation.”**

#### ③ On a Low-Mass Platform, Small Slip Still Matters

<img src="images/aebd4.png" alt="Tire Slip" width="800"/>

On a 1/10-scale, low-mass platform,  
even small amounts of slip can produce meaningful differences  
between the true vehicle speed and the speed reported by sensors.

So instead of completely ignoring slip,  
we chose to include a **simplified slip model** inside our dynamics block.

These are only part of the full model, but they share one common property:

> they are precisely the elements needed to explain  
> “how QCar actually moves in the real world.”

With this selection strategy, we were able to:

- remove unnecessary complexity, while  
- building a structure where the vehicle’s reaction to the next input is **predictable**.

---

### 6.4 What the Dynamics Model Gave Us – A Basis for Understanding “The Next Motion”

Dynamics modeling was not just about:

> “making the simulation prettier or more detailed.”

For the first time, this model allowed us to explain in physical terms:

- how a given steering and throttle command  
  will change speed and position in the next moment, and
- how changes in ESC duty  
  translate into actual changes in vehicle speed and by what ratio.

In other words, the dynamics model became:

> a **starting point for predictive reasoning**,  
> letting us project “how the vehicle can move next”  
> on top of the sensor’s current readings.

This physical understanding later became the **common foundation** for:

- SLAM-based localization,  
- advanced controllers such as MPC, and  
- AEB and evasive maneuver algorithms.

It marked the transition of our system from one that simply **observes** sensor values  
to one that can **actively predict its own future motion**—  
and that transition began with this **dynamics modeling** step.


---

## [7. ROS2 Integration – Building a Unified System](Product/ROS/)

<img src="images/Ros2.png" width="450" alt="ROS2 Integration – Multi-Node System"/>

Once individual modules were functioning, our next task was to **integrate them into a single system**. The main difficulty was that each module operated on a completely different time scale:

- Control and sensor modules required millisecond-level real-time performance and were implemented in **C/C++**.
- Perception and inference modules operated on **Python**, designed under the assumption of hundreds of milliseconds of delay.

Synchronizing communication between these two worlds under a common timing framework was far from trivial.

Our initial idea was to unify all computation on **CUDA**, since the final deployment platform was an **NVIDIA Jetson** board. This seemed natural, but CUDA is a low-level GPU parallel computing framework, not a real-time robotics integration layer. Ensuring deterministic timing and implementing full sensor–actuator integration is structurally difficult and rarely practiced in robotics. We concluded that a CUDA-centric architecture was not realistic.

Next, we considered unifying everything under a **single programming language**:

- All in C/C++ → robust, but inhibits easy use of modern AI frameworks.  
- All in Python → simpler development, but sacrifices strict real-time behavior.

Moreover, simply unifying the language does not fundamentally solve synchronization and communication problems. So this approach also fell short.

The third attempt was to integrate everything under **MATLAB**:

- Sensors via MATLAB Toolboxes,  
- AI models via Deep Learning Toolbox,  

with the goal of operating all components inside a single environment.

However, two critical limitations appeared:

1. Models like SCNN and YOLO, which include **custom message-passing layers**, often failed during ONNX conversion due to structural conflicts or opset mismatches. They could not be loaded reliably into MATLAB, and there was no reliable way to test whether they were functioning correctly.
2. MATLAB Toolboxes focus on industrial-grade, widely used sensors. Supporting **RPLiDAR** and **RealSense** directly was either not possible or severely limited. We tried TCP-based communication as a workaround, but this made the system overly complex and introduced persistent data-format mismatches.

Through these experiences, we realized that MATLAB-based integration would be difficult to maintain for a full-scale autonomous driving stack.

At this point, we revisited the question:

> “What do real robotics projects use to integrate heterogeneous systems?”

The answer was **ROS (Robot Operating System)**.

ROS is a meta-operating system that:

- connects multiple languages,
- manages multiple sensors and nodes, and
- provides standardized communication patterns.

It inherently addresses many of the synchronization and communication issues we had faced. It also supports both **C++** and **Python** nodes, allowing each module to leverage its natural strengths.

With standardized components for sensor drivers, message types, and TF coordinate transforms, ROS allowed us to focus on higher-level logic instead of low-level plumbing.

In particular, **ROS2 Humble** uses a DDS-based QoS mechanism, which improved how we met real-time requirements among sensor, control, and inference modules.

Of course, ROS is not without drawbacks. As the system scales, DDS buffers can saturate, callback queues can introduce delays, and the broadcast-style architecture can burden network bandwidth. We mitigated these issues by carefully designing QoS profiles, differentiating topics with strict timing constraints from those requiring higher reliability.

As a result, the entire system finally formed a **cohesive, organically integrated architecture**.

---

## 8. Expanding the Goal – Toward Full Autonomous Driving

<img src="images/acc.png" width="450" alt="Quanser"/>

Our initial goal of building a fully autonomous system did not end with implementing individual functions. During AEB development, we encountered a crucial realization:

> Even a single function like braking only works correctly when **perception–decision–control** are tightly coupled.

To detect hazards, sensor interpretation must be stable. To determine braking timing, the vehicle state must be correctly estimated. Finally, in the control stage, a precise controller based on a realistic vehicle model is required.

This experience fundamentally changed our perspective. We realized that the essence of autonomous driving is not the completion of individual features but the **design of the entire stack**.

Naturally, a question arose:

> “Where does our stack stand right now?”

We wanted an **objective benchmark**, not just our own satisfaction. Domestic competitions were considered, but most focused on specific functions and did not evaluate the entire system holistically.

We sought an environment where perception, planning, and control could be viewed as a single system and evaluated against **industrial standards**. We concluded that such a structure was difficult to find domestically and turned our attention abroad.

We discovered the **Quanser Autonomous Car Competition (ACC)**, which provided exactly this type of environment.

<img src="images/quan.jpg" width="450" alt="Quanser Track"/>

The mission theme of ACC is the **autonomous taxi**. This is more than just moving a car; taxis must:

- determine precise stopping positions,
- replan paths in response to changing road conditions, and
- consider pedestrians, intersections, traffic lights, and surrounding vehicles simultaneously.

Even in scaled form, this corresponds to a demanding urban full-stack autonomous driving problem.

To solve such a problem, it is not enough to implement a few algorithms. One must integrate **sensor fusion**, **path planning**, **decision making**, and **controller design**.

ACC was exactly the **“full stack validation stage”** we were searching for.

The fact that the competition is run alongside the **American Control Conference (ACC)** also significantly influenced our decision. This is not just a student contest; it takes place in the same environment where researchers and experts from around the world present their latest work. We could:

- observe international research trends firsthand,  
- see which technologies are actively used in industry and academia, and  
- gain insight into which direction we should grow toward.

Our aim was not simply to obtain a good rank. We wanted to:

- elevate our **technical mindset**,  
- transition from function-level development to **system-level design**,  
- reinterpret team roles from a systems perspective, and  
- experience development and testing processes similar to those used in industry.

Ultimately, ACC showed us clearly **where we stood** and **how far we needed to go**. Experiencing international standards directly allowed us to concretely envision the direction and level of the autonomous driving system we needed to build.

---

## 9. Perception – Beginning to See the World ([SCNN](Product/Perception/SCNN/) & [YOLOv8s](Product/Perception/YOLO/))

<img src="images/SCNN11.gif" width="450" alt="SCNN – Lane Tracking in Real Time"/>

Autonomous driving starts with **seeing the world**. Even if a vehicle can move, it cannot make decisions or respond to its environment without perception. Building a perception module that acts as the vehicle’s “eyes” was therefore essential.

Among various perception tasks, **lane detection** plays a central role: it defines the drivable region and provides a reference for steering control and trajectory generation. For this reason, lane detection is one of the most fundamental perception functions in autonomous driving.

Initially, we attempted lane detection using OpenCV-based image processing techniques such as **Gaussian Blur**, **Canny Edge**, and **Hough Transform**. However, the system became unstable in:

- sharp curves,
- partially occluded or faded lane markings, and
- noisy conditions.

When we tried to approximate lanes as functions, incomplete or noisy input often produced distorted curves that deviated from the actual lane shape.

These experiences made it clear that pure image processing alone would not suffice for robust lane detection, and that a new approach was needed.

Deep learning models structurally addressed many of these limitations. Because they can **learn lane shapes and patterns from data**, they are more robust to varying brightness, broken lines, and complex curvature. We expected that deep learning would provide **stable lane inference**, especially in curved segments.

We initially experimented with a custom CNN-based lane detector. However, after discussions with our advisor, we decided it would be more efficient to analyze existing well-established models and adapt them to our 1/10-scale environment.

We compared major lane detection models such as **PolyLaneNet**, **UFLD (Ultra Fast Lane Detection)**, and **SCNN (Spatial CNN)** by studying their research papers and experimental results. In terms of addressing our issues—lane discontinuity and curvature—**SCNN** emerged as the most suitable choice.

<img src="images/SCNN2.png" width="450" alt="SCNN mdel architecture"/>

Our core problem could be summarized as a lack of **continuity**. SCNN addresses exactly this through a spatial message passing structure: it propagates features in the **up, down, left, and right** directions within the CNN, sharing spatial context and enforcing continuity along lanes.

However, simply importing a pretrained SCNN model did not solve our problem:

- The dataset used for pretraining and the 1/10-scale track environment differed greatly in layout, lane width, field of view, and texture.
- As a result, we faced strong **domain gaps**, and initial tests showed poor lane detection performance.

Despite these difficulties, we decided to leverage the pretrained model because:

- Our team size and project timeline did not allow us to build a large labeled dataset from scratch, and  
- Training from scratch would have been prohibitively time-consuming.

Our strategy was to exploit the **general features** of the pretrained model and adjust only what was necessary for our environment.

To do this, we:

1. Visualized feature maps produced by different SCNN layers using track images from our environment.  
2. Analyzed what features were being extracted and at which layers.  
3. Decided which layers could be kept (mainly lower-level features) and which needed adaptation (mostly higher-level layers).

Because our track did not require full four-lane highway detection—only differentiation between opposing single lanes— we simplified the output structure. We adjusted batch size, training schedule, and resolution to match our dataset size and available compute resources, optimizing the training pipeline accordingly.

Through these adjustments, SCNN eventually achieved stable tracking of continuous lanes even in curves and partially missing regions. Compared to our unstable initial models, it was as if scattered points had been smoothly connected into a solid, continuous line.
 
 <img src="images/yolo11.gif" width="450" alt="YOLO - object detection"/>

On the object detection side, we needed robust detection of traffic lights and signs. We evaluated multiple object detection architectures and selected **YOLOv8s**, considering the trade-off between accuracy and runtime performance on our embedded platform.

However, the pretrained YOLO model did not directly recognize our 1/10-scale track signs and signals due to differences in:

- physical size,
- design and color patterns, and
- environmental appearance.

To close this domain gap, we:

- collected a custom dataset focused on track-specific objects,  
- fine-tuned YOLOv8s on these samples, and  
- applied data augmentation to cover different angles and lighting conditions.

Additionally, each YOLO version requires specific dependencies, and aligning these with our **ROS2 + Jetson + Unreal** stack caused repeated compatibility issues. We had to choose a version that was:

- accurate enough, yet  
- light enough for real-time inference without interfering with control loops.

We then defined output structures compatible with **Stateflow** so that detection results (e.g., traffic light state) could be directly consumed by the control logic.

In the end, YOLOv8s became a practical module that **directly enabled decisions**, not just a standalone perception block.

---

## [10. Simulator Constraints & Early Control – Xytron Preliminary Stage](Simulations/Xytron%20Simulation//)

<img src="images/xyts2.png" width="450" alt="Xytron - control"/>
The preliminary round (Xytron) was held entirely in a simulator, but with **severely limited sensors**:

- 180° front LiDAR, and  
- a single RGB camera.

SLAM and waypoint-based planning were not available. We had to design a system that essentially **“drives based only on what it sees”**.

At that time, we implemented a lightweight driving algorithm combining:

- **SCNN-based lane detection**, and  
- a **PID controller**.

We used the front camera image to detect lane centers, defined the pixel offset between image center and lane center as steering error, and fed that error into a PID controller. This formed a simple vision-based feedback loop for steering.

At low speeds, the vehicle drove reasonably stably. However, as speed increased, oscillations and overshoot became severe. Gain tuning alone could not guarantee consistent high-speed performance.

This experience clearly demonstrated the limitations of simple **pixel-offset-based lane centering**.

These lessons directly shaped our next steps in the Quanser phase. With access to SLAM-based pose estimation and waypoint-based planning, our control design evolved through **Stanley**, **Pure Pursuit**, and finally **MPC**, transitioning from simple offset compensation to fully trajectory-aware control.

---


## 11. After Perception – “We can see the lane… but how do we actually drive?”

<img src="images/waypoint.png" width="800" alt="waypoint"/>

Once we were able to detect the lane, we ran into a more fundamental question:  
**“Seeing the lane is good. But how do we actually move using this information?”**

SCNN visualizes the structure of the lane very well.  
However, the controller that actually moves the vehicle does not use images.  
It needs a **geometric target** – a trajectory that answers:

> “Where should the vehicle go next?”

In other words, between **seeing (Perception)** and **moving (Control)**,  
there must be an intermediate layer that connects the two worlds.

---

## 11.1 From Lanes to a Drivable Path – The Thought Process

For an autonomous vehicle to move naturally,  
it needs a **sequence of target points** that it can follow over time.

This line of thought started from a very simple question:

- In the end, an autonomous vehicle moves by following a sequence of target points.  
- So we need some way to represent those target points.  
- Then what if we represent the driving path as a sequence of points?

Through this reasoning, we arrived at a concept widely used in autonomous driving research:  
**waypoints**.

We decided to:

- extract the **lane centerline**,  
- generate points along that center, and  
- connect these points to form the **actual driving path** the vehicle should follow.

---

## 11.2 A More Important Insight – A Waypoint Is Not Just a “Point,” It Is a “Language”

Introducing waypoints is not just about dropping points on a map.  
What we really wanted was to convert lane information from a visual form into a  
**coordinate-based structural representation** that a controller can understand.

In that sense, a waypoint is not just raw position data. It:

- expresses the **flow of the lane** in mathematical form,  
- naturally links the vehicle’s direction of travel, and  
- serves as a **“language of the path”** that the controller can read and act on.

So a waypoint is not just a technical trick to place points.  
It is a **structured language for expressing a stable, drivable path**.

Once we saw it that way, the conclusion was clear:

Even if perception is accurate and the controller is powerful,  
if this “path language” is not well structured,  
the vehicle will not move stably.

---

## 11.3 After Introducing Waypoints – How the Idea Expanded

Lane detection turned out to be just the **starting point of perception**,  
and by itself it was not enough to produce actual driving behavior.

For an autonomous vehicle to really move, we needed to:

- convert **pixel-based perception**  
- into **metric, real-world coordinates**,  
- ensure those coordinates formed a **continuous path**, and  
- represent that path in a form that is **robust to sensor noise**.

By introducing waypoints, we finally obtained the pipeline:

> **Lane → Path → Control Input**

Once that flow was in place, the next questions naturally emerged:

- How do we know the vehicle’s exact position on the track? → **SLAM**  
- How do we generate the global path along the entire track? → **RRT Global Path**  
- How do we select paths under different scenarios? → **Decision Layer, DQN (RL)**  
- How do we avoid obstacles smoothly and then return to the lane? → **PH-based avoidance maneuvers**

In this sense, waypoints were not just another technical element.  
They became the **starting point of the entire path planning architecture**  
that we eventually built.



---
## 12. Beginning Trajectory Tracking Control – Stanley Controller

When we first moved the vehicle in the Xytron competition simulator,  
we immediately felt the limitations of the PID controller.  
On straight segments it could more or less keep the lane, but:

- vibrations when entering a corner  
- overshoot and undershoot  
- unstable trajectory tracking  

kept appearing.

At that moment it became clear:

**“Pure feedback control alone is not enough to build autonomous driving.”**

We needed a controller that could overcome PID’s structural limitations,  
and the first candidate that came to mind was the **Stanley controller**.

---

### 12.1 First Design for Stanley: SCNN + Depth-Based 3D Waypoints
<img src="images/control5.png" width="450" alt="Stanley with SCNN and Depth pipeline"/>

Our reasons for choosing Stanley were clear:

- actually used on the winning vehicle of the **2005 DARPA Grand Challenge**  
- optimized for **sensor-based waypoint tracking**  
- proven stability at **high speeds**  
- highly compatible with the real-time autonomous driving architecture we wanted to build  

The Stanley controller computes steering from:

- **cross-track error**  
- **heading error**  
- **lane curvature**

Unlike simple PID error-feedback,  
these variables have a clear geometrical interpretation, which is a major advantage.  
However, at this point a structural problem appeared.

---

### 12.2 Structural Limitations of 2D Information

Camera images and SCNN masks are fundamentally **2D planar information**.  
In other words, the curvature you see on the screen is just a **curve in pixel space**, and:

- it is unrelated to the **physical curvature (1/m)** defined in real-world metric space  
- therefore, using only 2D information, it is impossible to compute  
  **cross-track error**, **heading error**, and **curvature**  
  in the actual vehicle coordinate frame in the way Stanley requires

To overcome this limitation, we introduced a **depth camera**.

Depth tells us **how far each pixel is from the camera (in meters)**.  
When combined with the SCNN mask, each lane pixel can be converted into a  
**3D real-world coordinate** in the vehicle frame.

Thus, the SCNN output is no longer just a 2D contour, but can be reinterpreted as

> **“the full 3D geometric structure of the lane on the road.”**

Based on this, we designed the following 3D waypoint pipeline:

1. **Detect lane mask with SCNN**  
2. **Use the depth camera to estimate the metric distance of each pixel**  
3. **Fuse the two to generate 3D waypoints**  
4. **Feed the generated waypoints into the Stanley controller**

If this pipeline had functioned as intended,  
Stanley could have become the most powerful steering controller for both theory and practice.

---

### 12.3 An Unexpected Obstacle – Complete Shutdown Due to Depth Camera Bug

<img src="images/depthissue.png" width="800" alt="Quanser depth camera issue"/>

However, during experiments we encountered a **critical issue**.  
The QLabs depth camera was bugged and did not provide valid depth values.

The output looked like this:

- constant `0`  
- constant `∞`  
- random, frame-to-frame inconsistent values  

In other words, **all meaningful distance information was essentially lost**.

Without depth:

- SCNN **pixel coordinates cannot be converted to world coordinates**  
- without world coordinates → **no 3D waypoint generation for Stanley**

As a result,  
the **Stanley-based control architecture had to be completely abandoned** in the early stage.

This forced us to go back to the beginning and  
**fundamentally re-evaluate which alternative steering control methods were realistically usable** under our constraints.


---

## 13. Evolution of Control – [Pure Pursuit](Product/Control/Steering%20&%20Breaking%20Control/) and [MPC](Product/Control/MPC/)
<img src="images/control19.gif" width="450" alt="Control Stack – Pure Pursuit and MPC"/>

Due to the depth camera bug, the Stanley controller became infeasible, so our next candidate was **MPC (Model Predictive Control)**.

While analyzing various MATLAB path-tracking examples,  
we repeatedly found MPC being used for combined longitudinal–lateral control.

The structure of **“prediction-based optimal control”** was extremely appealing.

---

### 13.1 Looking for Another Path: Testing MPC (Model Predictive Control)

MPC:

- predicts future states  
- considers steering, acceleration, and constraints simultaneously  
- selects the optimal input over a prediction horizon  

In theory, it was the closest to the “smooth and predictive control” we wanted.

However, during implementation we encountered the following barriers.

#### 1) The Biggest Issue: Lack of an Accurate Vehicle Model
MPC is highly sensitive to model errors.  
Yet the following parameters were essentially unknown:

- tire cornering stiffness  
- yaw moment of inertia \(I_{zz}\)  
- true steering and acceleration response  

These values were not provided in documentation,  
and we did not have enough time or equipment to estimate them experimentally.

In other words,  
we faced the situation:

> **“This controller only works if you have a good model, but we cannot build a model we trust.”**

#### 2) Difficulties in Designing the Cost Function
In MPC, designing the Q and R weights is crucial.  
But since the model was inaccurate:

- no Q/R combination produced consistent behavior  
- there was no reliable tuning criterion  

In conclusion:

- MPC is powerful,  
- but under undergraduate-level constraints in equipment, time, and modeling,  
  it was impossible to guarantee the “competition-grade stability” we needed.

Therefore, MPC was ultimately excluded from the final controller set.

---

### 13.2 Final Choice: Pure Pursuit + Global Waypoints + Localization
<img src="images/control14.png" width="450" alt="SLAM + Waypoint + Pure Pursuit"/>

Summarizing:

- Stanley → infeasible due to depth camera bug  
- MPC → unable to guarantee stability due to lack of an accurate model  

Realistically, only one option remained: **Pure Pursuit**.

Advantages of Pure Pursuit:

- does not require precise 3D lane geometry  
- does not require a high-order vehicle dynamics model  
- can generate stable steering using only a look-ahead point  
- low computational cost and excellent real-time performance  

Thus,  
considering our sensing setup, modeling limitations, simulator environment, and competition rules,  
**Pure Pursuit was the simplest yet only robust choice.**

---

### 13.3 Issues in Pure Pursuit and Our Mitigation Strategy

When we applied Pure Pursuit to real driving, two issues appeared repeatedly.

**1) Oscillation on Straight Segments**

Cause:

- even small errors were translated into overly large steering commands  
- the steering wheel oscillated left and right  

Mitigation strategy:

- classify the track into straight vs. curved sections  
- embed segment type information in the waypoint map  
- for straight segments:  
  - use a large look-ahead distance  
  - apply stronger steering saturation  

**2) Inside Drift on Curved Segments**

Cause:

- if the look-ahead distance is too long in a curve, the controller behaves as if tracking a straight line  
- the vehicle cuts into the inside of the curve  
- at higher speeds, understeer becomes more severe  

Mitigation strategy:

- for curved segments:  
  - use a shorter look-ahead distance  
  - set a lower target velocity  

This allowed us to:

- respond more quickly to curvature changes  
- secure sufficient steering authority  

With different tuning strategies for straight and curved sections,  
Pure Pursuit overcame both straight-line oscillation and inside drift,  
achieving stable waypoint tracking.

---

### 13.4 Conclusion – The Most Realistic Answer After Many Failures

After trying and failing with several controllers, we arrived at the following conclusion:

- depth camera bug → Stanley infeasible  
- insufficient model accuracy → MPC infeasible  
- stable control based on global waypoints and localization → achievable only with Pure Pursuit  

We did **not** choose Pure Pursuit simply because “it is the easiest”.  
We chose it because:

> **It was the only controller we could responsibly rely on under our conditions.**

At the same time, Pure Pursuit is fundamentally a **tracking** controller:  
it follows a given path, but does not decide where to go.

This experience made us acutely aware of the importance of:

- reinforcement-learning-based path planning, and  
- high-accuracy localization,

on top of Pure Pursuit.


---

## 14. Localization (SLAM) and Stabilization (Kalman Filter)

<img src="images/local2.jpg" width="450" alt="SLAM Trajectory with Noise and Stabilization"/>

> “If I don't know where I am, I can't know where to go.”

Autonomous vehicle control ultimately begins from this simple statement.  
Just as humans infer their location from surrounding buildings, road structures, and the path they have traveled,  
an autonomous vehicle must also understand **where it currently is** in order to decide **what to do next**.

In the early stage of development, we estimated the trajectory using **IMU integration** and **odometry**.  
However, drift accumulated rapidly over time, and the position estimate collapsed.  
This naturally raised an important question:

> “Is it possible to build the map *and* estimate my position at the same time?”

The answer was **SLAM (Simultaneous Localization and Mapping)**.

---

### 14.1 Introducing SLAM – Solving Mapping and Localization Together

SLAM uses sensor data to:

- extract environmental features  
- construct a map from those features  
- estimate the robot’s pose within that map  

and performs all three simultaneously.

Implementing a full SLAM system from scratch was unrealistic.  
After discussions with our advisor, we decided to **adopt and adapt existing open-source SLAM solutions** for the QCar2 environment.

From analyzing TurtleBot examples and other robotics case studies, the most widely used options were:

- **AMCL** – requires a pre-built map but provides strong localization  
- **Cartographer** – builds the map and localizes simultaneously using only LiDAR  

Because our track was a closed-loop indoor environment where both **mapping** and **localization** were required,  
we selected **Cartographer**.  
It achieved relatively accurate and stable position estimation for our system.

---

### 14.2 A Desire for Better Accuracy — “Can we refine SLAM even further?”

Sensor data always includes noise, which introduced momentary instability in SLAM output.  
During earlier MATLAB studies, we encountered examples using a **Kalman Filter** to stabilize noisy measurements.

Naturally, a question arose:

> “If we apply a Kalman Filter after SLAM, wouldn’t the pose estimate become smoother and more accurate?”

Inspired by this idea, we attempted to apply a **Kalman Filter as a post-processing layer** over SLAM output.

---

### 14.3 Attempting the Kalman Filter — and Confronting Fundamental Issues

The experiment did not unfold as expected.  
Everything boiled down to a single principle:

> **A Kalman Filter is not a ‘noise remover’; it is a state estimator that requires accurate models.**

To work properly, it requires:

- an accurate **state transition model** (vehicle dynamics)  
- an accurate **observation model** (how sensor measurements relate to the true state)  
- realistic **noise covariance matrices** \(Q\) and \(R\)  

At the undergraduate level, we could not reliably build any of these.

---

#### (1) The vehicle dynamics model was not accurate

Our QCar2’s real motion depends on complex physical variables:

- tire slip  
- cornering stiffness  
- moment of inertia \(I_{zz}\)  

These values were not provided by the manufacturer, and we had no practical means to measure them accurately.  
Even small errors in these parameters caused the Kalman Filter to **increase** the estimation error rather than reduce it.

---

#### (2) The biggest challenge — “We do not know how to model the sensor noise”

We discovered that the most fundamental obstacle was our inability to model realistic noise.  
Typical examples assume:

- Gaussian noise  
- independence  
- linearity  

But real LiDAR noise is **not Gaussian**, **not independent**, and **not linear**, and we had no dataset to characterize it.

Designing the noise covariance \(R\) was essentially equivalent to:

> “Choosing numbers without any statistical justification.”

We were applying a Kalman Filter **without knowing the noise model**, which defeats the purpose of using the filter.

---

#### (3) Actual result — performance got worse

With inaccurate dynamics and noise models, the Kalman Filter behaved exactly as theory predicts:

- less stable than SLAM alone  
- less accurate  
- sometimes significantly divergent  

Applying a Kalman Filter on top of an unreliable model is like  
**building a complex structure on a weak foundation** — the result becomes worse, not better.

---

### 14.4 Choosing Practical Stability Over Theoretical Complexity — Decimal Truncation

At this point, we changed direction.  
Instead of forcing a Kalman Filter over poor models, we adopted a **simple, robust, and practical approach**:

> **We stabilized SLAM output by dropping excessive decimal precision from the pose.**

This method is not theoretically elegant,  
but in real-world control it effectively suppressed over-reactive steering due to tiny SLAM fluctuations.  
In practice, it provided **far more stable behavior than the Kalman Filter**,  
with zero additional modeling assumptions.


---

## 15. Path Planning – Combining [RRT](Product/Decision/Path%20Planning/RRT/) and [DQN](Product/Decision/Path%20Planning/DQN/)

<img src="images/rrt1.png" width="450" alt="RRT Exploration and Global Path Skeleton"/>
<img src="images/rrt10.png" width="450" alt="Global Path and PH-Based Avoidance Paths"/>

While building the autonomous driving system, we gradually developed **Perception** and **Localization**.  
In earlier stages, we used SCNN-based lane detection to extract the track centerline, combined it with depth information, and generated waypoints for driving. We initially attempted to use the Stanley controller, but due to limitations of the depth camera, we ultimately adopted the **Pure Pursuit controller** as our steering method.

However, by design, Pure Pursuit operates on **local waypoints** visible to the vehicle at each time step.  
This revealed a critical limitation:

> Local waypoints alone were not enough – we needed a **Global Path** that covered the entire track.

To generate such a Global Path, we first had to know the vehicle’s pose accurately in an **absolute coordinate frame**.  
For this, we introduced Cartographer-based SLAM and completed our **Localization module**.

Once SLAM allowed us to obtain a stable pose on the map frame, a new question appeared:

> “Now that we can see the lanes and know our position,  
> where should the vehicle go next, and in **what order**?”

We were no longer just following a lane centerline.  
We now had to operate like a real taxi, visiting multiple **pickup/dropoff points** within a limited time and doing so as efficiently as possible.

In other words, two high-level planning problems emerged:

- How to generate a **global path** (Global Path Planning), and  
- How to **optimize the visiting order** of multiple rides.

---

### 15.1 Problem Definition – “How do we move a taxi most efficiently within a time limit?”

The main mission of the Quanser ACC competition was to operate a **taxi on an urban-style track**.  
The organizers provided an official list of Rides (A, B, C … T) that had to be executed within 5 minutes.  
Each Ride (A–T) contained fixed **pickup/dropoff points**. For example:

- Ride A: 1 → 8  
  - Node 1 = \([0.269, -0.049, 90°]\)  
  - Node 8 = \([-0.749, 1.077, 180°]\)

During the 5-minute mission, the vehicle must repeatedly:

1. Visit the Ride’s Pickup → Dropoff in order  
2. Return to the Taxi Hub  
3. Start the next Ride

Thus, the problem was not simply “shortest path between two points,” but rather:

> Given multiple Ride locations,  
> in what **order** and via which **paths** should we visit them  
> to achieve **maximum profit within the time limit**?

This is a combined **combinatorial optimization + path planning** problem.

---

### 15.2 Why RRT? – RRT as a “Path Generator”

The track is a complex **continuous space** with features such as:

- curves with high curvature,  
- roundabouts,  
- intersections,  
- bidirectional single-lane segments, and  
- segments where lane boundaries twist and merge.

We judged that purely grid-based search methods (e.g., Dijkstra on a uniform grid) would struggle to fully capture:

- the clearance between vehicle width and lane boundaries, and  
- the continuous curvature and physically feasible trajectories.

While exploring candidate methods, we focused on  
**RRT (Rapidly-Exploring Random Tree)**.

RRT:

- samples directly in continuous space,  
- can quickly generate obstacle-avoiding paths, and  
- scales relatively well to higher-dimensional spaces.

We confirmed through papers and references that RRT is widely used in real autonomous driving research.

As a result, for global path design, we adopted the following direction:

> “Define the entire track as a free space,  
> and generate paths by sampling this region using RRT.”

In other words, instead of classic graph search,  
we decided to use **RRT as a sampling-based path generator**.

---

### 15.3 How We Applied RRT – Offline, Segmented, and Smoothed

<img src="images/rrt11.png" width="450" alt="RRT- smoothing"/>
#### 1) Why run RRT offline?

We did **not** use RRT as a real-time online planner that generates paths during driving.  
Despite its strengths, RRT’s stochastic nature introduces several issues:

- Each run can produce slightly different paths (randomness),  
- Generated paths are not guaranteed to be optimal, and  
- Some runs may yield very inefficient or even unreasonable paths.

So we changed the strategy:

> “Run RRT **offline** multiple times,  
> and only keep the **best paths**.”


The selected paths were then processed with:

- Elastic Band, and  
- Spline-based smoothing

to generate a **final global path** that the controller could follow stably.

With this, **there was no need to run RRT at the competition site.**  
During actual driving, the vehicle simply followed a **pre-validated Global Path**, eliminating the randomness problem at the system level.

#### 2) Why segment the track instead of treating it as a single space?

Initially, we tried:

> “Let’s treat the entire track as one big free space and run RRT once.”

However, we ran into the following issues:

- The tree tended to grow only in certain regions (e.g., long straight segments),  
- Intersections, roundabouts, and complex structures made the definition of “free space” ambiguous and tangled.

To address this, we changed to a segmented approach:

1. **Divide the track into segments**  
   - between intersections,  
   - roundabout regions,  
   - straight segments,  
   - curved segments, etc.  
2. Define each segment as an independent free space  
3. Run **RRT separately on each segment**

We then stitched together the **best paths for each segment**  
to form a single **Global Path** over the full track.

---

### 15.4 Converting to a Directed Graph – Reinterpreting Data Structures & Algorithms

<img src="images/dqn1.png" width="450" alt="dqn directed graph"/>

The Global Path obtained from RRT is a set of **continuous coordinates**.  
But to solve the taxi problem, simply following that path is not enough.

We needed a structure capable of representing decisions like:

- Which direction to take at intersections  
- How to encode one-way constraints and prevent wrong-way travel  
- How to enforce that all pickup/dropoff nodes for each Ride are visited

For these decision-making processes, a **directed graph** representation of the track, with:

- **nodes (Vertices)** and  
- **edges (Directed Edges)**,

was much more appropriate than raw continuous coordinates.

Based on the Global Path, we:

1. Defined branch points, merges, and intersections as **nodes**,  
2. Linked them using **directed edges** that respect actual drivable directions,  
3. Structurally removed the possibility of wrong-way driving by design.

In doing so, concepts from our **Data Structures & Algorithms** coursework naturally came into play.

- Defining graphs,  
- Constructing nodes and edges,  
- Structuring them so that path search is possible  

all became an example of applying **graph modeling concepts learned in class to a real track.**

---

### 15.5 Why DQN? – Optimizing the Ride Visit Order

Once the directed graph is prepared,  
the shortest path between any two nodes can be computed using standard algorithms such as **Dijkstra, BFS, or DFS**.

However, our core problem was not just “shortest path.”

> Given multiple pickup/dropoff locations,  
> **in what order** should we visit them  
> to maximize profit within 5 minutes?

This is not a single-path problem but a **visit order optimization** problem.  
The classical algorithms we learned are not designed to solve this directly:

- **BFS / DFS**  
  → would require exploring nearly all permutations; not scalable.  
- **Dijkstra**  
  → only gives the shortest path between a single pair of nodes.

In other words, “finding paths” was solved,  
but “**deciding visit order**” remained unresolved.

At this point, it became clear that our previous knowledge alone was not sufficient.  
We therefore introduced **Reinforcement Learning (RL)** – a framework where a policy is learned through interaction and rewards.

We referred to:

- Sutton & Barto, *Reinforcement Learning: An Introduction*  
- Oilseok & Lee, *파이썬으로 만드는 인공지능 (Building AI with Python)*

After understanding the structure of Q-learning,  
we concluded that this problem could be modeled as a **DQN (Deep Q-Network)**.

We chose DQN for the following reasons:

- Decisions on the directed graph are **discrete actions** (“from the current node, choose one of the outgoing edges”),  
  which matches well with **Q-learning–based methods** like DQN.
- The visit order problem induced by many Ride combinations is extremely hard to encode as mere hand-crafted rules.  
  Instead, we design a **reward function**, and let **DQN learn an optimal visit policy** from experience.

We defined the problem as:

- **State**: set of remaining destinations, current node  
- **Action**: which node to visit next  
- **Reward**: savings in travel distance, penalties for any illegal moves (e.g., wrong-way), bonuses for completing Rides

This allowed us to learn not just shortest paths, but:

> A **visit-order policy** that aims to maximize profit within the time horizon.

---

### 15.6 Why Offline DQN Instead of Real-Time DQN? – Turning It Into an Operational Strategy

Initially, we tried running DQN **in real time** to compute the sequence of nodes for each Ride on the fly.  
But experiments showed:

- Total number of nodes: about 60  
- Time for DQN to compute an optimal sequence for a single Ride: **around 10–15 seconds**

Since multiple Rides have to be completed within a 5-minute window,  
real-time computation was clearly **inefficient**.

We therefore changed the strategy:

- For all Rides A–T,  
  **pre-compute the optimal node sequence with DQN offline**, and
- During the competition:  
  - input the Ride ID,  
  - → load the precomputed node sequence,  
  - → start driving immediately.

<p align="center">
  <img src="images/dqn3.png" alt="DQN scenario 2 path" width="750"/>
</p>

<p align="center">
  <img src="images/dqn4.png" alt="DQN scenario 2 result" width="750"/>
</p>

This converted DQN from an online planner into an **offline policy generator** and  
significantly improved operational efficiency.

---

### 15.7 Final Architecture – Roles of RRT, Directed Graph, and DQN

Combining the above components, the final structure is:

1. **RRT** generates Global Paths offline.  
2. The Global Path is converted into **segment-wise directed graphs**.  
3. **DQN** runs on this graph to determine the sequence of nodes / waypoints to visit.  
4. For the resulting node sequence, the corresponding **waypoint JSON files are loaded in order**.  
5. Each waypoint sequence is tracked using the **Pure Pursuit controller**  
   (details are described in the control section).

---

### 15.8 Significance of This Section in the Overall Project

In the path planning module, we:

- Applied **graph modeling and search concepts** from our Data Structures & Algorithms coursework to a real-world track,  
- Used **RRT** to generate **global paths in a complex continuous environment**, and  
- Tackled the difficult **visit-order optimization** problem using DQN,  
  guided by reinforcement learning textbooks and research.

In this sense, the path planning module is:

> A concrete example where undergraduate-level fundamentals  
> and more advanced concepts were  
> **integrated and adapted to a real competition environment**,  

and it represents **one of the most technically complete and mature components** of the entire project.


---

## [16. Avoidance Maneuver – TDM + TG–PH Hybrid Framework](Product/Control/Avoidance%20Control/)

<img src="images/avo10.gif" width="450" alt="Driving situation"/>

Even after implementing AEB, one question kept recurring.

In real driving, **full emergency braking** is not always the best or only action. When a pedestrian suddenly appears or a cone or stopped vehicle blocks the lane, drivers usually:

- avoid the obstacle, and then  
- return to their original lane.

Our system, however, did not have this second stage. We could brake hard, but not **“avoid and rejoin the flow.”** We had no answer to:

> “How do we pass the obstacle, not just stop in front of it?”

This marked the beginning of our work on **avoidance maneuvers**.

### 16.1 Problem 1 – The Vehicle Cannot Decide *When* and *Where* to Avoid

The first deficiency was conceptual:

- The vehicle could detect obstacles,  
- but could not decide whether the situation required avoidance or deceleration,  
- nor which direction (left/right) was safer.

To formalize this vague judgment, we analyzed the paper:

> “Decision Making and Hybrid-Based Trajectory Planning for Autonomous Vehicle in Various Highway Scenarios” (Yoon et al.)

This work introduced a **TDM (Tactical Decision Making)** structure that considers:

- ego speed
- relative speed (v_rel)
- TTC (Time-To-Collision) 
- front/rear gaps and adjacent-lane gaps (spatio-temporal gaps)

The core idea is to encode **action choices** (e.g., keep lane, brake, change lane) as **cost structures** over these variables.

Adopting TDM allowed our vehicle to answer:

- “Is deceleration sufficient, or is avoidance necessary?”  
- “If avoidance is needed, is the left or right side safer?”  
- “Given current speed, is the available gap sufficient to merge?”

<p align="center">
  <img src="images/avo1.png" alt="Decision Topology" width="450"/>
</p>

In other words, TDM provides a systematic framework for answering:

> “Under what conditions should which action be taken?”

However, deciding that avoidance is necessary does not specify **how** to avoid.

### 16.2 Problem 2 – Even After Deciding to Avoid, the Vehicle Lacks a Concrete Path

After implementing TDM, the vehicle could logically conclude:

- “I should avoid now.”  
- “The left side is safer.”

But still lacked answers to:

<p align="center">
<img src="images/avo2.png" width="450" alt="Avoding Situation"/>
</p>

- “How far sideways must I move to avoid a collision?”  
- “How do I quantify the efficiency of the avoidance path?”  
- “How do I smoothly return to the original waypoint flow afterward?”

A naive approach might be to “turn the steering wheel hard left and then correct later,” but we wanted a path described in terms of vectors and equations.

The TDM paper includes a **Trajectory Generator (TG)**, but it operates in the **Frenet coordinate system**, generating an entirely new polynomial trajectory.

Our system, however, was built entirely on **waypoint-based driving**. We wanted to:

- follow the track’s centerline most of the time, and  
- temporarily deviate around obstacles and then return to the original flow.

In other words, we needed a method to:

- preserve the global waypoint structure,  
- locally modify it around obstacles, and  
- describe the modified segment as a smooth, drivable curve.

We therefore decided to **separate the tasks** of:

1. deciding **how much to shift (offset)**, and  
2. describing **how to move along that offset (curve)**.

### 16.3 A Second Clue from PH Curves – Avoiding “Unnecessary Motion”

We turned to another paper:

> “A Study on Collision Avoidance Method Using Offset Curves of Pythagorean Hodograph Curves” (2023)

This work combines two key ideas:

1. Computing the **minimum lateral offset** needed to avoid an obstacle, and  
2. Using **Pythagorean Hodograph (PH)** curves to generate efficient avoidance and return paths.

First, the paper expresses the relative position of the obstacle and ego vehicle as a vector and calculates the distance required to pass safely, adding vehicle width, obstacle width, and safety margins to define a **minimum avoidance offset** \(\delta\).

This answers the question:

> “How far sideways must we move to avoid collision?”

Second, PH curves are used to generate a smooth transition that:

- starts from the original centerline path,  
- deviates by offset \(\delta\) around the obstacle, and  
- smoothly returns to the centerline.

We chose PH curves because they provide a mathematically well-behaved, **efficient avoidance path** that only deviates as much as necessary.

In the context of waypoint-based driving, we can:

- compute new waypoints offset by \(\delta\) near the obstacle, and  
- connect original–offset–original segments via PH curves.

This yields an avoidance path that:

- precisely reflects the required offset, and  
- remains free of unnecessary oscillations.

### 16.4 Reconstructing Two Papers into a Single Framework

We combined these concepts into a three-step hybrid framework:

<img src="images/avo5.png" width="450" alt="Diagram"/>

1. **Decision (TDM)** – When and where to avoid  
   - Use \(v_{\text{rel}}\), TTC, and spatio-temporal gaps to decide whether to avoid.  
   - Determine which direction (left/right) offers a safer gap.

2. **Offset Calculation** – How much to avoid  
   - Express the relative pose between vehicle and obstacle as vectors.  
   - Decompose into longitudinal and lateral components and add vehicle/obstacle sizes plus safety margins to derive the minimum offset \(\delta\).

3. **Waypoint–PH Trajectory Generation** – How to move and come back  
   - Around the obstacle, shift global centerline waypoints by \(\delta\) to create a local avoidance path.  
   - Connect original centerline, avoidance segment, and return segment with PH curves to create a single continuous path.

By separating **decision**, **offset**, and **trajectory**, we were able to:

- decide when and where to avoid (TDM),
- quantify how far to move sideways, and
- generate a smooth path that minimally disrupts the original waypoint flow (PH).

### 16.5 What We Ultimately Built

The crucial point is that we did not simply “chain two papers together.” We:

- reinterpreted TDM as a **decision structure**,  
- reinterpreted PH as a **geometric tool for waypoint-based avoidance**, and  
- integrated both into our existing waypoint-centric system.

Thus, what we created is not “Paper A + Paper B,” but a unified **system-level interpretation** where:

- TDM provides a framework for **risk-aware decision making**, and  
- PH provides a framework for **geometrically sound avoidance trajectories**.

This required the ability to decompose problems, find relevant concepts, and recombine them in a way that fit our system’s architecture—an essential step toward research-level thinking.


<table>
  <tr>
    <td align="center" valign="top">
      <img src="images/avo6.gif" alt="PH trajectory" width="400"/><br/>
      <b>Large Obstacle</b>
    </td>
    <td align="center" valign="top">
      <img src="images/avo7.gif" alt="PH trajectory" width="400"/><br/>
      <b>Small Obstacle</b>
    </td>
  </tr>
</table>

<table>
  <tr>
    <td align="center" valign="top">
      <img src="images/avo8.gif" alt="PH trajectory" width="400"/><br/>
      <b>Right direction</b>
    </td>
    <td align="center" valign="top">
      <img src="images/avo9.gif" alt="PH trajectory" width="400"/><br/>
      <b>Left direction</b>
    </td>
  </tr>
</table>


In summary:

- **PH curves** became our essential geometric tool for **waypoint-based avoidance maneuvers**.
- **TDM** became our core structure for **mathematically judging risk and the need for avoidance**.

By reinterpreting and connecting these concepts, we strengthened our capability to lift paper-level ideas into **system-level implementations** as an undergraduate team.

---

## [17. Hardware System – “How do we bring an autonomous driving stack into the real world?”](Product/Circuit/)

<img src="images/cir.png" width="450" alt="Circuit"/>

In simulation, perception and control algorithms alone are enough to define “autonomous driving.”  
But the moment we tried to run our stack on a real vehicle, a completely different question appeared:

> “What kind of **vehicle hardware** do we need to trust this autonomous driving software in the real world?”

It was no longer about “one board that moves the car.”

We needed:

- a boost converter,
- Jetson power and I/O circuitry,
- a VESC-based drive stage,
- sensor interfaces,
- and PCB layouts that tie everything together,

so that the physical system can **reliably sustain what the algorithms expect it to do**.

---

### 17.1 Power Stage – “Is it enough to just boost 7.4 V to 19 V?”

The initial question was deceptively simple:

> “We just need to boost 2S Li-ion (7.4 V) to 19 V. Can’t we just copy a reference boost design from the market?”

Once we actually analyzed the boost behavior, that question changed immediately.

- In **voltage-mode control**, the **RHP zero** fundamentally limits how wide we can set the bandwidth.
- Increasing L and C reduces ripple but also worsens size, cost, and transient response.
- In other words, simply “using bigger parts” is not enough to meet the **power quality and response** required by an autonomous system.

So we had to ask a different question:

> “Instead of oversizing components, what if we change the **control structure and topology**?”

The first answer was to move to **current-mode control (PCMC, Peak Current Mode Control)**:

- We include the inductor current in the feedback loop, making the effective plant closer to a first-order system.
- We add slope compensation to suppress subharmonic oscillation.

This allowed us to reframe the boost converter: not as a “fundamentally hard-to-control topology,” but as a **well-behaved target under current-mode control**.

Then the next question followed:

> “Even with current-mode control, do we really have to dump all ripple and device stress into a single phase?”

Our solution was a **2-phase, 180° interleaved boost**:

- The inductor ripple currents of the two phases partially cancel, reducing **input and output ripple current**.
- Inductors and capacitors share current, **distributing thermal and electrical stress**.
- The effective switching frequency doubles, allowing us to hit the same ripple targets with **smaller L and C**.

After this design process, our view of the power stage changed completely:

> “This booster is not just a ‘7.4 V → 19 V converter.’  
> It is a **power architecture** that addresses RHP zeros, ripple, and device stress using  
> current-mode control (PCMC) and a 2-phase interleaved topology.”

---

### 17.2 Jetson and Sensors – “How far should we trust the reference design?”

When we first planned to mount a Jetson, our instinct was:

> “Wouldn’t it be safest to copy the NVIDIA reference board 1:1?”

The reference design already includes power sequencing, USB, and CSI port configurations.  
At first glance, copying it exactly seemed like the **lowest-risk option**.

But once we laid out the BOM, the issues became clear:

- Some power ICs and hub chips were **very expensive**.
- Several components had **EOL or unstable supply**, making long-term procurement unreliable.
- For a research/education platform that might go through multiple revisions and re-spins, this was not sustainable.

So we had to reformulate the question:

> “If we want to match the reference design’s performance,  
> how do we do it with **components that we can continuously source**?”

Our conclusion was:

1. Keep the **electrical specifications** (voltage, current, ripple, transient behavior) aligned with the reference.  
2. Rebuild the circuit using **components that are realistically available and stable in the local supply chain**.

In practice:

- Power ICs were replaced with **equivalent-grade devices** that matched the required specs.
- The USB hub was switched to a chip with solid documentation and **stable distribution**, and we redesigned the Jetson–hub–sensor chain around it.

In short:

> “The Jetson power/I-O board preserves the reference design’s electrical requirements,  
> but is rebuilt with components that we can **actually buy, repair, and recreate** over time.”

---

### 17.3 Motor Drive – “How do we create an output we can trust?”

On the motor side, the starting question was:

> “How can we make motor control for an autonomous vehicle **stable and trustworthy**?”

In this system, the motor is not just a part that spins the wheels.  
It is the **final actuator** that turns perception and control outputs into real-world motion.  
If motor control is unstable, safety logic is weak, or Jetson–VESC communication breaks, the entire stack collapses.

We therefore made an early design choice:

> “Rather than building our own motor control core,  
> let’s **build our hardware around a proven core**.”

The answer was:

- the open-source **VESC reference**, and  
- the **F1TENTH VESC ROS nodes** (such as `vesc_driver`).

That is:

- Low-level FOC control, current/voltage/temperature sensing, and protection logic are delegated to **VESC**, which is already well tested.
- High-level commands between Jetson and VESC flow through **ROS nodes**, using topics and services.

The next question was about **physical connectivity**:

> “The software stack is robust, but how do we connect things physically  
> so that we have a **control path that does not randomly break**?”

Our first thought was to simply wire Jetson and VESC using external cables and exposed connectors.  
However, in a real vehicle:

- vibration and shock,
- repeated plugging/unplugging,
- and constrained cable routing

mean that even a slightly loose connector can **break communication**—which directly turns into unpredictable motor behavior and safety risk.

So we changed our answer:

> “Pull the VESC–Jetson link **as far inside the PCB as possible**.”

In the final design:

- Critical interfaces between Jetson and VESC are connected via **internal PCB traces**, not long external cables.
- Exposed, frequently-handled cable segments are **minimized**.

The result is a VESC–Jetson pair that:

- is integrated logically through the **VESC–ROS stack**, and  
- is connected physically through a **robust internal PCB path**,

forming a **single, hard-to-break control pathway**.

Summarizing:

> “Our VESC-based drive stage places a proven motor-control core underneath  
> a custom power, sensing, communication, and wiring design that is tailored to our vehicle,  
> raising the stability and trustworthiness of motor control.”

---

### 17.4 PCB Layout – “How much attention does a single trace deserve?”

Near the end of the schematic design, another question arose:

> “How can we ensure that this circuit, once laid out on a PCB,  
> will behave **stably in the real world**?”

At higher switching frequencies, **traces and vias are no longer just ‘wires’.**  
They carry resistance as well as **parasitic inductance (L)** and **parasitic capacitance (C)**.

So layout design inevitably led to the question:

> “How do we close loops and route traces so that we can **trust this board** later?”

We based our layout on two core principles:

1. **“Minimize loop area, but don’t create excessive parasitic capacitance.”**  
2. **“Always route signals together with their return paths.”**

More concretely:

- The boost switching loop, gate driver–MOSFET loop, and current sensing loop were  
  routed to close within **tight, compact areas** to reduce parasitic L.
- At the same time, we avoided over-compressing them to the point that parasitic C to adjacent traces/planes would explode;  
  we tuned spacing and loop size to find a **balanced middle ground**.
- Trace corners were routed at **45° instead of 90°**, reducing local impedance jumps, heating, and waveform distortion in high-frequency paths.

Additionally:

- High-speed and sensitive signals were placed with their **GND return paths directly under/next to them**,  
  minimizing the signal–return loop area and reducing both radiated EMI and susceptibility to external noise.
- USB and CSI lines were routed so that they would **not run long, parallel segments with power lines**, mitigating unwanted coupling.
- TVS/ESD arrays were placed **as close as possible** to each external connector,  
  clamping ESD and surge events at the port itself.
- Power GND and signal GND were not mixed arbitrarily across the board;  
  they were **joined at carefully chosen single-point connections**.

In summary:

> “This is not just a board with a clean schematic.  
> It is a PCB whose traces and loops are laid out with **parasitic L/C and long-term reliability in mind**.”

---

### 17.5 Summary – Not just “combined references”, but “hardware shaped by questions”

At a glance, the hardware in this project might look like:

- a 2-phase interleaved boost converter,
- a Jetson power/I-O board,
- and a VESC-based drive unit.

Just three PCBs.

But beneath that, there is a long chain of questions and answers:

- “Can we really power this system stably with a boost converter?”
- “How far should we copy the Jetson reference, and where must we diverge?”
- “How do we make motor control that is both safe and robust against disconnection?”
- “How should we draw even a single trace so that we can trust it years later?”

Because of that, we think this work is best described as:

> “A hardware stack that evolved in response to our own questions —  
> built on proven references like VESC and Jetson,  
> but re-engineered in circuitry and layout to become a  
> **‘question-driven autonomous driving hardware stack.’**”

---

## 18. Conclusion and Achievements

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


Over two semesters (one year), we designed and implemented the entire pipeline from **theory → simulation → real vehicle**. Starting with Unreal-based simulation and ending with ROS2-based real QCar2 driving, we experienced the integration of all stages of autonomous driving into a single system.

Our key achievements include:

- **ACC Student Competition**: selected as one of the **top 12 teams** and advanced to Stage 2, finishing **7th overall**.  
- **Kookmin University Autonomous Driving Competition**: ranked in the **top 25 out of 126 teams** and qualified for the finals, but had to withdraw due to schedule conflicts with ACC.  
- Completion and open-sourcing of a **ROS2-based autonomous driving platform** on GitHub.

The final system was implemented as a ROS2-integrated stack comprising:

- a **camera–LiDAR–IMU fusion-based perception module**,  
- **SCNN and YOLO-based decision modules**, and  
- a **Pure Pursuit-based control module**.

These modules operated in real time and were successfully deployed on the QCar2 hardware.

However, the true essence of our work lies not in the visible end results but in the **process of turning theory into functioning systems**.

Concepts we had learned from textbooks—control theory, SLAM, deep learning—behaved in unexpected ways in real-world conditions. Multiple approaches existed for each function, and we constantly had to compare and test them to see which best addressed the overall system’s complexity and constraints.

Every time we solved a problem, we asked:

- “From which direction should we approach this problem?”  
- “Which method is most efficient under our constraints?”

The answers often came from the latest research papers and technical references, which we then adapted to our own system (e.g., SCNN, SLAM post-processing, TDM+PH).

As the system grew more complex, the number of variables exploded. A minor bug in design or algorithm could cause unexpected behavior, and sometimes the root cause lay in places we had not anticipated. We learned that systematically narrowing down possibilities and isolating error sources is itself a crucial engineering skill.

Most importantly, we experienced firsthand how **mathematical theories**—control theory, SLAM, deep learning—translate physical reality into **computable forms**. To make intuitive phenomena like “the lane curves” or “the lead vehicle is getting closer” accessible to computers, we had to express them in terms of coordinates, slopes, accelerations, and noise models.

This process of reconstructing reality into manageable units was where we experienced the greatest growth.

We also realized that real systems are full of noise and imperfections, and even small disturbances can affect overall stability. Studying and partially implementing Kalman Filters, MPC, and SLAM internals made us aware of both the limits of our current level and the need for deeper study.

This project became a starting point that clarified our future research direction. All team members gained practical experience with:

- **ROS2**,  
- **Simulink**, and  
- **TensorFlow**,

and learned how to design and integrate an entire autonomous driving pipeline.

Most importantly, we did not just assemble technologies. We designed and implemented **the learning process itself**:

- from PID to MPC,  
- from RRT to RL-enhanced RRT,  
- from generic CNNs to SCNN and LSTM.

Every step forward started from the question:

> “Why should we use this?”

> “We built a car that can see lanes, make decisions, and move.  
> But more importantly, through that process, we became engineers who can **see, decide, and act** on our own.”

---

## 19. Summary

The KDAS2025 project evolved from simple simulation-based control experiments into a **fully integrated AI-driven autonomous vehicle system**.

Step by step, we:

- overcame control instability with MPC,  
- improved localization with Cartographer and pose stabilization techniques,  
- enhanced perception via SCNN and YOLOv8s,  
- refined planning with RRT, DQN, and PH-based trajectory generation, and  
- integrated the entire stack over ROS2 on QCar2 hardware.

All code, data, and detailed documentation are organized under the `Product/` directory and related paths referenced above.

This repository captures not only the final system, but also the **entire process** through which we learned to design, implement, and validate a complete autonomous driving stack from first principles.

---

## Team Roles

### Su Chul Kim (Project Lead & System Architect)
- Designed the full autonomous driving pipeline architecture and led the project’s technical direction.
- Implemented Pure Pursuit + Stanley controllers and developed an MPC-based trajectory tracking module.
- Built a PID auto-tuner for the vehicle platform and developed Kalman Filter–based state estimation.
- Performed SLAM testing and integrated results with Kalman Filter evaluation.
- Led the design and development of the DQN-based reinforcement learning driving model.
- Contributed to RRT-based global path planning and led the development of the PH (Pythagorean Hodograph) curve–based collision avoidance planner.
- Integrated SCNN lane detection and YOLO object detection modules with planning and control stacks.
- Led the creation of AEB (Unreal Engine) and MATLAB simulation test environments.
- Oversaw ROS2-based system integration and QCar2 real-vehicle testing.
- Managed team schedules, code reviews, and technical sessions throughout the project.


### Kiheon Kwak (SLAM & ROS System Lead)
- Led SLAM tuning and map-building pipeline optimization.
- Managed overall ROS system node configuration and data-flow architecture.
- Contributed to AEB (Unreal Engine) and MATLAB implementation.
- Participated in vehicle dynamic modeling development.
- Oversaw QCar2 real-vehicle testing and ROS–hardware integration.


### Su Min Kim (Perception & Planning Engineer)
- Led development and optimization of the SCNN-based lane detection module.
- Led design and implementation of the RRT-based global path planner.
- Contributed to development of the DQN reinforcement-learning driving model.


### Jin Young Jeong (Computer Vision & Dynamic Modeling Engineer)
- Led YOLO-based object detection module development and optimization.
- Led dynamic vehicle modeling and parameter identification.
- Participated in developing the PH-based collision-avoidance curve generation module.


### Heung Sik Jeon (Electronics & Circuit Lead)
- Led design, wiring, and fabrication of the vehicle’s entire electronic and power-circuit system.
