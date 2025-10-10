# **SCNN-Based Lane Detection for 1/10-Scale Autonomous Driving**

---

## 1. Overview

This project implements a lane detection system using **SCNN (Spatial CNN)** for both a 1/10-scale autonomous platform and the QLabs simulation environment.  
Instead of reusing off-the-shelf code, we rebuilt the entire pipeline — **dataset construction, training pipeline, post-processing, depth projection, and waypoint generation** — so that the output can be fed directly into a driving controller.

<img src="../../../images/SCNN1.gif" alt="SCNN Overview" width="800"/>

**Motivation.** Classical OpenCV-based lane detection struggled under illumination changes, sharp curvature, and lane discontinuities. SCNN was chosen because its **row/column message passing** preserves line continuity and handles curves more robustly.

---

## 2. Why SCNN (vs. classical vision)

OpenCV (thresholding/edge + Hough) is sensitive to noise and texture changes, making it unreliable for real driving.  
SCNN’s spatial message passing aggregates features along rows/columns, improving continuity on curved lanes and broken markings.

<img src="../../../images/SCNN2.png" alt="SCNN vs Classical Vision" width="800"/>

---

## 3. Transfer Learning for Domain Adaptation

Applying CULane-pretrained SCNN “as-is” to a 1/10-scale domain produced limited performance due to scale and texture shift.  
We therefore adopted **transfer learning**:

- Keep reusable **low-level features** (edges, textures).
- **Re-train higher layers** on our dataset.
- Compare three strategies: keep VGG16 backbone only, unfreeze progressively, and full fine-tuning.  
  The best compromise: reuse early VGG16 layers, **fine-tune upper layers** on our domain-specific data.

<img src="../../../images/SCNN3.png" alt="Transfer Learning Strategy" width="800"/>

---

## 4. Feature Visualization and Validation

During training we visualized intermediate **feature maps** to see how layers responded.  
After fine-tuning with our dataset, **later layers** reacted strongly to lane edges and curvature while **early layers** remained similar — confirming our transfer strategy.

<img src="../../../images/SCNN4.png" alt="Feature Maps Before/After Fine-tuning" width="800"/>

---

## 5. Output Representation (Binary Mask → Centerline → Waypoints)

The network outputs a **1-channel binary mask** for lane pixels.  
Binary masks are label-efficient, work naturally with SCNN’s segmentation head, and make **post-processing** straightforward:

1. From the mask, extract **left/right lanes** and compute a **centerline**.  
2. Convert centerline pixels to metric coordinates via **Depth projection**.  
3. Generate **(x, y) waypoint** sequences for the controller (e.g., Pure Pursuit/Stanley).

<img src="../../../images/SCNN5.png" alt="Binary Mask to Centerline and Waypoints" width="800"/>

---

## 6. Training Pipeline (Rebuilt)

We reimplemented the entire training stack for our dataset (images + binary masks).

**Network**
- **Input:** 3×H×W RGB  
- **Backbone:** VGG16_BN  
- **SCNN** message passing layers  
- **Head:** Segmentation head → **1-channel logit** (binary mask)

**Loss / Optimizer**
- **BCEWithLogitsLoss** (handles class imbalance; stable training)  
- **AdamW** (`lr=1e-3`, `weight_decay=1e-2`) to reduce overfitting

**Data / Splits**
- Train/Val/Test = **7 : 1.5 : 1.5**  
- Preprocessing: resize to **480×640**, ToTensor, (optional) normalization/augmentation

**Metrics**
- **mIoU** on validation/test sets

<img src="../../../images/SCNN6.png" alt="Training Pipeline" width="800"/>

---

## 7. Training Rules and Forward/Backward Flow

**Forward**
- Resize → Model forward → 1-channel logit

**Loss**
- `BCEWithLogitsLoss` (logits in, internal sigmoid)

**Backward**
- Compute gradients by chain rule  
- **Weight Decay** via AdamW

**Update**
- Optimizer step; learning-rate schedule optional

<img src="../../../images/SCNN7.png" alt="Forward/Backward and Optimization" width="800"/>

---

## 8. Post-processing

To stabilize detection and produce control-ready centerlines:

1. **Morphology** (Dilation/Closing) – remove noise and mend small gaps  
2. **Row-wise Midpoint Sampling** – collect lane candidates per scanline (with light outlier rejection)  
3. **DBSCAN** clustering – merge fragments and split **left/right** lanes  
4. **Cubic polynomial fit** for each side – model the lane trajectories  
5. **Centerline** – evaluate both polynomials on a common y-grid and average x  
6. **Spline interpolation & smoothing** – produce a smooth centerline for control

<img src="../../../images/SCNN8.png" alt="Post-processing Steps" width="800"/>

> Result: The centerline and waypoints are smooth along curves and stable enough for direct use by the controller.

---

## 9. Depth Projection to Vehicle Coordinates

We project the (center/centerline) pixels to **3D camera coordinates** using depth and intrinsics \((f_x, f_y, c_x, c_y)\), then map to the **vehicle frame** using calibrated extrinsics \((R, t)\). Finally, we **sample waypoints** at fixed spacing for robust tracking.

Note: The algorithm was implemented and validated conceptually; however, QLabs’ depth module had official constraints that prevented full simulator deployment. The mapping works on real depth sensors (e.g., **RealSense D435i**).

<img src="../../../images/SCNN9.png" alt="Depth to Vehicle Coordinates" width="800"/>

---

## 10. Evaluation and Results

**Quantitative**
- Validation **IoU ≈ 0.84–0.86** after stable convergence (train/val loss decreasing).

**Qualitative**
- Robust detection on tight curves; centerline remains continuous after post-processing.  
- On real and simulated tracks, **centerline-based steering** achieved **no lane departures**.

<img src="../../../images/SCNN10.gif" alt="Quantitative and Qualitative Results" width="800"/>

**Controller Integration**
- The SCNN centerline was fed to a PID/geometry-based controller (see **XYTRON PID** section) and yielded smooth lane-center driving.

---

## 11. Conclusion and Future Work

We built a complete SCNN lane detection stack tailored to the 1/10-scale domain — from dataset and training pipeline to post-processing and waypoint generation — and demonstrated **stable lane-center driving** in both simulation and real environments.

Future directions:
- Fuse with **LiDAR/IMU** for multi-sensor robustness  
- Integrate with **MPC** for constraint-aware planning and control  
- Expand datasets and fine-tune higher layers further for extreme edge cases

<img src="../../../images/SCNN11.gif" alt="Conclusion and Future Directions" width="800"/>

