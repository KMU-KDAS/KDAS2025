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

## 6. 동역학 모델링 – `AEB_Dynamics` & `QCar_Dynamics`

<img src="images/control18.gif" width="450" alt="Vehicle Dynamics – Longitudinal/Lateral Modeling"/>

여러 제어 구조를 실험할수록 한 가지 의문이 반복해서 떠올랐다.  
센서가 말해주는 값만으로는 차량의 움직임을 온전히 제어할 수 없다는 점이었다.  
하지만 자율주행 알고리즘은 결국,

> “지금 넣은 조향·가속 입력이 앞으로 어떤 움직임을 만들 것인가?”

를 전제로 한다.  
즉, 센서가 말하는 **현재의 관측값** 위에,  
차량이 실제로 **어떻게 움직일 수 있는지에 대한 예측적 이해**가 필요했다.  
이 고민에서 동역학 모델링이 시작되었다.

---

### 6.1 첫 번째 모델 – Bicycle Model을 통해 이해한 ‘차가 움직이는 방식’

초기 단계에서는 MATLAB의 **Bicycle Vehicle Model**을 활용했다.  
이 모델은 일반적인 가솔린 차량의 거동을 기반으로 만들어져 있으며,

- 타이어–지면 상호작용,
- 차량의 기본 종·횡 방향 동역학

을 이해하기에 좋은 참고 구조였다.

우리가 이 모델을 사용한 이유는 단순했다.

- 이미 검증된 물리 모델이었고  
- 동역학 블록 구조를 빠르게 이해할 수 있었고  
- “차량이 어떤 원리로 움직이는지”를 학습하기에 적절했기 때문이다.

또한 우리의 실험 환경은 **평면 트랙**이었기 때문에,  
3DOF 중 z축(상하 방향) 거동은 제외하고,  
x–y 평면에 집중한 **2DOF 구성**으로 단순화해 사용했다.

다만 이 모델은 근본적으로 **풀사이즈 가솔린 차량**을 기준으로 만들어져 있었다.  
그래서 “차량 물리의 개념을 배우는 데는 적합했지만,  
실제로 우리가 제어해야 할 **QCar의 거동과는 거리가 있었다.**”

이 지점이, 우리가 **QCar 전용 동역학 모델(QCar_Dynamics)** 을 따로 구성하게 된 출발점이었다.

---

### 6.2 왜 QCar Dynamics를 다시 만들었는가 – 플랫폼이 바뀌면 물리식도 바뀐다

Quanser의 **QCar**는 일반 차량과 구조가 크게 다르다.

- BLDC 모터 + ESC 기반 구동
- 기계식 브레이크 없음
- 1/10 스케일 특유의 저질량·저속 플랫폼
- 굴림 저항과 모터 응답이 훨씬 크게 체감됨

이런 구조적 차이 때문에,  
기존 Bicycle Model만으로는 QCar의 **속도 변화, 감속 패턴, 슬립 현상**을 제대로 설명할 수 없었다.

그래서 우리는 QCar의 실제 거동을 설명하기 위해  
여러 물리식을 다시 검토하고, 그중에서

> “QCar의 움직임을 결정하는 데 꼭 필요한 관계식”

만을 선별해 **QCar_Dynamics** 블록을 별도로 구성했다.

---

### 6.3 QCar용 물리식을 고른 기준 – ‘예측 가능한 거동’을 만들기 위해

QCar Dynamics를 만들기 위해 검토한 물리식은 많았지만,  
우리가 원하는 것은 “모든 물리 현상”이 아니라

> 다음 움직임을 이해하는 데 필수적인 관계식

이었다. 핵심 기준은 다음 세 가지였다.

#### ① ESC Duty → 전압 → 전류 → 모터 토크 → 바퀴 회전 → 차량 속도

QCar에서 가장 중요한 특성은,  
**속도 변화가 ESC Duty 변화에 거의 직결된다는 점**이다.

따라서 이 전달 경로를 모델에 포함시키는 것은 필수였다.  
이 관계가 없으면 “왜 속도가 변하는지”를 해석할 수 없기 때문이다.

#### ② 감속은 브레이크가 아니라 ‘저항’으로 발생

QCar에는 별도의 브레이크가 없다.  
따라서 감속은

- 모터 역토크,
- 굴림저항,
- 공기저항

의 조합으로 설명된다.  
이 요소들을 포함해야만, SLAM·회피 기동·제어기가 사용하는 속도값이  
**“물리적으로 가능한 범위인지”** 판단할 수 있다.

#### ③ 저질량 플랫폼 특성상, 작은 슬립도 오차를 키운다

1/10 스케일 저질량 플랫폼에서는  
작은 슬립도 실제 속도와 센서가 보는 속도 사이에 의미 있는 차이를 만든다.  
그래서 슬립은 완전히 무시하기보다는,  
단순화된 형태로라도 모델에 포함시키는 것이 필요했다.

여기서 언급한 것들은 전체 모델의 일부에 불과하지만,  
공통점은 하나다.

> “QCar가 실제로 어떻게 움직이는지”를 설명하는 데 꼭 필요한 요소들만 선별했다는 점.

이 방식을 통해 우리는

- 불필요한 복잡성은 지우면서도  
- **다음 입력에서 차량이 어떻게 반응할지 예측 가능한 구조**를 만들 수 있었다.

---

### 6.4 동역학 모델이 만들어준 것 – ‘다음 움직임을 이해할 수 있는 기반’

동역학 모델링은 단순히

> “시뮬레이션을 더 정교하게 만드는 작업”

이 아니었다.

이 모델을 통해 우리는 처음으로,

- 우리가 넣은 조향·가속 명령이  
  다음 순간 어떤 속도·위치 변화를 만들 수 있는지,
- ESC Duty 변화가 실제 속도 변화에  
  어느 정도 비율로 반영되는지,

를 **물리식으로 설명할 수 있게** 되었다.

즉, 동역학 모델은

> 센서가 말해주는 “현재 값” 위에  
> “차량이 다음에 어떻게 움직일 수 있는지”를 그려볼 수 있게 해주는  
> **예측적 사고의 출발점**

이었다.

이 물리적 이해는 이후에 설계한

- SLAM 기반 위치 추정,  
- MPC와 같은 고급 제어기,  
- AEB 및 회피 기동 알고리즘

을 구성하는 데 있어 **공통의 기반**이 되었다.  
센서 값만 바라보던 시스템에서,  
“다음 움직임을 스스로 예측할 수 있는 시스템”으로 나아가는 첫 단계가 바로 이 **동역학 모델링**이었다.

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

## 10. Simulator Constraints & Early Control – Xytron Preliminary Stage

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

## 11. After Perception – “We See the Lane. Now How Do We Drive?”

<img src="images/SCNN10.gif" width="450" alt="Lane Mask to Waypoint Conceptual Transition"/>

Once we could reliably detect lanes, a more fundamental question emerged:

> “It’s good that we can see the lane, but how do we actually move using that information?”

SCNN provides lane information as **images**, but controllers require **numeric coordinates**. Perception alone is “seeing.” To move the vehicle, we must convert this information into mathematically defined targets—concrete points that the vehicle can follow.

We reasoned as follows:

- An autonomous vehicle moves by following a sequence of **continuous target points**.  
- We need a way to express these target points.  
- What if we represent the driving path as a sequence of **waypoints**?

This led us to the widely used concept of **waypoints** in autonomous driving: generating points along the lane center and connecting them into a drivable path.

### 11.1 Depth-Based Waypoint Generation – and Its Structural Limits

To generate waypoints, we must convert pixel coordinates into real-world coordinates (x, y). A natural idea was to use a **depth camera** and map SCNN’s pixel outputs to metric coordinates.

However, this is where we encountered a fundamental issue.

The QLabs simulator provider had already announced that **the depth camera was not functioning correctly**. We confirmed that the depth frames:

- contained very few valid values,
- intermittently produced invalid values like 0 and ∞, and
- exhibited highly inconsistent patterns from frame to frame.

In other words, the depth sensor was **unusable** for real-time coordinate conversion.

This was more than a sensor choice issue; it meant that our entire approach did not structurally fit the QLabs environment.

### 11.2 A More Important Lesson – Waypoints Are an “Language,” Not Just Points

Depth was not the only problem. We realized something deeper: even if depth had worked perfectly, the waypoints we were generating would have been unsuitable for control.

Sensor-based waypoints are inherently:

- noisy and unstable from frame to frame,  
- sensitive to small sensor perturbations, and  
- discontinuous, with frequent gaps between points.

From the controller’s perspective, this is not a **trackable path** but a **collection of jittering points**.

From this experience, we learned that **waypoints are not just points but a structural “language” for expressing stable, drivable paths**. Unless this language is used correctly, even the best perception or control algorithms cannot produce stable vehicle motion.

### 11.3 Our Conclusion from This Failure

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
## 12. 궤적 추종 제어의 시작 – Stanley Controller

Xytron 대회에서 차량을 처음 실제로 움직여 보았을 때,  
우리는 **PID 제어기의 한계를 뼈저리게 체감**했다.

- 직선에서는 어느 정도 정상적으로 주행했지만  
- 코너에 들어서는 순간 **진동**, **오버슈트**, **언더슈트**가 반복되었고  
- 차량은 안정적인 궤적을 유지하지 못했다.

그때 우리는 분명히 알 수 있었다.

> “단순 피드백만으로는 자율주행을 만들 수 없다.”

PID를 넘어선 보다 구조적인 제어 방식이 필요했다.  
그때 가장 먼저 떠올린 것이 바로 **Stanley 제어기**였다.

---

### 12.1 Stanley를 위한 첫 설계: SCNN + Depth 기반 3D Waypoint

<img src="images/control5.png" width="450" alt="SCNN + Depth 기반 Stanley 구조"/>

Stanley를 우선 고려한 이유는 명확했다.

- Stanley는 **2005 DARPA Grand Challenge 우승 차량에 실제 사용된 제어기**이며  
- 센서 기반 waypoint 추종, 고속 안정성 측면에서 이미 검증되었고  
- 우리가 만들고자 하는 **센서 기반 실시간 자율주행 구조**와 매우 잘 맞았다.

특히 Stanley는 다음 두 값을 기반으로 조향을 보정한다.

- 횡오차 (**cross-track error**)  
- 헤딩 오차 (**heading error**)  

PID보다 훨씬 안정적이고 **기하학적으로 정당한 조향 응답**을 만든다.

하지만 Stanley가 제대로 작동하기 위해 필요한 조건이 있었다:

> **정확한 3D 차선 경로(waypoint)가 있어야 한다는 것.**

이를 위해 우리는 아래와 같은 구조를 설계했다.

1. **SCNN으로 차선 마스크 검출**  
2. **Depth 카메라로 각 픽셀의 실제 거리 추정**  
3. **두 정보를 결합해 3D waypoint 생성**  
4. **생성된 waypoint를 Stanley 제어기의 입력으로 사용**

이 파이프라인이 제대로 작동한다면,  
Stanley는 이론적으로도, 실전에서도 가장 강력한 조향 제어기가 될 수 있었다.

---

### 12.2 예상 밖의 장애물 – Depth 카메라 버그로 인한 “전면 중단”

그러나 실험 과정에서 **결정적인 문제**가 발생했다.

> QLabs Depth 카메라가 시스템 버그로 인해  
> 올바른 depth 값을 제공하지 않았던 것이다.

실제 출력된 값은:

- 0  
- 무한대(∞)  
- 랜덤 노이즈  
- 프레임마다 불규칙한 값 변화  

즉, **실제 거리 정보를 완전히 잃은 상태**였다.

Depth 카메라 없이는:

- SCNN의 픽셀 좌표를  
- 실제 월드 좌표(3D waypoint)로  
- 변환할 수 없다.

그 말은 곧:

> 우리가 준비한 **Stanley 기반 제어 구조는 초기 단계에서 완전히 중단**될 수밖에 없었다는 의미였다.

---

### 12.3 다음 단계: Waypoint 생성을 위해 Localization에 집중하다

Stanley를 사용하려면 3D waypoint가 필요하고,  
3D waypoint를 만들려면 Depth가 반드시 필요했다.  
하지만 Depth가 **구조적으로 불가능한 환경**이었다.

따라서 우리는 전략을 재정비했다.

> **Depth 기반 waypoint → 불가능**  
> **Localization 기반 waypoint → 가능**

즉,  
Depth로부터 좌표를 얻는 대신,

- SLAM을 통해 차량의 위치를 안정적으로 추정하고  
- 트랙 전체의 전역 waypoint를 기준으로  
- 해당 위치에서 가장 가까운 waypoint를 추종하는 방식으로 전환했다.

이 결정은 이후 Pure Pursuit 선택과 SLAM–Waypoint 기반 구조 설계의  
핵심 기반이 되었다.

---

### 12.4 요약

- PID는 곡률 변화에 취약하여 실전에서 불안정  
- Stanley는 이론적으로 가장 강력한 선택  
- SCNN + Depth로 Stanley에 필요한 3D waypoint를 만들 계획이었음  
- 하지만 **QLabs Depth 버그 → 3D waypoint 생성 불가 → Stanley 중단**  
- 이후 시스템 구조를 **SLAM–waypoint 기반**으로 재구성하게 됨

Stanley는 실패했지만,  
이 실패는 이후 가장 현실적인 제어 구조를 찾는 중요한 출발점이 되었다.

---

## 13. Localization and Stabilization – SLAM + Kalman Filter

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

## 14. Path Planning – Combining RRT and DQN

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

## 15. 제어의 진화 – Pure Pursuit, 그리고 MPC

<img src="images/control19.gif" width="450" alt="Control Stack – Pure Pursuit and MPC"/>

Stanley 제어기의 적용이 어려워지자, 우리는 자연스럽게 다음 대안으로 **MPC(Model Predictive Control)** 를 검토하게 되었다.  
MATLAB에서 제공하는 여러 경로 추종 예제를 분석하는 과정에서, 복잡한 종·횡 통합 제어 문제에서 MPC가 자주 등장한다는 점을 확인했기 때문이다.  

특히 **“예측 기반 최적 제어”** 라는 구조는 충분히 매력적이었다.  
“우리 차량도 더 정교하게 제어할 수 있지 않을까?”라는 기대도 있었다.

---

### 15.1 다른 길을 찾다: MPC(Model Predictive Control) 테스트

실제로 MPC는:

- 미래 상태를 예측하고,
- 조향·가속·제약조건을 함께 고려하여,
- 특정 예측 지평선 내에서 **가장 합리적인 입력을 선택하는** 고급 제어 방식이다.

이론적으로는 우리가 원하던 “예측적이고 부드러운 제어”에 가장 잘 맞는 선택처럼 보였다.  
그러나 구현을 시도하면서, 이 방식이 **학부 수준 환경에서 안정적으로 다루기 어렵다**는 현실적인 벽을 마주하게 되었다.

#### 1) 가장 큰 문제: 정확한 차량 모델의 부재

MPC는 모델의 작은 오차에도 쉽게 불안정해지는 제어기다.  
하지만 우리 환경에서는 다음과 같은 핵심 파라미터를 **정확히 알 방법이 없었다.**

- 타이어 cornering stiffness  
- 관성모멘트 \( I_{zz} \)  
- 실제 조향 응답, 가속·감속 응답 특성  

이 값들은 문서로 제공되지 않았고, 이를 실험적으로 추정할 수 있는 설비와 시간도 부족했다.  
즉,

> “모델이 있어야 쓸 수 있는 제어기”임에도,  
> **신뢰할 수 있는 모델을 만들 수 있는 조건이 아니었다.**

#### 2) 비용함수(cost function) 설계의 난관

MPC가 요구하는 **비용함수(Q, R 가중치)** 설계도 큰 난관이었다.

- MATLAB 예제에서는 이미 잘 튜닝된 가중치가 제공되지만,  
- 실차 환경에서는 모든 Q·R을 **처음부터 직접 설정**해야 했다.

그러나 모델이 정확하지 않으니:

- 어떤 Q·R 조합도 이론과 실제 사이에서 일관되게 작동하지 않았고,
- 튜닝의 기준점 자체가 부족한 상황이었다.

결과적으로,

> MPC는 **이론적으로는 매우 강력하고**,  
> 구현 자체도 코드 레벨에서는 가능했지만,  
> **학부 수준 장비와 환경에서 “대회용 안정성”을 확보하기에는 한계가 명확했다.**

이에 우리는 MPC를 최종 제어기로 채택하지 않고,  
보다 **강건하고 실시간성이 확보되는 방식**으로 방향을 전환하게 되었다.

---

### 15.2 최종 선택: SLAM 위치 + 트랙 waypoint + Pure Pursuit

<img src="images/control14.png" width="450" alt="SLAM + Waypoint + Pure Pursuit"/>

정리하자면, 우리가 직면한 제약은 다음과 같았다.

- **Stanley**  
  → QLabs의 Depth 카메라 버그로 인해 **실행 자체가 불가능**  
- **MPC**  
  → 차량 모델 파라미터 정확도 부족으로 인해 **실용적인 안정성 확보가 어려움**

그 결과, 현실적으로 남은 선택지는 단 하나였다.

1. SLAM이 제공하는 **비교적 안정적인 차량 현재 위치**
2. 대회 트랙에 대해 **사전에 정의한 정적 waypoint 맵**

이 두 정보를 활용하면서도 **안정적으로 동작하는 제어기**,  
바로 **Pure Pursuit**였다.

Pure Pursuit의 장점은 우리가 가진 환경과 완벽히 부합했다.

- 정밀한 3D 차선 정보가 필요 없고  
- 복잡한 고차 동역학 모델이 없어도 되며  
- **look-ahead point** 만으로 안정적인 조향이 가능하고  
- 계산량이 적어 **실시간성이 뛰어나다**

즉, 우리의

- 센싱 환경,  
- 모델링 정확도,  
- 시뮬레이터 제약,  
- 대회 규칙

을 모두 고려했을 때,  
**Pure Pursuit는 가장 단순하면서도 가장 안정적으로 주행 가능한 유일한 선택**이었다.

---

### 15.3 Pure Pursuit의 문제점과 우리가 선택한 해결책

그러나 Pure Pursuit 역시 **구조적인 한계**가 있었고,  
실제 주행 과정에서 다음 두 가지 문제가 반복적으로 나타났다.

#### 1) 직선 구간에서의 흔들림(Oscillation)

직선 구간에서:

- 아주 작은 오차도 과도한 조향으로 반영되며,
- 핸들이 좌우로 흔들리는 진동 현상이 발생했다.

**해결 전략**

우리는 트랙 전체를 **직선 / 곡선 구간으로 분리**하고,  
waypoint 맵에 **구간 타입 정보**를 포함시켰다.

- **직선 구간에서는**  
  - Look-ahead distance를 크게 설정  
  - Steering saturation(최대 조향각 제한)을 강화  

를 적용해,  
불필요한 조향을 억제하고 **안정적인 직선 주행**을 확보했다.

#### 2) 곡선 구간에서의 안쪽 침투(Inside Drift)

곡선 구간에서는:

- Look-ahead가 너무 길면 경로를 **직선에 가깝게 추종**하는 경향이 생기고,
- 그 결과 차량이 커브 안쪽으로 말리는 현상이 나타났다.
- 속도가 높을수록 언더스티어와 saturation이 쉽게 발생했다.

**해결 전략**

- **곡선 구간에서는**  
  - Look-ahead distance를 짧게 설정  
  - Target velocity(목표 속도)를 낮게 설정  

을 통해:

- 곡률 변화에 더 빠르게 반응하고,  
- 무리 없는 조향 여유를 확보하도록 했다.

이렇게 **구간 타입(직선/곡선)에 따라 서로 다른 튜닝을 적용한 Pure Pursuit**는,

- 직선에서의 진동 문제와  
- 곡선에서의 안쪽 침투 문제를 모두 해결했으며,  

테스트 결과 **waypoint를 정확하게 추종하는 것**을 확인할 수 있었다.  
이 과정에서 **waypoint 자체의 전처리와 맵 구조 설계가 얼마나 중요한지**가 더욱 부각되었다.

---

### 15.4 결론 – 여러 실패 끝에 얻은 가장 현실적인 해답

여러 제어기를 실험하고, 실패하고, 다시 시도하는 과정을 거치며  
우리는 다음 사실을 받아들여야 했다.

- **Depth 버그**로 인해 Stanley 기반 구조는 **실현 불가능**했고  
- 정확한 동역학 파라미터 부족으로 MPC는 **신뢰도 있게 작동하기 어려웠으며**  
- **SLAM 위치 + 트랙 waypoint** 만으로도 안정적으로 작동하는 제어기는  
  **Pure Pursuit뿐이었다**

따라서 최종적으로 우리는 **Pure Pursuit 기반 제어 구조**를 채택했다.  
이 결정은 단순히

> “가장 간단해 보여서”

내린 선택이 아니라,

> **대회 환경에서 실제 차량을 가장 안정적으로 움직일 수 있는,  
> 유일하고 현실적인 선택이었다.**

여러 제어 이론과 구현을 직접 부딪혀 본 끝에,  
우리가 가진 조건 안에서 **“정말로 끝까지 책임질 수 있는 제어기”** 가 무엇인지를 선택한 결과가  
바로 이 Pure Pursuit였다.

---

## 16. 경로 계획 – RRT와 DQN의 결합

<img src="images/rrt1.png" width="450" alt="RRT Exploration and Global Path Skeleton"/>
<img src="images/rrt10.png" width="450" alt="Global Path and PH-Based Avoidance Paths"/>

자율주행 시스템을 구현하는 과정에서 우리는 **인지(Perception)** 와 **위치 추정(Localization)** 을 단계적으로 구축해 왔다.  
앞 단계에서는 SCNN 기반 차선 인지를 통해 트랙의 중심선을 추출하고, 이를 depth 정보와 결합해 waypoint를 생성하여 주행을 시도했다. 초기에는 Stanley 제어기를 사용했으나, depth 카메라의 한계로 인해 최종적으로 **Pure Pursuit 제어기**로 전환하였다.

그러나 Pure Pursuit 제어기의 특성상, 차량이 보고 있는 **국지적(local) waypoint** 만으로는 한계가 있었다.  
트랙 전체를 아우르는 **Global Path** 가 필요하다는 문제가 드러난 것이다.

Global Path를 생성하기 위해서는 먼저 차량의 위치를 **절대좌표계에서 정확하게 파악**해야 했고, 이를 위해 우리는 Cartographer 기반 SLAM을 도입하여 **Localization 모듈**을 완성했다.

SLAM을 통해 차량의 위치를 지도 좌표계에서 안정적으로 얻을 수 있게 되었지만, 이제 새로운 문제가 나타났다.

> “차선도 보고, 위치도 알고 있는데  
> 이제 차량을 **어디로**, **어떤 순서로** 움직여야 하지?”

단순히 차선 중심을 따라가는 것이 아니라, 실제 택시처럼 여러 **Pickup·Dropoff 지점**을 제한된 시간 안에 가장 효율적으로 방문해야 했다.  
즉, 전역 경로를 어떻게 생성할지(**Global Path Planning**)와  
여러 Ride의 방문 순서를 어떻게 최적화할지라는 **고난도의 경로 계획 문제**가 본격적으로 등장한 것이다.

---

### 16.1 문제 정의 – “제한 시간 내에 택시를 가장 효율적으로 움직이려면?”

Quanser ACC 대회의 메인 미션은 **도시형 트랙 내에서 택시를 운용**하는 것이었다.  
대회 측은 5분 동안 수행해야 하는 공식 Ride 리스트(A, B, C … T)를 제공했으며,  
각 Ride(A~T)는 **고정된 Pickup/Dropoff 지점**을 포함하고 있었다. 예시는 다음과 같다.

- Ride A: 1 → 8  
  - Node 1 = \([0.269, -0.049, 90°]\)  
  - Node 8 = \([-0.749, 1.077, 180°]\)

이에 따라 차량은 5분 동안 다음 과정을 반복해야 한다.

1. Ride의 Pickup → Dropoff 지점을 순서대로 방문하고  
2. Taxi Hub로 복귀한 뒤  
3. 다음 Ride를 수행하는 과정을 반복

즉, 문제는 단순히 “두 점 사이 최단 경로”가 아니라,

> 여러 Ride의 지점을 어떤 **순서**와 **경로**로 방문해야  
> 제한된 시간 안에 **최대 수익**을 낼 수 있는가?

라는 **조합 최적화 + 경로 계획 문제**였다.

---

### 16.2 왜 RRT를 선택했는가 – ‘경로 생성기’로서의 역할

트랙은 다음과 같은 요소를 포함한 복잡한 **연속 공간(Continuous Space)** 구조였다.

- 곡률이 큰 커브
- roundabout
- 교차로
- 양방향 단일차선 구조
- 경계가 꼬이는 구간

격자 기반 탐색 방식(Dijkstra 등)만으로는

- 차량의 폭과 차선 경계 사이의 여유,
- 연속적인 커브 형태와 실제 주행 가능한 궤적

을 충분히 반영하기 어렵다고 판단했다.

그래서 우리는 “경로를 탐색하는 방법”을 찾아보다가  
**RRT(Rapidly-Exploring Random Tree)** 에 주목하게 되었다.

RRT는:

- 연속 공간을 직접 샘플링하면서  
- 장애물을 피하는 경로를 빠르게 생성할 수 있고  
- 고차원에서도 잘 동작한다는 점 때문에  

실제 자율주행 연구에서도 널리 쓰이는 방법이라는 것을 논문과 자료를 통해 확인했다.

결론적으로, 우리는 전역 경로(Global Path)를 설계할 때 다음과 같은 방향을 잡았다.

> “트랙 전체를 자유공간(Free Space)으로 정의하고,  
> 그 안을 RRT로 샘플링해서 경로를 만드는 구조로 가자.”

즉, 기존 그래프 탐색 대신  
**샘플링 기반의 경로 생성기**로서 RRT를 사용하기로 한 것이다.

---

### 16.3 RRT 적용 방식 – 오프라인, 구간 분할, 그리고 스무딩

#### 1) 왜 RRT를 오프라인에서 돌렸는가?

우리는 RRT를 **실시간으로 실행해 경로를 생성하며 달리는 용도**로 사용하지 않았다.  
RRT는 강력하지만, 무작위 샘플링 특성 때문에 다음과 같은 문제가 있다.

- 실행마다 경로가 조금씩 달라지고 (RRT의 무작위성)  
- 샘플링 기반이라 생성된 경로가 **최적 경로라는 보장이 없으며**  
- 일부 실행에서는 **비효율적이거나 이상한 경로**가 나올 수도 있다.

그래서 전략을 바꿨다.

> “RRT는 오프라인에서 여러 번 수행하여  
> **가장 좋은 경로만 선택하는 방식**으로 사용하자.”

그리고 선택된 경로는

- Elastic Band  
- Spline smoothing  

과 같은 기법으로 부드럽게 다듬어,  
제어기에 넣어도 안정적으로 동작하는 **최종 Global Path**를 만들었다.

이렇게 하면 **대회 현장에서는 RRT를 실행할 필요가 없다.**  
실제 주행 시에는 **이미 검증된 Global Path를 그대로 따라가기만 하면 되기 때문에**,  
랜덤성 문제를 구조적으로 해결할 수 있었다.

#### 2) 왜 트랙 전체를 한 번에 하지 않고, 구간별로 나누었는가?

초기에는

> “트랙 전체를 하나의 자유공간으로 두고 RRT를 돌려 보자”

고 시도했다. 그러나:

- 트리가 특정 영역(예: 긴 직선 구간)에만 몰려 자라거나  
- 교차로, roundabout 등의 복잡한 구조 때문에  
  자유공간의 정의가 뒤섞이는 문제가 발생했다.

그래서 다음과 같은 방식으로 접근했다.

1. 트랙을 **구간 단위**로 분할  
   - 교차로 사이  
   - roundabout  
   - 직선 구간  
   - 곡선 구간 등
2. 각 구간을 **별도의 자유공간**으로 정의한 뒤  
3. 구간별로 **독립적인 RRT**를 수행

이렇게 생성된 **구간별 최적 경로**들을 이어 붙여  
하나의 **Global Path**를 구성했다.

---

### 16.4 Directed Graph로의 변환 – 자료구조·알고리즘 수업의 재해석

RRT로부터 얻은 Global Path는 **연속적인 좌표들의 집합**이다.  
그러나 택시 문제를 풀기 위해서는 단순히 경로를 따라 가는 것만으로는 부족했다.

우리는 다음과 같은 의사결정을 해야 했다.

- 교차로에서 **어떤 방향으로 갈 것인지**  
- 역주행이 허용되지 않는 **일방통행 구조**를 어떻게 반영할지  
- 각 Ride의 **pickup/dropoff 지점을 반드시 방문해야 하는 제약**을 어떻게 관리할지

이러한 의사결정은 연속 좌표보다는,  
트랙을 **노드(Node)** 와 **간선(Edge)** 로 표현한 **directed graph** 구조가 더 적합했다.

따라서 우리는 Global Path를 기반으로:

1. 분기점/합류점/교차로를 **노드(Node)** 로 정의하고  
2. 실제 주행 가능한 방향만을 **방향성 간선(Directed Edge)** 으로 연결하여  
3. **역주행 가능성은 애초에 그래프 구조에서 제거**하였다.

이 과정에서 자료구조·알고리즘 수업에서 배웠던 개념들이 자연스럽게 활용되었다.

- 그래프를 정의하고,  
- 노드와 간선을 구성하고,  
- 경로 탐색이 가능한 형태로 구조화하는 과정 자체가  

수업에서 이론적으로 배웠던 **그래프 모델링을 실제 트랙에 적용**한 사례가 된 것이다.

---

### 16.5 왜 DQN인가 – Ride 순서 최적화 문제

Directed Graph가 준비되면,  
두 노드 간 최단 경로는 학부 수업에서 배운 **Dijkstra / BFS / DFS** 와 같은 기존 알고리즘으로 충분히 계산 가능하다.

그러나 우리가 풀어야 하는 핵심 문제는 “최단 경로”가 아니었다.

> 여러 개의 pickup/dropoff 지점을  
> **어떤 순서로 방문해야 5분 동안 최대 수익을 얻는가?**

이 문제는 **단일 경로 탐색**이 아니라 **순서 최적화 문제**이며,  
우리가 배운 기존 알고리즘만으로는 해결이 불가능했다.

- **BFS / DFS**  
  → 모든 순열을 탐색해야 하므로 규모가 커지면 확장 불가  
- **Dijkstra**  
  → 한 쌍의 지점 간 최단경로만 제공  

즉, “경로 찾기”는 해결되었지만,  
“**방문 순서 결정**”이라는 근본적인 문제는 여전히 남아 있었다.

이 지점에서, 기존 지식만으로는 한계가 분명했기 때문에  
우리는 “정책을 스스로 학습하는 방식”인 **강화학습(Reinforcement Learning)** 을 도입하기로 했다.

참고한 자료는 다음과 같다.

- Sutton & Barto, *Reinforcement Learning: An Introduction*  
- 오일석 & 이진선, *파이썬으로 만드는 인공지능*

이를 통해 Q-learning 구조를 이해한 뒤,  
이 문제를 **DQN(Deep Q-Network)** 으로 모델링할 수 있다는 결론에 도달하였다.

DQN을 선택한 이유는 다음과 같다.

- Directed Graph에서의 의사결정은  
  “현재 노드에서 갈 수 있는 몇 개의 방향 중 하나를 고르는”  
  **이산적(discrete) 행동 구조**이기 때문에,  
  Q-learning 기반인 **DQN이 문제의 특성과 가장 잘 맞았다.**
- 여러 Ride 조합이 만들어내는 방문 순서 결정 문제는  
  **규칙 기반으로 직접 정의하기 어렵다.**  
  그래서 우리는 **보상 함수**만 설계하고,  
  그 보상 신호를 기반으로 **DQN이 경험을 통해 최적 방문 순서를 학습**하도록 하는 구조를 택했다.

이를 바탕으로 문제를 다음과 같이 정의했다.

- **상태(state)** : 남은 경유지 집합, 현재 위치  
- **행동(action)** : 다음에 방문할 노드 선택  
- **보상(reward)** : 이동 거리 절약, 역주행 패널티, Ride 완료 보상  

이렇게 해서 단순 거리 최적화가 아닌,

> “최대 수익을 달성하는 방문 순서 정책”

을 학습할 수 있었다.

---

### 16.6 왜 실시간이 아니라 오프라인 DQN인가 – 운영 전략으로의 전환

초기에는 DQN을 **실시간으로 실행**하여,  
경유해야 하는 경유지의 노드 번호를 넣고 **경로의 순서를 계산하는 방식**을 시도했다.  
하지만 실험해 보니:

- 노드 전체 개수: 약 60개  
- DQN이 한 Ride의 최적 주행 시퀀스를 계산하는 데: **약 10~15초**

Ride는 5분 동안 여러 개를 수행해야 했기 때문에,  
**실시간 계산은 비효율적**이라는 결론에 도달했다.

따라서 전략을 다음과 같이 바꾸었다.

- 모든 Ride A~T에 대해  
  **DQN이 최적의 시퀀스를 미리 계산**해 두고,
- 대회에서는  
  - Ride 번호 입력  
  - → 미리 계산된 노드 시퀀스 불러오기  
  - → 주행 시작  

이 구조를 통해 현장에서는 **즉시 차량이 출발할 수 있도록** 운영 효율을 높일 수 있었다.

---

### 16.7 최종 구조 정리 – RRT, Directed Graph, DQN의 역할

이제 RRT, Directed Graph, DQN 각각이 어떤 역할을 했는지 정리하면,  
최종 구조는 다음과 같다.

1. **RRT 기반 Global Path 생성 (오프라인)**  
2. Global Path를 바탕으로 **구간별 Directed Graph 구성**  
3. **DQN이 그래프 위에서 다음에 방문할 노드/경유지 시퀀스를 결정**  
4. 선택된 노드에 대응하는 **waypoint JSON을 순서대로 로딩**  
5. 각 waypoint 시퀀스를 **Pure Pursuit 제어기**로 추종  
   (자세한 내용은 제어기 항목 참조)

---

### 16.8 이 섹션이 프로젝트 전체에서 갖는 의미

경로 계획 파트에서 우리는:

- 자료구조·알고리즘 수업에서 배운 **그래프 모델링과 탐색 개념**을  
  실제 트랙 구조에 적용했고,
- **RRT**를 활용해 복잡한 연속 공간에서의 **전역 경로 생성**을 구현했으며,
- 기존 접근법으로 해결하기 어려운 **방문 순서 최적화 문제**를  
  강화학습 교재와 논문을 바탕으로 **DQN 기반 정책 결정 구조**로 확장했다.

즉, 이 경로 계획 모듈은

> 학부에서 배운 기초 지식과  
> 추가로 학습한 고급 개념을  
> 실제 트랙의 제약 조건에 맞추어 **통합적으로 구현해낸 완성형 사례**이며,

본 프로젝트의 **핵심 기술적 성취를 가장 잘 보여주는 부분**이라고 할 수 있다.


---

## 17. Avoidance Maneuver – TDM + TG–PH Hybrid Framework

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

Even after implementing AEB, one question kept recurring.

In real driving, **full emergency braking** is not always the best or only action. When a pedestrian suddenly appears or a cone or stopped vehicle blocks the lane, drivers usually:

- avoid the obstacle, and then  
- return to their original lane.

Our system, however, did not have this second stage. We could brake hard, but not **“avoid and rejoin the flow.”** We had no answer to:

> “How do we pass the obstacle, not just stop in front of it?”

This marked the beginning of our work on **avoidance maneuvers**.

### 17.1 Problem 1 – The Vehicle Cannot Decide *When* and *Where* to Avoid

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

### 17.2 Problem 2 – Even After Deciding to Avoid, the Vehicle Lacks a Concrete Path

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

### 17.3 A Second Clue from PH Curves – Avoiding “Unnecessary Motion”

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

### 17.4 Reconstructing Two Papers into a Single Framework

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

### 17.5 What We Ultimately Built

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
