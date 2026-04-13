<div align="center">
<img src="https://github.com/hiperiondev/CLASS-E_DESIGN_CALCULATOR/raw/main/images/logo.png" width="500">

# Class-E RF Power Amplifier Design Calculator

> **Excel/LibreOffice Calc spreadsheet for optimum Class-E switching amplifier design and harmonic Low-Pass Filter calculation — based on Sokal/Raab equations (WA1HQC · QEX Jan/Feb 2001)**

**Author:** Emiliano Gonzalez LU3VEA — lu3vea@gmail.com  
**License:** GPL v3  
**File:** `ClassE_Amplifier_Calculator.xlsx`

---
</div>

## Table of Contents

1. [What Is a Class-E Amplifier?](#1-what-is-a-class-e-amplifier)
2. [Why This Calculator?](#2-why-this-calculator)
3. [Spreadsheet Structure](#3-spreadsheet-structure)
4. [Theory Background](#4-theory-background)
   - [Class-E Operating Principle](#41-class-e-operating-principle)
   - [ZVS and ZDVS Conditions](#42-zvs-and-zdvs-conditions)
   - [Sokal/Raab Design Equations](#43-sokalraab-design-equations)
   - [Low-Pass Filter Theory](#44-low-pass-filter-theory)
5. [Quick Start](#5-quick-start)
6. [Input Parameters Explained](#6-input-parameters-explained)
7. [Output / Calculated Values](#7-output--calculated-values)
8. [Sheet 1 — Class-E Calculator](#8-sheet-1--class-e-calculator)
9. [Sheet 2 — LPF Direct 5-poles](#9-sheet-2--lpf-direct-5-poles)
10. [Sheet 3 — LPF Direct 7-poles](#10-sheet-3--lpf-direct-7-poles)
11. [Sheet 4 — MOSFET Reference Table](#11-sheet-4--mosfet-reference-table)
12. [Iterative Calculation: Why & How](#12-iterative-calculation-why--how)
13. [Component Selection Guidelines](#13-component-selection-guidelines)
    - [C1 — Shunt Capacitor](#131-c1--shunt-capacitor)
    - [C2 — Series Resonant Capacitor](#132-c2--series-resonant-capacitor)
    - [L2 — Series Resonant Inductor](#133-l2--series-resonant-inductor)
    - [RFC — RF Choke](#134-rfc--rf-choke)
    - [LPF Components](#135-lpf-components)
14. [MOSFET Selection Guide](#14-mosfet-selection-guide)
15. [Regulatory Compliance (Harmonics)](#15-regulatory-compliance-harmonics)
16. [Simulation Workflow (LTspice)](#16-simulation-workflow-ltspice)
17. [Known Limitations](#17-known-limitations)
18. [References](#18-references)
19. [Licence](#19-licence)

---

## 1. What Is a Class-E Amplifier?

A **Class-E power amplifier** is a highly efficient single-ended **switching (non-linear) RF power amplifier** topology. It was invented by Nathan O. Sokal (WA1HQC) and Alan D. Sokal, first described in their landmark 1975 paper in the *IEEE Journal of Solid-State Circuits* ("Class E — A New Class of High-Efficiency Tuned Single-Ended Switching Power Amplifiers"), and later refined with practical design equations in Sokal's 2001 *QEX* article.

Unlike linear amplifiers (Class A, AB, B), where the active device operates in its linear region and dissipates significant power, a Class-E amplifier operates its transistor as a **hard switch** — the device is either fully ON or fully OFF. Under ideal optimum (ZVS + ZDVS) conditions, the switch voltage is zero at turn-on and the voltage slope is also zero at that instant, meaning **no energy stored in the output capacitor is wastefully dissipated**. This makes theoretical drain efficiency approach **100%**, with practical efficiencies of 80–95% routinely achievable in HF QRP designs.

Key characteristics:
- Optimum switching topology (ZVS + ZDVS)
- Theoretical efficiency → 100%; practical: 80–95%
- Peak drain voltage ≈ 3.56 × Vcc (must be within MOSFET V_DSS rating)
- Best suited for single-frequency CW/WSPR/data transmitters
- Widely used in QRP HF ham radio beacons (WSPR, QRSS, CW) from 136 kHz through VHF

---

## 2. Why This Calculator?

Designing a Class-E amplifier from scratch requires solving a set of coupled nonlinear equations involving:

- Output power, supply voltage, operating frequency, and loaded Q
- Component values (C1, C2, L2, RFC) that depend on efficiency η
- Efficiency η that depends on the component values and ESR losses
- MOSFET parasitic capacitance (Coss) that absorbs part of the required C1

This spreadsheet automates all of those calculations and adds:

- **Iterative convergence** of the η ↔ I_dc circular dependency
- **Coss subtraction** so you know the exact *external* capacitor to add at the drain
- **Std 5% E-series rounding** for every component value
- **Voltage and current rating warnings** for the MOSFET and LPF capacitors
- **RFC self-resonance and saturation checks**
- **A full asymmetric 5-element Chebyshev LPF** designed at the correct source impedance R_opt (not 50 Ω) — Sheet 2
- **A full asymmetric 7-element Chebyshev LPF** for stricter harmonic rejection (≥−65 dBc at 2f) — Sheet 3
- **A curated MOSFET reference table** covering popular HF/VHF/VLF devices

---

## 3. Spreadsheet Structure

The workbook contains **four sheets**:

| Sheet | Purpose |
|---|---|
| **Class-E Calculator** | Main design sheet. Enter parameters here; get all component values. |
| **LPF Direct 5-poles** | 5-element Chebyshev LPF designed at the actual drain impedance R_opt. Linked automatically to the main sheet. |
| **LPF Direct 7-poles** | 7-element Chebyshev LPF designed at R_opt for ≥14 dB additional stopband rejection vs. the 5-pole design. Linked automatically to the main sheet. |
| **MOSFET Reference Table** | Datasheet parameters for 16 common MOSFETs used in HF Class-E designs. |

---

## 4. Theory Background

### 4.1 Class-E Operating Principle

The classic Class-E amplifier (Sokal Fig. 2) consists of:

```
Vcc ──── RFC ──────────────────────┬──── L2 ──── C2 ──── R (load)
                                   │ ← Drain node
                             ┌─────┤
                            [SW]   C1
                             │     │
                            GND   GND
```

> **Note:** `SW` (the MOSFET) and `C1` are both connected between the **Drain node** and GND. `RFC` connects Vcc to the Drain node in series only — it does **not** appear as a shunt component.

- **RFC** (RF choke): presents very high impedance at the operating frequency so the DC supply current flows with minimal RF ripple
- **C1** (shunt capacitor, drain to GND): shapes the drain voltage waveform; absorbs MOSFET Coss
- **L2–C2** (series resonant network): forms a bandpass that passes only the fundamental to the load while filtering harmonics
- **R** (optimum load resistance): calculated from Vcc, P_out, and η; differs from antenna impedance (50 Ω) — hence the need for a matching network or LPF impedance transformation

### 4.2 ZVS and ZDVS Conditions

For **Zero Voltage Switching (ZVS)**, the transistor must turn on when its drain-to-source voltage is exactly zero:

```
V_ds(t_on) = 0
```

For **Zero Derivative Voltage Switching (ZDVS)** — also written ZDS or ZVDS in some literature — the slope of the drain voltage must also be zero at turn-on:

```
dV_ds/dt|_(t_on) = 0
```

When both conditions are met simultaneously (optimum Class-E), the transistor dissipates no power when switching, yielding maximum efficiency. Any deviation from the optimum (wrong component values, frequency error, load mismatch) violates ZVS/ZDVS and rapidly degrades efficiency.

### 4.3 Sokal/Raab Design Equations

The calculator implements the **Sokal AACD 2001 closed-form equations** (QL-corrected polynomial fits to Table I numerical solutions). The key equations are:

**Optimum Load Resistance (Eq. 6a, 3rd-order polynomial, ±0.01%):**

```
R = (Veff² / P_out) × 0.576801 × (1.0000086 − 0.414395/QL − 0.577501/QL² + 0.205967/QL³)
```

where `Veff = Vcc − Vsat` (effective voltage swing).

**C1 — Shunt Capacitor (Eq. 7, QL-corrected):**

```
C1 = [QL-poly / (5.4466 × ω × R)] + 0.6 / (ω² × L_RFC)
```

where `QL-poly` is the QL-dependent polynomial correction factor from Sokal Eq. 7. The `+0.6/(ω²·L_RFC)` term is the RFC correction — it is dynamically linked to the RFC value in the sheet, not a fixed offset.

**C2 — Series Resonant Capacitor (Eq. 9, rational fit ±0.072%):**

```
C2 × ω × R = [1/(QL − 0.104823)] × [1.00121 + 1.01468/(QL − 1.7879)]
```

Valid for QL ≥ 1.7879. RFC correction −0.2/(ω²·L_RFC) applied.

**L2 — Series Resonant Inductor (Eq. 10):**

```
L2 = QL × R / ω
```

This is the physical coil to wind. `L_ser = L2 − 1/(ω²·C2)` is the net inductive excess of the L2-C2 branch — it is **informational only** and not a separate component.

**Efficiency Model (Sokal Eq. 2, extended for real component losses):**

```
η = R / [R + ESR_L2 + ESR_C2 + 1.365 × Ron + 0.2116 × ESR_C1 + (ESR_RFC × I_dc²)/P_out]
```

η feeds back into I_dc = P_out/(Vcc × η), creating the circular dependency resolved by iterative calculation.

### 4.4 Low-Pass Filter Theory

#### Why a LPF is needed

A Class-E amplifier, by nature of its hard-switching operation, generates **rich harmonic content** at the drain. Without a LPF, the harmonics would radiate from the antenna, violating FCC Part 97 (−43 dBc limit) and IARU recommendations (−50 dBc). A well-designed LPF passes the fundamental with minimal insertion loss (<0.1–0.5 dB) while attenuating harmonics to compliant levels.

#### Chebyshev vs Butterworth

A **Chebyshev (Type I, equiripple) LPF** is preferred over Butterworth for harmonic suppression because:

- It achieves a steeper roll-off for the same filter order
- It allows a small controlled passband ripple (0.1 dB in this design) to gain significantly more stopband attenuation
- For a 5-element design, Chebyshev 0.1 dB provides ≈10–15 dB more rejection at 2f than the same-order Butterworth

The Chebyshev prototype g-values for 5-element, 0.1 dB ripple are:

```
g1 = 1.1468,  g2 = 1.3712,  g3 = 1.9750,  g4 = 1.3712,  g5 = 1.1468
```

#### Why standard 50 Ω LPF tables are WRONG for direct drain connection

A textbook Chebyshev LPF designed for 50 Ω source and 50 Ω load works correctly **only when the source impedance actually is 50 Ω**. In a Class-E amplifier connected directly to the drain, the source impedance is **R_opt** (typically 5–50 Ω), not 50 Ω. The source-side shunt capacitor must be:

```
C_source = g2 / (R_opt × ωc)          [CORRECT]
C_source = g2 / (50Ω × ωc)            [WRONG — makes C ≈2× too small, loses 10–20 dB at 2f]
```

This is why the **LPF Direct (R_opt)** sheet computes an **asymmetric ladder** with Z_source = R_opt and Z_load = 50 Ω, using the geometric mean reference impedance `Z0_eff = √(R_opt × Z_load)` for the series inductors and independent scaling for each shunt capacitor.

The cutoff frequency is set to **fc = 1.40 × f** to keep fundamental insertion loss below 0.1 dB while providing useful harmonic rejection at 2f.

---

## 5. Quick Start

### Step 1 — Enable Iterative Calculation

The spreadsheet uses a small circular reference (η → I_dc → RFC loss → η) that requires iterative calculation to converge.

**Excel:**
> File → Options → Formulas → Enable Iterative Calculation  
> Set Maximum Iterations: **100**, Maximum Change: **0.0001**

**LibreOffice Calc:**
> Tools → Options → LibreOffice Calc → Calculate  
> Enable: Iterations, set to **100** iterations, Minimum Change: **0.0001**

### Step 2 — Enter Your Parameters

In the **Class-E Calculator** sheet, edit the blue **INPUT PARAMETERS** cells:

| Cell | Parameter | Example |
|---|---|---|
| Vcc | Supply voltage (V) | 5 |
| P_out | Desired output power (W) | 0.4 |
| f | Operating frequency (MHz) | 7 |
| Q_L | Loaded Q factor | 5 |
| Vo | Transistor Vsat (0 for MOSFET) | 0 |
| ESR_L2 | L2 winding ESR (Ω) | 0.05 |
| ESR_C2 | C2 series ESR (Ω) | 0.01 |
| ESR_C1 | C1 series ESR (Ω) | 0.01 |
| ESR_RFC | RFC winding ESR (Ω) | 0.1 |
| Z_out | Antenna/load impedance (Ω) | 50 |
| Ciss | MOSFET Ciss from datasheet (pF) | 60 |
| Coss | MOSFET Coss from datasheet (pF) | 12 |
| C_oss scale | Bias-point scale factor | 0.4 |

> **Note on ESR_C1 and ESR_C2:** These capacitor ESR values feed directly into the Sokal efficiency formula (Section 4.3). For NP0/C0G capacitors, typical values are 0.01–0.05 Ω and are often negligible. For X7R or other lossy dielectrics, ESR can reach 0.1–0.5 Ω and must be measured. Setting them to 0 is acceptable for initial estimates with low-ESR capacitors.

### Step 3 — Converge the Iteration

Press **F9** repeatedly (F9 recalculates the whole workbook; Shift+F9 recalculates only the active sheet) until the convergence indicator in cell C6 shows:

```
✅ Iterative calc CONVERGED — values stable
```

### Step 4 — Read the Component Values

The **CLASS-E COMPONENT VALUES** section gives you:
- C1_ext — external shunt capacitor to add (after subtracting Coss)
- C2 — series resonant capacitor
- L2 — series resonant inductor (one coil only)
- RFC — RF choke minimum inductance
- Nearest E12/E24 5% standard values for each

### Step 5 — Read the LPF Values

Switch to the **LPF Direct (R_opt)** sheet (Sheet 2) for the 5-element Chebyshev LPF, or the **LPF Direct 7-pole** sheet (Sheet 3) for higher harmonic rejection. Each sheet provides:
- L1, L3, L5 (and L7 for the 7-pole) — series inductors
- C2, C4 (and C6 for the 7-pole) — shunt capacitors
- Estimated harmonic attenuation at 2f and 3f

### Step 6 — Simulate

Before building, simulate in **LTspice** (free, from Analog Devices). Verify:
- ZVS achieved (V_drain → 0 at switch turn-on)
- Drain peak voltage within V_DSS / safety_factor
- Output power matches target
- Harmonic attenuation meets regulatory requirements

---

## 6. Input Parameters Explained

### Supply Voltage (Vcc)
DC supply voltage in volts. Typical QRP designs use 5–13.8 V. Higher Vcc reduces I_dc but increases peak drain voltage (V_pk ≈ 3.56 × Vcc). Ensure your MOSFET V_DSS exceeds V_pk × safety factor (typically 3×).

### Output Power (P_out)
Target RF power delivered to the load, in watts. This is the power at the antenna connector after the LPF. Note that the MOSFET sees higher dissipation when η < 1.

### Operating Frequency (f)
Fundamental transmit frequency in MHz. For HF QRP, typical values are 1.8, 3.5, 7, 10, 14, 18, 21, 24, 28 MHz (amateur bands). For VLF/LF/MF experimentation: 0.136, 0.472 MHz.

### Loaded Q Factor (Q_L)
The Q factor of the L2–C2 series resonator, defined as:
```
Q_L = 2π·f·L2 / R
```
Choosing Q_L involves trade-offs:
- **Higher Q_L (10–15):** Better harmonic rejection from the tank alone, narrower bandwidth, more critical tuning, higher L2 and lower C2 values, more sensitive to component tolerances
- **Lower Q_L (3–5):** Broader bandwidth, less harmonic filtering, easier to build, more tolerant of frequency variation
- **Recommended for HF (3–14 MHz):** Q_L = 5–10

### Transistor Vsat (Vo)
The saturation voltage of the switching device:
- **MOSFET:** 0 V (ideal switch, Rdson loss modelled separately in η)
- **BJT:** 0.1–1 V (collector-emitter saturation voltage)

### ESR_L2
The winding series resistance of the L2 inductor, measured at the operating frequency with an LCR meter or VNA. This parameter enters the Sokal efficiency equation with coefficient 1.0 — errors here directly corrupt η, R, C1, C2, and I_dc. Typical values:
- Small air-core solenoid: 0.03–0.08 Ω
- Wound toroid (HF): 0.05–0.15 Ω

### ESR_C2 and ESR_C1
The series resistance of the resonant capacitor C2 and shunt capacitor C1, respectively. These values enter the Sokal efficiency equation directly (ESR_C2 with coefficient 1.0; ESR_C1 with coefficient 0.2116). For NP0/C0G or silver-mica types, ESR is typically 0.01–0.05 Ω and can be left at 0.01 for initial design. Lossy dielectrics (X7R, Z5U) can have ESR of 0.1–0.5 Ω — measure with VNA if efficiency accuracy matters.

### ESR_RFC
The winding series resistance of the RF choke. RFC loss is approximately `ESR_RFC × I_dc²`. Typical values:
- Air-core toroid: 0.05–0.1 Ω
- Wound ferrite (HF): 0.1–0.5 Ω
- Small ferrite pot cores: up to 1 Ω

### Coss Scale Factor
MOSFET output capacitance (Coss) is highly **bias-dependent** — it decreases sharply at higher Vds. The scale factor corrects the datasheet value (measured at a fixed Vds, typically 25 V) to the effective value at Vds ≈ 0.5 × Vcc:

| Device Type | Recommended Scale Factor |
|---|---|
| Si planar (BS170, 2N7000) | 0.40 |
| Si vertical DMOS (IRF510, IRFZ44N) | 0.35 |
| RF LDMOS (RD16HVF1, BLF574) | 0.85 (use Coss,er from datasheet if available) |
| Unknown / conservative | 0.25 |

For best accuracy, measure Coss with a VNA at Vds ≈ 0.5 × Vcc, or use the manufacturer's energy-equivalent Coss,er if provided.

### V_DSS Safety Factor
Multiplier applied to the nominal peak drain voltage to determine the minimum required MOSFET V_DSS:
```
V_DSS_min = V_pk_nominal × safety_factor
```
Recommendations:
- **2.0×** — steady-state optimum-tuned operation only
- **3.0×** — HF bench work with some tolerance for mistuning
- **4.0×** — severe mistuning or VLF high-voltage designs

Note: Under severe mistuning, V_pk can reach 5× Vcc. The sheet includes a worst-case check for this condition.

---

## 7. Output / Calculated Values

### Angular Frequency ω
```
ω = 2π × f   [rad/s]
```
Used internally in all component formulas.

### Optimum Load Resistance R
The Class-E drain load impedance for which the circuit achieves ZVS + ZCS. This is **not** the antenna impedance (50 Ω) — it is the correct source impedance for the LPF. Typical: 5–50 Ω for HF QRP.

### DC Supply Current I_dc
```
I_dc = P_out / (Vcc × η)   [A]
```
Average current drawn from the supply. Depends on the converged η.

### C1_ext — External Shunt Capacitor
This is the capacitor you actually need to **purchase and solder** at the drain:
```
C1_ext = C1_total − Coss_effective
```
where `Coss_effective = Coss_datasheet × scale_factor`. If C1_ext is negative, the MOSFET's own Coss already exceeds the required C1 — this is a common situation with larger vertical DMOS devices.

### L2 — Series Resonant Inductor
Wind **one coil only** with this inductance value. The sheet may also show `L_ser` and `L_total` rows — these are informational only and equal L2. Do **not** wind separate coils for L_ser.

### RFC — RF Choke
Minimum choke inductance is:
```
RFC_min = 30 × R / ω
```
The sheet uses `50 × R / ω` in practice for margin. Critical checks:
- **SRF (self-resonant frequency) must be > 10× operating frequency** — at HF, a large RFC can self-resonate within the band and act as a capacitor
- **Core saturation** — verify the core does not saturate at I_dc

---

## 8. Sheet 1 — Class-E Calculator

This is the main design sheet. The layout is divided into sections:

| Section | Description |
|---|---|
| **Header** | Convergence status indicator and iterative calculation instructions |
| **MOSFET MODEL** | Selected device name and V_DSS / I_D compliance check |
| **INPUT PARAMETERS** | User-editable cells (blue): Vcc, P_out, f, Q_L, ESR values, MOSFET data |
| **INTERMEDIATE CALCULATIONS** | ω, R, I_dc, V_eff — internal values |
| **CLASS-E COMPONENT VALUES** | C1, C2, L2, RFC, L_ser — all with units, standard values, and notes |
| **PERFORMANCE / EFFICIENCY** | η, P_dissipated, V_pk, I_pk, thermal estimates |
| **MOSFET RATING CHECKS** | V_DSS, I_D compliance with safety factors |
| **LPF (50 Ω, main sheet)** | Optional L-network + 5-element LPF for use when not using the LPF Direct sheet |

> **Note:** The main sheet also provides an L-network impedance transformer (from R_opt to 50 Ω) followed by a 50 Ω symmetric Chebyshev LPF. This is an alternative to the **LPF Direct (R_opt)** sheet — the two approaches should not be cascaded in series. Choose one or the other.

---

## 9. Sheet 2 — LPF Direct 5-poles

This sheet computes the **correct** 5-element Chebyshev LPF when the filter is connected **directly to the drain**, with no intermediate L-network. All parameters are automatically linked from the main sheet.

### Key design decisions

| Parameter | Value | Reason |
|---|---|---|
| Filter type | Chebyshev Type I | Best stopband attenuation per element |
| Ripple | 0.1 dB | Minimal passband insertion loss |
| Order | 5 elements | Good harmonic rejection; practical to build |
| Cutoff fc | 1.40 × f | Keeps fundamental ≤0.1 dB insertion loss |
| Source Z | R_opt | Correct for direct drain connection |
| Load Z | 50 Ω | Standard antenna impedance |

### Component equations

```
L1 = g1 × Z0_eff / ωc           [g1 = 1.1468]
C2 = g2 / (R_opt × ωc)          [g2 = 1.3712, uses R_opt — NOT 50Ω]
L3 = g3 × Z0_eff / ωc           [g3 = 1.9750]
C4 = g4 / (Z_load × ωc)         [g4 = 1.3712, uses Z_load = 50Ω]
L5 = g5 × Z0_eff / ωc           [g5 = 1.1468, by symmetry L5 = L1]

Z0_eff = √(R_opt × Z_load)       [geometric mean reference impedance]
```

Note C2 ≠ C4 when R_opt ≠ Z_load (asymmetric terminations).

### Performance estimates

The sheet provides estimated harmonic attenuation at 2f and 3f. These estimates assume an **ideal R_opt source** — in practice, the actual rejection will differ slightly depending on MOSFET output impedance at harmonic frequencies. Verify with LTspice simulation before final build.

If the 5-element LPF fails the regulatory requirement (FCC −43 dBc, IARU −50 dBc), switch to the **LPF Direct 7-pole** sheet (Sheet 3), which uses a 7-element Chebyshev filter for ≥14 dB additional stopband rejection, or lower the cutoff frequency fc.

---

## 10. Sheet 3 — LPF Direct 7-poles

This sheet computes a **7-element Chebyshev LPF** directly coupled to the drain, using R_opt as the source impedance. It provides approximately 14 dB more stopband rejection than the 5-pole design, targeting −65 dBc or better at 2f for strict IARU compliance. All parameters are automatically linked from the main sheet.

### Key design decisions

| Parameter | Value | Reason |
|---|---|---|
| Filter type | Chebyshev Type I | Best stopband attenuation per element |
| Ripple | 0.1 dB | Minimal passband insertion loss |
| Order | 7 elements | Superior harmonic rejection; recommended for IARU −50 dBc compliance |
| Cutoff fc | 1.40 × f | Keeps fundamental ≤0.1 dB insertion loss |
| Source Z | R_opt | Correct for direct drain connection |
| Load Z | 50 Ω | Standard antenna impedance |

### Component equations

```
L1 = g1 × Z0_eff / ωc           [g1 = 1.1812]
C2 = g2 / (R_opt × ωc)          [g2 = 1.4228, uses R_opt — NOT 50Ω]
L3 = g3 × Z0_eff / ωc           [g3 = 2.0967]
C4 = g4 / (Z0_eff × ωc)         [g4 = 1.5734, uses Z0_eff (geometric mean)]
L5 = g5 × Z0_eff / ωc           [g5 = 2.0967, by symmetry L5 = L3]
C6 = g6 / (Z_load × ωc)         [g6 = 1.4228, uses Z_load = 50Ω]
L7 = g7 × Z0_eff / ωc           [g7 = 1.1812, by symmetry L7 = L1]

Z0_eff = √(R_opt × Z_load)       [geometric mean reference impedance]
```

Note: C2 ≠ C4 ≠ C6 due to asymmetric terminations. The centre shunt capacitor C4 uses Z0_eff (geometric mean), unlike the 5-pole design where only source and load impedances are used.

### Performance estimates

The sheet provides estimated harmonic attenuation at 2f and 3f. These estimates assume an **ideal R_opt source** — verify with LTspice simulation. The 7-pole design targets −65 dBc or better at 2f and is the recommended choice when strict IARU −50 dBc compliance must be met with margin.

⚠ **Asymmetric filter warning**: Attenuation formulas assume equal source termination at R_opt. For impedance ratios n = Z_load/R_opt > 1.5, actual stopband depth may differ ±3–6 dB from Chebyshev predictions. Verify with LTspice.

> **Choosing between 5-pole and 7-pole:** Use the 5-pole (Sheet 2) for most QRP designs. Use the 7-pole (Sheet 3) when the 5-pole sheet flags 🚨 non-compliance, when stricter IARU margins are required, or when operating near 2f.

---

## 11. Sheet 4 — MOSFET Reference Table

A quick-reference table of 16 MOSFETs commonly used in HF Class-E designs. Columns:

| Column | Description |
|---|---|
| Part No. | Device name |
| Package | Physical package |
| V_DSS | Drain-source breakdown voltage (V) |
| I_D | Drain current (A) |
| R_ds(on) | On-resistance (Ω) |
| C_iss / C_oss | Input/output capacitance (pF) — Vds-dependent |
| P_D | Package power dissipation (W) |
| f_T | Unity gain frequency (MHz) |
| Typical P_RF | Achievable RF output power (W) |
| RF Freq Range | Suitable operating frequency range |
| Applications | Notes on typical use cases |
| Coss Scale | Recommended bias-point scale factor for this design |
| Device Type | Si-planar / Si-DMOS / LDMOS |

### Device selection highlights

| Application | Recommended Device |
|---|---|
| QRP WSPR beacon, 20m–80m, <0.5W | BS170 (TO-92), 2N7000 |
| QRP CW, all HF bands, 1–30W | IRF510 (TO-220) — the classic ham MOSFET |
| Medium power, 40–80m, 10–80W | IRF520, IRF530 |
| High power, 160m–40m, 50–150W | IRF540, IRFP250 |
| VLF/LF/MF (136 kHz, 472 kHz) | IRFZ44N, IXFN55N50 |
| RF-optimised HF/VHF, 10m–40m | RD16HVF1 (LDMOS) |
| Professional HF, 100–300W | BLF574, MRF101AN (LDMOS) |

> ⚠ **VN66AF is obsolete** (discontinued ~2010). Replace with BS170 or 2N7000 in new designs.

---

## 12. Iterative Calculation: Why & How

### The circular dependency

Three quantities form a feedback loop:

```
η (efficiency) → I_dc = P_out/(Vcc·η) → RFC_loss = ESR_RFC × I_dc² → η
```

This circular reference cannot be solved in a single pass — it requires iteration. The other components (R, C1, C2, L2, RFC) depend only on fixed inputs and are **not** part of this loop; they are valid from the first calculation pass.

### How to converge

1. Open the spreadsheet and enable iterative calculation (see Quick Start)
2. Enter your parameters
3. Press **F9** (recalculate) once or twice
4. Check the convergence indicator in cell C6:
   - `✅ Iterative calc CONVERGED — values stable` → done
   - Still showing the old value or not converging → keep pressing F9
5. For very high ESR_RFC or unusual operating points, convergence may require 3–5 F9 presses

### Why it converges

The RFC loss term `ESR_RFC × I_dc²` is typically a small fraction of P_out for well-designed HF QRP circuits. The feedback gain is much less than 1, so the iteration always converges. If it does not converge, check that ESR_RFC is not unrealistically large.

---

## 13. Component Selection Guidelines

### 13.1 C1 — Shunt Capacitor

- **Type:** NP0/C0G ceramic or silver-mica
- **Voltage rating:** ≥2× V_pk (i.e. ≥ 2 × 3.56 × Vcc); use 100 V minimum for HF QRP
- **Value:** Use standard E12/E24 value nearest to C1_ext. Slight deviation from optimum shifts the ZVS point but does not catastrophically fail the circuit
- **Note:** The MOSFET's Coss (scaled) absorbs part of the required C1. Install only C1_ext as an external component

### 13.2 C2 — Series Resonant Capacitor

- **Type:** Polypropylene film (WIMA FKP, MKP) or silver-mica
- **Voltage rating:** ≥2× V_pk — this capacitor sees the full drain voltage swing
- **Stability:** Use C0G/NP0 or polypropylene; avoid X7R/Z5U which drift with temperature and cause frequency instability
- **Value:** Nearest 5% E-series value; small deviations can be absorbed by slightly adjusting L2

### 13.3 L2 — Series Resonant Inductor

- **Core:** T68-6 (yellow, mix 6) or T50-6 iron powder toroid for 7–30 MHz; T68-2 (red) for 1.8–10 MHz
- **Wire:** Use appropriate AWG for I_rms (see LPF sheet for current rating guidance)
- **Winding:** Wind as a close-wound solenoid or on toroid; leave a small gap between start and finish
- **SRF check:** Self-resonant frequency must be >10× operating frequency
- **Verification:** Measure finished inductance with LCR meter or VNA at operating frequency before installation; trim by spreading/compressing turns

### 13.4 RFC — RF Choke

- **Critical requirement:** SRF > 10× operating frequency (e.g. >70 MHz for a 7 MHz design)
- **Core:** Ferrite mix 43 or 61 works well at HF; avoid iron powder for large RFC values (too lossy)
- **Winding Q:** Aim for unloaded Q > 100 at operating frequency
- **Saturation:** Core must not saturate at the DC bias current I_dc; check manufacturer's AL values
- **Practical rule:** RFC reactance must be ≥ 30× R; this spreadsheet uses 50× R for margin

### 13.5 LPF Components

- **Inductors (L1, L3, L5 for 5-pole; L1, L3, L5, L7 for 7-pole):** T50-6 or T68-6 toroids; verify SRF > 10×f; wind to calculated µH value
- **Capacitors (C2, C4):** NP0/C0G or silver-mica rated ≥ 2× V_pk (≥ 100 V for 5–15 V QRP designs)
- **Lead dress:** Keep leads short; separate input and output physically to avoid coupling
- **Shielding:** For best harmonic rejection, house the LPF in a tinplate enclosure

---

## 14. MOSFET Selection Guide

### Minimum V_DSS
```
V_DSS_required = V_pk_nominal × safety_factor = 3.56 × Vcc × 3.0
```
For Vcc = 5 V: V_DSS_required ≈ 53 V → BS170 (60 V) is marginal; IRF510 (100 V) is safe.

### Gate drive requirements
- Gate capacitance (Ciss) must be charged/discharged completely within the switch transition time
- For MOSFET drivers, ensure the gate drive voltage exceeds V_gs(th) by a comfortable margin (2–3×)
- Use a gate damping resistor (R_gate = 10–47 Ω) to prevent oscillation in the gate loop

### MOSFET fT vs operating frequency
For efficient switching, the MOSFET's unity gain frequency fT should be ≥ 10× the operating frequency. Devices like IRFZ44N (fT ≈ 30 MHz) are not suitable above 3 MHz. LDMOS devices (fT > 500 MHz) work well to 50 MHz and beyond.

---

## 15. Regulatory Compliance (Harmonics)

| Regulation | Harmonic limit |
|---|---|
| FCC Part 97 (USA) | −43 dBc for P > 5W; −40 dBc for P ≤ 5W |
| IARU recommendation | −50 dBc |
| CEPT/EU (amateur) | −43 dBc minimum; −50 dBc recommended |

A 5-element Chebyshev LPF provides approximately:
- 2nd harmonic (2f): −17 to −25 dBc (depends on R_opt, fc choice)
- 3rd harmonic (3f): −38 to −55 dBc

For compliance at high power or strict IARU compliance, consider:
1. Switching to the **7-element Chebyshev LPF** (Sheet 3 — LPF Direct 7-pole), which provides ≥14 dB additional rejection
2. Lowering fc from 1.40×f to 1.25×f (increases fundamental insertion loss slightly)
3. Adding a separate **harmonic trap** (series LC to GND tuned to 2f or 3f)
4. Verifying with a **spectrum analyser** — always measure before transmitting

---

## 16. Simulation Workflow (LTspice)

LTspice (free from Analog Devices: https://www.analog.com/en/resources/design-tools-and-calculators/ltspice-simulator.html) is the recommended tool for pre-build validation. *(Note: if the URL above redirects, search "LTspice download Analog Devices" to find the current download page.)*

### Basic simulation model

1. Create a voltage source `V1 = PULSE(0 {Vgs_on} 0 {tr} {tf} {Ton} {T})` for the gate drive
2. Use an ideal switch (`SW`) or MOSFET SPICE model from the manufacturer
3. Add Coss as a fixed capacitor in parallel with the switch drain-source
4. Use the computed C1_ext, C2, L2, RFC from the spreadsheet as starting values
5. Run `.tran` simulation for ≥20 RF cycles to reach steady state
6. Plot V(drain) and I(drain) — verify ZVS (V_drain → 0 at switch turn-on)
7. Add the LPF and run `.fourier` or `.meas` to check harmonic levels

### Tuning in simulation

- If V_drain does not reach zero before turn-on: increase C1 slightly
- If V_drain goes below zero: decrease C1 slightly
- Fine-tune L2 ±5–10% to optimise ZVS and output power simultaneously
- Match output power to target by adjusting C2 around the calculated value

---

## 17. Known Limitations

- The **efficiency model** is based on the Sokal Eq. 2 linear ESR model. Nonlinear loss mechanisms (core hysteresis, skin effect at VHF, Coss loss) are partially accounted for via the Coss loss model (`p_coss = 0.5 × Coss_eff × V_pk² × f × k_coss`) but not fully modelled
- The **harmonic attenuation estimates** in the LPF sheet assume an ideal resistive source at R_opt. Real MOSFET output impedance at harmonic frequencies is reactive and different from R_opt — always verify with simulation or measurement
- The **5-element LPF** may not achieve FCC/IARU compliance at all operating points. The sheet flags non-compliance with 🚨 warnings. A 7-element design is recommended for strict compliance
- **Coss is Vds-dependent**. The scale factor approach is an approximation. For best accuracy, use manufacturer-specified Coss,er (energy-equivalent) or measure at Vds ≈ 0.5×Vcc
- The calculator assumes **D = 0.5 (50% duty cycle)**, which is required for the Sokal equations. Non-50% duty cycle designs require different equations not implemented here
- The calculator is valid for **QL ≥ 1.7879** (lower bound of Sokal Table I rational fit). QL < 1.79 is outside the valid design space for optimum Class-E
- The **ZVS/ZDVS conditions** can be violated at high impedance ratios (Z_load/R_opt > 10). Always verify with LTspice simulation when R_opt is very low (< 5 Ω)

---

## 18. References

1. **Sokal, N. O.** — "Class-E RF Power Amplifiers," *QEX Magazine*, No. 204, Jan/Feb 2001, pp. 9–20. American Radio Relay League. *(Primary reference for all design equations in this calculator)*

2. **Sokal, N. O.** — "Class-E High-Efficiency RF/Microwave Power Amplifiers: Principles of Operation, Design Procedures, and Experimental Verification," in *Analog Circuit Design* (AACD 2001), Kluwer Academic, 2002. Available: https://people.eecs.berkeley.edu/~culler/AIIT/papers/radio/Sokal%20AACD5-poweramps.pdf

3. **Sokal, N. O. and Sokal, A. D.** — "Class E — A New Class of High-Efficiency Tuned Single-Ended Switching Power Amplifiers," *IEEE Journal of Solid-State Circuits*, Vol. SC-10, No. 3, pp. 168–176, June 1975. *(Original patent/publication of the Class-E topology)*

4. **Raab, F. H.** — "Idealized Operation of the Class E Tuned Power Amplifier," *IEEE Transactions on Circuits and Systems*, Vol. CAS-24, No. 12, pp. 725–735, December 1977.

5. **Williams, A. B. and Taylor, F. J.** — *Electronic Filter Design Handbook*, 4th ed., McGraw-Hill, 2006. *(Source for asymmetric Chebyshev LPF g-values and ladder scaling)*

6. **Fajardo, A. and de Sousa, F. R.** — "Design of the Class-E Power Amplifier with Finite DC Feed Inductance under Maximum-Rating Constraints," *Applied Sciences*, Vol. 11, No. 9, 2021. DOI: 10.3390/app11093727. Available: https://www.mdpi.com/2076-3417/11/9/3727

7. **Dobbs, G. (G3RJV)** — "A Short Guide to Harmonic Filters for QRP Transmitter Output," GQRP Club. Available: https://www.gqrp.com/harmonic_filters.pdf

8. **VK1SV Class-E for Beginners** — Practical Class-E design tutorial with LTspice examples. Available: https://people.physics.anu.edu.au/~dxt103/class-e/

9. **VK2ZAY Online Class-E Calculator** — Web-based calculator based on Sokal equations. Available: https://people.physics.anu.edu.au/~dxt103/calculators/class-e.php

10. **RF Cafe — Chebyshev Prototype Element Values** — Tables of normalised Chebyshev g-values. Available: https://www.rfcafe.com/references/electrical/cheby-proto-values.htm

11. **Engineering LibreTexts — Chebyshev Lowpass Approximation** — Theoretical background. Available: https://eng.libretexts.org/Bookshelves/Electrical_Engineering/Electronics/Microwave_and_RF_Design_IV:_Modules_(Steer)/02:_Filters/2.05:_The_Chebyshev_Lowpass_Approximation

12. **Kazimierczuk, M. K.** — *RF Power Amplifiers*, 2nd ed., Wiley, 2015. *(Comprehensive textbook covering Class-E theory, ZVS/ZDVS conditions, and MOSFET parasitic effects at RF frequencies)*

13. **Kee, S. D., Aoki, I., Hajimiri, A., and Rutledge, D.** — "The Class-E/F Family of ZVS Switching Amplifiers," *IEEE Transactions on Microwave Theory and Techniques*, Vol. 51, No. 6, pp. 1677–1690, June 2003. *(Extends the Class-E theory to Class-E/F topologies; useful background for harmonic tuning)*

14. **Kazimierczuk, M. K. and Puczko, K.** — "Exact Analysis of Class E Tuned Power Amplifier at any Q and Switch Duty Cycle," *IEEE Transactions on Circuits and Systems*, Vol. CAS-34, No. 2, pp. 149–159, February 1987. *(Exact closed-form analysis without the infinite-Q assumption; useful when QL < 5)*

15. **Wetherhold, E. (W3NQN)** — "Second-Harmonic-Optimized Low-Pass Filters," *QST*, February 1999, pp. 44–48. American Radio Relay League. *(Introduces the Chebyshev-with-a-zero CWAZ topology; context for choosing standard 50 Ω vs. direct-drain LPF approaches)*

---

## 19. Licence

This spreadsheet and associated documentation are released under the **GNU General Public Licence v3 (GPL v3)**.

```
Copyright (C) LU3VEA — lu3vea@gmail.com

This program is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the
GNU General Public License for more details.
```

Full licence text: https://www.gnu.org/licenses/gpl-3.0.html

---

*73 de LU3VEA*
