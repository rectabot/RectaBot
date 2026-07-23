# Reference Schematics and Design Notes (RectaBot)

This document collects guidelines and reference schematics for the key IC components from our `BOM.csv`. We focus on maximizing the use of passive SMD components (mostly 0603 and 0805) from the JLCPCB "Basic" catalog, since they are essentially free to assemble.

## 1. Power rail (24V → 5V): TPS5430DDAR
**Datasheet:** [TI TPS5430DDAR](https://www.ti.com/lit/ds/symlink/tps5430.pdf)
The TPS5430 requires a few key passive components to operate as a stable "Buck" (step-down) converter.

**Reference connections:**
- **VIN (Pin 7)**: connects to 24V. Requires a ceramic cap to GND (C_in), typically `10uF / 50V` (0805).
- **EN (Pin 3)**: can be left open (floating) for automatic start, or connected to VIN through a 100k resistor.
- **PH (Pin 8)**: power switch output.
  - A **Schottky diode (e.g. SS34 or B340A)** runs from GND to PH (cathode on PH).
  - The **inductor (L1)** is 33 µH to 47 µH (must handle 3A+, CD75 package or larger).
- **BOOT (Pin 1)**: cap between BOOT and PH pins. Standard value: `10 nF / 50V` (0603).
- **VOUT (after inductor L1)**: requires output filtering, i.e. a `220 uF` (electrolytic) cap in parallel with a `10uF` ceramic.
- **VSENSE (Pin 4)**: feed the output voltage via a divider.
  - Formula: `Vout = 1.221V * (R1/R2 + 1)`.
  - For a 5.0V output: **R1 = 10 kΩ**, **R2 = 3.24 kΩ** (1% precision).

*BOM passives:* C_in and C_out components are easy to find in the JLCPCB Basic catalog. The 10k (0603) and 3.24k (0603 1%) resistors are also Basic.

## 2. LDO (5V → 3.3V): AMS1117-3.3
**Datasheet:** [AMS1117](http://www.advanced-monolithic.com/pdf/ds1117.pdf)
Trivial and ultra-cheap!
- **IN**: +5V (from the TPS5430). Requires a `10uF` cap to GND.
- **OUT**: +3.3V. Requires a `22uF` tantalum or `10uF` ceramic cap to GND for stability.
- **GND**: ground.

## 3. Axis buffers (3.3V → 5V): 74HC14D
RectaBot v1.0 uses the **74HC14D** (Schmitt-trigger HEX INVERTER) instead of the 74HCT245. Reasons: Basic on JLCPCB, the clean Schmitt input filters noise, smaller SOIC-14 package.

**Datasheet:** [74HC14D](https://assets.nexperia.com/documents/data-sheet/74HC_HCT14.pdf)
- **VCC (Pin 14)**: +5V (add a `100nF` decoupler very close to the chip).
- **GND (Pin 7)**: ground.
- **Inputs 1A, 2A, ..., 6A**: connected to MCU GP signals (3.3V TTL). The 74HC14D inputs accept 3.3V as logic 1 because the Schmitt threshold (VT+ ~3V at VCC=5V) can be crossed with 3.3V. **NOTE:** strictly speaking, the 74HC14D wants VIH≥3.5V at VCC=5V; in practice 3.3V works reliably, but for maximum safety the 74**HCT**14D (TTL-compatible, VIH=2V) can be used.
- **Outputs 1Y, 2Y, ..., 6Y**: INVERTED 5V output → goes to STEP-/DIR-/EN- pins of the DM556 (through a 330Ω series resistor). The DM556 common wire goes to the **+5V** rail (Common Anode topology).
- **Three 74HC14D chips** (U7, U10, U15) cover 18 signals: 5×STEP + 5×DIR + 5×EN + 3×coolant (MIST, FLOOD, VAC).
- **Firmware settings (validated on the reference board):** the 74HC14D inverts, but with Common Anode drivers the STEP pulse lands correctly with no invert. In grblHAL set:
  - `$2=0` — STEP invert OFF (Common Anode already accounts for the inversion; `$2`≠0 holds the opto on at idle → missed steps)
  - `$4=15` — ENABLE invert on all axes (7 = 3-axis, 15 = 4-axis, 31 = 5-axis)
  - `$5=0` — LIMIT invert OFF · `$6=1` — PROBE invert ON
  - `$3` — DIRECTION invert is **per-machine** (set from how each axis actually moves), not a fixed value
  - Coolant invert is controlled via `MIST_INVERT` and `FLOOD_INVERT` in the driver configuration

## 4. Ethernet controller: WIZnet W5500
**Datasheet:** [W5500](https://www.wiznet.io/wp-content/uploads/wiznethome/Chip/W5500/Document/W5500_ds_v109e.pdf)
The Ethernet section requires slightly more careful routing of the MDI/MDO lines.
- It typically uses a 25 MHz crystal (e.g. HC49/US, often Extended on JLCPCB).
- **RJ45 connector**: the easiest path is a connector with integrated magnetics (e.g. HanRun HR911105A), which makes the job much simpler — i.e. you don't have to spread components around the board for the isolation transformer.
- **Power**: it runs at 3.3V; place `100nF` decoupling caps around every VCC pin on the chip! The W5500 draws up to 150mA under full load.
- **Reference schematic**: follow Figure 29 (page 61) in the official W5500 datasheet linked above ("W5500 Reference Schematic with RJ45"). Copy the passives from there: `49.9 Ohm` and `33 Ohm` resistors that are JLCPCB `Basic`.

## 5. Communication: SP3485EN-L/TR (RS485 for VFD and RS422 for the Pendant)
RectaBot v1.0 uses the **SP3485EN-L/TR** (MaxLinear) — a 3.3V RS-485/RS-422 transceiver. The advantage over the MAX485: it runs directly at 3.3V (no level shift needed), low power, up to 10 Mbps. **Datasheet:** [SP3485](https://www.maxlinear.com/document/2098)

**RS485 (Half Duplex for VFD/Modbus)** — 1× SP3485 (U19)
- Power **3.3V** (and a `100nF` decoupler near pin 8).
- RO (pin 1) to `GP25` (RP2350 RX).
- DI (pin 4) to `GP24` (RP2350 TX).
- RE (pin 2) and DE (pin 3) tied together and connected to the Modbus Direction pin `GP26`.
- A (pin 6) and B (pin 7) to connector CN22; **120Ω** terminator via jumper.
- **TVS protection:** `PSM712-LF-T7` (D4) on the A/B line **at the connector**, NOT next to the SP3485.
- No galvanic isolation in v1; if VFD noise becomes a problem, consider SI8621 + B0505S for v2.

**RS422 (Full Duplex for the Pendant display)** — 2× SP3485 (U21 TX, U22 RX)
- Although the SP3485 is not natively RS-422, it is used as two separate chips in unidirectional mode:
  - **U21 (TX):** RE=GND (always receive disabled), DE=VCC (always transmit enabled) → transmit only
  - **U22 (RX):** RE=GND (always receive enabled), DE=GND (always transmit disabled) → receive only
- U21's A/B outputs go to the pendant (pendant RX side) — differential pair 1.
- Z/Y (the A/B from U22's perspective) receive from the pendant (pendant TX side) — differential pair 2.
- **120Ω 0603 terminator** on the input side of U22 (R111).
- **TVS protection:** `PSM712-LF-T7` (D5, D6) on each pair — **at connector CN32**.
- Pins: `GP28` (UART0 TX → U21 DI), `GP29` (UART0 RX ← U22 RO).

## 6. Spindle 0-10V Converter + EN/DIR (CN39)

RectaBot v1 integrates a **0-10V analog speed reference** and **open-drain EN/DIR digital signals** for universal compatibility with most VFDs (Huanyang, Lapond, Delta, Hitachi, etc.).

### CN39 Pinout (4-pin KF2EDGR-3.5)
| Pin | Net | Function | Voltage |
|---|---|---|---|
| 1 | `VFD_10V` | Analog speed (0-10V) | 0-10V DC |
| 2 | `VFD_EN` | Run/Enable (active LOW open-drain) | 0V active / floating inactive |
| 3 | `VFD_DIR` | Direction (active LOW open-drain) | 0V active / floating inactive |
| 4 | `GND` | Signal/Analog ground | 0V |

### 6.1 PWM-to-0-10V Converter (LM358DR2G)

**Topology (non-inverting amplifier with a 2-stage RC filter + Zener clamp):**

- **U23 = LM358DR2G** (Basic, SOIC-8). VCC = 24V_IN (rail-to-rail is not an issue on a 24V supply).
- **2-stage RC filter:** R116 (10k) + C81 (100nF), R117 (10k) + C83 (100nF) → cutoff 159Hz × 2, attenuation 88dB at 25kHz PWM.
- **Gain network:** R122 (10k) + R118 (10k trim pot) + R120 (10k) → safe gain range **1.5-3.0** (max 9.9V from 3.3V PWM).
- **Overvoltage protection:** R123 (1kΩ) in series + **D7 = 1N4742A 12V Zener (TH, hand-soldered)** + C84 (10nF) → clamps to 12V max regardless of failure.
- **LED indicator:** U23B unity buffer follows VFD_10V → R121 (2k) → LED6 (4mA at 10V).

**Calibration:** tune trim pot R118 with the multimeter so that at 100% PWM you get exactly 10.0V on CN39 pin 1.

### 6.2 EN/DIR open-drain buffers (2× 2N7002)

**Topology (open-drain MOSFET buffer):**

- **Q1, Q2 = 2N7002** (Basic, SOT-23) N-MOSFET, 60V Vds rating.
- **R125, R126 (1kΩ)** gate series — current limit during transients.
- **R124, R127 (10kΩ)** gate pull-down — keeps the MOSFET OFF during MCU reset/boot (failsafe).
- **Source** pins to GND.
- **Drain** pins to CN39 pin 2 (VFD_EN) and pin 3 (VFD_DIR).

**Operational logic:**
| MCU GP30/GP31 | MOSFET | CN39 pin | VFD sees |
|---|---|---|---|
| HIGH (3.3V) | ON | Pulled to GND (~0V) | **ACTIVE** (sinking input triggered) |
| LOW (0V) | OFF | Floating (high-Z) | INACTIVE (VFD pull-up holds HIGH) |
| Reset/boot | OFF (pull-down failsafe) | Floating | INACTIVE ✓ |

**MCU protection:** the 2N7002 Drain blocks up to 60V → if the VFD has a 24V internal pull-up on the FOR/REV terminal, **voltage does NOT come back into the RP2350B pin**.

### 6.3 VFD compatibility

Sinking digital inputs (~85% of the market): Huanyang HY01/HY02, Lapond SVD-EM, ATO, Delta VFD-M/EL, Hitachi WJ200 — **work directly** with the 2N7002 open-drain.

Sourcing/PNP digital inputs (industrial, ~15%): Siemens G120, ABB ACS580, Schneider ATV — need an **external 2-channel relay module** ($3 on AliExpress). For v2, an **integrated relay tier** is planned (Omron G6K-2P-Y).

**Detailed instructions per VFD model:** see [VFD_Wiring_Guide.md](VFD_Wiring_Guide.md).
