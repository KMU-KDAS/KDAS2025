<img src="images/kdasmain.gif" alt="KDAS" width="800"/>

# KDDAS2025: Kookmin Autonomous Driving System

*Kookmin University Autonomous Driving Research Team – Alpha Project 2024–2025*

The **Kookmin Autonomous Driving System (KMU-KDAS)** is a full-stack autonomous RC-car platform developed for the **Quanser Autonomous Car Competition (ACC)**. This repository documents our journey from basic vehicle dynamics simulation to a fully integrated ROS2-based AI driving system running on the QCar2 hardware.

---

## 1. Overall Results (Module GIF Summary)

Below are the final results showing each module operating in real time. Click on each caption to open the corresponding folder under `Product/`.

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
      <img src="images/rrt1.png" width="300"><br>
      <b><a href="Product/Decision/path%20planning/">Path Planning (RRT + DQN)</a></b>
    </td>
    <td align="center">
      <img src="images/SCNN11.gif" width="300"><br>
      <b><a href="Product/Decision/SCNN/">Lane Tracking Visualization</a></b>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="images/control19.gif" width="300"><br>
      <b><a href="Product/Control/Steering Control & Breaking Control/">Control QCar</a></b>
    </td>
    <td align="center">
      <img src="images/control18.gif" width="300"><br>
      <b><a href="Product/Decision/SCNN/">Control simulation</a></b>
    </td>
  </tr>
</table>

---

## 2. Development Story and Repository Overview

This section summarizes the final system modules, representative figures, and links them to the directory structure under `Product/`. It complements the detailed narrative in the later sections.

### 2.1 Overall Results (GIF Summary – Text-Only)

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

<img src="images/SCNN10.gif" width="450" alt="Perception – Sensor Integration Overview"/>

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
      <b><a href="Product/Decision/SCNN/">SCNN – Lane Detection</a></b>
    </td>
    <td align="center">
      <img src="images/SCNN11.gif" width="260"><br>
      <b><a href="Product/Decision/SCNN/">SCNN – Lane Tracking</a></b>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="images/yolo9.gif" width="260"><br>
      <b><a href="Product/Decision/YOLO/">YOLOv8s – Object Detection</a></b>
    </td>
    <td align="center">
      <img src="images/yolo11.gif" width="260"><br>
      <b><a href="Product/Decision/YOLO/">YOLOv8s – Traffic Light & Sign</a></b>
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
      <b><a href="Product/Decision/path%20planning/RRT/">RRT </a></b>
    </td>
    <td align="center">
      <img src="images/rrt10.png" width="260"><br>
      <b><a href="Product/Decision/path%20planning/">RRT </a></b>
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

## 4. First Challenge – Motor Control (PID Autotuning)

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

To overcome this inefficiency and uncertainty, we searched for a more systematic method to derive gains and adopted **PID Autotuner**. PID Autotuner computes gains based on the actual system response, providing more stable and consistent control performance compared to manual tuning.

---

## 5. AEB System Design and Simulation – `Matlab_AEB_Unreal`

<img src="images/control19.gif" width="450" alt="AEB Braking Response – Simulated Scenario"/>

> “If we can move the vehicle, how do we make it stop safely?”

Once we had implemented forward motion via motor control, the next task was **stopping in dangerous situations**. This is where **AEB (Autonomous Emergency Braking)** comes in—a system that allows the vehicle to detect hazards and perform deceleration or full braking at the right moment.

However, testing a control algorithm that assumes potential collisions in a real vehicle poses obvious risks: vehicle damage, sensor failure, and high cost. We therefore needed a **virtual environment** capable of replacing real-world crash scenarios.

To this end, we integrated **MATLAB/Simulink** with **Unreal Engine 3**, building a 3D simulation environment where the AEB controller could be tested and validated. In this environment, heterogeneous elements such as:

- discrete-time sensors,
- fixed-step controllers, and
- continuous-time vehicle dynamics

could be aligned on a single time axis, closely approximating real driving conditions.

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

## 6. Dynamics Modeling – `AEB_Dynamics` & `QCar_Dynamics`

<img src="images/control18.gif" width="450" alt="Vehicle Dynamics – Longitudinal/Lateral Modeling"/>

Our motivation for vehicle dynamics modeling was not limited to computing braking points for AEB. To increase the reliability of sensor-based systems such as **SLAM** and **MPC**, we needed a mathematical prediction model that could verify whether sensor outputs were physically plausible and correct them when necessary.

In other words, we sought to understand **“how the vehicle actually moves”** at a fundamental physical level, beyond what sensors merely report.

### 6.1 Modeling Direction

The primary objective of our modeling was to represent vehicle speed and its braking/acceleration behavior in a physically valid manner.

Instead of implementing every equation from scratch, we decided it would be more efficient and robust to leverage MATLAB’s built-in **Bicycle Vehicle Model**. This block can express full 3DOF vehicle motion—longitudinal (x), lateral (y), and vertical (z).

However, our experimental environment was a flat track without slopes. The vertical axis (z) would add complexity without contributing meaningful effects. Thus, we adopted a **2DOF (x–y plane)** model and built our dynamics block around it.

The Bicycle Vehicle Model is based on a **single-track representation**, where the front and rear wheels are each merged into a single representative wheel. On top of this structure, we integrated:

- Pacejka’s **Magic Formula**,  
- **longitudinal slip** modeling, and  
- friction–velocity relationships

to model realistic vehicle behavior.

As a result, the speed and acceleration profiles generated in simulation could be aligned with physical intuition and sensor readings.

### 6.2 Reconstructing a QCar-Specific Model

After the preliminary phase, our actual test platform was Quanser’s **QCar**, which differs significantly from a conventional gasoline vehicle:

- It uses a **BLDC motor** and **ESC (Electronic Speed Controller)** as actuators.
- It has **no mechanical braking system**.
- It is a **1/10-scale platform** with much smaller mass, inertia, and aerodynamic effects.

Therefore, the default Bicycle Model alone could not offer sufficiently accurate predictions.

We explicitly introduced the longitudinal equation of motion:

$$
m\dot{v}_x = \sum F_t - F_{\text{roll}} - F_{\text{aero}} - mg \sin\theta
$$

and added a motor torque model consistent with QCar’s actuation:

$$
T_m = k_t I_m = k_t \frac{V_{\text{duty}} - k_e \omega_m}{R}
$$

This model describes the physical pathway:

> ESC duty command → motor torque → vehicle speed

In simulation, this reconstruction reproduced QCar’s deceleration and response characteristics realistically, forming the basis for later controller designs and parameter tuning.

### 6.3 Significance of the Dynamics Model

Dynamics modeling here was not just about creating a simulation block. High-level control structures such as SLAM and MPC can only be trusted if their underlying physical models are consistent and well understood.

With our dynamics model, we gained a mathematical foundation to:

- correct sensor data, and  
- generate predictively stable control signals.

In this sense, the vehicle model served as a **bridge between perception and control**—a medium that translates physical reality into mathematics.

---

## 7. ROS2 Integration – Building a Unified System

<img src="images/Ros2.gif" width="450" alt="ROS2 Integration – Multi-Node System"/>

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

<img src="images/acc.png" width="450" alt="KDAS – Autonomous Taxi Mission"/>

Our initial goal of building a fully autonomous system did not end with implementing individual functions. During AEB development, we encountered a crucial realization:

> Even a single function like braking only works correctly when **perception–decision–control** are tightly coupled.

To detect hazards, sensor interpretation must be stable. To determine braking timing, the vehicle state must be correctly estimated. Finally, in the control stage, a precise controller based on a realistic vehicle model is required.

This experience fundamentally changed our perspective. We realized that the essence of autonomous driving is not the completion of individual features but the **design of the entire stack**.

Naturally, a question arose:

> “Where does our stack stand right now?”

We wanted an **objective benchmark**, not just our own satisfaction. Domestic competitions were considered, but most focused on specific functions and did not evaluate the entire system holistically.

We sought an environment where perception, planning, and control could be viewed as a single system and evaluated against **industrial standards**. We concluded that such a structure was difficult to find domestically and turned our attention abroad.

We discovered the **Quanser Autonomous Car Competition (ACC)**, which provided exactly this type of environment.

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

## 9. Perception – Beginning to See the World (SCNN & YOLOv8s)

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

## 10. After Perception – “We See the Lane. Now How Do We Drive?”

<img src="images/SCNN10.gif" width="450" alt="Lane Mask to Waypoint Conceptual Transition"/>

Once we could reliably detect lanes, a more fundamental question emerged:

> “It’s good that we can see the lane, but how do we actually move using that information?”

SCNN provides lane information as **images**, but controllers require **numeric coordinates**. Perception alone is “seeing.” To move the vehicle, we must convert this information into mathematically defined targets—concrete points that the vehicle can follow.

We reasoned as follows:

- An autonomous vehicle moves by following a sequence of **continuous target points**.  
- We need a way to express these target points.  
- What if we represent the driving path as a sequence of **waypoints**?

This led us to the widely used concept of **waypoints** in autonomous driving: generating points along the lane center and connecting them into a drivable path.

### 10.1 Depth-Based Waypoint Generation – and Its Structural Limits

To generate waypoints, we must convert pixel coordinates into real-world coordinates (x, y). A natural idea was to use a **depth camera** and map SCNN’s pixel outputs to metric coordinates.

However, this is where we encountered a fundamental issue.

The QLabs simulator provider had already announced that **the depth camera was not functioning correctly**. We confirmed that the depth frames:

- contained very few valid values,
- intermittently produced invalid values like 0 and ∞, and
- exhibited highly inconsistent patterns from frame to frame.

In other words, the depth sensor was **unusable** for real-time coordinate conversion.

This was more than a sensor choice issue; it meant that our entire approach did not structurally fit the QLabs environment.

### 10.2 A More Important Lesson – Waypoints Are an “Language,” Not Just Points

Depth was not the only problem. We realized something deeper: even if depth had worked perfectly, the waypoints we were generating would have been unsuitable for control.

Sensor-based waypoints are inherently:

- noisy and unstable from frame to frame,  
- sensitive to small sensor perturbations, and  
- discontinuous, with frequent gaps between points.

From the controller’s perspective, this is not a **trackable path** but a **collection of jittering points**.

From this experience, we learned that **waypoints are not just points but a structural “language” for expressing stable, drivable paths**. Unless this language is used correctly, even the best perception or control algorithms cannot produce stable vehicle motion.

### 10.3 Our Conclusion from This Failure

Seeing the lane is only the beginning of perception and does not automatically produce drivable behavior.

To move an autonomous vehicle, we must:

- convert pixel-based perception to **real-world coordinates**,  
- ensure these coordinates are **continuous**, and  
- represent them as a **noise-resilient path structure** suitable for control.

Since the QLabs depth sensor was structurally unusable and sensor-based waypoint generation could not guarantee stability, we concluded that this approach was fundamentally flawed.

This failure became a turning point that led us to more robust, structurally grounded path planning methods:

- SLAM-based pose estimation,
- RRT-based path planning, and
- PH-based avoidance maneuver generation.

The question,

> “We see the lane, but how do we drive?”

was not merely a technical issue. It taught us that perception outputs must be **translated into the “language of paths”**—a comprehensive reasoning process in its own right.

---

## 11. Localization and Stabilization – SLAM + Kalman Filter

<img src="images/local2.jpg" width="450" alt="SLAM Trajectory with Noise and Stabilization"/>

> “If I don’t know where I am, I can’t know where to go.”

Vehicle control begins with **accurate self-localization**. Just as humans construct a mental map and infer their current position from buildings, road shapes, and traveled routes, autonomous vehicles must make similar judgments based on sensor data.

Initially, we used **IMU integration** and **odometry** to estimate the trajectory. Over time, however, accumulated drift made it impossible to maintain accurate position estimates.

We asked:

> “Can we build the map and localize ourselves at the same time?”

The answer was **SLAM (Simultaneous Localization and Mapping)**.

SLAM uses sensor measurements to recognize environmental features and simultaneously:

- builds a map, and  
- estimates the vehicle’s pose within that map.

Implementing a complete SLAM system from scratch was not feasible within our constraints. In consultation with our advisor, we decided to analyze existing open-source SLAM solutions and adapt them to our environment instead.

From TurtleBot examples and various autonomous driving case studies, we found that **AMCL** and **Cartographer** were widely used:

- **AMCL** excels at probabilistic localization but requires a pre-built map.  
- **Cartographer** can build the map and localize in real time using only LiDAR, making it better suited to our closed-track environment.

We chose **Cartographer** and achieved relatively accurate real-time pose estimates.

However, in real-world operation, LiDAR noise and sensor timing offsets caused small fluctuations in the estimated pose. These fluctuations destabilized the controller and led to excessive corrections.

We had previously seen MATLAB examples that applied **Kalman Filters** to stabilize noisy sensor data. This led to the idea:

> “Can we apply a similar approach to stabilize SLAM-based pose estimates?”

We started experiments combining SLAM outputs with Kalman Filtering. In practice, however, results were not as consistent as we hoped:

- Designing an accurate prediction model was challenging at the undergraduate level.
- Model inaccuracies sometimes made the situation worse instead of better.

We learned the hard way that **poorly modeled complex filters can degrade performance**.

For this project, we chose a simpler but more robust method. We observed that:

- major jitter occurred from the second decimal place of SLAM coordinates, and  
- its magnitude was around **0.1 m**, significantly smaller than the vehicle length (**0.3 m**).

Thus, we decided to **truncate** SLAM coordinates to one decimal place, effectively quantizing the pose with 0.1 m resolution.

This approach is not as elegant as a Kalman Filter and does reduce positional resolution. However, it significantly suppressed overreactions in control and produced **stable and consistent behavior** in real driving.

---

## 12. Path Planning – Combining RRT and DQN

<img src="images/rrt1.png" width="450" alt="RRT Exploration and Global Path Skeleton"/>
<img src="images/rrt10.png" width="450" alt="Avoidance Paths – PH-Based Offset Around Obstacles"/>
Once we could localize ourselves accurately, the next question was:

> “How do we generate a path?”

Because our track was a **closed environment**, we applied **RRT (Rapidly-Exploring Random Tree)** to sample free space and generate global paths.

However, the taxi mission required more than continuous driving. The vehicle had to visit multiple pick-up and drop-off points in sequence:

> “Even on the same track, in what order should we visit locations to be most efficient?”

RRT is powerful for exploring space but not suited for **sequence optimization**. To address this, we integrated **DQN (Deep Q-Network)**-based reinforcement learning.

Our hybrid strategy was:

- RRT: explore free space and propose feasible path segments.  
- DQN: learn the order of visiting waypoints to minimize travel distance and time.

Through this combination, we constructed a conceptual **autonomous taxi service**, not just a simple autonomous vehicle.

---

## 13. Simulator Constraints & Early Control – Xytron Preliminary Stage

<img src="images/SCNN1.gif" width="450" alt="Xytron Simulator – Lane Following with SCNN + PID"/>

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

## 14. Evolution of Control – Stanley, Pure Pursuit, and MPC

<img src="images/control19.gif" width="450" alt="Control Stack – From Stanley to Pure Pursuit and MPC"/>

In the Quanser final stage, we worked in an environment without strong sensor constraints. With access to full information—vehicle position, speed, and reference path—we could now test more advanced controllers. To overcome the oscillation and overshoot limitations of PID, we moved toward more robust control schemes.

### 14.1 Stanley Controller

We first implemented a **Stanley Controller** in MATLAB/Simulink. Stanley corrects:

- lateral deviation (**cross-track error**), and  
- heading misalignment (**heading error**)

using a geometric formulation, which improves high-speed stability compared to simple PID steering.

We referenced the paper:

> “Trajectory Tracking Control of Autonomous Heavy-Duty Mining Dump Trucks with Uncertain Dynamic Characteristics” (Liang Chen et al.)

and implemented a **Modified Stanley Controller** incorporating a collaborative preview structure based on speed and curvature.

This modified Stanley controller produced a stable steering response even on tracks with large curvature variations.

### 14.2 Pure Pursuit Controller

Next, we introduced **Pure Pursuit**. Pure Pursuit computes curvature to a **lookahead point** at a fixed distance ahead, generating a steering command that geometrically tracks the desired path.

Its advantages include:

- low computational cost,  
- smooth trajectory following, and  
- strong real-time suitability for embedded systems.

In the QLabs competition environment, due to depth camera malfunctions, we could not rely on the SCNN+Depth coordinate input pipeline. Under these constraints, a Pure Pursuit-based control structure was the most practical choice.

We confirmed via Simulink simulations that steering and speed commands from the Pure Pursuit controller behaved as expected.

### 14.3 Model Predictive Control (MPC)

In the final real-vehicle integration stage, we moved to **MPC**. MPC formulates control as a constrained optimization problem over a prediction horizon, considering:

- steering angle limits,  
- steering rate limits, and  
- acceleration/deceleration constraints.

Compared to Stanley and Pure Pursuit, MPC offers smoother and more predictive control.

We enhanced this by:

- incorporating a **Kalman Filter (KF)** to improve state estimation reliability, and  
- integrating **LSTM-based short-term prediction** models to forecast future values of speed and other state variables, feeding these predictions into MPC.

The idea of LSTM arose from what we learned about **CAN (Controller Area Network)** in class:

> “Real vehicles host numerous sensors, and their data are exchanged over the CAN bus.”

We realized that in our system, diverse sensor streams from LiDAR, RGB, depth cameras, IMU, and CSI cameras could also be preprocessed and **learned as temporal sequences**.

By combining LSTM prediction with MPC, we obtained a controller that could account for near-future dynamics and environmental changes, improving performance in nonlinear and dynamic conditions.

---

## 15. Avoidance Maneuver – TDM + TG–PH Hybrid Framework

<table>
  <tr>
    <td align="center" valign="top">
      <img src="../../../images/avo6.gif" alt="PH trajectory" width="400"/><br/>
      <b>Large Obstacle</b>
    </td>
    <td align="center" valign="top">
      <img src="../../../images/avo7.gif" alt="PH trajectory" width="400"/><br/>
      <b>Small Obstacle</b>
    </td>
  </tr>
</table>

<table>
  <tr>
    <td align="center" valign="top">
      <img src="../../../images/avo8.gif" alt="PH trajectory" width="400"/><br/>
      <b>Right direction</b>
    </td>
    <td align="center" valign="top">
      <img src="../../../images/avo9.gif" alt="PH trajectory" width="400"/><br/>
      <b>Left direction</b>
    </td>
  </tr>
</table>

Even after implementing AEB, one question kept recurring.

In real driving, **full emergency braking** is not always the best or only action. When a pedestrian suddenly appears or a cone or stopped vehicle blocks the lane, drivers usually:

- avoid the obstacle, and then  
- return to their original lane.

Our system, however, did not have this second stage. We could brake hard, but not **“avoid and rejoin the flow.”** We had no answer to:

> “How do we pass the obstacle, not just stop in front of it?”

This marked the beginning of our work on **avoidance maneuvers**.

### 15.1 Problem 1 – The Vehicle Cannot Decide *When* and *Where* to Avoid

The first deficiency was conceptual:

- The vehicle could detect obstacles,  
- but could not decide whether the situation required avoidance or deceleration,  
- nor which direction (left/right) was safer.

To formalize this vague judgment, we analyzed the paper:

> “Decision Making and Hybrid-Based Trajectory Planning for Autonomous Vehicle in Various Highway Scenarios” (Yoon et al.)

This work introduced a **TDM (Tactical Decision Making)** structure that considers:

- ego speed,  
- relative speed \(v_{\text{rel}}\),  
- TTC (Time-To-Collision), and  
- front/rear gaps and adjacent-lane gaps (spatio-temporal gaps).

The core idea is to encode **action choices** (e.g., keep lane, brake, change lane) as **cost structures** over these variables.

Adopting TDM allowed our vehicle to answer:

- “Is deceleration sufficient, or is avoidance necessary?”  
- “If avoidance is needed, is the left or right side safer?”  
- “Given current speed, is the available gap sufficient to merge?”

In other words, TDM provides a systematic framework for answering:

> “Under what conditions should which action be taken?”

However, deciding that avoidance is necessary does not specify **how** to avoid.

### 15.2 Problem 2 – Even After Deciding to Avoid, the Vehicle Lacks a Concrete Path

After implementing TDM, the vehicle could logically conclude:

- “I should avoid now.”  
- “The left side is safer.”

But still lacked answers to:

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

### 15.3 A Second Clue from PH Curves – Avoiding “Unnecessary Motion”

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

### 15.4 Reconstructing Two Papers into a Single Framework

We combined these concepts into a three-step hybrid framework:

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

### 15.5 What We Ultimately Built

The crucial point is that we did not simply “chain two papers together.” We:

- reinterpreted TDM as a **decision structure**,  
- reinterpreted PH as a **geometric tool for waypoint-based avoidance**, and  
- integrated both into our existing waypoint-centric system.

Thus, what we created is not “Paper A + Paper B,” but a unified **system-level interpretation** where:

- TDM provides a framework for **risk-aware decision making**, and  
- PH provides a framework for **geometrically sound avoidance trajectories**.

This required the ability to decompose problems, find relevant concepts, and recombine them in a way that fit our system’s architecture—an essential step toward research-level thinking.

In summary:

- **PH curves** became our essential geometric tool for **waypoint-based avoidance maneuvers**.
- **TDM** became our core structure for **mathematically judging risk and the need for avoidance**.

By reinterpreting and connecting these concepts, we strengthened our capability to lift paper-level ideas into **system-level implementations** as an undergraduate team.

---

## 16. Conclusion and Achievements

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

## 17. Summary

The KDAS2025 project evolved from simple simulation-based control experiments into a **fully integrated AI-driven autonomous vehicle system**.

Step by step, we:

- overcame control instability with MPC,  
- improved localization with Cartographer and pose stabilization techniques,  
- enhanced perception via SCNN and YOLOv8s,  
- refined planning with RRT, DQN, and PH-based trajectory generation, and  
- integrated the entire stack over ROS2 on QCar2 hardware.

All code, data, and detailed documentation are organized under the `Product/` directory and related paths referenced above.

This repository captures not only the final system, but also the **entire process** through which we learned to design, implement, and validate a complete autonomous driving stack from first principles.
