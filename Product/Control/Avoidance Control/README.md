# Autonomous Evasive Maneuver (자율주행 회피기동)

Autonomous vehicles frequently encounter sudden obstacles in the front or on the sides while driving. In such cases, the vehicle must execute an **evasive maneuver** safely without collision and, to do so, it requires an algorithm that generates safe and efficient trajectories **in real time** as the environment changes.  
To simultaneously secure **efficiency** and **stability**, we design the evasive maneuver algorithm as a **hybrid architecture** split into two parts:

- **Tactical Decision Making (TDM):** high-level behavior decision (keep lane, change lane, decelerate, etc.)  
- **Trajectory Generator (TG):** dynamically feasible trajectory generation under vehicle constraints

Especially in TG, we introduce the **Pythagorean Hodograph (PH)** technique to overcome the limitations of conventional polynomial trajectory generation. A PH curve guarantees **$C^2$ continuity** (continuous curvature and curvature rate), which suppresses abrupt changes in steering angle and steering rate. Consequently, the proposed PH-augmented hybrid structure preserves computational efficiency while significantly improving steering stability and ride comfort.

---

## 1. Tactical Decision Making (TDM)

TDM is responsible for **high-level behavior decisions**. Among candidate behaviors such as lane keeping, lane change, and deceleration, TDM selects the safest and most efficient behavior for the current traffic context.

### (1) Decision Topology

**Reachability** between gaps is defined as

$$
\mathrm{Reach}_{i,j}=
\begin{cases}
\mathrm{T}, & \left| \mathrm{Gap}_i.s-\mathrm{Gap}_j.s \right| < \dfrac{\mathrm{Gap}_i.l+\mathrm{Gap}_j.l}{2} \\
\mathrm{F}, & \text{otherwise}
\end{cases}
$$

This compares the center distance of two gaps with their lengths; if their spatio-temporal regions overlap or are adjacent, the transition is considered **reachable** (True). Starting from the root node that contains the ego vehicle, we explore only reachable gaps, maximizing cost-computation efficiency.  
Let $N_R$ be the number of reachable transitions. Then the search complexity reduces to $O\!\left(N_R^{2}\right)$, making the decision process suitable for real-time operation.

<br/>

<img src="../../../images/avo1.png" width="660" alt="Basic Frenet topology and reachable gaps"/>

### (2) Cost Function

<br/>

<img src="../../../images/avo2.png" width="780" alt="Cost function overview"/>

The total cost for gap selection is

$$
J = C_{\mathrm{road}} + C_{\mathrm{traffic}} + C_{\mathrm{arrival}}
$$

**① Road Cost**

$$
C_{\mathrm{road}}^{\,i}=1-\frac{\mathrm{Gap}_i.l}{100}
$$

A shorter gap length implies a smaller safety region; hence a higher cost. In effect, $C_{\mathrm{road}} \propto 1/L_{\mathrm{gap}}$.

**② Traffic Cost**

$$
C_{\mathrm{traffic}}^{\,i}=C_{\mathrm{traffic,1}}^{\,i}+C_{\mathrm{traffic,2}}^{\,i}
$$

$$
C_{\mathrm{traffic,1}}^{\,i}=
\begin{cases}
0, & n_{\mathrm{car}}=0 \\
-\,w_{\mathrm{car}}\dfrac{\mathrm{Gap}_{\mathrm{car}}}{n_{\mathrm{car}}}, & \text{otherwise}
\end{cases}
$$

$$
C_{\mathrm{traffic,2}}^{\,i}=
\begin{cases}
c_f\dfrac{v_{\mathrm{ego}}}{v_{\mathrm{target}}}, & s_{\mathrm{target}}>\mathrm{Gap}_i.l \\
-\,c_b\dfrac{v_{\mathrm{target}}}{v_{\mathrm{ego}}}, & s_{\mathrm{target}}<\mathrm{Gap}_i.l
\end{cases}
$$

- $C_{\mathrm{traffic,1}}$: risk increases with the number of vehicles.  
- $C_{\mathrm{traffic,2}}$: dynamic interaction via relative speed ratio. For example, when $v_{\mathrm{ego}}>v_{\mathrm{target}}$, the rear-end risk increases and the cost becomes larger.  
Together, $C_{\mathrm{traffic}}$ quantifies the **trade-off** between traffic flow and safety.

**③ Arrival Cost**

$$
C_{\mathrm{A\to B}}^{\,i}=C_{\mathrm{long}}^{\,i}+C_{\mathrm{lat}}^{\,i}
$$

$$
C_{\mathrm{long}}^{\,i}
=1-\frac{\min\!\left(A_f,B_f\right)-\max\!\left(A_r,B_r\right)}{\mathrm{Gap}_A.l}
$$

$$
C_{\mathrm{lat}}^{\,i}=
\begin{cases}
\dfrac{\mathrm{LaneWidth}}{10}, & \Delta \mathrm{Idx}_{\mathrm{lane}}\le 1 \\
2.5, & \Delta \mathrm{Idx}_{\mathrm{lane}}> 1
\end{cases}
$$

- $C_{\mathrm{long}}$: difficulty based on the **overlap ratio** between gaps  
- $C_{\mathrm{lat}}$: penalty on the **number of lane changes**

<br/>

<img src="../../../images/avo3.png" width="760" alt="Four scenes for arrival cost from A to B"/>

---

## 2. 2D LiDAR–Vision Fusion Pipeline for Vehicle Candidates

This pipeline is configured as a **YOLO + Depth + 2D LiDAR** fusion system to robustly detect surrounding vehicles and to provide TDM with accurate **distance, velocity,** and **TTC** in real time.

### 1) Candidate Generation (YOLO + Depth)

**(a) YOLO object detection**  
Using YOLOv8 on the RGB camera image, we detect 2D bounding boxes of vehicles, pedestrians, and obstacles. Each detection (with ID) is published on `/yolo_objects`.

**(b) Depth-based range estimation**  
From the lower ROI of each bounding box, we obtain depth samples and apply mean/median with IQR filtering to extract the object’s average depth $z$. Using camera intrinsics $(f_x,f_y,c_x,c_y)$, points are **back-projected** to 3D $(X,Y,Z)$. The results are published as `/objects_3d` (`vision_msgs/Detection3DArray`).

- Sample several depth points near the lower center of the ROI  
- Remove outliers via IQR and adopt the median  
- Apply **EMA** to stabilize frame-to-frame range  
- Output: $(x,y,z)$ for each vehicle with class ID

At this stage, coarse vehicle candidates are created. The LiDAR node refines **range** and estimates **velocity** precisely.

### 2) LiDAR-based Range Refinement and Candidate Verification (LiDAR–TDM Bridge)

We take YOLO/Depth candidates and refine distance, bearing, and velocity using 2D LiDAR.

**(a) LiDAR gating**  
For each candidate at Depth-based position $(x,y)$, compute polar $(r_d,\theta_d)$. In `/scan`, define a gate window **centered** at $(r_d,\theta_d)$ with

$$
\theta_{\mathrm{gate}}=\pm 5^\circ,\qquad r_{\mathrm{gate}}=\pm 0.5\,\mathrm{m}
$$

Collect distances inside the gate and compute the **median** $r_{\mathrm{med}}$ to correct the Depth estimate $r_d$.

**(b) Range-consistency check**

$$
\left|r_{\mathrm{med}}-r_d\right|\le \varepsilon(r)=\varepsilon_0+k_r\,r_d
$$

Symbols and meanings:

| Symbol | Meaning |
|---|---|
| $r_d$ | Range computed from camera Depth (via back-projection) |
| $r_{\mathrm{med}}$ | Median of LiDAR ranges within the gate |
| $\varepsilon(r)$ | Distance-proportional tolerance threshold |
| $\varepsilon_0$ | Base tolerance (near-range minimum) |
| $k_r r_d$ | Increase of tolerance with range |

We use $\varepsilon_0=0.3\,\mathrm{m}$ and $k_r=0.05$. If the difference exceeds the tolerance, the candidate is rejected as **failed correction**.

**(c) Lane assignment**

Using the object’s lateral coordinate $y$, assign a lane:

$$
\mathrm{lane\_id}=
\begin{cases}
0, & |y|<\dfrac{W_{\mathrm{lane}}}{2} \\
-1, & |y-W_{\mathrm{lane}}|<\dfrac{W_{\mathrm{lane}}}{2} \\
+1, & |y+W_{\mathrm{lane}}|<\dfrac{W_{\mathrm{lane}}}{2}
\end{cases}
$$

For QCar track, $W_{\mathrm{lane}}=0.8\,\mathrm{m}$.  
Each object is labeled with a lane ID, which TDM directly uses for gap and cost calculation.

### 3) Inter-frame Velocity and TTC (Temporal Differencing)

We compute relative approach speed $v_{\mathrm{rel}}$ and **TTC** from inter-frame position changes.

**(a) Relative speed**

Assume frame period $\Delta t\approx 0.1\,\mathrm{s}$. If the same ID is detected across two consecutive frames,

$$
v_{\mathrm{rel}}=\frac{x_{\mathrm{prev}}-x_{\mathrm{now}}}{\Delta t}
$$

- Approaching the ego: $v_{\mathrm{rel}}>0$  
- Receding or stationary: $v_{\mathrm{rel}}\le 0$

This sign convention matches the TDM cost design.  
$\Delta t$ is based on the LiDAR node publish rate (10 Hz via `rclpy.Timer(0.1 s)`).

**(b) TTC**

For distance $d=\sqrt{x^2+y^2}$ and $v_{\mathrm{rel}}$,

$$
\mathrm{TTC}=
\begin{cases}
\dfrac{d}{v_{\mathrm{rel}}}, & v_{\mathrm{rel}}>0 \\
\infty, & v_{\mathrm{rel}}\le 0
\end{cases}
$$

TTC quantifies frontal-collision risk and is directly used in the $C_{\mathrm{traffic}}$ term.

---

## 3. Gap Computation and MIO (Most Important Object) Selection

### (1) Lane-wise Gap Estimation

We compute longitudinal **gaps** between front/rear vehicles per lane so TDM can quantitatively evaluate safety and space availability for **lane change (LC)** or **keep**.

- **Front gap (current lane)**  
  For vehicles with $x_d>0$ in the current lane $\bigl(\mathrm{lane}(d)=0\bigr)$, define

  $$
  \mathrm{Gap}_{\mathrm{front}}=\min_{\{d:\ \mathrm{lane}(d)=0,\ x_d>0\}} x_d
  $$

  This is the ego lane’s **forward safety margin** and is used in $C_{\mathrm{traffic}}$ and $C_{\mathrm{road}}$.

- **Lateral lane-change gap (adjacent lane)**  
  For adjacent lane $l\in\{-1,+1\}$, compute nearest front/back distances $D_F$ and $D_B$ in that lane. After subtracting ego length and safety buffer $L_{\mathrm{safe}}$, define the admissible entry gap as

  $$
  \mathrm{Gap}_{l}=
  \begin{cases}
  \mathrm{Gap}_{l}^{\text{prev}}, & \text{$D_F$ or $D_B$ missing in the current frame} \\
  0, & \text{missing persists for $N$ frames or more} \\
  \max\!\bigl(0,\ D_F+D_B-L_{\mathrm{safe}}\bigr), & \text{both $D_F$ and $D_B$ observed}
  \end{cases}
  $$

  Here $l=-1$ is the left lane and $l=+1$ is the right lane. A larger value indicates a safer entry and is used in $C_{\mathrm{road}}$ and $C_{\mathrm{arrival}}$.

- **Publish**  
  We publish the computed gaps as

  $$
  \bigl[\ \mathrm{Gap}_{\mathrm{front}},\ \mathrm{Gap}_{\mathrm{left}},\ \mathrm{Gap}_{\mathrm{right}}\ \bigr]^{\!\top}
  $$

  on `/tdm/gaps` (`Float32MultiArray`) so TDM can judge space availability and safety in real time.

### (2) MIO Selection

Within $\pm 30^\circ$ in front of ego’s heading, select the **closest** object as **MIO**. We publish its relative speed and TTC on `/tdm/mio_vrel` and `/tdm/mio_ttc`.

---

## 4. Trajectory Generator (TG)

TG turns the TDM decision into a dynamically feasible trajectory. We independently model longitudinal and lateral motions using polynomials and evaluate candidates with a cost function to select the optimum.

### (1) Longitudinal Direction

$$
s(t)=a_0+a_1 t+a_2 t^2+a_3 t^3+a_4 t^4
$$

The initial conditions use the ego’s current position, speed, and acceleration; the terminal conditions use the MIO’s speed and a safe stopping distance. To satisfy boundary conditions, solve

$$
\begin{bmatrix}
\dot s(T)-\dot s(0)-\ddot s(0)T \\
\ddot s(T)-\ddot s(0)
\end{bmatrix}
=
\begin{bmatrix}
3T^2 & 4T^3 \\
6T   & 12T^2
\end{bmatrix}
\begin{bmatrix}
a_3\\ a_4
\end{bmatrix}
$$

We sample multiple $T$ values and evaluate safety and comfort of each candidate.

### (2) Lateral Direction

$$
d(t)=b_0+b_1 t+b_2 t^2+b_3 t^3+b_4 t^4+b_5 t^5
$$

The terminal point is either the lane center or an evasive target. Smoothness is evaluated via acceleration and **jerk** limits. The coefficients satisfy

$$
\mathbf{M}
\begin{bmatrix}
b_3\\ b_4\\ b_5
\end{bmatrix}
=
\mathbf{R}(d_0,d_T,\dot d_0,\dot d_T)
$$

where $\mathbf{M}$ is a time-dependent coefficient matrix set by $T$.

### (3) TG Cost Function

$$
J^{(i)}=w_v C_v+w_a C_a+w_j C_j+w_k C_k+w_{\mathrm{ttc}} C_{\mathrm{ttc}}
$$

$$
C_j=\frac{1}{T}\int \left(\frac{j_{\mathrm{long}}(t)^2}{j_{\max,\mathrm{long}}^2}
+\frac{j_{\mathrm{lat}}(t)^2}{j_{\max,\mathrm{lat}}^2}\right)\,dt
$$

Here $C_j$ is the sum of squared longitudinal and lateral jerks, quantifying **trajectory smoothness**.

---

## 5. PH-Augmented Lateral Trajectory

While the polynomial TG is efficient, curvature and curvature rate can be discontinuous, which may induce steering oscillation and overshoot. We therefore extend the **lateral** trajectory with a **Pythagorean Hodograph (PH)** curve.

### (1) Definition and Properties

A PH curve is defined by

$$
\mathbf{r}(t)=\int_{0}^{t}\!\!\left(u(\tau)^2-v(\tau)^2,\; 2\,u(\tau)v(\tau)\right)\,d\tau
$$

where $u(t)$ and $v(t)$ are polynomials. The speed magnitude is a perfect square,

$$
\|\dot{\mathbf{r}}(t)\|=u(t)^2+v(t)^2
$$

so the **arc length** admits a closed form

$$
L(t)=\int_{0}^{t}\!\left(u(\tau)^2+v(\tau)^2\right)\,d\tau
$$

and the curvature

$$
\kappa(t)=\frac{2\left(\dot u\,v-u\,\dot v\right)}{\left(u^2+v^2\right)^2}
$$

is polynomial in $t$. Consequently, both curvature and curvature-rate are **continuous ($C^2$)**.

### (2) Application Procedure and Cost Coupling

- **Target from lateral error**  
  Using ego–obstacle lateral error, set an evasive target $d^\*$ that maintains at least a minimum clearance $\Delta d_{\mathrm{safe}}$:

  $$
  d^\*=d_{\mathrm{ego}}\pm \Delta d_{\mathrm{safe}}
  $$

- **PH guide and TG integration**  
  Construct $\mathbf{r}(t)$ to satisfy boundary conditions $(x_0,y_0,\psi_0)\rightarrow(x^\*,y^\*,\psi^\*)$, then plug it into the TG cost. PH trajectories naturally yield **lower $C_j$ and $C_k$**, thus suppressing jerk and curvature without extra constraints.

- **Benefits**  
  Guarantees continuity of steering angle and curvature-rate, improving **steering stability**.

<br/>

<img src="../../../images/avo4.png" width="760" alt="PH-based evasive path illustration"/>

We verified the algorithm using arbitrary parameter values as a functional test.

---

## 6. Implementation

We implement the TDM–TG architecture in a two-lane environment. The full system consists of **Perception → TDM → TG**, operating in real time to decide and generate evasive trajectories.

<br/>

<img src="../../../images/avo5.png" width="820" alt="System architecture diagram"/>

### (1) Perception and Data Fusion

To robustly output distances in real time from YOLOv8 detections, we design a **confidence-based fusion** of Camera, Depth, and LiDAR. A single sensor (Depth or LiDAR) alone often suffers from noise or missed detections; thus we combine them with distance-dependent weighting.

**Implementation ideas**

- YOLOv8 provides real-time object class and position. Depth samples from the center and lower parts of each box are filtered by mean/median (IQR) to get an initial distance $d_{\mathrm{depth}}$  
- LiDAR ranges $d_{\mathrm{LiDAR}}$ are computed using rays near the object’s bearing $\theta$  
- Distance-based weight $\alpha(d)$ blends the two: emphasize camera within $3\,\mathrm{m}$ and LiDAR beyond that. The fused range is

  $$
  d_{\mathrm{fused}}=\alpha(d)\,d_{\mathrm{depth}}+\bigl(1-\alpha(d)\bigr)\,d_{\mathrm{LiDAR}},\qquad \alpha(d)\in[0,1]
  $$

- Apply **EMA** to stabilize frame-to-frame variation; in outliers or missed detections, hold the previous frame value  
- Use a lane mask to classify each object as **current** or **adjacent** lane, and pass only valid objects to TDM

### (2) TDM Implementation

We design a **cost-based decision** that considers **road situation, vehicle density,** and **arrival stability**, rather than just distance thresholds.

**Implementation ideas**

- Behavior set: **Keep Lane** vs **Change Lane**  
- For each lane, compute three partial costs:
  - $C_{\mathrm{road}}$: increases when gap length $L_{\mathrm{gap}}$ is small  
  - $C_{\mathrm{traffic}}$: proportional to number of vehicles $N_{\mathrm{veh}}$ and relative speed $v_{\mathrm{rel}}$  
  - $C_{\mathrm{arrival}}$: reflects path length and number of lane changes required to the goal
- Final decision cost:

  $$
  J=C_{\mathrm{road}}+C_{\mathrm{traffic}}+C_{\mathrm{arrival}}
  $$

- Compare current vs. adjacent lanes. If the difference exceeds a threshold $\Delta J_{\mathrm{th}}$, perform a lane change:

  $$
  \mathrm{Action}=
  \begin{cases}
  \text{Change Lane}, & J_{\mathrm{adjacent}} < J_{\mathrm{current}}-\Delta J_{\mathrm{th}} \\
  \text{Keep Lane}, & \text{otherwise}
  \end{cases}
  $$

### (3) TG Implementation

Using the TDM decision, TG generates the drivable trajectory in real time. In a two-lane scenario, the behavior is either **Keep** or **Change**, and the lateral terminal condition is

$$
y(T)=
\begin{cases}
y_{\mathrm{center,current}}, & \text{Keep} \\
y_{\mathrm{center,current}} \pm W_{\mathrm{lane}}, & \text{Change}
\end{cases}
$$

**Implementation ideas**

- Replace conventional lateral polynomials with **PH** so that both curvature $\kappa$ and curvature-rate $\dot\kappa$ are continuous  
- PH curve:

  $$
  \mathbf{r}(t)=\int_{0}^{t}\!\left(u(\tau)^2-v(\tau)^2,\; 2\,u(\tau)v(\tau)\right)d\tau
  $$

  with polynomial $u(t),v(t)$

- Closed-form arc length (distance-based control without numerical integration):

  $$
  L(t)=\int_0^t \bigl(u(\tau)^2+v(\tau)^2\bigr)\,d\tau
  $$

- Curvature:

  $$
  \kappa(t)=\frac{2(\dot u\,v-u\,\dot v)}{(u^2+v^2)^2}
  $$

These ensure continuity of $\kappa$ and $\dot\kappa$, suppressing abrupt changes of steering angle $\delta$ and steering rate $\dot\delta$.

---

## 7. Experiments

### PH Segment Parameters

**Base form**

$$
u(t)=U,\qquad v(t)=c\,t(1-t)
$$

$$
\dot{\mathbf r}(t)=(x'(t),y'(t))=\bigl(u^2-v^2,\; 2uv \bigr)
$$

**End constraints**

$$
v(0)=v(1)=0 \quad \Rightarrow \quad y'(0)=y'(1)=0
$$

Thus the start/end headings are along the $x$-axis (steering angle $0^\circ$).

**Closed-form solution** for target arc length $L_{\mathrm{target}}$ and lateral offset $D$:

$$
s=\frac{1}{2}\left(L_{\mathrm{target}}+\sqrt{L_{\mathrm{target}}^{\,2}+\frac{12}{10}D^{2}}\right)
$$

$$
U=\sqrt{s},\qquad c=\frac{3D}{U}
$$

**Parameter table**

| Variable | Value | Unit | Description |
|---|---:|:---:|---|
| $D$ | $\pm 0.8$ | m | Lateral shift (left $+$ / right $-$) |
| $U$ | by formula | – | Constant term of $u(t)$; sets longitudinal length vs. lateral shift ratio |
| $c$ | $3D/U$ | – | Scale of $v(t)$; controls curvature magnitude |

Using the TDM–TG structure in various two-lane scenarios with different gap sizes and obstacle configurations, we test whether TDM decides lane changes and whether TG generates **smoothly connected waypoints** using PH curves. The following figures show trajectory visualizations for representative cases.

<table>
  <tr>
    <td align="center">
      <img src="../../../images/avo6.gif" width="460" alt="Case A-1"><br/>
      <sub>Case A-1</sub>
    </td>
    <td align="center">
      <img src="../../../images/avo7.gif" width="460" alt="Case A-2"><br/>
      <sub>Case A-2</sub>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="../../../images/avo8.gif" width="460" alt="Case B-1"><br/>
      <sub>Case B-1</sub>
    </td>
    <td align="center">
      <img src="../../../images/avo9.gif" width="460" alt="Case B-2"><br/>
      <sub>Case B-2</sub>
    </td>
  </tr>
</table>

---

## 8. Summary

By feeding **YOLO+LiDAR** perception outputs into TDM and passing the **cost-based decision** to a **PH-augmented TG**, we confirmed that the system generates **stable and smooth** evasive trajectories. The proposed architecture demonstrates strong **real-time performance, computational efficiency, and steering stability**, making it a practical evasive maneuver algorithm for road environments.
