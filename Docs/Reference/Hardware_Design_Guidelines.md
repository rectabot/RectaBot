# RectaBot Hardware Design Guidelines (KiCad / EasyEDA)

This document is a reference for laying out the PCB based on the approved architecture.

## 1. Layer stackup (4-layer PCB)
Given the choice of 4 layers and the need to minimize EMI (especially because of the VFD), the recommended *Layer Stackup* is:
- **Top Layer (Signal / Power routing):** signals for components on the top side, routing of short logic lines, and power.
- **Layer 2 (GND plane):** solid (uninterrupted) ground plane. Crucial for short return paths and for isolation under the W5500 SPI lines, as well as for cooling the TPS5430DDAR.
- **Layer 3 (Power plane / Signal):** islands (polygons) for +5V and +3.3V supplies. Optionally routing of control lines.
- **Bottom Layer (Signal):** routing of longer signal lines, e.g. from the microcontroller to connectors.

## 2. Power (Power Tree)
- **TPS5430DDAR (C9864) Buck Converter (24V → 5V):** design according to Texas Instruments recommendations. The *Exposed Pad* under the chip **must** be stitched with vias (thermal vias) into Layer 2 and the Bottom layer to efficiently dissipate heat and avoid overheating. The inductor should be shielded and placed as close to the chip as possible.
- **3.3V LDO (e.g. AMS1117-3.3 — C6186):** drops the voltage from 5V to 3.3V. Given the small voltage drop (1.7V difference), thermal design is extremely safe.

## 3. Communication (Isolation and EMI)
- **RS485 Modbus:** RectaBot v1.0 uses the **SP3485EN-L/TR** transceiver (works directly at 3.3V — no level shift needed as with MAX485). TVS protection via **PSM712-LF-T7** diodes on the A/B outputs — placed at the connector, not at the IC. For additional protection against VFD noise, digital isolators (e.g. SI8621) can be used with a separate isolated DC/DC supply (B0505S series) if a noise problem appears.
- **RS422 (Pendant):** SP3485EN-L/TR is used (two chips — one TX, one RX) for the Full Duplex link. Pay attention to matching and the termination resistor (120Ω) especially on the input side (RX) of a long cable.
- **W5500 (Ethernet):** route the SPI lines (RX, CS, SCK, TX) close together and always over a solid GND plane without interruption to preserve signal integrity. Place the RJ45 port as far as possible from high-voltage outputs/VFD connectors.

## 4. Axes (5V Common Anode to DM556)
- The grblHAL RP2350 chip outputs I/O signals at 3.3V. RectaBot v1.0 uses the **74HC14D** (Schmitt-trigger HEX **inverter**) for the 3.3V→5V level shift. Power the buffers from +5V from the main buck converter.
- Control outputs (Step, Dir, Enable) run in *Common Anode* mode: the DM556 optocoupler's `+` lead is permanently tied to +5V, and the 74HC14D pulls the `−` lead to GND to turn the opto on.
- The 74HC14D inverts, but the Common Anode opto conducts on a LOW cathode, so a grbl active-high step pulse lands correctly at the opto with **no** firmware step invert: **`$2=0`** (validated on the reference board — `$2`≠0 holds the opto on at idle and causes missed steps). The **ENABLE** line does need inverting because of the driver's ENA polarity: **`$4`** = all axes (7 for 3-axis, 15 for 4-axis, 31 for 5-axis). Limit/probe opto inputs use **`$5=0`** / **`$6=1`**. Direction (`$3`) is per-machine — set it from how each axis actually moves.
- *Heads up:* **the STEP pins must be consecutive.** grblHAL drives them from PIO
  (`STEP_PORT = GPIO_PIO`, `STEP_PINS_BASE = 8`), and a PIO state machine writes one
  contiguous field of bits across a pin range — so `GP8`-`GP12` is a hard requirement, not
  a preference. **Dir (`GP13`-`GP17`) and Enable (`GP18`-`GP22`) are ordinary GPIO**
  (`DIRECTION_PORT = GPIO_OUTPUT`, `GPIO_MAP`), individually mapped: they are numbered in
  runs for tidiness, and may be moved when drawing a new board. Direction only has to
  settle before the pulse (`$29` setup time), which needs no PIO.
- On the RP2350B a PIO instance sees a **32-GPIO window** — `0`-`31` by default, or
  `16`-`47` via `pio_set_gpio_base()`. Keep the step pins inside one window; `GP8`-`GP12`
  sits well within the default one, so no base shifting and no splitting across PIO
  instances.

### Alternatives considered for v2 (NOT in v1):
- **74HCT245** (octal bus transceiver) — does not invert, but uses more pins and does not clean up noise like a Schmitt trigger.
- **74LVC07A** (open-drain buffer) — works with external pull-up resistors to 5V, adds 6 resistors, but does not invert.
- v1 kept the **74HC14D** because it is Basic on JLCPCB (free assembly) and provides a clean Schmitt input.

## 5. Spindle Interface (0-10V + EN/DIR)

RectaBot v1 implements a **universal VFD interface** on connector CN39 (4-pin KF2EDGR-3.5):
- Pin 1: `VFD_10V` — analog speed reference 0-10V DC
- Pin 2: `VFD_EN` — Run/Enable, active LOW open-drain
- Pin 3: `VFD_DIR` — Direction, active LOW open-drain
- Pin 4: `GND` — signal ground

### 5.1 PWM → 0-10V Converter (LM358DR2G)

**Topology:** non-inverting op-amp with a two-stage RC filter + Zener clamp + LED indicator.

**Components and layout:**
- **U23 = LM358DR2G** (Basic, SOIC-8) — dual op-amp, VCC = 24V_IN; rail-to-rail is not an issue on a 24V supply.
- **Filter stage 1:** R116 (10k) + C81 (100nF) — cutoff 159Hz, attenuates the PWM carrier.
- **Filter stage 2:** R117 (10k) + C83 (100nF) — second-order filtering, total 88dB at 25kHz PWM.
- **Gain network:** R122 (10k) + R128 (10k trim pot PV37W) + R120 (10k) → safe gain range **1.5-3.0** (max 9.9V from 3.3V PWM).
- **Overvoltage clamp:** R123 (1kΩ) in series + **D7 = 1N4742A 12V Zener (TH, hand-soldered)** + C84 (10nF) — clamps to 12V max even if the op-amp fails.
- **LED indicator:** the U23B section configured as a unity buffer for VFD_10V → R121 (2k) → LED6 (visual indicator of the real speed).

**Calibration:** tune trim pot R128 with the multimeter so that at 100% PWM (M3 S24000) you get exactly **10.0V** on CN39 pin 1.

### 5.2 EN/DIR open-drain buffers (2× 2N7002)

**Topology:** N-MOSFET open-drain, gate pull-down failsafe.

**Components:**
- **Q1, Q2 = 2N7002** (Basic, SOT-23) — N-MOSFET, 60V Vds rating, sufficient for any industrial VFD with up to 24V internal pull-up.
- **R125, R126 (1kΩ)** — gate series resistor, limits current during transients.
- **R124, R127 (10kΩ)** — gate pull-down, keeps the MOSFET OFF during MCU reset/boot (failsafe).

**Operational logic:**
| MCU pin (GP30/GP31) | MOSFET state | CN39 pin voltage | VFD sees |
|---|---|---|---|
| HIGH (3.3V) | ON (closed) | Pulled to GND (~0V) | **ACTIVE** (sinking input triggered) |
| LOW (0V) | OFF (open) | Floating / high-Z | INACTIVE (VFD pull-up holds HIGH) |
| MCU reset/boot | OFF (pull-down failsafe) | Floating | INACTIVE ✓ failsafe |

**Why this is safe:**
- The 2N7002 Drain blocks up to **60V** → if the VFD has a 24V internal pull-up, **voltage does NOT come back into the RP2350B pin**.
- The pull-down R124/R127 guarantees the spindle **never starts by itself** during MCU reset, BOOT, firmware update, or crash.
- Open-drain topology means you can connect multiple VFDs in parallel (e.g. dual spindle), each with its own pull-up.

### 5.3 PCB layout notes for the Spindle section

- **PWM signal routing** (GP27 → R116): short trace, do NOT route across the isolation void. Spindle PWM stays in the GND_MCU domain.
- **U23 LM358 decoupling:** 100nF (C82) very close to pin 8 (VCC). Without it, oscillations occur at high LED current.
- **D7 Zener placement:** hand-soldered TH (DO-41) → place **exactly between R123 and CN39 pin 1**, with the cathode (band) facing the CN39 side.
- **R128 trim pot:** place it so the adjust screw is accessible **without disassembling the board** — top-adjust orientation.
- **Ground reference:** CN39 pin 4 (GND) must be a **dedicated ground trace** to U23 GND, not shared with the noisy step/dir ground.
- **TVS on VFD_EN/VFD_DIR:** optional in v1 (2N7002 60V Vds is sufficient), TVS array planned in v2 for better EMC compliance.

### 5.4 VFD compatibility (summary)

| VFD Family | Digital Input Type | Works out-of-the-box? |
|---|---|---|
| Huanyang HY01/HY02 | Sinking (NPN) | ✅ Yes |
| Lapond SVD-EM | Sinking (NPN) | ✅ Yes |
| Delta VFD-M/EL | Sinking (NPN) | ✅ Yes |
| Hitachi WJ200 | Sinking (NPN) | ✅ Yes |
| ATO inverter | Sinking (NPN) | ✅ Yes |
| Siemens G120 | Sourcing (PNP) | ⚠️ Needs external relay module |
| ABB ACS580 | Sourcing (PNP) | ⚠️ Needs external relay module |
| Schneider ATV | Sourcing (PNP) | ⚠️ Needs external relay module |

**Detailed instructions by VFD model:** [VFD_Wiring_Guide.md](VFD_Wiring_Guide.md).

### v2 planned improvement
- Add an optional **2× Omron G6K-2P-Y relay** for out-of-the-box PNP/sourcing VFD compatibility (integrated relay tier).
