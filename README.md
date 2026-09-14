<div align="center">
  <img src="Images/3dboard view-ai generated.jpeg" width="100%" alt="Unipolar Stepper Motor Driver PCB — AI-Generated 3D Render"/>
</div>

---

<div align="center">

# Unipolar Stepper Motor Driver

### A Hardware-Logic Stepper Motor Driver — No Microcontroller Required

[![Build Status](https://img.shields.io/badge/build-passing-brightgreen?style=flat-square)](.)
[![KiCad Version](https://img.shields.io/badge/KiCad-8.0-blue?style=flat-square&logo=kicad)](https://www.kicad.org/)
[![License: CERN-OHL-S v2](https://img.shields.io/badge/License-CERN--OHL--S%20v2-blue?style=flat-square)](LICENSE)
[![Hardware Status](https://img.shields.io/badge/Hardware-Prototype%20v1.0-orange?style=flat-square)](Fabrication/)
[![PCB Layers](https://img.shields.io/badge/PCB-2--Layer%20FR4-lightgrey?style=flat-square)](Hardware/)
[![Open Source Hardware](https://img.shields.io/badge/OSHW-Certified-brightgreen?style=flat-square)](https://www.oshwa.org/)

</div>

---

## Abstract

This repository documents a fully discrete, hardware-logic-based unipolar stepper motor driver implemented as a two-layer PCB designed in **KiCad 8.0**. The design deliberately eliminates any dependence on a microcontroller or programmable device, instead realizing the sequencing, direction-control, and power-switching functions entirely through classical digital and analog ICs operating in open-loop.

The signal path is built around a **NE555 timer** configured in astable mode to generate a continuous clock pulse train, a **74LS194 4-bit universal shift register** to implement the four-phase commutation sequence with hardware-selectable direction, and a **7407 hex buffer** (open-collector, non-inverting) to drive the base networks of four **TIP120 NPN Darlington power transistors**. Each transistor drives one motor winding, and a **1N4001 rectifier diode** is placed in a flyback configuration across each winding terminal to suppress the inductive voltage transient at turn-off.

The PCB layout enforces a strict separation of signal and power domains, with independent trace widths calculated from IPC-2221 current-carrying capacity standards. A bottom-layer ground copper pour provides a low-impedance return path, improves thermal dissipation from the power transistors, and reduces susceptibility to radiated emissions from the switching power stage.

---

## Table of Contents

1. [Theoretical Background](#1-theoretical-background)
2. [Circuit Architecture](#2-circuit-architecture)
3. [PCB Layout and Routing Strategy](#3-pcb-layout-and-routing-strategy)
4. [3D Visualization](#4-3d-visualization)
5. [Bottom Layer and Grounding Strategy](#5-bottom-layer-and-grounding-strategy)
6. [Fabrication and Manufacturing Specifications](#6-fabrication-and-manufacturing-specifications)
7. [Repository Structure](#7-repository-structure)
8. [Getting Started](#8-getting-started)
9. [Contributing](#9-contributing)
10. [License](#10-license)

---

## 1. Theoretical Background

The design is derived from a reference circuit published in a practical electronics reference text. The original schematic was adopted as the conceptual baseline and subsequently re-implemented in KiCad 8.0 with component-level refinements, improved footprint selections, and a production-intent PCB layout.

<div align="center">

<table>
  <thead>
    <tr>
      <th align="center">Original Concept — Reference Schematic</th>
      <th align="center">KiCad Implementation — Production Schematic</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td align="center">
        <img src="Images/Schematic From Practical Electronics Book.png" width="460" alt="Original Schematic from Practical Electronics Reference"/>
        <br/>
        <sub><b>Figure 1.</b> Reference schematic sourced from a practical electronics textbook. This diagram defines the four-phase unipolar commutation sequence using shift-register logic.</sub>
      </td>
      <td align="center">
        <img src="Images/schematic diagram.png" width="460" alt="KiCad Schematic Implementation"/>
        <br/>
        <sub><b>Figure 2.</b> KiCad 8.0 schematic implementation (<code>Hardware/Stepper_Motor_Driver.kicad_sch</code>). Hierarchical net labels, ERC-clean, with annotated reference designators.</sub>
      </td>
    </tr>
  </tbody>
</table>

</div>

### Logic Flow

The commutation sequence operates as follows:

| Stage | Active IC | Function |
|---|---|---|
| **1 — Clock Generation** | NE555 (Astable) | Generates a square wave whose frequency is set by R1, R2, and C1. Motor speed is directly proportional to this frequency. |
| **2 — Sequence Generation** | 74LS194 (SRSR / SLSL modes) | The shift register's parallel outputs (QA–QD) cycle through the four-phase binary pattern `1000 → 0100 → 0010 → 0001` (or its reverse). The DIR input pin selects left-shift or right-shift to reverse motor direction. |
| **3 — Signal Buffering** | 7407 (Open-Collector Buffer) | The 74LS194 TTL outputs source insufficient current to reliably drive the TIP120 base resistors at full speed. The 7407 provides open-collector buffering, raising the effective drive current while keeping the logic stage isolated from the power stage. |
| **4 — Power Switching** | TIP120 (Darlington NPN) | Each transistor switches one motor winding to GND. The Darlington configuration (hFE > 1000 typ.) ensures full saturation with the base current available from the 7407 buffer stage. |
| **5 — Flyback Suppression** | 1N4001 (Rectifier Diode) | At turn-off, the collapsing magnetic field in each motor winding generates a counter-EMF spike. The 1N4001 clamps this spike by providing a recirculating path, protecting the TIP120 collector junctions. |

---

## 2. Circuit Architecture

### Component Selection Rationale

| Reference Designator | Component | Key Specification | Design Rationale |
|---|---|---|---|
| U1 | NE555 | Astable, f = 1/(0.693 * C * (R1 + 2R2)) | Industry-standard timer; single supply, wide Vcc range (4.5V–16V), robust output stage. |
| U2 | 74LS194 | 4-bit bi-directional shift register, 20 MHz max | TTL compatible with NE555 output; hardware direction control via S0/S1 mode pins. |
| U3 | 7407 | Hex non-inverting open-collector buffer, 30V/40mA per output | Decouples TTL logic levels from the power-stage base drive; no logic inversion preserves 74LS194 phase sequencing. |
| Q1–Q4 | TIP120 | NPN Darlington, Vce(sat) = 2V, Ic(max) = 5A, hFE > 1000 | Sufficient current gain to switch inductive motor loads; TO-220 package enables heatsinking. |
| D1–D4 | 1N4001 | 50V, 1A rectifier | Standard flyback suppression for inductive loads at the switching frequencies used. |

### Power and Signal Domain Separation

The board operates with two distinct voltage rails:

- **VCC (Logic):** 5V DC, supplied to U1, U2, U3, and pull-up/bias resistors.
- **VMOT (Motor Power):** 5V–12V DC (depending on motor specification), supplied exclusively to the TIP120 collectors and motor winding return paths.

Separating these rails at the connector level prevents switching noise from the power stage from propagating into the sensitive 74LS194 clock and data inputs.

---

## 3. PCB Layout and Routing Strategy

<div align="center">
  <img src="Images/pcb routing.png" width="780" alt="PCB Routing — Front Copper Layer"/>
  <br/>
  <sub><b>Figure 3.</b> Front copper layer (F.Cu) routing. Signal traces in the logic domain are clearly distinguishable from the heavier power traces in the motor drive stage.</sub>
</div>

<br/>

### Trace Width Calculation

Trace widths were determined using the **IPC-2221A** external-conductor current-carrying capacity formula:

```
I = k * dT^0.44 * A^0.725
```

Where:
- `I` = maximum current (A)
- `dT` = allowable temperature rise above ambient (deg C)
- `A` = cross-sectional area (mils sq) = width (mils) x copper thickness (mils)
- `k` = 0.048 for external conductors (1 oz / 35 um copper)

The following width classes were established and applied uniformly:

| Domain | Trace Width | Max Current Capacity | Applied To |
|---|---|---|---|
| **Signal / Logic** | 0.3 mm – 0.5 mm | ~0.5 A – 0.9 A | NE555 RC network, 74LS194 data/clock, 7407 inputs/outputs |
| **Power / Motor** | 1.5 mm | ~2.5 A | TIP120 emitter-to-GND, motor winding supply rails, VMOT distribution |

### Routing Guidelines Applied

- **Signal integrity:** All clock and data lines in the logic domain were routed with minimum stub length. The 74LS194 clock input was given priority routing to minimize capacitive loading.
- **Via minimization:** Signal traces were routed on a single layer where possible to avoid introducing via parasitics in the clock path.
- **Power trace topology:** Star topology was used for power distribution to the four TIP120 collector pads, ensuring equal impedance paths to each motor winding drive point.
- **Pad clearance:** A minimum clearance of 0.2 mm (8 mil) was maintained between all copper objects, with an expanded 0.4 mm clearance enforced between power and signal domains.
- **Thermal relief:** Through-hole pads connected to the bottom-layer copper pour include thermal relief spokes to ensure reliable soldering without excessive heat sinking.
- **Design Rule Check (DRC):** The design was verified DRC-clean under KiCad 8.0 rules prior to Gerber export.

---

## 4. 3D Visualization

The following renders were exported from KiCad 8.0's integrated 3D viewer using the VRML/STEP component models.

<div align="center">

<table>
  <tbody>
    <tr>
      <td align="center">
        <img src="Images/3d view.png" width="300" alt="3D View — Front Isometric"/>
        <br/>
        <sub><b>Figure 4.</b> Front isometric view. Component placement, silk screen annotation, and TIP120 TO-220 package orientation are clearly visible.</sub>
      </td>
      <td align="center">
        <img src="Images/3d view 1.png" width="300" alt="3D View — Top Down"/>
        <br/>
        <sub><b>Figure 5.</b> Top-down view. Illustrates the spatial separation between the logic ICs (left cluster) and the power transistor array (right cluster).</sub>
      </td>
      <td align="center">
        <img src="Images/3d view 2.png" width="300" alt="3D View — Perspective"/>
        <br/>
        <sub><b>Figure 6.</b> Perspective view. Demonstrates board height profile, dominated by the electrolytic capacitor and TIP120 transistors in TO-220 packages.</sub>
      </td>
    </tr>
  </tbody>
</table>

</div>

---

## 5. Bottom Layer and Grounding Strategy

<div align="center">
  <img src="Images/3d back.png" width="780" alt="PCB Bottom Layer — GND Copper Pour"/>
  <br/>
  <sub><b>Figure 7.</b> Bottom copper layer (B.Cu) showing the solid GND copper pour. The pour fills virtually the entire bottom layer, providing a continuous low-impedance ground plane.</sub>
</div>

<br/>

### Copper Pour Implementation

The bottom layer (B.Cu) is defined as a single net fill assigned to the **GND** net. The fill was configured with the following parameters in KiCad:

| Parameter | Value |
|---|---|
| Net | GND |
| Fill Type | Solid |
| Minimum Width | 0.25 mm |
| Pad Connection | Thermal Relief (4 spokes, 0.5 mm spoke width) |
| Island Removal | Remove isolated islands |
| Priority Level | 0 (highest fill priority) |

### Functional Rationale

A solid GND pour on the bottom layer serves three distinct engineering purposes in this design:

1. **Low-Impedance Return Path:** The four TIP120 transistors switch currents up to several amperes through motor windings. A solid ground plane provides a near-zero-inductance return path for these pulsed currents, preventing GND bounce that would corrupt the TTL logic levels at U2 and U3.

2. **Thermal Dissipation:** The TIP120 in TO-220 package dissipates power as P_D = Vce(sat) * Ic. At Vce(sat) approximately 2V and Ic = 1A per winding, each transistor dissipates approximately 2W. The copper pour thermally couples to the PCB substrate and distributes this heat across a larger area, supplementing (but not replacing) any external heatsink attached to the TO-220 tab.

3. **EMI Reduction:** Switching transients from the inductive motor windings generate common-mode noise. The solid copper pour, when connected to protective earth (PE) in the system enclosure, provides a low-impedance path for displacement currents to return to their source, reducing radiated emissions from the PCB.

---

## 6. Fabrication and Manufacturing Specifications

Gerber files and drill files are pre-generated and available in the `Fabrication/` directory, ready for immediate submission to any standard PCB fabrication service.

### Gerber File Manifest

| File | Layer Description |
|---|---|
| `Stepper_Motor_Driver-F_Cu.gbr` | Front copper layer |
| `Stepper_Motor_Driver-B_Cu.gbr` | Back copper layer (GND pour) |
| `Stepper_Motor_Driver-F_Mask.gbr` | Front solder mask |
| `Stepper_Motor_Driver-B_Mask.gbr` | Back solder mask |
| `Stepper_Motor_Driver-F_Silkscreen.gbr` | Front silkscreen (component references) |
| `Stepper_Motor_Driver-B_Silkscreen.gbr` | Back silkscreen |
| `Stepper_Motor_Driver-F_Paste.gbr` | Front solder paste |
| `Stepper_Motor_Driver-B_Paste.gbr` | Back solder paste |
| `Stepper_Motor_Driver-Edge_Cuts.gbr` | Board outline |
| `Stepper_Motor_Driver-PTH.drl` | Plated through-hole drill file |
| `Stepper_Motor_Driver-NPTH.drl` | Non-plated through-hole drill file |
| `Stepper_Motor_Driver-job.gbrjob` | Gerber job file |

### Recommended PCB Fabrication Parameters

| Parameter | Specification |
|---|---|
| **Board Material** | FR4 (Tg 130 deg C minimum) |
| **Board Thickness** | 1.6 mm |
| **Copper Weight** | 1 oz (35 um) — all layers |
| **Layers** | 2 |
| **Minimum Trace Width** | 0.3 mm |
| **Minimum Clearance** | 0.2 mm |
| **Minimum Drill Size** | 0.8 mm |
| **Surface Finish** | HASL (Lead-free) or ENIG |
| **Solder Mask Color** | Green (standard); any color accepted |
| **Silkscreen** | White |
| **PCB Quantity (minimum)** | 5 (typical fab minimum) |

### Generating Gerbers from Source

1. Open `Hardware/Stepper_Motor_Driver.kicad_pro` in KiCad 8.0.
2. Open the PCB Editor (PCBnew).
3. Navigate to **File > Fabrication Outputs > Gerbers (.gbr)**.
4. Set the output directory to `Fabrication/`.
5. Select all layers and enable **Use drill/place file origin**.
6. Click **Plot**, then **Generate Drill Files** from the same dialog.
7. Verify the output against the file manifest above.

---

## 7. Repository Structure

```
Unipolar Stepper Motor Driver/
|
|-- Hardware/                           # KiCad 8.0 source files
|   |-- Stepper_Motor_Driver.kicad_pro  # Project file
|   |-- Stepper_Motor_Driver.kicad_sch  # Schematic
|   |-- Stepper_Motor_Driver.kicad_pcb  # PCB layout
|
|-- Fabrication/                        # Production-ready Gerber and drill files
|   |-- *.gbr                           # Gerber layers
|   |-- *.drl                           # Drill files
|   |-- *-job.gbrjob                    # Gerber job descriptor
|
|-- Images/                             # Documentation images and renders
|   |-- 3dboard view-ai generated.jpeg  # AI-enhanced 3D banner render
|   |-- Schematic From Practical        # Original reference schematic
|       Electronics Book.png
|   |-- schematic diagram.png           # KiCad schematic export
|   |-- pcb routing.png                 # PCB front copper routing
|   |-- 3d view.png                     # KiCad 3D render -- isometric
|   |-- 3d view 1.png                   # KiCad 3D render -- top-down
|   |-- 3d view 2.png                   # KiCad 3D render -- perspective
|   |-- 3d back.png                     # KiCad 3D render -- bottom layer
|
|-- Docs/                               # Additional documentation
|-- README.md                           # This file
|-- CONTRIBUTING.md                     # Contributor guidelines
|-- LICENSE                             # CERN-OHL-S v2 license text
```

---

## 8. Getting Started

### Prerequisites

- [KiCad 8.0](https://www.kicad.org/download/) installed on your system.
- A standard Gerber viewer (e.g., gerbv, KiCad Gerber Viewer) for verifying fabrication files.

### Viewing the Design

```bash
# Clone the repository
git clone https://github.com/<your-username>/unipolar-stepper-motor-driver.git

# Open the KiCad project file
# Navigate to: Hardware/Stepper_Motor_Driver.kicad_pro
# Open with KiCad 8.0
```

### Bill of Materials (BOM) Summary

| Qty | Reference | Part | Package |
|---|---|---|---|
| 1 | U1 | NE555P | DIP-8 |
| 1 | U2 | 74LS194N | DIP-16 |
| 1 | U3 | 7407N | DIP-14 |
| 4 | Q1-Q4 | TIP120 | TO-220 |
| 4 | D1-D4 | 1N4001 | DO-41 |
| 2 | R1, R2 | Resistor (per NE555 design) | Axial |
| 1 | C1 | Capacitor (per NE555 design) | Radial |
| — | — | Decoupling capacitors (100nF per IC) | Radial / Disc |

> Refer to the schematic (`Hardware/Stepper_Motor_Driver.kicad_sch`) for the complete BOM with exact values, net names, and reference designators.

---

## 9. Contributing

Contributions to this project are welcome and encouraged. Please read the [CONTRIBUTING.md](CONTRIBUTING.md) file for detailed guidelines on how to report hardware bugs, propose schematic changes, follow PCB layout conventions, and submit pull requests for KiCad source files.

---

## 10. License

This hardware design is released under the **CERN Open Hardware Licence Version 2 — Strongly Reciprocal (CERN-OHL-S v2)**.

You are free to study, modify, distribute, and manufacture this design, provided that any modifications or derivative works are released under the same license. Commercial use is permitted under these terms.

Full license text is available in the [`LICENSE`](LICENSE) file in this repository.

> CERN-OHL-S v2: [https://ohwr.org/cern_ohl_s_v2.txt](https://ohwr.org/cern_ohl_s_v2.txt)

---

<div align="center">

Designed with KiCad 8.0 | Licensed under CERN-OHL-S v2 | Open Source Hardware

</div>
