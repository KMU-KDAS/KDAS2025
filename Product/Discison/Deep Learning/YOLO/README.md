# YOLOv8 for Traffic Signals & Road Signs (Autonomous RC / QCar & QLabs)

> Real-time object detection for **traffic lights (Red/Yellow/Green), STOP signs, roundabouts**, etc.  
> Chosen model: **YOLOv8-s** (speed–accuracy balance for embedded/real-time).

---

## 1. Model Selection

We evaluated object detectors for on-vehicle camera streams and selected **YOLOv8**.  
Reasons:

1) **Speed–Accuracy Balance**  
- v8 is lighter than very recent versions (v9–v11) while keeping strong accuracy.  
- Lower inference latency is critical for real-time driving; v8 hits the sweet spot.

2) **Efficiency & Portability**  
- Multi-scale predictions with an optimized backbone yield high performance per FLOP.  
- Small variants (**n**, **s**) run in real time on moderate GPU/CPU.

3) **Project Practicality**  
- v8 has mature docs, examples, and community support → fast adoption and easier debugging.

### Available Model Sizes
- **nano (n)**: ultra-light, fastest, lowest accuracy  
- **small (s)**: still real-time, noticeably better accuracy than *n*  
- **medium (m)**: balanced, needs stronger GPU  
- **large (l)**: higher accuracy, slower  
- **xlarge (x)**: best accuracy, heavy; not ideal for real-time

> **Final choice:** **YOLOv8-s** (meets real-time speed and field accuracy requirements).

---

## 2. Dataset Construction

We did **targeted curation** instead of using public datasets verbatim:
- Captured demo videos with our **Intel RealSense** on the vehicle.
- Extracted **horizon** and **ORB features** from frames and matched against public images.
- Kept only scenes that closely resemble our deployment environment to improve generalization.

Directory layout:

```
YOLOv8s/
├── data.yaml
├── images/
│   ├── train/
│   ├── valid/
│   └── test/
└── labels/
    ├── train/
    ├── valid/
    └── test/
```

Split: **Train : Valid : Test = 8 : 1 : 1**  
- We prioritized **enough train samples per class** (anchor stability & feature diversity).  
- 10% validation was sufficient to monitor loss/mAP; increasing it would starve training.

### `data.yaml` (example)
```yaml
train: ../images/train
valid: ../images/valid
test:  ../images/test

nc: 5
names: ['Red', 'Green', 'Yellow', 'roundabout', 'stop']
```

> We **do not** merge traffic-light colors into one class.  
> - A single “traffic_light” class tends to overfit lamp shapes rather than **color semantics**.  
> - Control logic (Stateflow, Pure Pursuit, etc.) needs **explicit color decisions**.  
> - Per-color training (Red/Yellow/Green) improves robustness under lighting/viewpoint changes.

---

## 3. Training

### 3.1 Data Augmentation Strategy

| Augmentation | When to Use | Description |
|:-------------|:------------|:-------------|
| **hsv_h** | When lighting or hue changes (e.g., indoor/outdoor, weather) | Adjust hue to handle color tone variation |
| **hsv_s** | When saturation varies due to weather or camera exposure | Apply saturation adjustment |
| **hsv_v** | When brightness varies (day/night, indoor/outdoor) | Adjust value to manage illumination changes |
| **scale** | When object–camera distance changes | Learn recognition over multiple scales |
| **translate** | When object position varies in image | Apply image translation for spatial robustness |
| **flipud** | When vertical inversion may occur | Vertical flip learning |
| **fliplr** | When left/right symmetry is possible | Horizontal flip (e.g., bidirectional lanes) |
| **degrees** | When small rotations are common | Apply rotation augmentation |
| **shear** | When perspective skew or slant may appear | Handle geometric distortion |
| **perspective** | When viewing angle changes (top/side/front) | Perspective compensation |
| **blur** | When motion blur occurs (speed, night) | Improve recognition under blur |
| **noise** | When sensor noise or low-light noise exists | Add Gaussian or salt–pepper noise |
| **cutout** | When occlusions occur | Simulate partial object hiding |
| **mosaic** | When background diversity is low | Combine four images for varied backgrounds |
| **mixup** | When class boundaries are ambiguous | Mix images for smoother boundary learning |

---

### 3.2 Optimizer: AdamW

We used **AdamW** instead of classical SGD for more stable convergence and built-in regularization.  
AdamW explicitly decouples weight decay from the gradient update:

$$
\theta \leftarrow \theta - \eta \cdot \left( \frac{m^t}{\sqrt{v^t} + \epsilon} + \lambda \theta \right)
$$

where  
- $\eta$ : learning rate  
- $m^t, v^t$ : first and second moment estimates  
- $\lambda$ : weight decay coefficient  
- $\epsilon$ : numerical stability term  

This improves both **training stability** and **generalization** by preventing overfitting on specific features (color, shape, location).

---

## 4. Results

### 4.1 Metrics (Validation)
Overall **Precision/Recall** were near 1.0 with strong **mAP@0.5** → reliable detection across classes.

<p align="center">
  <img src="../../../images/yolo2.png" alt="Validation metrics" width="780"/>
</p>

### 4.2 Qualitative Detections (Samples)

<table>
  <tr>
    <td align="center"><img src="../../../images/yolo3.png" alt="Sample det #1" width="420"/><br/><b>Sample #1</b></td>
    <td align="center"><img src="../../../images/yolo4.png" alt="Sample det #2" width="420"/><br/><b>Sample #2</b></td>
  </tr>
</table>

---

## 5. Simulation-First Pipeline (QLabs)

Competition schedules shifted, so we first built a **simulation-ready** model:
- Labeled images captured from **QLabs** scenes.

<p align="center">
  <img src="../../../images/yolo5.png" alt="Labeling examples" width="780"/>
</p>

**New training with sim data → metrics:**

<p align="center">
  <img src="../../../images/yolo6.png" alt="Sim-trained metrics" width="780"/>
</p>

<table>
  <tr>
    <td align="center"><img src="../../../images/yolo7.png" alt="Sim det #1" width="420"/><br/><b>QLabs Detection #1</b></td>
    <td align="center"><img src="../../../images/yolo8.png" alt="Sim det #2" width="420"/><br/><b>QLabs Detection #2</b></td>
  </tr>
</table>

<table>
  <tr>
    <td align="center"><img src="../../../images/yolo9.gif" alt="Sim clip #1" width="420"/><br/><b>Video (Sim) #1</b></td>
    <td align="center"><img src="../../../images/yolo10.gif" alt="Sim clip #2" width="420"/><br/><b>Video (Sim) #2</b></td>
  </tr>
</table>

---

## 6. ROS2 Node for Real-Time Control

We deployed a ROS2 node to bridge detection → motion:

- Runs the YOLOv8-s model at **10 FPS**.  
- Publishes a **binary topic per class id** (1/0) when an object is detected.  
- **Hold-off logic:**  
  - Threshold window = **4 s** (avoid sticky STOP oscillations).  
  - Once detected, **suppress re-publish for 10 s** to prevent loops.  
- MATLAB/Simulink subscribes to the topic and feeds **Stateflow** (stop/go logic).

---

## 7. Real Vehicle Transfer (QCar)

After the qualifier, we trained on **QCar camera data** and validated on videos:

<p align="center">
  <img src="../../../images/yolo11.gif" alt="QCar driving demo (detections)" width="780"/>
</p>

The model generalized from simulation to reality and **reliably recognized diverse objects**, enabling direct use in:
- **Stop decision**, **evasive maneuvering**, and **speed control** pipelines.

---

## 8. Repro Notes

- Keep **per-color** traffic-light classes (`Red`, `Yellow`, `Green`) for control clarity.  
- Prefer **YOLOv8-s** for real-time on embedded laptops/mini-PCs.  
- Curate data to your optics (FoV, mounting), lighting, and sign set.  
- Use **AdamW + mild augmentations**; avoid heavy geometric warps that distort semantics.

---
