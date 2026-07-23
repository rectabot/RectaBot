# RectaBot v1.0 — Product Specification

**Industrial-grade 5-axis CNC controller with integrated network, USB, isolated I/O, and spindle 0-10V interface.**

---

## 📑 Table of Contents

- [Product Overview](#-product-overview)
- [Key Features](#-key-features)
  - [Motion Control](#motion-control)
  - [Spindle Control](#spindle-control-new-in-v10)
  - [Isolated I/O](#isolated-io)
  - [Connectivity](#connectivity)
  - [Power Distribution](#power-distribution)
  - [Safety & EMI](#safety--emi)
- [Connector Pinout](#-connector-pinout)
  - [Power & USB](#power--usb)
  - [Stepper Motors](#stepper-motors-x-y-z-a-b)
  - [Spindle / Laser](#spindle--laser)
  - [Isolated Inputs](#isolated-inputs-24v_iso-domain)
  - [Auxiliary / Communication](#auxiliary--communication)
- [Electrical Specifications](#-electrical-specifications)
  - [Power](#power)
  - [Stepper Outputs](#stepper-outputs-per-axis)
  - [Spindle 0-10V Output](#spindle-0-10v-output)
  - [Spindle EN/DIR Outputs](#spindle-endir-outputs)
  - [Isolated Inputs](#isolated-inputs)
  - [Communication](#communication)
- [Compliance & Reliability](#-compliance--reliability)
- [Compatibility](#-compatibility)
  - [Stepper Drivers](#stepper-drivers)
  - [VFD Compatibility](#vfd-variable-frequency-drives)
  - [Laser Modules](#laser-modules)
  - [CNC Machines](#cnc-machines)
- [Onboard Components Summary](#-onboard-components-summary)
- [Mechanical](#-mechanical)
- [Firmware](#-firmware)
- [In the Box](#-in-the-box)
- [Documentation](#-documentation)
- [Support](#-support)
- [Legal](#-legal)
- [Revision History](#-revision-history)

---

## 📦 Product Overview

RectaBot is a professional CNC mill controller designed for hobby-pro and small workshop applications. Built around the powerful **Raspberry Pi RP2350B** microcontroller running open-source **grblHAL** firmware, it combines industrial-grade isolation, multi-axis stepper control, and modern connectivity in a single 150×100mm PCB.

| Property | Value |
|---|---|
| **Model** | RectaBot v1.0 |
| **Form factor** | 150×100mm 4-layer PCB |
| **MCU** | Raspberry Pi RP2350B (QFN-80, dual ARM Cortex-M33 @ 150MHz) |
| **Firmware** | grblHAL (open-source) |
| **Power input** | 24V DC, 2A max |
| **Operating temperature** | -10°C to +60°C |
| **PCB layers** | 4 (top signal, GND plane, power plane, bottom signal) |
| **Input isolation** | Optical isolation on all 10 field inputs (LTV-217-B-G, 5kV rated, 24V_ISO domain). Outputs share MCU ground in v1. |

---

## 🎯 Key Features

### Motion Control
- ✅ **5-axis stepper control** (X, Y, Z, A, B) via PIO-based step generation
- ✅ **Common Anode** drive topology for DM556 / TB6600 / DM860 driver compatibility
- ✅ **5V logic-level** STEP/DIR/EN signals (via 74HC14D Schmitt trigger buffer)
- ✅ Hardware-accelerated step pulses up to **300kHz per axis**
- ✅ Y-axis auto-squaring (B-axis as ganged Y2)

### Spindle Control (NEW in v1.0)
- ✅ **0-10V analog speed output** for VFD compatibility (LM358 op-amp with trim calibration)
- ✅ **Open-drain Enable & Direction signals** (2N7002 MOSFET, 60V tolerant)
- ✅ **Overvoltage protection** (12V Zener clamp on 0-10V output)
- ✅ Compatible with **Huanyang, Lapond, Delta, ATO, Hitachi** VFDs out-of-the-box
- ✅ PWM range: 0-25kHz, supports laser modulation on shared output
- ✅ Visual brightness LED indicator follows actual VFD output

### Isolated I/O
- ✅ **10 optically isolated inputs** (24V_ISO domain, LTV-217 optocouplers, 5kV component rating)
  - 5× limit switches (X_MIN, Y_MIN, Z_MAX, A_MIN, B_MIN)
  - E-Stop, Probe, Cycle Start, Feed Hold, Door safety
- ✅ **3 SSR-compatible outputs** (Mist M7, Flood M8, Vacuum M62/M63)
- ✅ **24V_ISO isolated supply** (B2424S-2WR3, 2W isolated DC-DC)
- ✅ Hardware debounce (100nF parallel to optocoupler LED)

### Connectivity
- ✅ **Ethernet 10/100 Mbps** via W5500 + RJ45 with integrated magnetics
  - Bob Smith termination for EMI suppression
  - 100Ω diff impedance controlled TX/RX pairs
- ✅ **USB-C 2.0 Full-Speed** for programming, debugging, host control
  - VBUS detection on GP47
  - CC1/CC2 5.1kΩ pull-down (device mode)
  - 22Ω series termination on D+/D-
- ✅ **RS-485 Modbus** for VFD spindle and external I/O expansion (SP3485)
- ✅ **RS-422 Pendant interface** for RectaPad (full duplex, dual SP3485)
- ✅ **MicroSD card** for G-code file storage and configuration

### Power Distribution
- ✅ **24V → 5V buck converter** (TPS5430DDAR, 3A capability)
- ✅ **5V → 3.3V LDO** (AMS1117-3.3, 800mA)
- ✅ **24V_ISO isolated rail** (83mA for opto LED inputs)
- ✅ **5 power LED indicators** (24V, 5V, 3V3, USB, ISO status)
- ✅ **+5V ORing diode** (SS54) — buck 5V and USB-C 5V coexist without back-feed

### Safety & EMI
- ✅ **4-layer stackup** with solid GND plane (L2) for return paths
- ✅ **PSM712 TVS arrays** on RS-485/422 + Bob Smith on RJ45/USB-C shells
- ✅ **Input-side isolation barrier** (2mm copper void) — separates MCU ground from 24V_ISO input domain. Stepper/Spindle/AUX outputs share MCU ground in v1 (full I/O isolation planned for v2).
- ✅ **Failsafe spindle defaults** during MCU reset (open-drain MOSFETs OFF by default)
- ✅ **12V Zener clamp** on spindle 0-10V output (protects VFD from MCU failure)

---

## 🔌 Connector Pinout

### Power & USB
| Connector | Type | Function |
|---|---|---|
| **CN42** | 2-pin screw (3.5mm) | 24V DC input |
| **USBC1** | USB-C | Programming, debug, host control |
| **RJ1** | RJ45 | Ethernet 10/100 Mbps |

### Stepper Motors (X, Y, Z, A, B)
| Connector | Pinout | Function |
|---|---|---|
| **CN34** | X axis (4-pin) | STEP, DIR, EN, GND (5V Common Anode) |
| **CN35** | Y axis (4-pin) | STEP, DIR, EN, GND |
| **CN36** | Z axis (4-pin) | STEP, DIR, EN, GND |
| **CN37** | A axis (4-pin) | STEP, DIR, EN, GND |
| **CN38** | B axis (4-pin) | STEP, DIR, EN, GND (Y2 tandem) |

### Spindle / Laser
| Connector | Pinout | Function |
|---|---|---|
| **CN39** | 4-pin (3.5mm) | VFD_10V, VFD_EN, VFD_DIR, GND |
| **CN33** | 2-pin (3.5mm) | Laser PWM, GND |

### Isolated Inputs (24V_ISO domain)
| Connector | Function |
|---|---|
| **CN22** | DOOR (safety door input, GP42) |
| **CN23** | FEED HOLD |
| **CN24** | CYCLE START |
| **CN25** | PROBE |
| **CN26** | E-STOP |
| **CN27** | B_LIMIT |
| **CN28** | A_LIMIT |
| **CN29** | Z_LIMIT |
| **CN30** | Y_LIMIT |
| **CN31** | X_LIMIT |

### Auxiliary / Communication
| Connector | Function |
|---|---|
| **CN40** | AUX (+5V, VAC, FLOOD, MIST, GND) — 5-pin SSR outputs |
| **CN41** | RS-422 Pendant (24V/TX+/TX−/RX+/RX−/GND) — 6-pin |
| **CN32** | RS-485 Modbus (GND/A+/B-) |
| **CARD1** | MicroSD socket (SPI mode) |

---

## ⚙️ Electrical Specifications

### Power
| Parameter | Min | Typ | Max | Unit |
|---|---|---|---|---|
| Input voltage | 12 | 24 | 30 | V DC |
| Input current (no load) | — | 150 | 250 | mA |
| Input current (typical) | — | 400 | 800 | mA |
| Input current (worst case all outputs active) | — | — | 2000 | mA |
| 5V rail output | 4.9 | 5.0 | 5.1 | V |
| 3.3V rail output | 3.25 | 3.30 | 3.35 | V |
| 24V_ISO rail output | 23 | 24 | 25 | V |
| 24V_ISO max current | — | — | 83 | mA |

### Stepper Outputs (per axis)
| Parameter | Value |
|---|---|
| Logic level (after 74HC14D) | 5V CMOS |
| Output drive current | ±25mA peak per pin |
| STEP pulse minimum width | 1µs |
| STEP frequency max | 300kHz |
| DIR setup time | 5µs (configurable) |

### Spindle 0-10V Output
| Parameter | Value |
|---|---|
| Output voltage range | 0 - 10V DC |
| Output linearity | ±0.5% (after trim calibration) |
| Output impedance | ~1kΩ (series limit + Zener) |
| Output ripple at 25kHz PWM | <5mV |
| Trim adjustment range | Gain 1.5x to 3.0x |
| Overvoltage clamp | 12V (1N4742A Zener) |
| Settling time (0→10V) | <20ms |

### Spindle EN/DIR Outputs
| Parameter | Value |
|---|---|
| Output type | Open-drain (N-MOSFET) |
| Max sink current | 300mA |
| Max applied voltage | 60V DC |
| ON-state Vds | <100mV at 100mA |
| OFF-state leakage | <1µA |
| Failsafe default (MCU reset) | OFF (high-Z) via pull-down |

### Isolated Inputs
| Parameter | Value |
|---|---|
| Input voltage range | 12-30V DC (24V nominal) |
| Input current | ~5mA at 24V |
| Isolation rating | 5kV (LTV-217) |
| Response time (with debounce cap) | <1ms |
| Logic threshold | Optocoupler current ~3mA |

### Communication
| Interface | Speed | Distance | Notes |
|---|---|---|---|
| Ethernet | 100Mbps | 100m | RJ45 with integrated magnetics |
| USB-C | 12Mbps (Full-Speed) | 3m | Device mode, CDC + UF2 |
| RS-485 Modbus | up to 1Mbps | 1200m | Half-duplex, 120Ω termination |
| RS-422 Pendant | up to 1Mbps | 100m | Full-duplex differential |
| SPI (microSD) | 25MHz | <30mm | Mode 0, on-board |

---

## 🛡️ Compliance & Reliability

### EMI/EMC Design
- 4-layer PCB with solid GND plane for low-impedance return paths
- Edge stitching vias every 5mm (Faraday cage effect)
- Diff pair impedance control (USB 90Ω, Ethernet 100Ω, RS-422/485 100Ω)
- TVS protection on all external connectors (RS-485, RS-422, USB shell, Ethernet shell)
- Bob Smith termination on Ethernet RJ45 + USB-C shells

### Protection Features
- ⚠️ Note: the 24V input has **no** reverse-polarity protection in v1 (planned for v2)
- ESD protection on USB data lines (22Ω series + Bob Smith chassis coupling)
- TVS arrays on RS-485/422 lines (PSM712-LF)
- Optical isolation on all field inputs (LTV-217-B-G, 5kV component rating); outputs share MCU ground in v1
- Spindle output overvoltage clamp (12V Zener)
- Open-drain MOSFET protection for EN/DIR (60V tolerance)
- Failsafe spindle OFF during MCU reset/boot

### Quality Standards
- **CE/RoHS compliant** components (Pb-free, halogen-free where available)
- **Industrial-grade** electrolytics and ceramics (X7R dielectric)
- **Operating range:** -10°C to +60°C (component-limited)
- **Storage range:** -25°C to +85°C

---

## 🧰 Compatibility

### Stepper Drivers
- ✅ Leadshine **DM556** (most common)
- ✅ Leadshine **DM860**
- ✅ MakerBase **TB6600**
- ✅ DQ542MA, DQ860MA
- ✅ Any Common Anode optocoupler driver (5V signal)

### VFD (Variable Frequency Drives)
- ✅ **Huanyang** HY01/HY02 (most common, 0-10V + RUN/DIR)
- ✅ **Lapond** SVD-EM series
- ✅ **Delta** VFD-M, VFD-EL
- ✅ **ATO** generic VFDs
- ✅ **Hitachi** WJ200
- ⚠️ **Siemens G120, ABB ACS580** (industrial PNP): requires external relay module
- ✅ Modbus RS-485 spindle control (any VFD with Modbus support)

### Laser Modules
- ✅ Diode laser modules with TTL PWM input (Sculpfun, Atomstack, NEJE)
- ✅ Up to 25kHz PWM modulation rate
- ✅ 3.3V TTL signal level

### CNC Machines
- ✅ Router-style mills (drewno, MDF, aluminijum)
- ✅ Plasma cutters (with appropriate driver interface)
- ✅ Engraving machines
- ✅ XY plotters and pen-plotters
- ⚠️ Large industrial machines: confirm motor driver compatibility

---

## 🔬 Onboard Components Summary

### Critical IC's
| Designator | Part | Function |
|---|---|---|
| U1 | TPS5430DDAR | 24V→5V buck converter (3A) |
| U2 | AMS1117-3.3 | 5V→3.3V LDO (800mA) |
| U3 | B2424S-2WR3 | Isolated 24V DC/DC for I/O domain |
| U4 | RP2350B | Main MCU (dual M33 @ 150MHz, 48 GPIO) |
| U5 | W5500 | Ethernet controller (SPI) |
| U6 | W25Q128JVSIQ | 128Mbit (16MB) QSPI flash |
| U7, U10, U15 | 74HC14D | Stepper level shifter (3.3V→5V) |
| U8-U20 | LTV-217-B-G | Isolated input optocouplers (×10) |
| U19, U21, U22 | SP3485EN-L/TR | RS-485/422 transceivers (×3) |
| U23 | LM358DR2G | 0-10V spindle op-amp |
| Q1, Q2 | 2N7002 | Spindle EN/DIR open-drain buffers |
| RJ1 | J1B1211CCD | Integrated RJ45 + magnetics (WIZnet) |

### Protection & Filtering
- 3× PSM712-LF-T7 TVS arrays (RS-485, RS-422)
- 1× 1N4742A Zener (spindle output clamp)
- 1× SS54 Schottky (+5V ORing)
- 2× SS34 Schottky (buck output)
- 1× 1nF/2kV ceramic (Bob Smith chassis coupling)
- 2× crystal oscillators (12MHz MCU, 25MHz Ethernet)

---

## 📐 Mechanical

| Property | Value |
|---|---|
| PCB dimensions | 150 × 100 mm |
| PCB thickness | 1.6 mm |
| Copper weight | 1oz outer / 0.5oz inner |
| Surface finish | HASL (lead-free) or ENIG |
| Solder mask | Green (default), other colors available |
| Silkscreen | White, with component reference designators |
| Mounting holes | 5× total — 4× M3 (3.2mm drill) at corners + 1× central support hole on the diagonal (3.5mm drill, 7.5mm annular pad) |
| Mounting hole keepout | 6mm clearance |
| Connector spacing | All field connectors on board edges |

### Connector Specifications
- **Screw terminals:** KEFA KF2EDGR 3.5mm pitch (3/4/5/6 pin variants) — TH, soldered manually
- **Ethernet:** WIZnet J1B1211CCD RJ45 with shielded shell + integrated magnetics — TH, soldered manually
- **USB-C:** Korean Hanyong TYPE-C-31-M-12 (24-pin SMD with TH alignment posts) — **SMT, JLCPCB assembly**
- **MicroSD:** Push-push SMD socket — SMT, JLCPCB assembly

---

## 🔧 Firmware

### grblHAL Configuration
- **Board map:** `Firmware/boards/my_machine_map.h`
- **Steps:** configurable per axis via `$100-$104`
- **Max rate:** configurable per axis via `$110-$114`
- **Acceleration:** configurable per axis via `$120-$124`
- **Max spindle RPM:** `$30` (default 24000)
- **Invert masks (validated):** `$2=0` (step — Common Anode needs no invert), `$4` = enable invert on all axes (7/15/31 for 3/4/5-axis), `$5=0` (limit), `$6=1` (probe). `$3` (direction) is per-machine.

### Supported G-code
- G0-G3 (rapid, linear, arc moves)
- G4 (dwell)
- G10 (coordinate system)
- G17-G19 (plane selection)
- G20/G21 (inch/mm)
- G28/G30 (home)
- G38.x (probe)
- G54-G59 (coordinate systems)
- G90/G91 (absolute/relative)
- G92 (set position)
- G93/G94 (inverse/units per min)
- M0-M2 (program flow)
- M3/M4/M5 (spindle on CW/CCW/off)
- M7/M8/M9 (mist, flood, coolant off)
- M30 (program end)
- M62/M63 (auxiliary output control)

### Real-time Commands
- `?` Status query (position, feed, spindle)
- `~` Cycle resume
- `!` Feed hold
- `Ctrl-X` Soft reset

### Communication Modes
- USB CDC (default, virtual serial)
- Ethernet TCP (Telnet on port 23, websocket on port 80)
- Ethernet UDP (broadcast for discovery)

---

## 📦 In the Box

- 1× RectaBot v1.0 PCB (assembled)
- Quick Start Guide
- Connector pinout reference card
- Link to online documentation

### Optional Accessories (separately sold)
- RectaPad (handheld controller with LCD + touch)
- Pre-made wiring harness for common VFDs
- Power supply 24V/5A
- Enclosure (DIN rail or panel mount)

---

## 📚 Documentation

- [Pinout Reference](Pinout.md)
- [Hardware Design Guidelines](Hardware_Design_Guidelines.md)
- [Schematic References](Schematic_References.md)
- [VFD Wiring Guide](VFD_Wiring_Guide.md)

---

## 🆘 Support

- **GitHub:** github.com/rectabot (TBD)
- **Documentation:** docs.rectabot.com (TBD)
- **Community:** Discord "RectaBot Users" (TBD)
- **Email:** support@rectabot.com (TBD)

---

## ⚖️ Legal

- **License (Hardware):** CERN-OHL-S v2 (Strong reciprocal open hardware)
- **License (Firmware):** GPL v3 (grblHAL base)
- **Warranty:** 12 months limited (EU consumer rights)
- **CE marking:** Self-certified per EMC Directive 2014/30/EU (DoC available)
- **RoHS:** Compliant (lead-free components used where available)

---

## 📊 Revision History

| Version | Date | Changes |
|---|---|---|
| **v1.0** | 2026-05-28 | Initial production release with integrated 0-10V spindle interface, 2N7002 EN/DIR protection, Zener clamp |
| v0.9 | 2026-04-20 | Pre-production prototype (no 0-10V, no protection MOSFETs) |
| v0.5 | 2026-03-15 | Schematic complete, layout draft |
| v0.1 | 2026-02-01 | Concept phase |

---

*RectaBot is an open-source project designed and manufactured in Serbia by Filip Perić.*

*Document version 1.0 — last updated 2026-05-28.*
