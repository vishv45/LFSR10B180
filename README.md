# 10-Bit Linear Feedback Shift Register (LFSR)
**SCL 180nm CMOS | Transistor-Level Full Custom Physical Design**

## Overview
This repository contains the full-custom, transistor-level physical implementation and post-silicon characterization of a 10-bit Linear Feedback Shift Register (LFSR) in the SCL 180nm CMOS process node. 

To maintain stringent control over device geometry, active-area sharing, and parasitic routing penalties, automated standard-cell synthesis methodologies were intentionally bypassed. The project encompasses the complete Backend-of-Line (BEOL) workflow: schematic capture of fundamental logic primitives, custom standard cell layout utilizing Euler path optimization, top-level datapath integration, and Siemens Calibre sign-off (DRC, LVS, PEX).

📝 **[View the 2-Page IP Datasheet](Docs/LFSR10B180_Datasheet.pdf)**
📝 **[View the Full Project Report](Docs/LFSR_Group10.pdf)**

## Key Architectural Features
* **Universal NAND Logic:** 100% static-CMOS NAND topology, eliminating pass-transistor logic vulnerabilities and ensuring a full 1.8V rail-to-rail output swing.
* **Maximal Length Sequence (MLS):** Utilizes the Galois Field primitive polynomial `P(x) = x^10 + x^7 + 1` to generate a 1023-state sequence.
* **Custom 6-NAND Sequential Logic:** Implements an edge-triggered 7474-style architecture, eradicating the localized clock-skew found in standard master-slave setups.
* **Hardware-Level Initialization:** Utilizes bifurcated `Dff7474_prst` and `Dff7474_rst` standard cells to instantaneously hardwire a deterministic `1100000000` seed upon asynchronous reset, bypassing the critical path delays of datapath multiplexers.

## Cryptographic & Security Applications
Beyond general pseudo-random number generation (PRNG), this LFSR macro is architected as a foundational primitive for hardware security modules (HSMs) and trusted memory systems.
* **Deterministic Seeding:** The custom hardwired initialization sequence (bypassing multiplexer logic) inherently protects the macro against fault-injection attacks attempting to force a "zero-lock" stall state.
* **Stream Cipher Foundation:** The high-frequency 1.75 GHz physical datapath is optimized to serve as a fast keystream generator for lightweight stream ciphers (e.g., Trivium or A5/1 style topologies) in secure IoT edge devices.

## Post-Layout Performance (PEX Extracted)
| Parameter | Description | Value |
| :--- | :--- | :--- |
| **Technology Node** | SCL CMOS | 180nm |
| **Supply Voltage (VDD)** | Nominal | 1.8 V |
| **Maximum Frequency (Fmax)** | CLK to Datapath | 1.75 GHz |
| **Dynamic Power** | Active PRBS switching | 246.1 µW |
| **Static Power** | Leakage (Clock = 0V) | 18.4 nW |
| **Total Area** | Silicon Bounding Box | 7855.28 µm² |
| **Setup / Hold Time** | Internal Feedback limits | 78.5 ps / 14.2 ps |
*Sign-off: 100% DRC and LVS clean in Siemens Calibre (Metal density errors waived for isolated IP extraction).*

## Tools Used
* **Schematic & Layout:** Cadence Virtuoso
* **Analog/Mixed-Signal Simulation:** Cadence ADE Explorer (Spectre RF)
* **Physical Verification:** Siemens Calibre (DRC, LVS, PEX)

## Pin Configuration
| Pin Name | Direction | Description |
| :--- | :--- | :--- |
| **VDD** | Input | 1.8V Core Supply |
| **GND** | Input | 0.0V Reference |
| **CLK** | Input | System Clock (Rising-edge active) |
| **PRESET_BAR** | Input | Asynchronous active-low initialization |
| **Q** | Output | 1-bit pseudo-random serial output |

## Module Hierarchy
The top module (`lfsr_10bit`) is built from the ground up using fundamental logic gates:
```text
lfsr_10bit
|-- Dff7474_preset (x2 instances)
|   |-- nand_3input (x4 instances)
|   |-- nand_2input (x2 instances)
|
|-- Dff7474_reset (x8 instances)
|   |-- nand_3input (x4 instances)
|   |-- nand_2input (x2 instances)
|
|-- xor_n (x1 instance)
|   |-- nand_2input (x4 instances)


