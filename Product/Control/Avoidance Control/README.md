# Autonomous Evasive Maneuver (AEM)

Autonomous vehicles frequently encounter sudden obstacles ahead or to the side during driving.  
In such cases, the vehicle must perform an **evasive maneuver** stably without collision, requiring an algorithm that can generate **safe and efficient trajectories in real time** as the environment changes.

To jointly achieve **efficiency** and **stability**, we design the evasive maneuver algorithm as a **hybrid architecture** consisting of two modules:

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

$$
\mathrm{Reach}_{i,j} =
\begin{cases}
\mathbf{T}, & \bigl|\,\mathrm{Gap}_i.s - \mathrm{Gap}_j.s\,\bigr| < \dfrac{\mathrm{Gap}_i.l + \mathrm{Gap}_j.l}{2} \\
\mathbf{F}, & \text{otherwise.}
\end{cases}
$$

If the center distance between two gaps is less than half the sum of their lengths (i.e., the spatio-temporal regions overlap or are adjacent), the transition is **reachable**.

We explore only the gaps reachable from the **root node containing the ego vehicle**, maximizing efficiency.  
Let $N_R$ be the number of reachable state transitions; then the complexity is reduced to **$O(N_R^2)$**, enabling real-time operation.

![Frenet coordinate system](../../../images/avo1.png)

---

### (2) Cost Function

The **total cost** for gap selection is

$$
J = C_{\text{road}} + C_{\text{traffic}} + C_{\text{arrival}}.
$$

#### (a) Road Cost

$$
C_{\text{road}}^{(i)} = 1 - \frac{\mathrm{Gap}_i.l}{100}.
$$

Shorter gaps imply smaller safe regions and thus higher cost ($C_{\text{road}} \propto 1/L_{\text{gap}}$).

#### (b) Traffic Cost

$$
C_{\text{traffic}}^{(i)} = C_{\text{traffic,1}}^{(i)} + C_{\text{traffic,2}}^{(i)}.
$$

$$
C_{\text{traffic,1}}^{(i)} =
\begin{cases}
0, & n_{\text{car}}=0,\\
-\,w_{\text{car}}\dfrac{\mathrm{Gap}_{\text{car}}}{n_{\text{car}}}, & \text{otherwise.}
\end{cases}
$$

$$
C_{\text{traffic,2}}^{(i)} =
\begin{cases}
c_f\dfrac{v_{\text{ego}}}{v_{\text{target}}}, & s_{\text{target}} > \mathrm{Gap}_i.l,\\
-\,c_b\dfrac{v_{\text{target}}}{v_{\text{ego}}}, & s_{\text{target}} < \mathrm{Gap}_i.l.
\end{cases}
$$

- $C_{\text{traffic,1}}$: increases as the **number of cars** grows.  
- $C_{\text{traffic,2}}$: reflects **dynamic speed interaction**; if $v_{\text{ego}} > v_{\text{target}}$, rear-end risk increases.

#### (c) Arrival Cost

$$
C_{\text{A}\to\text{B}}^{(i)} = C_{\text{long}}^{(i)} + C_{\text{lat}}^{(i)}.
$$

$$
C_{\text{long}}^{(i)} = 1 - \frac{\min(A_f, B_f) - \max(A_r, B_r)}{\mathrm{Gap}_A.l}.
$$

$$
C_{\text{lat}}^{(i)} =
\begin{cases}
\dfrac{\text{LaneWidth}}{10}, & \Delta \mathrm{Idx}_{\text{lane}} \le 1,\\
2.5, & \Delta \mathrm{Idx}_{\text{lane}} > 1.
\end{cases}
$$

- $C_{\text{long}}$: measures overlap ratio (difficulty to move A→B).  
- $C_{\text{lat}}$: penalizes excessive lane changes.

![Arrival cost evaluation](../../../images/avo3.png)

---

## 2. 2D LiDAR Processing Pipeline (with Vision Fusion)

We employ **YOLO + Depth + 2D LiDAR** fusion to provide TDM with accurate **distance, velocity, and TTC** estimation.

### (1) Vision Candidate Generation (YOLO + Depth)

- Detect objects via **YOLOv8** and publish `/yolo_objects`.  
- Compute median depth $z$ (IQR filtering) and back-project $(f_x,f_y,c_x,c_y)$ → $(X,Y,Z)$.  
- Publish `/objects_3d` (`vision_msgs/Detection3DArray`).

### (2) LiDAR Gating and Range Check

Convert vision coordinates $(x,y)$ → polar $(r_d,\theta_d)$.  
Select LiDAR beams within $\theta_{\text{gate}}=\pm5^\circ$, $r_{\text{gate}}=\pm0.5\,\text{m}$, compute median $r_{\text{med}}$.

Range check condition:

$$
|r_{\text{med}} - r_d| \le \varepsilon(r) = \varepsilon_0 + k_r r_d.
$$

| Symbol | Meaning |
|:--|:--|
| $r_d$ | Depth-based range (camera back-projection) |
| $r_{\text{med}}$ | Median LiDAR range inside gate |
| $\varepsilon(r)$ | Distance-proportional tolerance |
| $\varepsilon_0$ | Base tolerance |
| $k_r r_d$ | Linear increase with range |

Default: $\varepsilon_0 = 0.3\,\text{m}$, $k_r = 0.05$.

### (3) Lane Assignment

$$
\mathrm{lane\_id} =
\begin{cases}
0, & |y| < \tfrac{W_{\text{lane}}}{2},\\
-1, & |y - W_{\text{lane}}| < \tfrac{W_{\text{lane}}}{2},\\
+1, & |y + W_{\text{lane}}| < \tfrac{W_{\text{lane}}}{2}.
\end{cases}
$$

(QCar track: $W_{\text{lane}}=0.8\,\text{m}$.)

---

## 3. Relative Velocity and TTC

For each tracked object:

$$
v_{\text{rel}} = \frac{x_{\text{prev}} - x_{\text{now}}}{\Delta t}, \quad \Delta t \approx 0.1\,\text{s}.
$$

$$
\mathrm{TTC} =
\begin{cases}
\dfrac{d}{v_{\text{rel}}}, & v_{\text{rel}} > 0,\\
\infty, & v_{\text{rel}} \le 0.
\end{cases}
$$

---

## 4. Trajectory Generator (TG)

TG converts TDM output into a dynamically feasible trajectory.

### (1) Longitudinal

$$
s(t) = a_0 + a_1t + a_2t^2 + a_3t^3 + a_4t^4.
$$

Boundary condition system:

$$
\begin{bmatrix}
\dot s(T)-\dot s(0)-\ddot s(0)T\\
\ddot s(T)-\ddot s(0)
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
$$

### (2) Lateral

$$
d(t) = b_0 + b_1t + b_2t^2 + b_3t^3 + b_4t^4 + b_5t^5.
$$

### (3) Cost

$$
J = w_vC_v + w_aC_a + w_jC_j + w_kC_k + w_{\text{ttc}}C_{\text{ttc}},
$$

$$
C_j = \frac{1}{T}\int\!\left(\frac{j_{\text{long}}^2}{j_{\text{max,long}}^2}+\frac{j_{\text{lat}}^2}{j_{\text{max,lat}}^2}\right)dt.
$$

---

## 5. PH (Pythagorean Hodograph) Extension

PH curves ensure continuous curvature:

$$
\mathbf{r}(t)=\!\int_0^t(u^2-v^2,2uv)d\tau,\quad
L(t)=\!\int_0^t(u^2+v^2)d\tau,\quad
\kappa(t)=\frac{2(\dot u v - u\dot v)}{(u^2+v^2)^2}.
$$

---

## 6. Implementation

Full stack: **Perception → TDM → TG**, all in real time.

![System diagram](../../../images/avo5.png)

---

## 7. Experiments

**Base PH segment**

$$
u(t)=U,\quad v(t)=c\,t(1-t),\quad
\dot r(t)=(u^2-v^2,2uv).
$$

$$
s=\tfrac{1}{2}\!\left(L_{\text{target}}+\sqrt{L_{\text{target}}^2+\tfrac{4}{10}\,3D^2}\right),\quad
U=\sqrt{s},\quad c=\frac{3D}{U}.
$$

| Var | Value | Unit | Desc |
|:--|--:|:--:|:--|
| $D$ | ±0.8 | m | Lateral shift |
| $U$ | via formula | – | Longitudinal scale |
| $c$ | $3D/U$ | – | Curvature scale |

<table>
<tr>
<td align="center"><img src="../../../images/avo6.gif" width="360"/></td>
<td align="center"><img src="../../../images/avo7.gif" width="360"/></td>
</tr>
<tr>
<td align="center"><img src="../../../images/avo8.gif" width="360"/></td>
<td align="center"><img src="../../../images/avo9.gif" width="360"/></td>
</tr>
</table>

---

## 8. Summary

By fusing **YOLO + LiDAR** with cost-based **TDM** and PH-based **TG**,  
the system achieves real-time evasive maneuvers with high steering stability and smoothness.
