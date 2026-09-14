# Contributing to the Unipolar Stepper Motor Driver

Thank you for your interest in contributing to this open-source hardware project. This document outlines the standards, workflows, and expectations for contributions to ensure a consistent, high-quality design that meets the engineering and documentation standards of the open-source hardware community.

All contributors are expected to adhere to the technical guidelines described herein. Contributions that do not meet these standards will be requested to be revised before merging.

---

## Table of Contents

1. [Code of Conduct](#1-code-of-conduct)
2. [How to Contribute](#2-how-to-contribute)
3. [Reporting Hardware Bugs](#3-reporting-hardware-bugs)
4. [Proposing Schematic Changes](#4-proposing-schematic-changes)
5. [PCB Layout Guidelines](#5-pcb-layout-guidelines)
6. [Documentation Standards](#6-documentation-standards)
7. [Pull Request Process](#7-pull-request-process)
8. [KiCad File Conventions](#8-kicad-file-conventions)
9. [Licensing](#9-licensing)

---

## 1. Code of Conduct

This project follows the [Contributor Covenant Code of Conduct](https://www.contributor-covenant.org/). By participating, you agree to uphold a respectful, inclusive, and collaborative environment. Disrespectful, harassing, or exclusionary behavior will not be tolerated.

---

## 2. How to Contribute

Contributions to this hardware project can take several forms:

- **Hardware bug reports:** Identifying errors in the schematic or PCB layout.
- **Schematic changes:** Proposing improvements to component selection or circuit topology.
- **PCB layout changes:** Improving routing, footprints, or design-for-manufacturability (DFM).
- **Documentation improvements:** Correcting or expanding the README, Docs/, or inline comments.
- **Fabrication verification:** Reporting manufacturing results, DFM feedback, or assembly notes.

### Workflow Overview

```
Fork repository  -->  Create feature branch  -->  Make changes  -->  Self-verify  -->  Open PR
```

1. **Fork** the repository on GitHub.
2. **Create a branch** from `main` with a descriptive name:
   ```bash
   git checkout -b fix/ne555-frequency-calculation
   git checkout -b feature/add-enable-pin-logic
   ```
3. **Make and verify** your changes according to the guidelines below.
4. **Commit** with a clear, structured commit message (see Section 7).
5. **Open a pull request** against the `main` branch.

---

## 3. Reporting Hardware Bugs

Hardware bugs in a PCB design can range from incorrect net connections in the schematic to DRC violations in the layout, incorrect footprint assignments, or manufacturing-stage failures.

### What Constitutes a Hardware Bug

| Category | Examples |
|---|---|
| **Schematic Error** | Incorrect component pin mapping, missing pull-up/pull-down resistor, wrong power net assignment, ERC violation. |
| **PCB Layout Error** | DRC violation, trace width below specification, insufficient creepage/clearance, missing via, incorrect pad size. |
| **Footprint Mismatch** | Pad pitch does not match physical component datasheet, drill size incompatible with component lead diameter. |
| **Fabrication Issue** | Gerber file missing a layer, incorrect drill origin, non-plated hole assigned to plated net. |
| **Functional Error** | Incorrect commutation sequence behavior, insufficient base drive current, missing flyback diode. |

### How to Report

1. Search existing [GitHub Issues](../../issues) to confirm the bug has not already been reported.
2. Open a new issue using the **Hardware Bug Report** template.
3. Provide the following information in your report:

```
**Component or Net Affected:**
(e.g., Q2 — TIP120, Net: GND, Ref: D3)

**Description of the Bug:**
(Clear, concise description of the observed or suspected problem.)

**Steps to Reproduce / Evidence:**
(Schematic screenshot, DRC report, Gerber layer screenshot, oscilloscope capture, etc.)

**Expected Behavior:**
(What the circuit should do according to the design intent.)

**Observed Behavior:**
(What actually happens, or what the DRC/ERC reports.)

**Severity:**
[ ] Critical — Prevents fabrication or causes circuit damage
[ ] Major — Causes functional failure under expected operating conditions
[ ] Minor — Cosmetic or marginal-condition issue

**KiCad Version:**
**Operating System:**
```

---

## 4. Proposing Schematic Changes

Schematic changes alter the electrical design of the circuit. All such proposals require a detailed technical justification and must preserve the fundamental design intent: a hardware-logic, microcontroller-free four-phase unipolar stepper motor driver.

### Proposal Requirements

Before submitting a PR with schematic modifications, open a **GitHub Discussion** or **Issue** of type "Schematic Change Proposal" that includes:

1. **Motivation:** Why is the change needed? What problem does it solve, or what improvement does it offer?
2. **Affected Nets and Components:** List all reference designators and net names that will be changed.
3. **Electrical Analysis:** Provide supporting calculations or simulation results (LTspice, KiCad Simulator, or equivalent).
4. **Backward Compatibility:** Does the change affect the BOM, footprints, or Gerber output in a way that invalidates existing fabricated boards?
5. **Datasheet References:** Link to or attach relevant datasheet sections for any new or modified components.

### Acceptable Schematic Changes

- Component value adjustments with documented calculation rationale (e.g., NE555 timing resistor change with updated frequency derivation).
- Additional decoupling capacitors with placement rationale.
- Footprint corrections that align pad geometry with component datasheet specifications.
- Addition of optional test points or jumper configurations, provided they do not alter the default circuit behavior.

### Changes Requiring Board-Level Discussion First

- Replacing any of the core ICs (NE555, 74LS194, 7407, TIP120).
- Adding a microcontroller or programmable device of any kind.
- Altering the commutation sequence logic.
- Changing power supply architecture.

---

## 5. PCB Layout Guidelines

All PCB layout changes must comply with the following constraints. Changes that violate these rules will not be accepted.

### 5.1 Trace Width Rules

| Domain | Minimum Width | Maximum Width | Basis |
|---|---|---|---|
| Signal / Logic | 0.3 mm | 0.5 mm | IPC-2221A, < 0.5A, dT = 10 deg C |
| Power / Motor | 1.5 mm | No limit | IPC-2221A, up to 2.5A, dT = 10 deg C |
| GND Fill (B.Cu) | 0.25 mm (fill min width) | Solid fill | Minimum fill width parameter |

Do not reduce power trace widths below 1.5 mm without providing a revised IPC-2221A calculation in the PR description.

### 5.2 Clearance and Creepage

| Rule | Value |
|---|---|
| Minimum copper-to-copper clearance | 0.2 mm |
| Power-to-signal domain clearance | 0.4 mm (recommended) |
| Board edge clearance | 0.5 mm minimum |
| Pad-to-pad clearance (different nets) | 0.2 mm minimum |

### 5.3 Design Rule Check (DRC)

- **All PRs containing PCB layout changes must include a DRC report showing zero errors and zero unconnected nets.**
- To generate a DRC report in KiCad: **Inspect > Design Rules Checker > Run DRC > Save Report**.
- Attach the `.rpt` file to your pull request.
- Warnings are acceptable only if accompanied by a written justification explaining why each warning is benign.

### 5.4 Via Rules

| Parameter | Value |
|---|---|
| Minimum via drill diameter | 0.8 mm |
| Minimum via annular ring | 0.2 mm |
| Via usage in signal domain | Minimize; use only when layer change is unavoidable |
| Via usage in power domain | Acceptable for layer transitions; use multiple vias in parallel for currents above 1A |

### 5.5 GND Copper Pour

- The B.Cu GND pour must be maintained as a solid fill covering the maximum available board area.
- Do not introduce isolated copper islands disconnected from the GND net.
- Thermal relief spokes must be preserved on all through-hole pads connected to the pour.
- Pour minimum width must not be reduced below 0.25 mm.

### 5.6 Component Placement

- Maintain logical grouping: logic-domain ICs (U1, U2, U3) should remain spatially separated from the power-domain components (Q1–Q4, D1–D4).
- Keep the NE555 RC timing network (R1, R2, C1) in close proximity to U1 to minimize stray capacitance on the timing pins.
- Flyback diodes (D1–D4) must be placed as close as possible to the corresponding TIP120 collector-emitter pads.
- Decoupling capacitors (100 nF) must be placed within 2.5 mm of the VCC pin of each IC.

### 5.7 Silkscreen

- All reference designators must remain visible and must not overlap pads or copper features.
- Silkscreen line width must not be reduced below 0.12 mm.
- Board outline must be present on the Edge.Cuts layer as a closed polygon.

---

## 6. Documentation Standards

- All documentation is written in **Markdown with embedded HTML** for layout control.
- Do not use casual language. Maintain an academic, professional tone consistent with the existing README.
- Any new images added to `Images/` must use descriptive filenames in lowercase with hyphens (e.g., `pcb-routing-v2.png`).
- Tables must be used for structured data (component lists, parameter tables, file manifests).
- When referencing a specific schematic net, use code formatting: `` `GND` ``, `` `VMOT` ``.
- When referencing a component reference designator, use bold: **U2**, **Q3**.

---

## 7. Pull Request Process

### Commit Message Format

Use the following format for all commits:

```
<type>(<scope>): <short imperative summary>

<body — optional, explains the why, not the what>

<footer — optional, references issues, breaking changes>
```

**Type values:**

| Type | Use For |
|---|---|
| `fix` | Correcting a hardware bug, schematic error, or layout violation |
| `feat` | Adding a new capability (test point, optional jumper, etc.) |
| `refactor` | Layout optimization without functional change |
| `docs` | Documentation-only changes |
| `fab` | Changes to Fabrication/ output files |
| `chore` | Project file updates, library updates, version bumps |

**Examples:**

```
fix(pcb): increase D3 pad clearance to meet 0.2mm DRC minimum

The pad clearance on D3 was 0.15mm, causing a DRC error when
the design rules file was applied. Increased to 0.2mm.

Closes #14
```

```
feat(schematic): add optional motor-enable jumper on VMOT rail

Adds JP1 to allow the motor power supply to be interrupted
via a jumper without disconnecting the logic VCC rail.
Documented in README Section 2.
```

### PR Checklist

Before requesting review, confirm all of the following:

**Schematic Changes:**
- [ ] ERC (Electrical Rules Check) passes with zero errors in KiCad.
- [ ] All modified nets are listed in the PR description.
- [ ] Datasheet references provided for any new or changed components.
- [ ] Schematic exported as PNG and attached to the PR for visual review.

**PCB Layout Changes:**
- [ ] DRC passes with zero errors and zero unconnected nets. DRC report attached.
- [ ] Trace widths conform to the domain rules in Section 5.1.
- [ ] GND pour has been rebuilt (Edit > Fill All Zones) after layout changes.
- [ ] 3D render exported and attached for visual confirmation.
- [ ] Gerber files in `Fabrication/` have been regenerated.

**Documentation Changes:**
- [ ] Markdown renders correctly (test locally with a Markdown previewer).
- [ ] No emojis. Professional tone maintained throughout.
- [ ] Images referenced in documentation exist in the `Images/` directory.

**All PRs:**
- [ ] Branch is up-to-date with `main`.
- [ ] Commit history is clean (squash fixup commits before requesting review).
- [ ] PR description clearly explains the problem being solved and the approach taken.

### Review Process

1. PRs require at least one approving review from a project maintainer.
2. Hardware layout changes that affect the power domain require review by a maintainer with PCB design experience.
3. The CI/CD pipeline (if configured) must pass before merging.
4. Maintainers may request changes. Respond to all review comments before re-requesting review.
5. Approved PRs are merged using **Squash and Merge** to maintain a clean commit history on `main`.

---

## 8. KiCad File Conventions

### File Format

- All KiCad files must be committed in their native KiCad 8.0 format (`.kicad_sch`, `.kicad_pcb`, `.kicad_pro`).
- Do not commit KiCad backup files (`*.kicad_sch-bak`, `*.kicad_pcb-bak`). These are excluded by `.gitignore`.
- Do not commit KiCad lock files (`*.lck`).

### Library Management

- Prefer KiCad's built-in standard symbol and footprint libraries over external or project-local libraries where possible.
- If a custom symbol or footprint is required, it must be included in the repository under a `Libraries/` directory (create if it does not exist) and documented in the PR.
- Custom symbols must include accurate pin numbers, pin types, and reference prefix. Custom footprints must match the component datasheet's recommended land pattern.

### Schematic Conventions

- All power symbols must use KiCad's built-in power flags (`PWR_FLAG`) on VCC and GND nets to suppress ERC "pin not connected" warnings on power nets.
- Use hierarchical net labels for all inter-sheet connections if the schematic is ever expanded to multiple sheets.
- Reference designators must be annotated (no `U?`, `R?`, etc.) and must not conflict within the design.

### PCB Conventions

- The board origin (drill/place file origin) must be set to the lower-left corner of the board outline.
- All footprints must be locked after final placement (`Right-click > Lock`) to prevent accidental movement.
- The Edge.Cuts board outline must be a single closed polyline with zero gaps.

---

## 9. Licensing

By submitting a contribution to this repository, you agree that your contribution will be licensed under the **CERN Open Hardware Licence Version 2 — Strongly Reciprocal (CERN-OHL-S v2)**, the same license that governs the existing project. You confirm that you have the legal right to make the contribution under these terms.

Full license text: [LICENSE](LICENSE)

---

<div align="center">

Thank you for contributing to open-source hardware.

</div>
