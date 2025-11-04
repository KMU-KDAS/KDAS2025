# Autonomous Evasive Maneuver (AEM)

Autonomous vehicles frequently encounter sudden obstacles ahead or to the side during driving.  
In such cases, the vehicle must perform an **evasive maneuver** stably without collision, and this requires an algorithm that can generate **safe and efficient trajectories in real time** as the environment changes.

To jointly achieve **efficiency** and **stability**, we design the evasive maneuver algorithm as a **hybrid architecture** consisting of two parts:

- **Tactical Decision Making (TDM)** — high-level behavior selection (keep lane, change lane, decelerate, etc.)
- **Trajectory Generator (TG)** — dynamically feasible trajectory synthesis for the chosen behavior

In particular, to overcome the limitations of classical polynomial trajectory generators, the TG stage adopts the **Pythagorean Hodograph (PH)** technique.  
PH curves have **$C^2$ continuity** (continuous curvature and curvature-rate), which suppresses sudden changes in steering angle and steering rate.  
Thus, by introducing PH into the hybrid architecture, we preserve the computational efficiency of polynomial methods **while substantially improving steering stability and ride comfort**.

---

## 1. Tactical Decision Making (TDM)

TDM is responsible for **high-level behavior decisions**.  
Among the candidate actions such as lane keeping, lane changing, and deceleration, it selects the **safest and most efficient** action for the current traffic context.

### (1) Decision Topology

The **reachability** between gaps is defined as

\[
\mathrm{Reach}_{i,j} \;=\;
\begin{cases}
\mathbf{T}, & \bigl|\,\mathrm{Gap}_i.s - \mathrm{Gap}_j.s\,\bigr| < \dfrac{\mathrm{Gap}_i.l + \mathrm{Gap}_j.l}{2} \\
\mathbf{F}, & \text{otherwise}
\end{cases}
\tag{1}
\]

That is, if the center distance between two gaps is less than half the sum of their lengths (i.e., the spatio-temporal regions overlap or are adjacent), the transition is **reachable**.

We explore only the gaps that are reachable from the **root node containing the ego vehicle**, which maximizes the efficiency of cost evaluation.  
Let $N_R$ be the number of reachable state transitions; then the search complexity is reduced to **$O(N_R^2)$**, making the topology suitable for real-time decision making.

![Figure 1: Frenet coordinate system of the algorithm](./-avo1.png)

### (2) Cost Function

The **total cost** for gap selection is

\[
J \;=\; C_{\text{road}} + C_{\text{traffic}} + C_{\text{arrival}}.
\tag{2}
\]

**① Road Cost**

\[
C_{\text{road}}^{(i)} \;=\; 1 - \frac{\mathrm{Gap}_i.l}{100}.
\tag{3}
\]

Shorter gaps imply smaller safe regions and therefore receive higher cost (i.e., \( C_{\text{road}} \propto 1/L_{\text{gap}} \)).

**② Traffic Cost**

\[
C_{\text{traffic}}^{(i)} \;=\; C_{\text{traffic,1}}^{(i)} + C_{\text{traffic,2}}^{(i)}.
\tag{4}
\]

\[
C_{\text{traffic,1}}^{(i)} \;=\;
\begin{cases}
0, & n_{\text{car}}=0,\\
-\,w_{\text{car}}\,\dfrac{\mathrm{Gap}_{\text{car}}}{n_{\text{car}}}, & \text{otherwise},
\end{cases}
\tag{5}
\]

\[
C_{\text{traffic,2}}^{(i)} \;=\;
\begin{cases}
c_f \,\dfrac{v_{\text{ego}}}{v_{\text{target}}}, & s_{\text{target}} > \mathrm{Gap}_i.l,\\[6pt]
-\,c_b \,\dfrac{v_{\text{target}}}{v_{\text{ego}}}, & s_{\text{target}} < \mathrm{Gap}_i.l.
\end{cases}
\tag{6}
\]

- \(C_{\text{traffic,1}}\): increases risk as the **number of cars** grows.
- \(C_{\text{traffic,2}}\): reflects **dynamic interaction** via speed ratio.  
  For example, if \( v_{\text{ego}} > v_{\text{target}} \), rear-end risk increases and the cost rises.

Overall, \( C_{\text{traffic}} \) **quantifies the trade-off** between traffic flow and safety.

**③ Arrival Cost**

\[
C_{\text{A}\to\text{B}}^{(i)} \;=\; C_{\text{long}}^{(i)} + C_{\text{lat}}^{(i)}.
\tag{7}
\]

\[
C_{\text{long}}^{(i)} \;=\; 1 - \frac{\min(A_f,\,B_f) - \max(A_r,\,B_r)}{\mathrm{Gap}_A.l},
\tag{8}
\]

\[
C_{\text{lat}}^{(i)} \;=\;
\begin{cases}
\dfrac{\text{LaneWidth}}{10}, & \Delta \mathrm{Idx}_{\text{lane}} \le 1,\\
2.5, & \Delta \mathrm{Idx}_{\text{lane}} > 1.
\end{cases}
\tag{9}
\]

- \(C_{\text{long}}\): evaluates longitudinal **overlap ratio** between gaps (difficulty of moving from A to B).  
- \(C_{\text{lat}}\): penalizes **number of lane changes**.

![Figure 2: Four scenes for computing the arrival cost from A to B](./-avo3.png)

---

### 2D LiDAR Processing Pipeline (with Vision Fusion)

We employ a **YOLO + Depth + 2D LiDAR** fusion pipeline to provide TDM with robust **distance, velocity, and TTC** estimates for surrounding vehicles in real time.

#### 1) Candidate Generation via Vision (YOLO + Depth)

**(a) YOLO detection.**  
Detect 2D bounding boxes of vehicles/pedestrians/obstacles using YOLOv8 on RGB images.  
Publish to `/yolo_objects` with persistent IDs.

**(b) Depth-based ranging.**  
From the lower ROI of each box, compute a robust depth \( z \) by mean/median with IQR filtering.  
Back-project to 3D with intrinsic parameters \((f_x,f_y,c_x,c_y)\) to obtain \((X, Y, Z)\).  
Publish as `vision_msgs/Detection3DArray` on `/objects_3d`.

Implementation notes:
- Sample multiple depth points at the lower-center ROI of the box.  
- Remove outliers via IQR, then take the **median**; smooth temporally with **EMA**.  
- Output per object: \((x,y,z)\) and class ID.

This stage produces **vision candidates**; the LiDAR node refines distance and velocity.

#### 2) LiDAR-based Refinement (LiDAR–TDM Bridge)

**(a) LiDAR Gating.**  
Convert each vision candidate position \((x,y)\) to polar \((r_d,\theta_d)\).  
From `/scan`, gate beams within \(\theta_{\text{gate}}=\pm5^\circ\) and \(r_{\text{gate}}=\pm0.5\text{ m}\) to obtain a local set of ranges, and compute the **median** \(r_{\text{med}}\).  
Use \(r_{\text{med}}\) to correct the vision range \(r_d\).

**(b) Range Consistency Check.**

\[
\bigl|r_{\text{med}} - r_d\bigr| \;\le\; \varepsilon(r) \;=\; \varepsilon_0 + k_r \, r_d,
\]

where:

| Symbol | Meaning |
|---|---|
| \(r_d\) | Depth-based range (from camera back-projection) |
| \(r_{\text{med}}\) | Median LiDAR range inside the gate |
| \(\varepsilon(r)\) | Distance-proportional tolerance |
| \(\varepsilon_0\) | Base tolerance (near-range minimum) |
| \(k_r r_d\) | Linear increase with distance |

We set \(\varepsilon_0 = 0.3\,\text{m}\), \(k_r = 0.05\).  
Exceeding the tolerance marks the candidate as a **failed correction**.

**(c) Lane Assignment.**  
Assign lane based on lateral position \(y\):

\[
\mathrm{lane\_id} \;=\;
\begin{cases}
0, & |y| < \dfrac{W_{\text{lane}}}{2},\\
-1, & \bigl|y - W_{\text{lane}}\bigr| < \dfrac{W_{\text{lane}}}{2},\\
+1, & \bigl|y + W_{\text{lane}}\bigr| < \dfrac{W_{\text{lane}}}{2}.
\end{cases}
\]

(QCar track: \(W_{\text{lane}} = 0.8\,\text{m}\).)  
The assigned `lane_id` is used for gap and cost computations in TDM.

#### 3) Temporal Differencing for Velocity & TTC

**(a) Relative speed.**  
With frame period \(\Delta t \approx 0.1\text{ s}\) (LiDAR publishes at 10 Hz via `rclpy.Timer(0.1)`), for the same tracked ID:

\[
v_{\text{rel}} \;=\; \frac{x_{\text{prev}} - x_{\text{now}}}{\Delta t}.
\]

- Approaching: \(v_{\text{rel}} > 0\)  
- Receding / stationary: \(v_{\text{rel}} \le 0\)

This sign convention matches TDM cost definitions.

**(b) Time-to-Collision (TTC).**  
For range \(d=\sqrt{x^2+y^2}\) and relative speed \(v_{\text{rel}}\):

\[
\mathrm{TTC} \;=\;
\begin{cases}
\dfrac{d}{v_{\text{rel}}}, & v_{\text{rel}} > 0,\\[6pt]
\infty, & v_{\text{rel}} \le 0.
\end{cases}
\]

TTC directly feeds into \(C_{\text{traffic}}\) of TDM as a quantitative collision-risk signal.

---

### 4) Gap Estimation and MIO (Most Important Object)

#### (1) Lane-wise Gap Estimation

We compute longitudinal **gaps** within each lane to support TDM’s keep-vs-change decisions.

- **Front gap (current lane).**  
Among vehicles with \(X>0\) in the current lane \((\mathrm{lane\_id}=0)\), the nearest longitudinal distance defines the **front gap**:
\[
\mathrm{Gap}_{\text{front}} = \min \{\, d \;:\; \text{lane}(d)=0,\; x_d>0 \,\}.
\]
This represents the ego’s forward safety margin and is used in \(C_{\text{traffic}}\) and \(C_{\text{road}}\).

- **Lateral lane-change gap (adjacent lane).**  
For \(\mathrm{lane\_id}=\pm1\), compute the nearest **front** and **back** clearances \(D_F, D_B\), then subtract the vehicle length & buffer \(L_{\text{safe}}\) to obtain the usable entry gap:
\[
\mathrm{Gap}_{\ell} =
\begin{cases}
0, & \text{if } D_F \text{ or } D_B \text{ missing for this frame, or for } N \text{ consecutive frames},\\
\max\!\bigl(0,\, D_F + D_B - L_{\text{safe}}\bigr), & \text{otherwise},
\end{cases}
\quad \ell\in\{-1,+1\}.
\]
A larger \(\mathrm{Gap}_{\ell}\) indicates safer/easier lane entry and contributes to \(C_{\text{road}}\) and \(C_{\text{arrival}}\).

**Publishing.**  
We publish the gap triplet
\[
[\ \mathrm{Gap}_{\text{front}},\ \mathrm{Gap}_{\text{left}},\ \mathrm{Gap}_{\text{right}}\ ]
\]
as `Float32MultiArray` on `/tdm/gaps` for real-time TDM evaluation.

#### (2) MIO Selection

Within the forward field-of-view of \(\pm 30^\circ\), select the **closest object** as the **MIO**.  
Publish \(v_{\text{rel}}\) and TTC of the MIO on `/tdm/mio_vrel` and `/tdm/mio_ttc`.

---

## 2. Trajectory Generator (TG)

TG converts the behavior chosen by TDM into a **kinematically and dynamically feasible** trajectory.  
We model **longitudinal** and **lateral** motions with independent polynomials and score candidates by a cost function.

### (1) Longitudinal Direction

\[
s(t) \;=\; a_0 + a_1 t + a_2 t^2 + a_3 t^3 + a_4 t^4.
\tag{10}
\]

Initial conditions are given by the ego’s current position/velocity/acceleration; terminal conditions use the MIO’s speed and a safety distance.  
Coefficients satisfying boundary conditions are obtained by the linear system

\[
\begin{bmatrix}
\dot{s}(T) - \dot{s}(0) - \ddot{s}(0)T \\
\ddot{s}(T) - \ddot{s}(0)
\end{bmatrix}
=
\begin{bmatrix}
3T^2 & 4T^3\\
6T & 12T^2
\end{bmatrix}
\begin{bmatrix}
a_3\\
a_4
\end{bmatrix}.
\tag{14}
\]

We sample multiple \(T\) values and evaluate each candidate for safety and comfort.

### (2) Lateral Direction

\[
d(t) \;=\; b_0 + b_1 t + b_2 t^2 + b_3 t^3 + b_4 t^4 + b_5 t^5.
\tag{15}
\]

The terminal point is the **lane center** or **avoidance target**.  
Smoothness is evaluated via acceleration/jerk limits.  
Coefficients satisfy

\[
\mathbf{M}\,[b_3,\ b_4,\ b_5]^\top = \mathbf{R}(d_0, d_T, \dot d_0, \dot d_T),
\]

where \(\mathbf{M}\) is determined by the time horizon \(T\).

### (3) TG Cost Function

\[
J^{(i)} \;=\; w_v C_v + w_a C_a + w_j C_j + w_k C_k + w_{\text{ttc}} C_{\text{ttc}},
\tag{19}
\]

\[
C_j \;=\; \frac{1}{T}\int \left( \frac{j_{\text{long}}(t)^2}{j_{\text{max,long}}^2}
+ \frac{j_{\text{lat}}(t)^2}{j_{\text{max,lat}}^2} \right) dt,
\tag{22}
\]

where \(C_j\) is the squared **jerk** sum (longitudinal + lateral), serving as a quantitative smoothness metric.

---

## 3. PH (Pythagorean Hodograph) Extension

Polynomial TG is efficient but can exhibit **discontinuous curvature and curvature-rate**, which leads to steering oscillation or overshoot.  
We therefore extend the lateral trajectory using **PH curves**.

### (1) Definition and Properties

A PH curve is defined by

\[
\mathbf{r}(t) \;=\; \int_0^t \bigl(u(\tau)^2 - v(\tau)^2,\;\; 2u(\tau)v(\tau)\bigr)\, d\tau,
\tag{PH-1}
\]

where \(u(t)\) and \(v(t)\) are polynomials. The speed magnitude is a **perfect square**:

\[
\|\dot{\mathbf{r}}(t)\| \;=\; u(t)^2 + v(t)^2.
\]

Hence the **arc length** admits a **closed form** (no numeric integration):

\[
L(t) \;=\; \int_0^t \bigl(u(\tau)^2 + v(\tau)^2\bigr)\, d\tau.
\tag{PH-2}
\]

The curvature is also polynomial:

\[
\kappa(t) \;=\; \frac{2\bigl(\dot u v - u \dot v\bigr)}{\bigl(u^2 + v^2\bigr)^2},
\]

so **curvature and curvature-rate are continuous** (\(C^2\) continuity).

### (2) Application Steps and TG Coupling

1) **Set avoidance target** from lateral error.  
Choose \(d^\*\) such that the lateral clearance exceeds \(\Delta d_{\text{safe}}\):
\[
d^\* \;=\; d_{\text{ego}} \pm \Delta d_{\text{safe}}.
\]

2) **Construct PH guide** and integrate with TG.  
Design \(\mathbf{r}(t)\) to meet boundary conditions \((x_0,y_0,\psi_0) \to (x^\*,y^\*,\psi^\*)\) and add to TG’s cost.  
PH trajectories naturally produce **lower \(C_j\) and \(C_k\)** (jerk & curvature terms), thus minimizing steering effort.

**Benefits:**  
- Continuity of steering angle and curvature-rate  
- Improved **steering stability**

![PH integration concept](./-avo4.png)

---

## 4. Implementation

This section describes applying the TDM–TG architecture to a **two-lane** road.  
The system comprises **Perception → TDM → TG**, running in real time to decide and synthesize evasive paths.

![System diagram](./-avo5.png)

### (1) Perception & Data Fusion

We designed a custom vision–LiDAR fusion to produce **stable real-time range**.

- Detect objects and classes with **YOLOv8**.  
  For each box, compute initial depth \(d_{\text{depth}}\) using mean/median with IQR at the bottom/center ROI.
- From LiDAR scans, select beams near the object bearing \(\theta\) to compute \(d_{\text{LiDAR}}\).
- Combine strengths by a **distance-dependent weight** \(\alpha(d)\):
  \[
  d_{\text{fused}} \;=\; \alpha(d)\, d_{\text{depth}} + \bigl(1-\alpha(d)\bigr)\, d_{\text{LiDAR}}, \quad \alpha(d)\in[0,1].
  \]
  Give more weight to **camera** at near range (< 3 m) and to **LiDAR** at far range.
- Apply **EMA** for temporal smoothing and hold previous values across brief dropouts.
- Classify each object’s lane (current vs. adjacent) using a lane mask and keep only **valid TDM inputs**.

### (2) TDM Implementation

We implement a **cost-based decision** that considers road geometry, traffic density, and arrival ease—not just distances.

- The action set is { **Keep Lane**, **Change Lane** }.
- For each lane, compute:
  - \(C_{\text{road}}\): increases as available **gap** \(L_{\text{gap}}\) shrinks.  
  - \(C_{\text{traffic}}\): proportional to **number of vehicles** \(N_{\text{veh}}\) and **relative speed** \(v_{\text{rel}}\).  
  - \(C_{\text{arrival}}\): penalizes **path length** and **number of lane changes** to reach the goal.
- Total:
  \[
  J \;=\; C_{\text{road}} + C_{\text{traffic}} + C_{\text{arrival}}.
  \]
- Decision rule:
  \[
  \text{Action} \;=\;
  \begin{cases}
  \text{Change Lane}, & \text{if } J_{\text{adjacent}} < J_{\text{current}} - \Delta J_{\text{th}},\\
  \text{Keep Lane}, & \text{otherwise}.
  \end{cases}
  \]

### (3) TG Implementation

Based on TDM’s decision, TG generates the actual path.

For a two-lane road, the lateral terminal condition is

\[
y(T) \;=\;
\begin{cases}
y_{\text{center,current}}, & \text{Keep Lane},\\
y_{\text{center,current}} \pm W_{\text{lane}}, & \text{Change Lane}.
\end{cases}
\]

We replace pure polynomials with **PH curves** to avoid discontinuities in steering angle:

\[
\mathbf{r}(t)=\int_0^t \bigl(u(\tau)^2 - v(\tau)^2,\; 2u(\tau)v(\tau)\bigr)d\tau,\qquad
L(t)=\int_0^t(u^2+v^2)\,d\tau,\qquad
\kappa(t)=\frac{2(\dot u v - u\dot v)}{(u^2+v^2)^2}.
\]

This ensures continuity in curvature and curvature-rate, thus suppressing spikes in steering angle \(\delta\) and steering rate \(\dot\delta\).

---

## 5. Experiments

### PH Segment Parameters

**Base form**
\[
u(t)=U,\qquad v(t)=c\,t(1-t), \qquad
\dot{\mathbf{r}}(t)=\bigl(x'(t),y'(t)\bigr)=\bigl(u^2-v^2,\; 2uv\bigr).
\]

**End constraints**
- \(v(0)=v(1)=0 \Rightarrow y'(0)=y'(1)=0\)  
  ⇒ heading aligned with the \(x\)-axis at both start and end (steering angle \(0^\circ\)).

**Closed-form solution** for total length \(L_{\text{target}}\) and lateral offset \(D\):

\[
s=\tfrac{1}{2}\Bigl(L_{\text{target}} + \sqrt{L_{\text{target}}^2 + \tfrac{4}{10}\cdot 3D^2}\Bigr),\quad
U=\sqrt{s},\quad
c=\frac{3D}{U}.
\]

| Variable | Value | Unit | Description |
|---|---:|:---:|---|
| \(D\) | \(\pm 0.8\) | m | Lateral shift (left + / right −) |
| \(U\) | via formula | – | Constant term of \(u(t)\); sets longitudinal length vs. lateral share |
| \(c\) | \(3D/U\) | – | Scale of \(v(t)\); controls curvature magnitude |

Using the TDM–TG framework, we tested various vehicle spacings and obstacle layouts in a two-lane environment.  
TDM decided lane changes based on gap size and traffic; TG generated **smooth PH-based waypoints** accordingly.

**Trajectory visualizations (per case):**

<table>
<tr>
<td align="center"><img src="./-avo6.gif" width="360"/></td>
<td align="center"><img src="./-avo7.gif" width="360"/></td>
</tr>
<tr>
<td align="center"><img src="./-avo8.gif" width="360"/></td>
<td align="center"><img src="./-avo9.gif" width="360"/></td>
</tr>
</table>

---

## 6. Summary

We integrated **YOLO + LiDAR** perception as inputs to **TDM**, and fed TDM’s **cost-based decisions** to a **PH-augmented TG**, producing **stable and smooth** evasive trajectories.  
The proposed structure shows excellent **real-time performance, computational efficiency, and steering stability**, making it a strong candidate for road-optimized autonomous evasive maneuvers.

---

### Notes on Equation Figures

Where equations were shown as images in the source, we have rendered them as LaTeX above.  
If you prefer to keep the screenshots for reference, place them at the indicated paths (e.g., `-avo2.png`, etc.), and we can add explicit `![eq](...)` links next to the corresponding formulas.
