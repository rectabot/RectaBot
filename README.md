# RectaBot — Industrial 5-Axis CNC Controller

**Industrial-grade open-source CNC controller za hobi i mali workshop. Bazirano na Raspberry Pi RP2350B mikrokontroleru sa grblHAL firmware-om.**

---

## 📊 Status projekta

| | |
|---|---|
| **Trenutna verzija** | v1.0 |
| **Status** | 🔵 Hardware u proizvodnji (JLCPCB) — bring-up sredinom juna 2026. |
| **Prva production batch** | 5 ploča (JLCPCB Economic PCBA + ENIG finish) |
| **Firmware** | grblHAL fork za RP2350B |
| **Form factor** | 150 × 100 mm, 4-layer PCB, 1.6mm |

---

## 🎯 Šta je RectaBot?

5-axis CNC kontroler dizajniran za hobi-pro i workshop primene, sa industry-grade galvanskom izolacijom i bogatom konektivnošću. Direktna alternativa skupim proprietary kontrolerima poput Centroid Acorn, Mach3 hardvera, ili LinuxCNC sa skupim Mesa karticama.

### Glavne karakteristike

- ⚙️ **5-axis stepper kontrola** (X, Y, Z, A, B) — PIO-based step generation do **300kHz po osi**
- 🔌 **Spindle 0-10V interface** sa overvoltage zaštitom — kompatibilno sa Huanyang/Lapond/Delta/Hitachi VFD-ovima
- 🛡️ **Optička izolacija ulaza** — svih 10 field ulaza ide preko LTV-217-B-G optokaplera na zasebnom 24V_ISO rail-u (full I/O izolacija planirana za v2)
- 🌐 **Ethernet 10/100 Mbps** preko W5500 sa Bob Smith terminacijom
- 💾 **MicroSD slot** za G-code fajlove i konfiguraciju
- 🔋 **24V napajanje** sa reverznom polaritetnom zaštitom (SS54 Schottky)
- 📡 **RS-485 Modbus + RS-422 Pendant** interfejsi
- 🚦 **10 izolovanih ulaza** + **3 SSR izlaza** (Mist/Flood/Vac)

**Detaljna specifikacija:** [Docs/Product_Specification_v1.0.md](Docs/Product_Specification_v1.0.md)

---

## 📂 Struktura repozitorijuma

```
RectaBot/
├── Docs/                        # 📚 Sva dokumentacija (čitati prvo!)
│   ├── Product_Specification_v1.0.md   # Kompletan datasheet/spec
│   ├── Quick_Start_Guide.md            # 🆕 Od neraspakovanog do prvog motora (2-3h)
│   ├── First_Power_On_Procedure.md     # 🆕 Bezbedan first power-on sa multimetrom
│   ├── Pinout.md                       # RP2350B pinmap (GP0-GP47)
│   ├── Schematic_References.md         # Referentne šeme za sve IC-eve
│   ├── Hardware_Design_Guidelines.md   # PCB layout smernice
│   ├── VFD_Wiring_Guide.md             # Povezivanje sa VFD-ovima
│   ├── Hand_Solder_Components.md       # 28 TH komponenti za ručno lemljenje
│   ├── Silkscreen_Layout_Guide.md      # Smernice za silkscreen markings
│   ├── BOM_v1.0_final.csv              # Bill of Materials (~80 komponenti)
│   ├── CPL_RectaBot_V1.0.csv           # Component Placement List za JLCPCB
│   └── LCSC_Additional_Order.csv       # TH parts za zasebnu LCSC porudžbinu
│
├── Firmware/                    # 🔧 grblHAL fork za RP2350B
│   ├── boards/my_machine_map.h         # Mapiranje pinova za RectaBot v1
│   ├── release/                        # 🆕 Build + Flash + Config dokumentacija
│   │   ├── README.md                   #   - Overview release-a
│   │   ├── BUILD.md                    #   - Kako kompajlovati firmware
│   │   ├── FLASH.md                    #   - Procedure za flashovanje preko USB
│   │   ├── CONFIG.md                   #   - grblHAL $$ parametri (sva 5 osa)
│   │   └── sample_config.txt           #   - Plain text za copy-paste u serial
│   └── ...                             # Standard grblHAL struktura
│
├── SmartPendant/                # 🎮 Sledeći projekat: ručni pendant
│   └── ...                             # RP2350 + LCD + enkoderi (u razvoju)
│
└── Hardware/                    # 🛠️ KiCad/EasyEDA fajlovi (planirano za open-source)
```

---

## 🔧 Hardver

### Centralne komponente

| Komponenta | Uloga |
|---|---|
| **Raspberry Pi RP2350B** (QFN-80) | Dual ARM Cortex-M33 @ 150MHz, 48 GPIO, 12 PIO state machines |
| **WIZnet W5500** | Hardver TCP/IP stack, 10/100 Ethernet |
| **74HC14D ×3** | Schmitt-trigger inverteri za 3.3V→5V level shift (svi step/dir/en signali) |
| **LTV-217-B-G ×10** | Optoizolatori za sve ulazne signale (limit switches, ESTOP, probe, controls) |
| **LM358DR2G** | Op-amp za PWM→0-10V spindle interface |
| **2N7002 ×2** | Open-drain MOSFET-i za VFD EN/DIR signale |
| **SP3485EN-L/TR ×3** | 3.3V native RS-485/RS-422 transceiver-i (1 Modbus + 2 Pendant) |
| **B2424S-2WR3** | Izolovani 24V→24V DC/DC za opto LED napajanje |

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

> **📦 Status:** Hardware tek treba da stigne (~14-16. jun 2026). Dokumentacija je spremna i čeka prvi bring-up za finaliziranje screenshot-ova i edge case-ova.

**Glavni vodiči:**
- **[Quick_Start_Guide.md](Docs/Quick_Start_Guide.md)** — Kompletan vodič od neraspakovanog do prvog motora (procena: 2-3h)
- **[First_Power_On_Procedure.md](Docs/First_Power_On_Procedure.md)** — Bezbednosna procedura za prvo uključivanje sa multimetrom

**Glavni koraci (skraćeno):**
1. Vizuelna inspekcija ploče (10 min)
2. Hand-soldering 28 TH komponenti ([lista](Docs/Hand_Solder_Components.md), 60-90 min)
3. Pre-Power-On continuity testovi (15 min)
4. Prvi Power-On + LED + voltage verifikacija (10 min)
5. Firmware flash preko BOOTSEL+USB (15 min)
6. grblHAL konfiguracija (`$$` parametri za RectaBot, 20 min)
7. Prvi axis test sa DM556 driver-om (10 min)
8. VFD povezivanje — vidi [VFD Wiring Guide](Docs/VFD_Wiring_Guide.md)

---

## 🤝 Acknowledgments

- **[grblHAL](https://github.com/grblHAL)** zajednica — za neverovatan open-source firmware
- **Raspberry Pi Foundation** — za RP2350 chip i pristupačnu cenu
- **JLCPCB & LCSC** — za pristupačnu prototype manufacturing
- **EasyEDA** — za CAD alat koji je omogućio ovaj projekat solo developeru

---

## 📜 License

License will be added before public release. Hardware design files (KiCad/EasyEDA) and firmware konfiguracija planiraju se kao **open hardware/open source**.

---

## 📬 Contact

**Project lead:** Filip Perić
**Email:** kingort@gmail.com
**Location:** Kragujevac, Srbija

---

*🤖 Ovaj README je deo solo-founder projekta. Više informacija o pojedinim aspektima dizajna naći ćeš u [Docs/](Docs/) folderu.*
