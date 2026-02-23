# Experiment 01: Common Source (CS) Amplifier Design and Analysis

## Overview
This repository contains the complete design, theoretical analysis, and LTspice simulation for a Common Source (CS) Amplifier. The experiment aims to design the amplifier within a specified power budget, verify its DC operating point, and evaluate its transient and AC responses.

## Objectives
* **Design Validation:** Calculate appropriate values for component parameters (like $R_D$ and MOSFET width $W$) to meet a specific power budget ($0.333 \text{ mA}$).
* **DC Analysis:** Verify that the MOSFET operates in the saturation region.
* **Transient & AC Analysis:** Determine the practical voltage gain and bandwidth (identifying the high cut-off frequency, $f_H$) and compare them against theoretical calculations.

## Repository Structure
* **`EXP-01-CS-Amplifier.asc`**: The main LTspice schematic file containing the circuit design.
* **`report.md`**: The comprehensive lab report detailing the mathematical verification, power budget calculations, gain validation, and comparison between theoretical and simulated results.
* **`images/`**: A directory containing all necessary simulation screenshots, including the circuit diagram, DC operating point, VTC curve, transient response, and AC analysis graphs.
