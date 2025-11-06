# Interleaved Boost Converter — Design and Verification

This README documents the complete design, analysis, simulation, and verification of a **2-phase interleaved boost converter** intended for small autonomous platforms (e.g., F1TENTH). We retain every equation, figure, and table from the original document.

---

## 0. Motivation

Most subsystems of a small autonomous platform (sensing, compute, actuation) have off-the-shelf alternatives. However, for **battery → system bus step-up conversion**, especially an **interleaved boost converter** meeting our specifications, suitable COTS options were not available.  
Therefore, we **designed and implemented** a 2-phase interleaved boost converter to step up from a battery (e.g., **2S Li-ion**) to a system bus (e.g., **19 V**).

We defined input variations and load profiles, evaluated topologies using **PSIM** simulations and datasheet analysis, and finally adopted **Peak Current Mode Control (PCMC)** with **slope compensation** to suppress high-duty subharmonics. A **180° inter-phase shift** was applied to reduce input/output ripple.

The overall flow was:

1) Parameter calculation 
2) **Open-Loop Test** 
3) Current sensing (CT/sense-resistor) and slope-compensation parameter design 
4) **Open-Loop Test** (re-run) 
5) Loop-stability review (compensator)  
6) **Closed-Loop Test**

Simulations confirmed rated-load operation achieving the target output (e.g., **19 V / 5 A**) and reduced ripple/thermal stress compared with a single-phase converter.

---

## 1. Converter Topology Selection and Rationale

### 1.1 Buck Converter

- Step-down; lowers $V_\text{out}$ with respect to $V_\text{in}$.  
- Our requirement is **$V_\text{out} > V_\text{in}$**, so **not suitable**.

<br/>
<img src="../../../images/con1.png" width="420" alt="Buck converter diagram"/>

### 1.2 Buck-Boost Converter

- Can step down or step up depending on duty.  
- If duty control fails, it can inadvertently switch to buck mode.  
- **Polarity inversion** between input and output is a drawback.  
- Due to polarity and fail-safe concerns, **not suitable**.

<br/>
<img src="../../../images/con2.png" width="420" alt="Buck-Boost converter diagram"/>

### 1.3 Boost Converter

- Step-up; increases $V_\text{out}$ beyond $V_\text{in}$.  
- Fits our requirement (**$V_\text{out} > V_\text{in}$**).

<br/>
<img src="../../../images/con3.png" width="420" alt="Boost converter diagram"/>

**Decision:** We selected the **Boost** topology.  
However, a single-phase boost imposes higher stress on the switch and leads to larger inductor current ripple, which increases component stress. To reduce device stress and ripple, we adopt **interleaving**.

---

## 2. Interleaving Method and Control Strategy

Interleaving drives **$N$ phases** in parallel with a phase shift of **$360^\circ/N$**.

<br/>
<img src="../../../images/con4.png" width="460" alt="Interleaved phases concept"/>

**Effects of interleaving:**
- Phase ripple currents cancel each other, **greatly reducing output ripple**.  
- Current sharing reduces device **peak/RMS currents** and **distributes heat**.  
- With reduced ripple and stress, component rating margins improve and **efficiency optimization** becomes easier.

We must also select a **control method**. Two typical strategies are **Voltage Mode Control (VMC)** and **Current Mode Control (CMC)**.

### 2.1 Voltage Mode Control (VMC)

<br/>
<img src="../../../images/con5.png" width="460" alt="Voltage Mode Control block diagram"/>

- Feedback output voltage, compare to reference in an error amplifier, and compare with a ramp to produce PWM duty; stabilizes $V_\text{out}$.  
- **Issue:** Phase-current balancing is not guaranteed → device mismatch, temperature drift, and wiring imbalance lead to **circulating currents**, **localized heating/saturation**, and loss of interleaving ripple-cancellation benefits.  
- Since interleaving **requires accurate inter-phase current balance**, VMC is **undesirable** here.

### 2.2 Current Mode Control (CMC)

<br/>
<img src="../../../images/con6.png" width="460" alt="Current Mode Control block diagram"/>

- Sense inductor/switch current, compare to a reference, end PWM at the intersection; indirectly regulates $V_\text{out}$.  
- A **per-phase current loop** directly regulates each phase current, making currents naturally equal despite device/temperature/wiring variations.  
- With **clock interleaving (phase synchronization)**, each phase’s current ramp stays consistent, enabling precise duty/phase control.  
- Because **current balancing is guaranteed**, **CMC is preferred** for interleaved operation.

**Decision:** We adopt **Current Mode Control** with **slope compensation**.

---

## 3. Specifications and Component Parameter Calculations

## Boost Converter Design Tool

| **Category** | **Item** | **Value** | **Unit** | **Remark** |
|--------------|-----------|-----------|-----------|-------------|
| **Input Data** | $V_{\text{in}}$ | 7.4 | V |  |
|  | $V_o$ | 19 | V |  |
|  | $V_{o,\text{ripple ratio}}$ | 1 | % of $V_o$ |  |
|  | $I_{o,\max}$ | 5 | A |  |
|  | Inductor current ripple ratio | 5 | % of $I_o$ |  |
|  | $f_{\text{sw}}$ | 300 | kHz |  |
| **Output Data** | $D_{\max}$ | 0.611 | – |  |
|  | $I_{L,\text{ripple}}$ | 0.642 | A |  |
|  | $V_{o,\text{ripple}}$ | 0.190 | V |  |
|  | $L_o$ | 23.461 | µH |  |
|  | $C_o$ | 53.555 | µF |  |
|  | $T_s$ | 3.333 | µs |  |
|  | Switch voltage stress | 19.000 | V |  |
|  | Switch current stress | 5.321 | A |  |
|  | Diode voltage stress | 19.000 | V |  |
|  | Diode current stress | 5.321 | A |  |
| **Minimum Rated Values** | Switch: rated voltage | 33.250 | V | 175% margin |
|  | Switch: rated current | 10.642 | A | 200% margin |
|  | Diode: rated voltage | 33.250 | V | 175% margin |
|  | Diode: rated current | 10.642 | A | 200% margin |

> If your repository uses different filenames, keep the relative paths and replace the `src` with your actual image paths.

## Converter Summary (Design Targets)

| Item                     | Value                                     | Unit  | Note |
|-------------------------|-------------------------------------------|:-----:|------|
| Topology                | 2-Phase Interleaved Boost, **PCMC**       |  –    | Peak Current Mode + slope comp. |
| $V_{\text{in}}$         | 7.4–12.6 (worst-case design at 7.4)       |  V    | 2S Li-ion example |
| $V_o$ / $I_o$           | 19 / 5                                     | V / A | System bus |
| Switching               | 300 kHz **per phase** (effective ≈ 600 kHz) | kHz | Interleaved |
| Phases $(N)$            | 2                                          |  –    | 180° phase shift |
| $C_o$ (total)           | 12 µF × 3 = 36 µF (MLCC, low ESR)         |  µF   | Example from sim/BOM |
| Ripple targets          | Inductor current 5%, Output voltage 1% (≤ 0.19 Vpp) |  – | Design constraint |

> With a **2-phase interleaved boost**, very stable voltage/current ripple is achieved; $L$ and $C_o$ can be reduced vs. single-phase while keeping ripple within targets.



## Key Definitions and Sizing Equations

- **ROA (Inductor current ripple ratio)**

$$
\mathrm{ROA}=\frac{\Delta i_L}{I_L}\times 100\%)
$$

- **ROV ($V_o$ ripple ratio)**

$$
\mathrm{ROV}=\frac{\Delta V_o}{V_o}\times 100\%)
$$

- **Duty ($D$) for an ideal boost**

$$
\frac{V_o}{V_{\text{in}}}=\frac{1}{1-D}
$$

- **Inductor average current**

$$
I_L=\frac{I_o}{1-D}
$$

- **Inductor ripple current**

$$
\Delta i_L=\mathrm{ROA}\cdot I_L
$$

- **Output ripple voltage**

$$
\Delta V_o=\mathrm{ROV}\cdot V_o
$$

- **Inductance**

$$
L=\frac{V_{\text{in}}}{\Delta i_L}\,D\,T_s
$$

- **Output capacitance**

$$
C_o=\frac{I_o}{\Delta V_o}\,D\,T_s
$$



**Note:** With a **2-phase interleaved boost**, we achieved very stable voltage/current ripple and could reduce the inductance and capacitance compared to a single-phase design, providing **cost and size benefits**.

---

## 4. Open-Loop Test

<br/>
<img src="../../../images/con7.png" width="460" alt="Open-loop test schematic/plot"/>

- Since $D>0.5$, **subharmonic oscillation** can arise; we add **slope compensation** to prevent it.  
- For control, we must sense current. We emulate the real **current transformer (CT)** used in hardware to ensure simulation behavior closely matches practical operation.

**Simulation results**

### Inductor

- **Inductor Current**

  <img src="../../../images/con8.png" width="460" alt="Inductor current (open loop)"/>

- **Inductor Voltage**

  <img src="../../../images/con9.png" width="460" alt="Inductor voltage (open loop)"/>

### Switch

<img src="../../../images/con10.png" width="460" alt="Switch waveforms (open loop)"/>

### Diode

<img src="../../../images/con11.png" width="460" alt="Diode waveforms (open loop)"/>

### Output

<img src="../../../images/con12.png" width="460" alt="Output node waveforms (open loop)"/>

- **Output Current**

  <img src="../../../images/con13.png" width="460" alt="Output current (open loop)"/>

- **Output Voltage**

  <img src="../../../images/con14.png" width="460" alt="Output voltage (open loop)"/>

**Observations**

- Phase currents/voltages operate with a **$180^\circ$ phase shift** → interleaving works correctly.  
- Even with **smaller $L$ and $C$** than single-phase sizing, measured ripples are **lower**: about **0.03 V** (voltage) and **0.01 A** (current).

---

## 5. Feedback Controller Design

### 5.1 Converter Transfer Function $G(s)$ (small-signal)

Reference: *Small Signal Modeling and Stability Analysis of N-Phase Interleaved Boost Converter*.

<br/>
<img src="../../../images/con15.png" width="450" alt="G(s) small-signal model and parameters"/>

### 5.2 Compensator Transfer Function $H(s)$ — Type-II

<br/>
<img src="../../../images/con16.png" width="450" alt="Type-II compensator design figure"/>

**Analytic form used in the document:**

$$
\frac{V_c(s)}{V_o(s)}=
-\frac{1}{R_1}\cdot \frac{sC_2R_2+1}{s\left(sC_1C_2R_2+C_1+C_2\right)}
$$

<br/>
<img src="../../../images/con17.png" width="450" alt="Bode plot of compensator H(s)"/>
<img src="../../../images/con18.png" width="450" alt="Controller parameter report (zeros/poles, component suggestions)"/>

---

## 6. Closed-Loop Test

<br/>
<img src="../../../images/con19.png" width="400" alt="Closed-loop test overview"/>

- An inrush current over **14 A** was observed; by adding a resistor and a diode, we reduced the inrush to **≤ 12 A**.

**Simulation results**

### Inductor

- **Inductor Current**

  <img src="../../../images/con20.png" width="460" alt="Inductor current (closed loop)"/>

  The inrush is around **11.25 A**. Considering a typical **200% current margin**, we design for **≈ 14 A**, which our sizing satisfies.

  <img src="../../../images/con21.png" width="460" alt="Phase current balance (closed loop)"/>

  Phase currents run with a **$180^\circ$** phase shift → interleaving verified.

- **Inductor Voltage**

  <img src="../../../images/con22.png" width="460" alt="Inductor voltage (closed loop)"/>

### Switch

<img src="../../../images/con23.png" width="460" alt="Switch waveforms (closed loop)"/>
<img src="../../../images/con24.png" width="460" alt="Additional switch plots (closed loop)"/>

### Diode

- **Diode Current**

  <img src="../../../images/con25.png" width="460" alt="Diode current (closed loop)"/>

- **Diode Voltage**

  <img src="../../../images/con26.png" width="460" alt="Diode voltage (closed loop)"/>

### Output

<img src="../../../images/con27.png" width="460" alt="Output node waveforms (closed loop)"/>

- **Output Current**

  <img src="../../../images/con28.png" width="460" alt="Output current (closed loop)"/>

- **Output Voltage**

  <img src="../../../images/con29.png" width="460" alt="Output voltage (closed loop)"/>

**Observations**

- $V_\text{out}$ ripple ≈ **0.05 V**, $I_\text{out}$ ripple ≈ **0.02 A** — despite smaller $L$ and $C$ than the single-phase baseline, ripple **further decreases** thanks to interleaving.  
- Output voltage/current overshoot remains within allowable limits → **stable operation**.

---

## 7. Bill of Materials (BOM) Notes and Selections

### 7.1 RMS Current Sizing

- **Switch RMS current**

  
$$
\mathrm{ROA} = \frac{\Delta i_L}{I_L} \times 100(\%)
$$


- **Output capacitor RMS current**

  
$$
\mathrm{ROV} = \frac{\Delta V_o}{V_o} \times 100(\%)
$$


Using the above:

- $I_{\mathrm{sw,rms}} \approx \mathbf{1.53\ A}$  
- $I_{C,\mathrm{rms}} \approx \mathbf{0.26\ A}$

### 7.2 PWM Controller — **UCC28220**

A 2-phase interleaved PWM controller that guarantees **180° phase shift**, supports **current-mode control**, includes **protection** and **slope compensation** — well-matched to our design.

<br/>
<img src="../../../images/con30.png" width="460" alt="UCC28220 block/typical application"/>

### 7.3 Gate Driver — **UCC27424**

UCC28220’s OUT pins supply ~1 V logic and only tens of mA, insufficient to quickly charge/discharge power-MOSFET gates.  
Using a dedicated **gate driver** supplies adequate gate voltage/current, reducing rise/fall times, **switching loss**, and **EMI**. UCC27424 is recommended in the UCC28220 datasheet and was selected.

<br/>
<img src="../../../images/con31.png" width="460" alt="UCC27424 driver figure"/>

### 7.4 MOSFET — **AG508EGD3HRBTL**

Selection criteria:

1) **$V_\mathrm{DS}$ rating**:  
   OFF-state MOSFET sees $V_\text{out}$ plus spikes → choose **$V_\mathrm{DS} > 1.75\times V_\text{out}$**  
   ⇒ $V_\mathrm{DS} > 19\text{ V}\times 1.75 = \mathbf{33.25\ V}$

2) **$I_\mathrm{D}$ rating**:  
   Conduction path carries inductor current → choose ≥ **$200\%$ of $I_{\mathrm{sw,rms}}$**  
   ⇒ $I_\mathrm{D} \ge 2\times 1.53\text{ A} = \mathbf{3.06\ A}$

3) **$R_\mathrm{DS(on)}$**: as low as possible to minimize conduction loss.

4) **$V_\mathrm{GS}$**: Driver (UCC27424) outputs ≥ **4.5 V**; select MOSFET fully ON at **$V_\mathrm{GS}\approx 4.5\text{ V}$** (logic-level, $V_\mathrm{GS,\text{th}}=2\sim4$ V).

5) **Switching speed / Gate charge $Q_g$**:  
   With **$f_s=300$ kHz**, choose **$Q_g \le 30$ nC**.

<br/>
<img src="../../../images/con32.png" width="460" alt="Selected MOSFET datasheet capture"/>

### 7.5 Diode — **SVM1045V2B_R2_00001** (Schottky)

1) **Reverse voltage $V_R$**:  
   At least **$1.75\times V_\text{out}$** ⇒ $V_R\ge \mathbf{33.25\ V}$

2) **Forward current $I_F$**:  
   ≥ **2×** $I_\text{out}$ ⇒ $I_F > \mathbf{10\ A}$ (for $I_\text{out}=5$ A)

3) **Forward drop $V_F$**:  
   Lower $V_F$ improves efficiency; **$V_F \le 1$ V** preferred.

4) **Reverse recovery**:  
   With **$f_s=300$ kHz**, select **$t_{rr}\le 100$ ns**.

<br/>
<img src="../../../images/con33.png" width="460" alt="Selected diode datasheet capture"/>

### 7.6 Output Capacitor — **RKS1V120MCNAFGGS**

1) **Voltage rating**:  
   ≥ **$1.75\times V_\text{out}$** ⇒ $V \ge \mathbf{33.25\ V}$

2) **Ripple current**:  
   Use **200% margin** on $I_{C,\mathrm{rms}}$  
   ⇒ $I_{\text{ripple,rms}} > 0.26\times 2 = \mathbf{0.52\ A}$

<br/>
<img src="../../../images/con34.png" width="460" alt="Selected capacitor datasheet capture"/>

### 7.7 Current Transformer — **CT02-100**

1) **Turns ratio**: set to **1:100** per design.  
2) **Current rating**: must exceed peak input current (e.g., **> 13 A**).

<br/>
<img src="../../../images/con35.png" width="460" alt="Current transformer capture"/>

### 7.8 Error Amplifier / Comparator — OPA2197IDGKR

1. **Crossover Frequency**

With voltage-loop $f_c \approx 2.396\ \mathrm{kHz}$, noise-gain $\mathrm{NG} \approx 5$, and margin $k = 10 \sim 20$,  
the required gain-bandwidth product (GBW) is:

$$
\mathrm{GBW}_{\min} = k \cdot f_c \cdot \mathrm{NG} = 0.24 \text{–} 0.48\ \mathrm{MHz}
$$

Choose **MHz-class (≥ 5–10 MHz)** op-amp → **OPAx197 family**.


2. **−3 dB Bandwidth**

With single-pole approximation:

$$
f_{-3\mathrm{dB}} \approx \frac{\mathrm{GBW}}{\mathrm{NG}}
$$

For **OPA2197 (GBW ≈ 10 MHz)**, we get about **2 MHz** at $\mathrm{NG}=5$, providing ample margin.


3. **Low Input Bias**

Voltage divider: $R_{\text{top}} = 100\ \mathrm{k\Omega}$, $R_{\text{bot}} = 7.34\ \mathrm{k\Omega}$.  
Bias-induced output error:

$$
\Delta V_{\text{out}} = I_B \cdot R_{\text{top}}
$$

With $I_B = 20\ \mathrm{nA}$:

$$
\Delta V_{\text{out}} \approx 2.0\ \mathrm{mV}\ (\approx 0.0105\%)
$$

This satisfies the required precision for OPA2197/OPA197.

<br/>
<img src="../../../images/con36.png" width="460" alt="OPA2197 datasheet capture"/>

---

## 8. PCB

### 8.1 Schematic

<img src="../../../images/con37.png" width="450" alt="PCB schematic"/>

### 8.2 Layout

<img src="../../../images/con38.png" width="450" alt="PCB layout"/>

---

## 9. Conclusion

- Phase currents and voltages operate with a **$180^\circ$ phase shift**, confirming correct **interleaving**.  
- Interleaving **reduces device stress and size** and **lowers ripple**, making it suitable even for **kW-class** high-power converters by improving stability, reducing converter volume, and cutting cost.

---
