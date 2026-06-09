# RectaBot — 5-Axis CNC Controller

Open-source 5-axis CNC controller based on the Raspberry Pi RP2350B microcontroller running grblHAL firmware. Designed for hobby and workshop applications.

---

## 📊 Project Status

| | |
|---|---|
| **Current version** | v1.0 |
| **Status** | 🔵 Hardware in production (JLCPCB) — bring-up mid-June 2026. |
| **First production batch** | 5 boards (JLCPCB Economic PCBA + ENIG finish) |
| **Firmware** | grblHAL fork for RP2350B |
| **Form factor** | 150 × 100 mm, 4-layer PCB, 1.6mm |

---

## 🎯 What is RectaBot?

A 5-axis CNC controller with optical input isolation, integrated Ethernet, and a VFD interface.

### Key features

- ⚙️ **5-axis stepper control** (X, Y, Z, A, B) — PIO-based step generation up to **300kHz per axis**
- 🔌 **Spindle 0-10V interface** with overvoltage protection — compatible with Huanyang/Lapond/Delta/Hitachi VFDs
- 🛡️ **Optically isolated inputs** — all 10 field inputs through LTV-217-B-G optocouplers on a dedicated 24V_ISO rail (full I/O isolation planned for v2)
- 🌐 **Ethernet 10/100 Mbps** via W5500 with Bob Smith termination
- 💾 **MicroSD slot** for G-code files and configuration
- 🔋 **24V power input** with reverse polarity protection (SS54 Schottky)
- 📡 **RS-485 Modbus + RS-422 Pendant** interfaces
- 🚦 **10 isolated inputs** + **3 SSR outputs** (Mist/Flood/Vac)

**Detailed specification:** [Docs/Product_Specification_v1.0.md](Docs/Product_Specification_v1.0.md)

---

## 📂 Repository structure

```
RectaBot/
├── Docs/                                # 📚 All documentation (read first!)
│   ├── Product_Specification_v1.0.md    # Complete datasheet/spec
│   ├── Quick_Start_Guide.md             # From unboxing to first motor (2-3h)
│   ├── First_Power_On_Procedure.md      # Safe first power-on with a multimeter
│   ├── Pinout.md                        # RP2350B pinmap (GP0-GP47)
│   ├── Schematic_References.md          # Reference schematics for all ICs
│   ├── Hardware_Design_Guidelines.md    # PCB layout guidelines
│   ├── VFD_Wiring_Guide.md              # VFD connection guide
│   ├── Hand_Solder_Components.md        # 28 TH components for hand-soldering
│   ├── BOM_v1.0_final.csv               # Bill of Materials (~80 components)
│   ├── CPL_RectaBot_V1.0.csv            # Component Placement List for JLCPCB
│   ├── LCSC_Additional_Order.csv        # TH parts for a separate LCSC order
│   ├── Gerber_RectaBot_1.0/             # Production-ready Gerber files
│   ├── Brand_Assets/                    # Logo files (SVG) under CERN-OHL-S
│   └── configurator/                    # Web-based grblHAL settings generator
│
└── (Submodules live in separate repos — see below)
```

## 🔗 Related repositories

RectaBot is split across separate GitHub repos for better modularity:

| Repo | Contents |
|---|---|
| **[rectabot/RectaBot](https://github.com/rectabot/RectaBot)** (this one) | Hardware design (Gerber), docs, BOM, brand assets, configurator tool |
| **[rectabot/RectaBot-firmware](https://github.com/rectabot/RectaBot-firmware)** | grblHAL fork with RectaBot board map + pre-built UF2 |
| **[rectabot/RectaPad](https://github.com/rectabot/RectaPad)** | Touchscreen pendant firmware (LVGL on RP2350) |

**Clone everything at once:**
```powershell
git clone https://github.com/rectabot/RectaBot.git
git clone https://github.com/rectabot/RectaBot-firmware.git
git clone https://github.com/rectabot/RectaPad.git
```

---

## 🔧 Hardware

### Core components

| Component | Role |
|---|---|
| **Raspberry Pi RP2350B** (QFN-80) | Dual ARM Cortex-M33 @ 150MHz, 48 GPIO, 12 PIO state machines |
| **WIZnet W5500** | Hardware TCP/IP stack, 10/100 Ethernet |
| **74HC14D ×3** | Schmitt-trigger inverters for 3.3V→5V level shift (all step/dir/en signals) |
| **LTV-217-B-G ×10** | Optocouplers for all input signals (limit switches, ESTOP, probe, controls) |
| **LM358DR2G** | Op-amp for PWM→0-10V spindle interface |
| **2N7002 ×2** | Open-drain MOSFETs for VFD EN/DIR signals |
| **SP3485EN-L/TR ×3** | 3.3V native RS-485/RS-422 transceivers (1 Modbus + 2 Pendant) |
| **B2424S-2WR3** | Isolated 24V→24V DC/DC for opto LED supply |

### Power Tree

```
24V DC IN ──┬──► SS54 (reverse protection) ──┬──► TPS5430 buck ──► +5V (3A)
            │                                 │
            ├──► B2424S-2WR3 isolation ──► +24V_ISO (opto LEDs)
            │
            └──► LM358 (op-amp supply, 24V rail-to-rail)

+5V ──► AMS1117-3.3 LDO ──► +3.3V (MCU, W5500, SP3485)
```

### System Block Diagram

```
                       ┌─────────────────────────────────────┐
                       │     RectaBot v1.0 — RP2350B MCU     │
                       │   (Dual Cortex-M33 @ 150MHz)        │
                       └─────────────────────────────────────┘
                                       │
       ┌───────────────────────┬───────┴───────┬───────────────────────┐
       │                       │               │                       │
       ▼                       ▼               ▼                       ▼
┌──────────────┐        ┌──────────────┐ ┌──────────────┐       ┌──────────────┐
│  MOTION      │        │  SPINDLE     │ │  AUX (SSR)   │       │  I/O & COMM  │
│              │        │              │ │              │       │              │
│ PIO step gen │        │ PWM @ 25kHz  │ │ GP0-2 →      │       │ SPI0 W5500   │
│ GP8-22 →     │        │ GP27 →       │ │ 74HC14D      │       │ UART1 RS485  │
│ 74HC14D ×3   │        │ LM358 op-amp │ │ Mist/Flood/  │       │ UART0 RS422  │
│ (Schmitt inv)│        │ + 2N7002 ×2  │ │ Vacuum       │       │ SPI1 microSD │
└──────┬───────┘        └──────┬───────┘ └──────┬───────┘       └──────┬───────┘
       │                       │                │                      │
       ▼                       ▼                ▼                      ▼
┌──────────────┐        ┌──────────────┐ ┌──────────────┐       ┌──────────────┐
│ CN34-CN38    │        │ CN39 (VFD)   │ │ CN40 (AUX)   │       │ RJ45 + USB-C │
│ 5× Stepper   │        │ 0-10V analog │ │ Mist/Flood/  │       │ + CN22 RS485 │
│ STEP/DIR/EN  │        │ EN/DIR open- │ │ Vacuum +     │       │ + CN41 RS422 │
│ (5V CMOS)    │        │ drain MOSFET │ │ 5V/VAC/GND   │       │ + microSD    │
└──────────────┘        └──────────────┘ └──────────────┘       └──────────────┘

     ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ INPUT-SIDE ISOLATION (10× OPTO) ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─

                              ┌─────────────────────┐
                              │  10× ISOLATED       │
                              │  INPUTS             │
                              │                     │
                              │ LTV-217-B-G opto    │
                              │ + 24V_ISO domain    │
                              │ (B2424S-2WR3 DC/DC) │
                              │                     │
                              │ Limits, ESTOP,      │
                              │ Probe, Controls     │
                              │                     │
                              │ → CN23-CN32         │
                              └─────────────────────┘

Note: v1 isolates input side only. Full I/O galvanic isolation is planned for v2.
```

---

## 🚀 Quick Start

> **📦 Status:** Hardware is yet to arrive (~14-16 June 2026). Documentation is ready and awaits the first bring-up to finalize screenshots and edge cases.

**Main guides:**
- **[Quick_Start_Guide.md](Docs/Quick_Start_Guide.md)** — Complete walkthrough from unboxing to first motor (estimated: 2-3h)
- **[First_Power_On_Procedure.md](Docs/First_Power_On_Procedure.md)** — Safety procedure for the first power-on with a multimeter

**Main steps (summary):**
1. Visual inspection of the board (10 min)
2. Hand-soldering 28 TH components ([list](Docs/Hand_Solder_Components.md), 60-90 min)
3. Pre-Power-On continuity tests (15 min)
4. First Power-On + LED + voltage verification (10 min)
5. Firmware flash via BOOTSEL+USB (15 min)
6. grblHAL configuration (`$$` parameters for RectaBot, 20 min) — use [Docs/configurator/](Docs/configurator/) to generate
7. First axis test with a DM556 driver (10 min)
8. VFD wiring — see [VFD Wiring Guide](Docs/VFD_Wiring_Guide.md)

---

## 🤝 Acknowledgments

- **[grblHAL](https://github.com/grblHAL)** community — for the incredible open-source firmware
- **Raspberry Pi Foundation** — for the RP2350 chip and accessible pricing
- **JLCPCB & LCSC** — for accessible prototype manufacturing
- **EasyEDA** — for the CAD tool that made this project possible for a solo developer

---

## 📜 License

- **Code, documentation:** MIT (see [LICENSE](LICENSE))
- **Hardware design files** (Gerber, BOM, CPL): CERN-OHL-S v2 (see [LICENSE.hardware](LICENSE.hardware))
- **Brand assets** (logo SVGs): CERN-OHL-S v2 (with derivative-work caveat — see [Docs/Brand_Assets/README.md](Docs/Brand_Assets/README.md))
- **grblHAL firmware fork:** GPL-3.0 (inherits from upstream, in the separate [RectaBot-firmware](https://github.com/rectabot/RectaBot-firmware) repo)

---

## 📬 Contact

**Project lead:** Filip Perić
**Email:** hello@rectabot.org
**Location:** Kragujevac, Serbia
