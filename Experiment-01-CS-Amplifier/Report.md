# Experiment 1: Common Source (CS) Amplifier

## Brief Theory
The Metal Oxide Semiconductor Field Effect Transistor (MOSFET) is widely used as a switch and an amplifier. Among its configurations, the **Common Source (CS) Amplifier** is highly preferred because it offers high voltage gain and good input impedance. 

For the MOSFET to act as a linear amplifier, it must be biased in the **Saturation Region** ($V_{DS} \ge V_{ov}$). Setting the correct Q-point (quiescent point) ensures maximum signal swing without the output clipping. A basic CS amplifier provides an inverted output, demonstrating a 180-degree phase shift relative to the input signal.
### Effect of Channel Length Modulation (CLM)
In ideal long-channel devices, the drain current is completely independent of V<sub>DS</sub> in the saturation region. However, in short-channel devices (like the 180nm node used in this lab), increasing V<sub>DS</sub> causes the depletion region at the drain to expand, effectively shortening the channel. This is known as **Channel Length Modulation (CLM)**.

* **Effect on Output Resistance:** CLM prevents the I<sub>D</sub> curve from being perfectly flat. This slope creates a finite internal output resistance (r<sub>o</sub>) inside the MOSFET.
* **Effect on Voltage Gain:** Because this internal resistance r<sub>o</sub> acts in parallel with the external drain resistor R<sub>D</sub>, it lowers the total effective load resistance. Therefore, CLM significantly degrades the overall voltage gain of the amplifier.

---
Q1. CS Amplifier design having a power budget of 0.5mW and supply voltage of 1.5V. Load capacitance of 1pF and tsmc018.lib file for LTspice.

CIRCUIT :

![Circuit Diagram](images/CIrcuit-diagram.png)

CALCULATION :

Power = Voltage * Current

Current = Power / Voltage = 0.5m / 1.5 = 0.333 mA (or 333.33 uA).

$\mu_n = 273.809\text{ cm}^2/\text{V}$, $\epsilon_r = 3.9$, $t_{ox} = 4.1\text{nm}$

$R_D = \frac{V_{DD} - V_{DS}}{I_D} = \frac{1.5\text{V} - 0.75\text{V}}{0.333\text{mA}} = 2.25\text{ k}\Omega$

**Validation of Operating Region:**
For the MOSFET to operate as a linear amplifier,conditions must be satisfied:
1. **Turn-On Condition (V<sub>GS</sub> > V<sub>T</sub>):** The gate-to-source voltage must exceed the threshold voltage to invert the channel. 
   * *Validation:* We designed V<sub>GS</sub> = 0.9V and the extracted V<sub>T</sub> = 0.366V. Since **0.9V > 0.366V**, the device is properly turned on.
2. **Saturation Condition (V<sub>DS</sub> ≥ V<sub>OV</sub>):** The overdrive voltage is V<sub>OV</sub> = V<sub>GS</sub> - V<sub>T</sub> = 0.534V. 
   * *Validation:* Our operating point sets V<sub>DS</sub> = 0.75V. Since **0.75V > 0.534V**, the device is successfully pushed deep into the saturation region.

$$
I_D = \frac{1}{2} k_n (V_{ov})^2
$$

With having L = 180nm and W = 2.75um, the drain current of (approx.) ID == 333 uA is calculated and verified.

1. DC Operating Point :

**A. DC Operating Point**
The DC bias points verify that the power and current match the calculated budget, and the device operates in saturation.

![DC Operating Point](images/DC-operating-point.png)

**B. Voltage Transfer Characteristics (VTC)**
A DC sweep of the input voltage ($V_{GS}$) demonstrates the transition from cut-off to saturation (linear region) and finally triode.

![Voltage Transfer Characteristic](images/DC-sweep.png)

**C. Transient Analysis**
A 1kHz AC signal was applied to observe the time-domain amplification and the 180-degree phase shift.

* **Input Voltage ($V_{in}$):** 20mV peak-to-peak
* **Output Voltage ($V_{out}$):** 46.35mV peak-to-peak
* **Calculated Gain:** 46.35mV / 20mV = **$|A_v| = 2.31 V/V$**=**-2.31 V/V**(since it is 180deg phase shift)
  
* **Vin:**
  
![Transient Vin](images/Transient-analysis-Vin.png)
* **Vout:**
  
![Transient Vout](images/Transient-analysis-Vout.png)

![Transient Both](images/Transient-analysis-both.png)

* **Both waves:** This graph shows that there is 180 degree phase shift. Green representing the output voltage and Red the input. Notice the gain in Output voltage

---
**D. AC Analysis (Frequency Response)**
An AC sweep was performed to find the bandwidth, which is heavily influenced by the 1pF load capacitance. 
* **Mid-band Gain:** ~7.3 dB
* **Upper Cut-off Frequency ($f_H$):** 79.6 MHz

**1.With load capacitor:**
![AC Analysis with 1pF Capacitor](images/AC-analysis-with-capacitor.png)


**2.without load capacitor:**
*(Note: Without the 1pF capacitor, the bandwidth artificially extends into the GHz range, showing how load capacitance creates the dominant pole).*
![AC Analysis without Capacitor](images/AC-analysis-without-capacitor.png)
* **Mid-band Gain:** ~7.3 dB
* **Upper Cut-off Frequency ($f_H$):** 88.3GHz

### **Effect of Load Capacitance on Frequency Response (AC Analysis)**

The AC analysis of the Common Source amplifier demonstrates how the amplifier responds to different frequencies. A key observation from the simulation is the drastic change in the **upper cutoff frequency ($f_H$)**, also known as the -3dB bandwidth, when a load capacitor ($C_L$) is introduced. 

This behavior can be explained by analyzing the poles of the circuit. The output node of the amplifier forms an inherently low-pass RC filter, where the resistance is the output resistance of the amplifier ($R_{out} = r_o || R_D$) and the capacitance is the total capacitance seen at the output node.

#### **Case 1: Without Load Capacitor ($C_L = 0$)**
* **Simulated -3dB Frequency:** 88.3 GHz
* **Reasoning:** When no external load capacitor is connected, the only capacitances present at the output node are the **intrinsic parasitic capacitances** of the MOSFET itself (specifically the drain-to-bulk capacitance, $C_{db}$, and the gate-to-drain Miller capacitance, $C_{gd}$). 
* Because these parasitic capacitances are extremely small (typically in the femtofarad range for the 180nm process), the resulting RC time constant is tiny. This pushes the dominant pole (the -3dB cutoff point) to an extremely high frequency (88.3 GHz), essentially limited only by the physical limits of the transistor model.

#### **Case 2: With Load Capacitor ($C_L = 1 \text{ pF}$)**
* **Simulated -3dB Frequency:** 79.61 MHz
* **Reasoning:** When the 1 pF load capacitor is connected between the output node ($V_{out}$) and ground, it adds a massive amount of capacitance compared to the transistor's internal parasitics. This external capacitor creates a **dominant pole** at the output, acting as a strong low-pass filter that severely restricts the high-frequency bandwidth of the amplifier. 

#### **Mathematical Verification**
The upper -3dB cutoff frequency ($f_H$) caused by the output node can be theoretically calculated using the standard RC pole equation:
$$f_H = \frac{1}{2\pi \cdot R_{out} \cdot C_{total}}$$

Where:
- R_out ≈ r_o || R_D ≈ 1.99 kΩ (calculated in the previous section).
- C_total = C_L + C_parasitic ≈ 1 pF (since 1 pF dominates the femtofarad parasitics).

Plugging in the primary values:
$$f_H = \frac{1}{2\pi \cdot 1.99 × 10^3 Ω \cdot 1 × 10^-12 F}$$

$$f_H ≈ 79.98 MHz$$

*Conclusion:* The theoretically calculated cutoff frequency of **79.98 MHz** is incredibly close to the simulated cutoff frequency of **79.61 MHz**. The minor discrepancy of roughly 0.37 MHz is easily accounted for by the transistor's actual internal parasitic capacitances (C_db and the Miller-multiplied C_gd), which add a fraction of a picofarad to the load. This conclusively proves that the 1 pF capacitor acts as the dominant pole, dictating the bandwidth of the amplifier.

---
### **Theoretical Calculations for Voltage Gain (Av)**

To calculate the theoretical voltage gain of the Common Source amplifier, we first determine the transconductance ($g_m$) of the MOSFET at our chosen operating point.

**Step 1: Calculate Transconductance ($g_m$)**
Using the first-order square-law approximation for a MOSFET in the saturation region:
$$g_m \approx \frac{2 I_d}{V_{gs} - V_{TH}}$$

Based on the circuit design and the `tsmc018.lib` SPICE model:
* **Id:** 332.97 µA (from DC operating point simulation)
* **Vgs:** 0.9V (Input DC bias)
* **VTH:** 0.366V (Nominal threshold voltage `VTH0` extracted from the TSMC 180nm library 

$$g_m = \frac{2 \times 332.97 \times 10^{-6}}{0.9 - 0.366}$$
$$g_m = \frac{665.94 \times 10^{-6}}{0.534}$$
$$g_m \approx 1.247 \text{ mA/V}$$

**Step 2: Calculate Ideal Voltage Gain**
If we assume the internal output resistance ($r_o$) is infinitely large, the ideal gain formula is:
$$A_v \approx -g_m R_D$$

Given our drain resistor is 2.25 kΩ:
$$A_v = -(1.247 \times 10^{-3}) \times (2.25 \times 10^3)$$
$$A_v \approx -2.80 \text{ V/V}$$
*(Note: The negative sign denotes the 180° phase inversion typical of a Common Source amplifier).*

**Step 3: Calculate Practical Voltage Gain (Including $r_o$)**

**3.1 Why We Can't Calculate λ by Hand**
In this design, we are using the TSMC 180nm BSIM3v3 model. If you look at the `tsmc018.lib` file, there is no simple $\lambda$ value given. Instead, the simulator calculates the output resistance ($r_o$) using complex short-channel parameters like `PCLM` and `PDIBLC`. Because these equations are too complicated to solve by hand, we have to extract our small-signal parameters directly from a simulation.

**3.2 Finding Parameters Using a DC Sweep**
To find our exact $r_o$ and $\lambda$, we ran an isolated DC sweep on the NMOS transistor. We set the gate bias to match our actual amplifier, which gave us a drain current of about 333 µA. 

In a 180nm device, the saturation curve never goes perfectly flat because of short-channel effects. Because the line is slightly curved, the slope changes depending on where you look. To find the exact resistance our AC signal will see at our specific operating point ($V_{DS} = 0.75\text{V}$), we placed our cursors tightly at 0.7V and 0.8V. This "brackets" our operating point to give us the exact slope.

<img width="1919" height="1008" alt="Screenshot 2026-02-28 124231" src="https://github.com/user-attachments/assets/d7e205f1-54a1-4bfb-8d6f-faa43ecdd173" />

From the LTspice cursors, we get:
* $I_D \approx 332.92 \ \mu\text{A}$
* Slope ($g_{ds}$) = $5.80584 \times 10^{-5}\text{ A/V}$

Now we can calculate the exact internal output resistance ($r_o$):
$$r_o = \frac{1}{g_{ds}} = \frac{1}{5.80584 \times 10^{-5}} \approx 17.22\text{ k}\Omega$$

And we can find our equivalent $\lambda$ for our hand calculations:
$$\lambda = \frac{g_{ds}}{I_D} = \frac{5.80584 \times 10^{-5}}{332.92 \times 10^{-6}} \approx 0.174\text{ V}^{-1}$$

**3.3 Theoretical Voltage Gain Calculation ($A_v$)**
Now that we have our true output resistance ($r_o = 17.22\text{ k}\Omega$), we can calculate the voltage gain. We will use the transconductance we already found for our operating point ($g_m = 1.247\text{ mA/V}$).

First, we find the total output resistance ($R_{out}$), which is $r_o$ in parallel with our drain resistor ($R_D = 2.25\text{ k}\Omega$):
$$R_{out} = r_o || R_D = \frac{17.22 \cdot 2.25}{17.22 + 2.25} \approx 1.99\text{ k}\Omega$$

Finally, we calculate the theoretical voltage gain:
$$A_v = -g_m \cdot R_{out}$$
$$A_v = -(1.247\text{ mA/V}) \cdot (1.99\text{ k}\Omega) \approx -2.48\text{ V/V}$$

**Conclusion:**
The calculated practical gain of **-2.48 V/V** almost matches the simulated transient analysis result of **-2.318 V/V** (magnitude), validating the SPICE simulation against theoretical models.
## Comparison with Other CS Amplifier Configurations

While this experiment utilizes a standard passive resistive load (R<sub>D</sub>), Common Source amplifiers in integrated circuits typically use active loads to save silicon area and improve performance. Here is how our resistive load compares to other standard configurations:

| Parameter | CS with Resistive Load (Current Lab) | CS with Diode-Connected Load | CS with Current Source Load |
| :--- | :--- | :--- | :--- |
| **Voltage Gain (A<sub>v</sub>)** | **Medium** (-g<sub>m</sub> * (R<sub>D</sub> &#124;&#124; r<sub>o</sub>)) | **Low** (-g<sub>m1</sub> / g<sub>m2</sub>) | **Very High** (-g<sub>m1</sub> * (r<sub>o1</sub> &#124;&#124; r<sub>o2</sub>)) |
| **Voltage Headroom** | **Poor** (Large DC drop across the physical resistor) | **Moderate** (Requires at least V<sub>T</sub> + V<sub>OV</sub> across the load) | **Excellent** (Can operate very close to the supply rails) |
| **Linearity** | **Good** (Resistance is perfectly linear) | **Moderate** (Gain relies on transconductance ratios) | **High** (Active load provides near-constant current) |
---
## Summary

* **Design Completion:** A Common Source (CS) Amplifier was successfully designed, simulated, and mathematically verified using the TSMC 180nm technology node.
* **Power Budget:** The measured power consumption was strictly maintained at **0.5 mW**, drawing a drain current (I<sub>D</sub>) of approximately 333 µA.
* **Voltage Gain:** The theoretically calculated practical gain of **-2.48 V/V** closely matched the simulated transient gain of **-2.31 V/V**.
* **Frequency Response:** The calculated upper cutoff frequency of **79.98 MHz** almost perfectly aligned with the simulated AC response of **79.61 MHz**.

## Inference

* **Importance of Short-Channel Parameters:** The close match in voltage gain proves that extracting the channel-length modulation parameter (λ) and internal output resistance (r<sub>o</sub>) is absolutely critical for accurate hand-calculations in 180nm short-channel devices.
* **Dominant Pole Concept:** The massive drop in bandwidth from 88.3 GHz (bare MOSFET) to 79.61 MHz mathematically proves that the external 1 pF load capacitor (C<sub>L</sub>) creates the dominant pole. It completely overrides the MOSFET's internal parasitic capacitances (C<sub>db</sub>, C<sub>gd</sub>) and acts as the primary bottleneck for amplifier speed.
* **Q-Point Optimization:** Forcing the MOSFET into saturation and successfully centering V<sub>DS</sub> at exactly 0.75V (half of the 1.5V supply) established the most stable operating window for the amplifier.

### Fundamental Theory & Conclusion

By fundamental principle, to achieve maximum gain and linearity, a MOSFET must operate deep within the saturation region. The Q-point (V<sub>DS</sub> and I<sub>D</sub>) must be chosen carefully so that the output signal can achieve a full, symmetrical 360-degree swing for any changes within the V<sub>GS</sub> input window. Centering this point prevents the output voltage from clipping and keeps the transistor from slipping into the triode or cut-off regions, thereby eliminating non-linear distortion. 

Therefore, the finalized design parameters (**V<sub>GS</sub>** = 0.9 V, **W** = 2.75 µm, **L** = 180 nm, **V<sub>DD</sub>** = 1.5 V, and **R<sub>D</sub>** = 2.25 kΩ) successfully fulfill the target 0.5 mW power budget while maintaining optimal, distortion-free amplification.
