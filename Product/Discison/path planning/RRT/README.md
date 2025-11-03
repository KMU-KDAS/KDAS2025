# RRT-Based Global Path Generation  
*(Rapidly-exploring Random Tree for Global Path Generation)*

---

## 1. Overview
This study pre-generates a **global driving path** for an autonomous vehicle operating in a **closed-track environment** using the **Rapidly-exploring Random Tree (RRT)** algorithm.

Since the track is closed and static, global path generation can be performed **offline (pre-generation)** before actual driving.

However, the standard RRT algorithm has two main limitations:

1. **Non-determinism** — Random sampling leads to different results each run, producing inconsistent paths.  
2. **No cost-awareness** — The path may not be smooth or optimal.

To address these, we:
- Divide the track into **Free Space Segments**,  
- Execute RRT within each segment independently, and  
- Apply **Elastic-Band Smoothing** for global continuity.

This ensures a continuous and reproducible global path suitable for closed-loop control.

---

## 2. Theoretical Background

### 2.1 Algorithm Concept

RRT explores the search space by iteratively expanding a tree toward random samples.

1. **Random Sampling**
   $$
   q_{\text{rand}} \sim \mathcal{U}(Q)
   $$
   Randomly sample a point \( q_{\text{rand}} \) in free space \( Q \).

2. **Nearest Node Search**
   $$
   q_{\text{near}} = \underset{q_i \in T}{\arg\min} \, \text{dist}(q_i, q_{\text{rand}})
   $$
   Find the nearest existing node in tree \( T \) using Euclidean or configuration-space distance.

3. **Tree Extension (Steer)**
   $$
   q_{\text{new}} = q_{\text{near}} + 
   \frac{(q_{\text{rand}} - q_{\text{near}})}{\|q_{\text{rand}} - q_{\text{near}}\|} \cdot \varepsilon
   $$
   Extend from \( q_{\text{near}} \) toward \( q_{\text{rand}} \) by a step size \( \varepsilon \).

4. **Collision Check**
   $$
   q_{\text{new}} \in C_{\text{free}}
   $$
   Verify that \( q_{\text{new}} \) lies within the free configuration space.

   **Line-segment collision test:**
   $$
   \text{CollisionFree}(q_{\text{near}}, q_{\text{new}}) =
   \begin{cases}
   \text{True}, & \text{if } (1-s)q_{\text{near}} + s q_{\text{new}} \notin O, \, \forall s \in [0,1] \\
   \text{False}, & \text{otherwise}
   \end{cases}
   $$
   where \( O \) is the obstacle region.

5. **Tree Update**
   $$
   T = T \cup \{(q_{\text{near}}, q_{\text{new}})\}
   $$
   If collision-free, add the new node and edge to the tree.

---

## 3. Defining Free Space and Problem Recognition

### 3.1 Track Characteristics
- Closed-loop track with **only an outer boundary** and no internal obstacles.  
- Visual markers (lights, signs) exist but do not affect physical occupancy.  
- Occupancy grid representation is insufficient for defining drivable regions.

### 3.2 Using SLAM-Derived Centerline
The **Cartographer** SLAM-generated centerline was used as the base.  
From this, **left and right lane boundaries** were created by offsetting the centerline by the lane width.  
The resulting polygon defines each segment’s **Free Space** corridor.

### 3.3 Problem: Global Free Space Overlap
Generating one large free-space polygon for the entire track caused **overlaps** at intersections and bi-directional sections.

<p align="center"><img src="../../../../images/rrt1.png" width="700"/></p>

---

## 4. Segment-wise Free Space Division and RRT Execution

### 4.1 Pseudo-code for Free Space Modeling
```python
# INPUT: waypoints_*.json (sorted)
# PARAM: ds=0.05, lane_width=0.4
# OUTPUT: segments_free_space, centerline, left/right boundaries

segments = [load_json_xy(f) for f in files]
center_raw = stitch(segments)  # merge & remove overlaps
centerline = resample_uniform(center_raw, step=ds)

tangent = normalize(gradient(centerline))
normal  = rotate90(tangent)
left_boundary  = centerline + 0.5 * lane_width * normal
right_boundary = centerline - 0.5 * lane_width * normal

breaks = []
for i in range(1, len(centerline)-1):
    kappa = curvature(centerline, i)
    dpsi  = heading(centerline, i) - heading(centerline, i-1)
    if (kappa > KAPPA_TH) or (abs(dpsi) > HEADING_TH) or \
       (arc(i) - arc(last_break) > L_MAX):
        breaks.append(i)

segments_idx = pairwise([0] + breaks + [len(centerline)-1])

segments_free_space = []
for (s, e) in segments_idx:
    L = left_boundary[s:e+1]
    R = right_boundary[s:e+1]
    corridor = polygon(np.vstack([L, R[::-1]]))
    segments_free_space.append(corridor)
```

### 4.2 Segment Examples
<table><tr>
<td><img src="../../../../images/rrt2.png" width="320"/></td>
<td><img src="../../../../images/rrt3.png" width="320"/></td>
<td><img src="../../../../images/rrt3.png" width="320"/></td>
</tr></table>

---

## 5. RRT Implementation and Analysis

### 5.1 Full Algorithm (Pseudo-code)
```text
Algorithm RRT(Q, q_start, q_goal):
    T.init(q_start)
    for i = 1 to N_max:
        q_rand ← Sample(Q_free)
        q_near ← Nearest(T, q_rand)
        q_new  ← Steer(q_near, q_rand, ε)
        if CollisionFree(q_near, q_new):
            T.add_vertex(q_new)
            T.add_edge(q_near, q_new)
            if ||q_new - q_goal|| < r_goal:
                return ExtractPath(T, q_new)
    return Failure
```

### 5.2 Visual Results
<table><tr>
<td><img src="../../../../images/rrt4.png" width="320"/></td>
<td><img src="../../../../images/rrt5.png" width="320"/></td>
<td><img src="../../../../images/rrt6.png" width="320"/></td>
</tr></table>

Green = drivable corridor,  
Blue = RRT tree,  
Red = final connected path.

### 5.3 Limitations
<table><tr>
<td><img src="../../../../images/rrt7.png" width="360"/></td>
<td><img src="../../../../images/rrt8.png" width="360"/></td>
</tr></table>

- Path is **feasible** but **non-smooth** and **non-optimal**.  
- Randomness causes slight path variations per run.  
- Requires **post-smoothing** for vehicle-grade driving.

---

## 6. Post-Processing and Final Global Path

### 6.1 Elastic-Band Smoothing (Pseudo-code)
```text
function ElasticBandSmoothing(P, corridor, centerline):
    repeat N iterations:
        for each internal point p_i in P:
            f_len    = p_{i-1} + p_{i+1} - 2p_i
            f_curv   = (p_{i-1} - 2p_i + p_{i+1})
            f_clear  = direction_to_interior(p_i, corridor)
            f_center = nearest(centerline, p_i) - p_i
            p_i ← p_i + step*(w_len*f_len + w_curv*f_curv + w_clear*f_clear + w_center*f_center)
            if p_i ∉ corridor:
                p_i ← project_inside(p_i, corridor)
    return P
```

### 6.2 Before/After Comparison
<table><tr>
<td><img src="../../../../images/rrt9.png" width="360"/></td>
<td><img src="../../../../images/rrt10.png" width="360"/></td>
</tr></table>

- Left: Raw RRT path (discontinuous)  
- Right: Smoothed path (continuous and drivable)

### 6.3 Global Path Visualization
<p align="center"><img src="../../../../images/rrt11.png" width="720"/></p>

The final path connects all local RRT outputs via spline interpolation, maintaining curvature continuity and lane clearance.

---

## 7. Conclusion

This work constructs a **global drivable path** by integrating:
- Segmented free-space modeling  
- Per-segment RRT exploration  
- Elastic-band post-smoothing  

The result is a **stable, continuous, and realistic** driving path for closed tracks.  
This path also serves as input to the **DQN-based path ordering** module, enabling a combined **sampling + learning** pipeline.

<p align="center"><img src="../../../../images/rrt12.png" width="720"/></p>
