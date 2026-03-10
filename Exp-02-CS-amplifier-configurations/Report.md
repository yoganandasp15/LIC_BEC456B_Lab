# Experiment 02: CS Amplifier Configurations using TSMC 180nm

## 1. Objective
To design and compare three different Common Source (CS) amplifier configurations using the TSMC 180nm parameter library in LTspice. The objective is to extract specific performance metrics, compare the configurations, and justify the interpretations.

---

## 2. Theory & Circuit Configurations
All three circuits utilize a PMOS active load ($M_2$) to provide a high output resistance, maximizing the intrinsic gain. The primary differences lie in the source degeneration networks applied to the input amplifying transistor ($M_1$).

### Configuration A: Resistor Degeneration ($R_S$)
* **Description:** A physical resistor $R_S$ is placed at the source of the input NMOS ($M_1$).
* **Purpose:** This provides ideal, perfectly linear source degeneration. It stabilizes the DC operating point and makes the amplifier highly linear, though it sacrifices some voltage gain. 
* **Trade-off:** Physical resistors consume a massive amount of silicon area in IC design and are subject to high manufacturing variations, making this configuration impractical for compact integrated circuits.
* **Theoretical Gain:** $$A_v=\frac{-g_{m1}}{1+g_{m1}R_S}$$

![Circuit Diagram](Images/Exp-2a/Exp-2a-circuit.png)

### Configuration B: Active Current Source Degeneration
* **Description:** The resistor is replaced by an NMOS ($M_3$) biased by a fixed DC voltage ($V_{B2}$), operating as a constant current source.
* **Purpose:** A MOSFET in saturation provides an extremely high output resistance ($r_{o3}$). This applies heavy degeneration, ensuring rock-solid DC bias stability without taking up the massive footprint of a physical resistor.
* **Trade-off:** The massive AC resistance looking into the drain of $M_3$ severely degrades the AC voltage signal. This is typically used for specific biasing control rather than maximizing signal amplification.

//![Config B Schematic](images_config_B/schematic_B.png)

### Configuration C: Diode-Connected Degeneration
* **Description:** The source of $M_1$ is tied to an NMOS ($M_3$) configured with its gate and drain shorted together (diode-connected).
* **Purpose:** This is the IC designer's space-saving alternative to Configuration A. The diode-connected MOSFET acts as a small-signal resistor with a value of approximately $1/g_{m3}$. 
* **Advantage:** Because $M_1$ and $M_3$ are built on the same silicon, their transconductances ($g_{m1}$ and $g_{m3}$) track each other across temperature and process variations. The gain becomes a stable ratio of these parameters, making the circuit incredibly resilient.
* **Theoretical Gain (Approximate):**
  $$A_v\approx\frac{-g_{m1}}{1+\frac{g_{m1}}{g_{m3}}}$$

## 3. Simulation Results & Analysis

### 3.1 Configuration A: Resistor Degeneration ($R_S$)

#### 1. Transistor Sizing & Design Parameters
In 180nm CMOS design, the aspect ratios ($W/L$) are the primary design variables. The following sizes were utilized to achieve the desired operating point:
* **$M_1$ (NMOS Amplifying Device):** W = 15.3µm, L = 180nm
* **$M_2$ (PMOS Active Load):** W = 56.35µm, L = 180nm

#### 2. DC Analysis & Saturation Verification
The operating point was set to ensure all transistors operate securely in the saturation region. A source resistor $R_S$ of 666.67 Ω was used.

* **Supply Voltage ($V_{DD}$):** 1.5 V
* **Drain Current ($I_D$):** 300.7 µA
* **Total DC Power Dissipation:** 451.05 µW

**Saturation Proof for $M_1$:**
To act as a linear amplifier, $M_1$ must satisfy $V_{DS} > V_{GS} - V_{TH}$ (or $V_{DS} > V_{OV}$).
From the SPICE output log:
* $V_{GS}$ = 0.610 V
* $V_{TH}$ = 0.474 V
* Overdrive Voltage ($V_{OV}$) = 0.610 - 0.474 = 0.136 V
* Actual $V_{DS}$ = 0.750 V

Since **0.750 V > 0.136 V**, the amplifying transistor $M_1$ is deeply in saturation.

![DC Operating Point](Images/Exp-2a/Exp-2a-DC-op.png)

#### 3. DC Sweep Analysis
A DC sweep was performed to identify the high-gain linear region of the amplifier's transfer characteristic. The chosen DC bias of $V_{in}$ = 0.81 V places the operating point dead center in this linear region.

![DC Sweep](Images/Exp-2a/Exp-2a-DC-sweep.png)

#### 4. Transient Analysis (Time Domain)
A 1 kHz sine wave with an amplitude of 10 mV (20 $mV_{p-p}$) was applied. 

* **Input Peak-to-Peak ($V_{in(p-p)}$):** 20 mV
* **Output Peak-to-Peak ($V_{out(p-p)}$):** 207.43 mV
* **Simulated Transient Gain:** 10.37 V/V

![Transient Response](Images/Exp-2a/Exp-2a-transient-Vout.png)

#### 5. AC Analysis (Frequency Domain)
An AC frequency sweep was performed to determine the small-signal gain and bandwidth.

* **AC Mid-Band Gain (Mag):** 20.826 dB
* **AC Linear Gain ($A_v$):** 10.99 V/V
* **Phase:** -180° (Inverting)

![AC Analysis](Images/Exp-2a/Exp-2a-AC-analysis.png)

### 6. Theoretical vs. Simulated Gain Comparison

<img width="476" height="639" alt="Exp-2a-log" src="https://github.com/user-attachments/assets/8ddb9375-9e01-43b4-bf9b-ed89548764d2" />


To verify our simulation, let's calculate the theoretical gain by hand using the small-signal values from the LTspice `.log` file. Since LTspice gives us drain-source conductance ($G_{ds}$), we just take the inverse to get the output resistance ($r_o$):

* $g_{m1} = 3.62 \text{ mA/V}$
* $r_{o1} = \frac{1}{G_{ds1}} = \frac{1}{1.00 \times 10^{-4}} = 10 \text{ k}\Omega$
* $r_{o2} = \frac{1}{G_{ds2}} = \frac{1}{7.45 \times 10^{-5}} = 13.42 \text{ k}\Omega$
* $R_S = 666.67 \text{ }\Omega$

As instructed, we are using the exact formula for a source-degenerated CS amplifier with an active load (taking channel length modulation into account where $\lambda \neq 0$):

$$A_v = \frac{-g_{m1}}{1 + g_{m1}R_S + \frac{R_S}{r_{o1}}} \times \left( [g_{m1} R_S r_{o1} + R_S + r_{o1}] \parallel r_{o2} \right)$$

**Step 1: Find the output resistance looking into the NMOS ($R_{out\_NMOS}$)**
First, we calculate the resistance looking down into $M_1$ with the source resistor included:
$$R_{out\_NMOS} = g_{m1} R_S r_{o1} + R_S + r_{o1}$$
$$R_{out\_NMOS} = (3.62\text{m})(666.67)(10\text{k}) + 666.67 + 10\text{k} \approx 34.8 \text{ k}\Omega$$

**Step 2: Find the total output resistance ($R_{out}$)**
Next, we put this in parallel with the PMOS load ($r_{o2}$):
$$R_{out} = 34.8\text{k} \parallel 13.42\text{k} = 9.69 \text{ k}\Omega$$

**Step 3: Find the effective transconductance ($G_{m(eff)}$)**
$$G_{m(eff)} = \frac{g_{m1}}{1 + g_{m1}R_S + \frac{R_S}{r_{o1}}}$$
$$G_{m(eff)} = \frac{3.62\text{m}}{1 + 2.413 + 0.067} = 1.04 \text{ mA/V}$$

**Step 4: Final Voltage Gain**
$$A_v = -G_{m(eff)} \times R_{out} = -1.04 \text{ mA/V} \times 9.69 \text{ k}\Omega = -10.07 \text{ V/V}$$

**Conclusion:** Our calculated theoretical gain magnitude is **10.07 V/V**, which is close to our Transient analysis gain of **10.37V/V** and simulated AC gain of **10.99 V/V**. 

**Why is there a small difference?**
Our hand calculation assumes $V_{BS} = 0$. However, because $R_S$ raises the source of $M_1$ above ground, it actually triggers the **body effect**. The LTspice log shows $g_{mb} = 7.96 \times 10^{-4} \text{ A/V}$. We ignored this in our hand formula to keep the math manageable, which explains the ~8% difference. 
Also, the simulated Transient gain (**10.37 V/V**) is slightly different from the AC gain because AC analysis is purely linear, while Transient analysis captures the slight non-linearities of the MOSFET during the 20 mV input swing.

---
# Experiment 2b: Common Source Amplifier with Active Load and Source Degeneration

## 1. Objective
To design, simulate, and analyze Configuration B—a Common Source (CS) amplifier utilizing an **active current source for source degeneration**—using the TSMC 180nm process in LTspice. The experiment follows a strict rules to evaluate its performance:

1. **DC Analysis:** Establish biasing strategies to fix the operating point, ensuring all transistors remain in the saturation region, and calculate the total DC power dissipation.
2. **DC Sweep:** Verify the transfer characteristics to secure a highly linear operating point.
3. **Transient Analysis:** Inject a small-signal input to verify time-domain linear amplification, and extract the voltage gain along with the maximum input and output voltage swings.
4. **AC Analysis:** Extract the frequency response metrics, including the midband Gain, 3dB Bandwidth, and Unity Gain Bandwidth (UGB).
5. **Mathematical Verification:** Compare the extracted simulation metrics against the exact small-signal theoretical model to justify the results and understand the trade-offs of active degeneration.

---

## 2. Circuit Diagram & Biasing Strategy
The circuit is a Common Source amplifier consisting of three MOSFETs. To keep all transistors in the saturation region, the DC bias voltages were calculated based on three initial design assumptions: a target drain current of **ID = 0.3 mA**, an overdrive voltage of **VOV = 0.25V**, and a degeneration voltage drop of **VRS = 0.30V**.

Based on these targets, the voltage sources were derived as follows:

* **M1 (NMOS Amplifying Transistor):** Assuming a typical 180nm NMOS threshold voltage (VTH ~ 0.36V), the required DC gate-to-source voltage is VGS = VTH + VOV = 0.61V. Since the source sits at 0.30V (VRS), the input DC bias is:
  **VIN** = VGS + VRS = 0.60V + 0.30V = **0.90V**.

* **M2 (NMOS Current Source):** M2 provides active source degeneration. Its source is grounded, meaning its gate bias (VB2) directly sets its VGS. To sink the target 0.3 mA, it requires a VGS of roughly 0.60V. After minor SPICE tuning for its specific VTH, this was locked at:
  **VB2 = 0.61V**.

* **M3 (PMOS Active Load):** Its source is tied to VDD (1.50V). To push exactly 0.3 mA down into the circuit, it requires an absolute |VGS| of approximately 0.64V based on our selected aspect ratio. Therefore, the gate bias is:
  **VB1** = VDD - |VGS| = 1.50V - 0.64V = **0.86V**.

![Circuit Diagram](Images/Exp-2b/Exp-2b-circuit.png)

---

## 3. Theoretical Formulation
To understand the behavior of this amplifier, we rely on its small-signal model. In this configuration, the physical source resistor is replaced by an active NMOS current source (M2). While M2 provides excellent DC bias stability, it introduces an extremely large small-signal output resistance ($r_{o2}$) at the source of the amplifying transistor (M1).

The standard, simplified gain equation for a source-degenerated amplifier is:
$$A_v \approx \frac{-g_{m1} r_{o3}}{1 + g_{m1} r_{o2}}$$

However, because the active degeneration resistance ($r_{o2}$) is exceptionally large, we cannot ignore the channel-length modulation of the main amplifier (M1). Ignoring the intrinsic output resistance ($r_{o1}$) leads to significant theoretical errors. Therefore, to accurately predict the midband gain and match our SPICE simulations, we must use the exact small-signal equation:
$$A_v = \frac{-g_{m1}}{1 + g_{m1} r_{o2} + \frac{r_{o2}}{r_{o1}}} \times \left( \left[ g_{m1} r_{o2} r_{o1} + r_{o2} + r_{o1} \right] \parallel r_{o3} \right)$$

*(Note: This exact theoretical model will be calculated using extracted SPICE parameters and compared against the simulated AC frequency response in Section 8).*

---

## 4. DC Analysis (Operating Point)
After setting up the bias voltages, a DC operating point simulation was performed. This step verifies that our hand calculations work in the actual SPICE model and ensures that all transistors are operating in the saturation region ($|V_{DS}| > |V_{DSAT}|$). It also allows us to calculate the circuit's total power dissipation.

![DC Operating Point](Images/Exp-2b/Exp-2b-op.png)

### DC Parameters Table
| Transistor | ID (mA) | VGS (V) | VTH (V) | VDS (V) | VDSAT (V) | Region |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **M1 (NMOS)** | 0.30 | 0.60 | 0.476 | 0.754 | 0.099 | Saturation |
| **M2 (NMOS)** | 0.30 | 0.61 | 0.500 | 0.300 | 0.094 | Saturation |
| **M3 (PMOS)** | -0.30 | -0.64 | -0.509 | -0.445 | -0.122 | Saturation |

**Total DC Power Dissipation:**
Since the entire circuit consists of a single branch drawing 0.30 mA from the 1.50V supply, the total DC power is calculated as:
$$P_{DC} = V_{DD} \times I_D = 1.50\text{ V} \times 0.30\text{ mA} = 0.45\text{ mW} \text{ (or } 450\text{ }\mu\text{W)}$$

**Verification of Biasing:**
* **Input Bias (VIN):** 0.90V
* **Output Bias (VOUT):** 1.054V

To confirm our design values, we can check the node voltages. The input DC bias (VIN) is 0.90V. Since M1 has a simulated VGS of 0.60V, the source voltage must be exactly 0.30V ($0.90\text{V} - 0.60\text{V}$). This perfectly matches the VDS of M2, showing that our active degeneration network is working exactly as we planned. 

Additionally, for the PMOS active load (M3), saturation is confirmed using absolute values ($|-0.445\text{V}| > |-0.122\text{V}|$). Because all three transistors satisfy the saturation condition, the circuit is ready to function as a linear amplifier.

---

## 5. DC Sweep Analysis (Voltage Transfer Characteristic)
While the operating point confirms the transistors are in saturation at exactly 0.90V, a DC sweep is necessary to visualize the overall Voltage Transfer Characteristic (VTC). This ensures that our chosen 0.90V bias does not sit too close to the cutoff or triode boundaries, but rather dead-center in the steepest (highest gain) linear amplification region.

![DC Sweep](Images/Exp-2b/Exp-2b-dc-sweep.png)
*Observation:* The curve confirms that 0.90V is an optimal bias point, residing safely within the linear segment of the VTC.

---

## 6. Transient Analysis (Time Domain)
Having secured a stable, high-gain DC operating point, we now inject a small-signal sine wave to observe the amplifier's real-time dynamic behavior and ensure the signal does not clip the boundaries identified in the DC sweep.

* **Input Signal:** Vin is a 1 kHz sine wave with a 10 mV peak amplitude (20 mV peak-to-peak) superimposed on our 0.90V DC offset.
* **Output Signal:** The amplified output is measured as **35.57 mV** peak-to-peak without distortion.

![Transient Input and Output](Images/Exp-2b/Exp-2b-transient-both.png)
![Transient Output Measurement](Images/Exp-2b/Exp-2b-transient-vout.png)

**Gain Calculation from Transient:**
$$A_v = \frac{V_{out(p-p)}}{V_{in(p-p)}} = \frac{35.57 \text{ mV}}{20 \text{ mV}} = 1.778 \text{ V/V}$$
$$A_v (\text{dB}) = 20 \log_{10}(1.778) \approx 5.00 \text{ dB}$$

---

## 7. AC Analysis (Frequency Domain)
While transient analysis confirms time-domain small-signal amplification, an AC frequency sweep is required to determine the bandwidth limits and the precise linear midband gain of the amplifier.

### Midband Gain
The measured midband AC gain is **5.08 dB**, which strongly correlates with our transient calculation.
![AC Gain](Images/Exp-2b/Exp-2b-AC-gain.png)

### 3dB Bandwidth
The upper cut-off frequency is located at the point where the gain drops by approximately 3 dB from its midband value.
* **Measured Frequency:** 238.23 MHz
![3dB Bandwidth](Images/Exp-2b/Exp-2b-AC-3db.png)

### Unity Gain Bandwidth (UGB)
The Unity Gain Bandwidth is the frequency at which the amplifier's gain drops to 0 dB.
* **Measured UGB:** 458.14 MHz
![Unity Gain Bandwidth](Images/Exp-2b/Exp-2b-AC-UGB.png)

*Note: The UGB is relatively close to the 3dB bandwidth because the initial midband gain is very low (5.08 dB). The amplifier only needs to attenuate the signal by ~5 dB to reach unity gain.*

---

## 8. Theoretical vs. Simulated Gain Verification
To verify that our simulator is behaving accurately according to semiconductor physics, we extract the small-signal parameters from the SPICE error log and run them through the exact analytical formula established in Section 3.

![SPICE Log](Images/Exp-2b/Exp-2b-log.png)

From the log, we extract:
* $g_{m1} = 3.85 \text{ mA/V}$
* $r_{o1} \approx 10 \text{ k}\Omega$ *(Output resistance of M1)*
* $r_{o2} = 5.26 \text{ k}\Omega$ *(Degeneration resistance from M2)*
* $r_{o3} = 10.47 \text{ k}\Omega$ *(Active load resistance from PMOS M3)*

**Step 1: Find the output resistance looking into the NMOS ($R_{out\_NMOS}$)**
First, we calculate the massive resistance looking down into M1 degenerated by M2:
$$R_{out\_NMOS} = g_{m1} r_{o2} r_{o1} + r_{o2} + r_{o1}$$
$$R_{out\_NMOS} = (3.85\text{m})(5.26\text{k})(10\text{k}) + 5.26\text{k} + 10\text{k} \approx 202.51\text{k} + 15.26\text{k} = 217.77 \text{ k}\Omega$$

**Step 2: Find the total output resistance ($R_{out}$)**
Next, we place this in parallel with the PMOS active load ($r_{o3}$):
$$R_{out} = 217.77\text{k} \parallel 10.47\text{k} = \frac{217.77 \times 10.47}{217.77 + 10.47} \approx 9.99 \text{ k}\Omega$$

**Step 3: Find the effective transconductance ($G_{m(eff)}$)**
$$G_{m(eff)} = \frac{g_{m1}}{1 + g_{m1} r_{o2} + \frac{r_{o2}}{r_{o1}}}$$
$$G_{m(eff)} = \frac{3.85\text{m}}{1 + 20.251 + 0.526} = \frac{3.85\text{m}}{21.777} \approx 0.1768 \text{ mA/V}$$

**Step 4: Final Voltage Gain**
$$A_v = -G_{m(eff)} \times R_{out} = -0.1768 \text{ mA/V} \times 9.99 \text{ k}\Omega = -1.766 \text{ V/V}$$
$$A_v \text{ (dB)} = 20 \log_{10}(1.766) = 4.94 \text{ dB}$$

**Mathematical Verdict:** Our calculated theoretical gain is **1.766 V/V (4.94 dB)**. This is remarkably close to our simulated Transient gain of **1.778 V/V (5.00 dB)** and simulated AC gain of **5.08 dB**. The highly marginal ~0.14 dB difference is primarily attributed to the body effect ($g_{mb}$) on M1 triggered by the elevated source voltage from M2, which is accurately captured in SPICE but omitted in our hand calculations for mathematical clarity.

---

## 9. Conclusion
The Configuration B Common Source Amplifier was successfully designed, simulated, and mathematically verified. The progressive workflow from hand-calculated bias design to DC analysis confirmed that the main amplifier, active load, and current source were all securely biased into the saturation region exactly as predicted. 

While replacing a physical source resistor with an active current source (M2) saves physical silicon area and provides exceptional bias stability, its massive degeneration resistance severely penalizes the voltage gain. As a result, the simulated midband gain dropped to **5.08 dB** (compared to earlier non-degenerated topologies). However, as a trade-off for the low gain, the heavy degeneration allows the circuit to exhibit a highly stable and broad operational bandwidth, successfully achieving a UGB of **458.14 MHz**. The tight correlation between our exact theoretical hand-calculations and the SPICE AC analysis confirms the validity of the small-signal model and concludes the experiment successfully.

---
# Experiment 2(c) — Common Source Amplifier with Current Mirror Load

**Course:** Linear Integrated Circuits Lab — BEC456B  
**Technology:** TSMC 180nm CMOS | **Simulator:** LTSpice XVII  
**VDD = 1.5V | P ≤ 0.5mW | CL = 1pF | L = 180nm**

---

## 1. Aim

The aim of this experiment is to design and simulate a Common Source (CS) amplifier using a PMOS current mirror as the active load, in TSMC 180nm CMOS technology using LTSpice. The specific objectives are:

1. Fix the DC operating point so that all transistors are in saturation and verify the power consumption is within the given budget.
2. Run a transient simulation to confirm the amplifier is working linearly without distortion.
3. Find the voltage gain, maximum output swing, and maximum input swing, and compare them with the theoretical values.
4. Run an AC simulation and extract the mid-band gain, 3dB bandwidth, and Unity Gain Bandwidth (UGB) with CL = 1pF.
5. Justify any differences between the theoretical and simulated results.

---

## 2. Circuit Description

![Circuit Diagram](Exp-2c-circuit.png)

The circuit uses three MOSFETs. M1 is the NMOS transistor doing the actual amplification — it is configured in common source, meaning the input goes to its gate and the output is taken from its drain. M2 is a PMOS transistor acting as the active load — instead of a resistor, we use M2 because it gives a much higher output resistance, which directly increases the gain. M3 is a diode-connected NMOS (gate tied to drain) placed at the source of M1, and it works as the current mirror reference to set the bias current accurately.

VDD = 1.5V is connected to the source of M2. The gate of M2 is biased at 0.86V through voltage source V5. The input signal is applied to the gate of M1 through V3, which is set as `SINE(1.23V, 10mV, 1kHz) AC 1`. The output Vout is taken at the common drain point of M1 and M2, and a load capacitance of CL = 1pF is connected there for the AC simulation.

---

## 3. Design Procedure

### 3.1 Given Specifications

- Supply voltage: VDD = 1.5V
- Maximum power dissipation: P ≤ 0.5mW
- Load capacitance: CL = 1pF
- Technology: TSMC 180nm CMOS
- Channel length for all transistors: L = 180nm

### 3.2 Step 1 — Finding the Drain Current

The first thing we did was find the maximum drain current allowed from the power budget:

$$I_D = \frac{P}{V_{DD}} = \frac{0.5 \times 10^{-3}}{1.5} = 0.333 \text{ mA}$$

So the target quiescent current is 0.333 mA. The simulation gave Id = 0.301 mA, which keeps the power at 0.452 mW — just within the 0.5mW limit.

### 3.3 Step 2 — Setting the Output DC Voltage

This was the main design decision — where to place Vout(DC) so we get the maximum possible output swing without any transistor going into triode.

For the output to swing without clipping, both M1 and M2 need to stay in saturation at all times. This means:

- **Lower limit of Vout** — how low can Vout go before M1 or M3 leaves saturation:
$$V_{out,min} = V_{DS,sat,M1} + V_{DS,sat,M3} \approx V_{ov,M1} + V_{ov,M3}$$

- **Upper limit of Vout** — how high can Vout go before M2 leaves saturation:
$$V_{out,max} = V_{DD} - |V_{DS,sat,M2}| = V_{DD} - |V_{ov,M2}|$$

We started with VDD/2 = 0.75V as a reference center, then used VDD − 0.6 = 0.9V as the target center accounting for the available headroom. We then distributed a swing of ±0.44V symmetrically, which gave us:

$$V_{out,DC} = V_{DD} - 0.44 = 1.5 - 0.44 = \mathbf{1.06 \text{ V}}$$

The simulation confirmed V(vout) = **1.06024 V** ✓

The reason Vout is placed closer to VDD rather than exactly at the middle is that the lower side has more headroom — the NMOS Vdsat values (M1 and M3 stacked) are smaller than the PMOS Vdsat (M2 alone), so we can afford to place Vout higher and still get a reasonable symmetric swing.

### 3.4 Step 3 — Setting the Input Bias (Vin DC)

Once we fixed Vout = 1.06V and Id = 0.301mA, we tuned Vin until the operating point settled correctly. From the BSIM3 log:

- Vgs(M1) = 0.610V, Vth(M1) = 0.514V → Vov = **0.096V**
- Vds(M1) = 0.440V >> Vov → **M1 is in saturation** ✓

So we set **Vin = 1.23V** as the DC bias.

### 3.5 Step 4 — Setting M2 Gate Bias

M2's gate is biased at VB1 = 0.86V via V5:

- Vgs(M2) = 0.86 − 1.5 = −0.64V
- |Vth(M2)| = 0.509V → |Vov(M2)| = 0.131V
- |Vds(M2)| = 0.440V > |Vov(M2)| → **M2 is in saturation** ✓

### 3.6 Step 5 — W/L Sizing

All transistors use L = 180nm as given. The widths were calculated using the drain current equation in saturation:

$$I_D = \frac{1}{2} \mu C_{ox} \frac{W}{L} V_{ov}^2$$

Each transistor was sized to carry the target Id at its respective Vov. M2 needs a much larger width because PMOS mobility is roughly half that of NMOS, so it needs more width to carry the same current. The final dimensions are:

- **M1 (NMOS):** W = 27.47 µm, L = 180nm → W/L = 152.6
- **M3 (NMOS):** W = 18.27 µm, L = 180nm → W/L = 101.5
- **M2 (PMOS):** W = 58.1 µm, L = 180nm → W/L = 322.8

---

## 4. DC Operating Point Results

![DC Operating Point](Exp-2c-dcop.png)

![BSIM3 MOSFET Parameters](Exp-2c-logs.png)

The .op simulation gives the node voltages and device parameters at the quiescent point. The important results are:

- V(vin) = 1.23V (M1 gate bias as set)
- V(vout) = **1.06024V** — exactly where we designed it ✓
- V(vrs) = 0.620V — this is the source of M1 / drain of M3

All three transistors carry **Id = 0.301 mA**, confirming the current mirror is working correctly.

**Power check:**
$$P = V_{DD} \times I_{D} = 1.5 \times 0.301 \times 10^{-3} = \mathbf{0.4515 \text{ mW} \leq 0.5 \text{ mW}} \checkmark$$

**Saturation check** from the BSIM3 log (Vds must be greater than Vdsat for each transistor):

- M1: Vds = 0.440V >> Vdsat = 0.089V ✅ Saturation
- M3: Vds = 0.620V >> Vdsat = 0.100V ✅ Saturation
- M2: |Vds| = 0.440V >> |Vdsat| = 0.122V ✅ Saturation

The small-signal parameters we will use for gain calculation: gm1 = 4.52 mS, gm3 = 3.89 mS, gds1 = 0.148 mS, gds2 = 0.0971 mS, gds3 = 0.114 mS.

---

## 5. Theoretical Gain Calculation

### 5.1 Computing ro Values

From the gds values in the BSIM3 log:

$$r_{o1} = \frac{1}{g_{ds1}} = \frac{1}{1.48 \times 10^{-4}} = \mathbf{6.757 \text{ k}\Omega}$$

$$r_{o2} = \frac{1}{g_{ds2}} = \frac{1}{9.71 \times 10^{-5}} = \mathbf{10.299 \text{ k}\Omega}$$

$$r_{o3} = \frac{1}{g_{ds3}} = \frac{1}{1.14 \times 10^{-4}} = \mathbf{8.772 \text{ k}\Omega}$$

$$\frac{1}{g_{m3}} = \frac{1}{3.89 \times 10^{-3}} = \mathbf{257.07 \ \Omega}$$

### 5.2 Applying the Gain Formula

We are using the full general formula where **all three λ ≠ 0** — meaning channel length modulation is considered for all three transistors M1, M2, and M3. None of them are treated as ideal. The formula from the sheet is:

$$A_V = \frac{-g_{m1}}{\left[1 + g_{m1}\left(\frac{1}{g_{m3}} \| r_{o3}\right) + \frac{1}{\left(\frac{1}{g_{m3}} \| r_{o3}\right) \times r_{o1}}\right]} \times \left[g_{m1}\left(\frac{1}{g_{m3}} \| r_{o3}\right)r_{o1} + r_{o1} + \left(\frac{1}{g_{m3}} \| r_{o3}\right)\right] \| r_{o2}$$

**Step 1 — Find (1/gm3 ∥ ro3):**

This is the impedance seen at the source of M1 due to M3 being diode-connected.

$$\frac{1}{g_{m3}} \| r_{o3} = \frac{257.07 \times 8772}{257.07 + 8772} = \mathbf{249.75 \ \Omega}$$

Since 1/gm3 (257Ω) is much smaller than ro3 (8.77kΩ), the parallel result is almost equal to 1/gm3 — so ro3 barely contributes here.

**Step 2 — Denominator:**

$$\text{Denom} = 1 + (4.52 \times 10^{-3})(249.75) + \frac{1}{249.75 \times 6757}$$

$$= 1 + 1.1289 + 0.000001 = \mathbf{2.1289}$$

The third term is essentially zero (≈10⁻⁶) so it doesn't affect anything practically.

**Step 3 — Numerator bracket (before the parallel with ro2):**

$$\text{Num} = (4.52 \times 10^{-3})(249.75)(6757) + 6757 + 249.75$$

$$= 7627.5 + 6757 + 249.75 = \mathbf{14634.25 \ \Omega}$$

**Step 4 — Parallel with ro2:**

$$14634.25 \| 10299 = \frac{14634.25 \times 10299}{14634.25 + 10299} = \mathbf{6044.71 \ \Omega}$$

**Step 5 — Final gain:**

$$A_V = \frac{-4.52 \times 10^{-3}}{2.1289} \times 6044.71 = \mathbf{-12.83 \text{ V/V}}$$

$$\boxed{|A_V|_{dB} = 20\log_{10}(12.83) = \mathbf{22.17 \text{ dB}}}$$

### 5.3 Output Resistance

$$R_{out} = r_{o1} \| r_{o2} = \frac{6757 \times 10299}{6757 + 10299} = \mathbf{4.09 \text{ k}\Omega}$$

---

## 6. Transient Analysis

![Transient Waveform — V(vout) and V(vin)](Exp-2c-transient.png)

**Setup:** Input = SINE(1.23V, 10mV, 1kHz), simulated for 10ms.

Looking at the waveform, the output (green) is clearly an amplified and inverted version of the input (blue). There's no distortion or clipping, which means all three transistors stayed in saturation throughout — exactly what we wanted.

The output swings from around 0.92V to 1.19V (about 270mV peak-to-peak) for a 20mV peak-to-peak input. So the gain from the transient is roughly:

$$|A_V|_{transient} \approx \frac{270}{20} = 13.5 \approx \mathbf{22.6 \text{ dB}}$$

This matches well with both the theoretical value (22.17 dB) and the AC simulation (23.04 dB) ✓

**Maximum output swing:**

The output can only swing as far as the transistors allow before they leave saturation. Using the Vdsat values from the BSIM3 log:

$$V_{out,min} = V_{DS,sat,M1} + V_{DS,sat,M3} = 0.089 + 0.100 = \mathbf{0.189 \text{ V}}$$

$$V_{out,max} = V_{DD} - |V_{DS,sat,M2}| = 1.5 - 0.122 = \mathbf{1.378 \text{ V}}$$

$$V_{out,swing,max} = 1.378 - 0.189 = \mathbf{1.189 \text{ V}_{pp}}$$

From the DC point of 1.06V — we have 0.318V of upward swing and 0.871V of downward swing. The asymmetry makes sense because the M1+M3 stack on the bottom has lower combined Vdsat than M2 alone on the top.

**Maximum input swing** (before the output clips):

$$V_{in,swing,max} = \frac{1.189}{12.83} \approx \mathbf{92.7 \text{ mV}_{pp}}$$

Our test input of 20mV pp is well within this, so no clipping — confirmed by the clean sinusoid in the transient plot ✓

---

## 7. AC Analysis

### 7.1 Mid-band Gain

![AC Gain — Mid-band](Exp-2c-gain.png)

The AC sweep shows a flat gain region from very low frequencies up to around 100MHz. The cursor placed in the flat region reads a mid-band gain of **23.036 dB (14.17 V/V)**.

Comparing with theory — we calculated **22.17 dB** using the full formula with all λ ≠ 0, and got **23.04 dB** from simulation. That's a difference of only **0.87 dB (~4%)**, which is a very good match for a hand calculation. The small error comes from the fact that the BSIM3 model in simulation additionally accounts for body effect on M1 (Vbs = −0.0537V shifts Vth), short-channel effects, and velocity saturation — none of which are captured in the long-channel formula.

### 7.2 3dB Bandwidth

![3dB Bandwidth](Exp-2c-3db-gain.png)

Two cursors are used — one placed in the mid-band (164.33 kHz, 23.036 dB) and one at the point where the gain has dropped by 3dB (421.70 MHz, 20.035 dB). The cursor ratio box shows exactly **3.001 dB** difference, confirming:

$$\mathbf{f_{3dB} = 421.7 \text{ MHz}}$$

We can also estimate where the dominant pole should be from the output RC time constant:

$$f_{p1} = \frac{1}{2\pi \cdot R_{out} \cdot C_L} = \frac{1}{2\pi \times 4090 \times 1 \times 10^{-12}} = \mathbf{38.9 \text{ MHz}}$$

The simulated bandwidth (421.7 MHz) is about 10× higher than this estimate. This happens because M3's source degeneration reduces the effective output resistance at Vout — the pole moves to a higher frequency. So the bandwidth is actually better than what the simple RC calculation would suggest, which is a useful side-effect of this configuration.

### 7.3 Unity Gain Bandwidth

![Unity Gain Bandwidth](Exp-2c-UGB.png)

The cursor is placed at the 0dB crossing point. It reads **6.7608 GHz** at a magnitude of 7.089 mdB which is essentially 0 dB, so:

$$\mathbf{UGB = 6.761 \text{ GHz}}$$

As a quick check using the gain-bandwidth product:

$$GBW = 14.17 \times 421.7 \text{ MHz} = \mathbf{5.98 \text{ GHz}}$$

The simulated UGB (6.76 GHz) is slightly higher than this because the gain doesn't roll off at a perfect −20dB/decade — higher-order poles start adding phase before we reach unity gain, pulling the actual 0dB crossing a bit higher than the ideal single-pole estimate.

The phase at UGB is −294.25°, which means there's 114.25° of phase shift beyond −180°. This tells us there are significant higher-order poles in the system, which is expected for a three-transistor circuit.

---

## 8. Summary of Results

| Parameter               | Theoretical          | Simulated            |
|-------------------------|----------------------|----------------------|
| Drain Current (Id)      | 0.333 mA             | 0.301 mA             |
| Power Dissipation       | 0.5 mW (max)         | 0.452 mW ✓           |
| V(vout) DC              | 1.06 V               | 1.06024 V ✓          |
| Voltage Gain            | 22.17 dB (12.83 V/V) | 23.04 dB (14.17 V/V) |
| 3dB Bandwidth           | 38.9 MHz (simple RC) | 421.7 MHz            |
| Unity Gain Bandwidth    | 5.98 GHz (GBW)       | 6.761 GHz            |
| Max Output Swing (P-P)  | 1.189 V              | —                    |
| Max Input Swing (P-P)   | ~92.7 mV             | —                    |
| Rout                    | 4.09 kΩ              | —                    |

---

## 9. Discussion

**On the gain:** The theoretical gain using the full formula (all λ ≠ 0, all transistors non-ideal) came out to 22.17 dB, and the simulation gave 23.04 dB — only 0.87 dB apart. This is a good result. The gain is lower than the simple gm1·(ro1∥ro2) = 25.3 dB estimate because M3's diode connection introduces source degeneration at M1's source. The degeneration impedance is (1/gm3 ∥ ro3) ≈ 249.75 Ω, which reduces the effective transconductance and brings the gain down. This is a trade-off — we lose some gain but gain biasing accuracy and linearity in return.

**On the bandwidth:** The 3dB bandwidth of 421.7 MHz is much higher than the 38.9 MHz simple RC pole estimate. Source degeneration from M3 lowers the effective Rout at the output node, which directly pushes the dominant pole to a higher frequency. So this topology naturally gives better bandwidth than a simple resistor-loaded CS stage.

**On the UGB:** The UGB of 6.76 GHz is quite high for a single-stage amplifier driving a 1pF load. This is mainly because gm1 is large (4.52 mS, from the wide M1 with W/L = 152.6) and the bandwidth is wide. The GBW product of ~5.98 GHz matches well with the simulated UGB.

**On saturation:** All three transistors were confirmed to be in deep saturation at the operating point — M1 (Vds/Vdsat = 4.9×), M3 (Vds/Vdsat = 6.2×), and M2 (|Vds|/|Vdsat| = 3.6×). This is important because the small-signal model (and hence all our gain calculations) is only valid in saturation.

**On the current mirror:** M3 carries the same current as M1 (0.301 mA each), confirming the mirror is working. The W/L ratio M1/M3 = 152.6/101.5 ≈ 1.50, but both carry equal current because the body effect on M1 (Vbs = −0.0537V) shifts its threshold voltage up slightly, adjusting the operating point until current equality is reached.

---

## 10. Conclusion

We designed a CS amplifier with a PMOS current mirror active load in TSMC 180nm using LTSpice, and all the design targets were met. The DC operating point was fixed at Vout = 1.06V with all three transistors in saturation and total power of 0.452 mW — within the 0.5mW budget. The theoretical gain from the formula (22.17 dB) matched the simulation (23.04 dB) with less than 1 dB error, which validates our design approach. The 3dB bandwidth came out to 421.7 MHz with CL = 1pF, and the UGB was 6.761 GHz — both much better than what a simple resistor load would give. The transient simulation confirmed clean linear amplification with no distortion, consistent with all transistors being in saturation.

The main trade-off in this configuration is that M3's source degeneration reduces gain compared to the ideal case. A future improvement would be to use a cascode current mirror, which removes the source degeneration effect while keeping accurate current biasing.

---

*Simulated using LTSpice XVII with TSMC BSIM3 180nm CMOS models.*  
*All data taken from exp-02-c.log and exp-02-c.raw output files.*



