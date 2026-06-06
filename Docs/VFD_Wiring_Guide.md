# RectaBot — VFD Wiring & Configuration Guide

**Version:** 1.0 | **Hardware:** RectaBot v1.0 | **Firmware:** grblHAL

This guide explains how to connect and configure popular Variable Frequency Drives (VFDs) to the RectaBot CNC controller for spindle control.

---

## 📐 CN39 Spindle Connector Pinout

RectaBot uses a 4-pin **CN39** connector (3.5mm pitch screw terminal) for spindle control:

| Pin | Net Name | Function | Signal Type | Typical VFD Terminal |
|---|---|---|---|---|
| **1** | `VFD_10V` | Analog speed reference | 0-10V DC | VI, AVI, AI1, AIN1, FV |
| **2** | `VFD_EN` | Run / Enable | Active LOW open-drain | FWD, FOR, X1, DI1, RUN |
| **3** | `VFD_DIR` | Direction select | Active LOW open-drain | REV, X2, DI2, DIR |
| **4** | `GND` | Signal & analog ground | 0V reference | GND, ACM, COM, DCM |

### ⚠️ Critical compatibility note

The **VFD_EN** and **VFD_DIR** pins use **2N7002 N-MOSFET open-drain output**. This means:
- When MCU asserts HIGH → MOSFET pulls pin to GND (active LOW)
- When MCU asserts LOW → MOSFET is OFF, pin is high-impedance (floating)

**Compatible VFD input types:**
- ✅ **NPN sinking inputs** (most Chinese VFDs: Huanyang, Lapond, Delta) — works directly
- ⚠️ **PNP sourcing inputs** (most industrial: Siemens, ABB, Schneider) — needs external relay module
- ✅ **Dry contact inputs** — works directly

**Maximum voltage tolerance on VFD_EN / VFD_DIR pins: 60V** (2N7002 Vds rating). Safe for any VFD with up to 48V digital input voltage.

---

## 🔌 1. Huanyang HY01 / HY02 (Most Common, ~50% of Hobby Market)

**Examples:** HY01-D751 (0.75kW), HY02-D152 (1.5kW), HY02-D222 (2.2kW)

### Wiring

```
RectaBot CN39                Huanyang VFD (HY01/HY02)
─────────────                ──────────────────────────
Pin 1  VFD_10V  ──────────►  VI    (analog speed input)
Pin 2  VFD_EN   ──────────►  FOR   (forward run terminal)
Pin 3  VFD_DIR  ──────────►  REV   (reverse run terminal)
Pin 4  GND      ─────┬────►  ACM   (analog common)
                     └────►  DCM   (digital common, jumper to ACM)
```

**Cable:** Use **shielded twisted pair** for VFD_10V + ACM. Connect cable shield to VFD chassis ground on one end only.

### Parameter Configuration

Enter VFD programming mode (PRG button), then set these parameters:

| Parameter | Value | Description |
|---|---|---|
| `PD001` | `1` | Run command source = external terminals |
| `PD002` | `1` | Frequency reference = external AVI (0-10V analog) |
| `PD003` | `400` | Main frequency (Hz) — adjust to your spindle's rated frequency |
| `PD004` | `400` | Base frequency (typically same as PD003) |
| `PD005` | `400` | Max output frequency (Hz) |
| `PD006` | `2.5` | Intermediate frequency for V/f curve |
| `PD007` | `0.5` | Min output frequency |
| `PD008` | `220` | Max voltage (V) — match your motor's rated voltage |
| `PD009` | `15` | Intermediate voltage |
| `PD010` | `8` | Min voltage |
| `PD011` | `120` | Frequency lower limit (Hz) — minimum spindle RPM |
| `PD014` | `5.0` | Acceleration time (s) |
| `PD015` | `5.0` | Deceleration time (s) |
| `PD044` | `1` | **Two-line mode 1** — FOR=RUN, REV=DIRECTION (CRITICAL!) |
| `PD045` | `0` | Disable jog terminals |
| `PD070` | `1` | Analog input 0-10V mode |
| `PD072` | `400` | Analog input max frequency |

### grblHAL Configuration

After flashing RectaBot firmware, set these via UART/USB (`$$` to view all settings):

```
$30=24000          ; Max spindle RPM (must match VFD PD005 × 60 / motor_poles)
$31=0              ; Min spindle RPM
$32=0              ; Laser mode disabled (we're using spindle)
```

### Test Procedure

1. Power on RectaBot, connect VFD per wiring diagram above
2. Power on VFD, verify display shows "0.0Hz" idle
3. Send G-code: `M3 S12000` (spindle on, forward, 12000 RPM)
4. Multimeter on CN39 pin 1 (VFD_10V) should read ~5V (50% of max)
5. VFD display should show ~200Hz (50% of 400Hz max)
6. Spindle should rotate clockwise (CW) at 12000 RPM
7. Send `M5` to stop
8. Send `M4 S12000` to test reverse — spindle should rotate counter-clockwise (CCW)

### Troubleshooting

| Symptom | Cause | Solution |
|---|---|---|
| Spindle doesn't start | PD001 wrong | Set PD001=1 (terminal control) |
| Speed control doesn't work | PD002 wrong | Set PD002=1 (AVI analog) |
| M4 (reverse) doesn't work | PD044 wrong | Set PD044=1 (two-line mode 1) |
| Spindle runs at wrong speed | Calibration | Adjust trim pot R118 on RectaBot until 100% PWM = 10.0V exactly |
| VFD alarm "ESC" | Both FOR+REV active | Check PD044=1 (NOT PD044=0) |

---

## 🔌 2. Lapond SVD-EM (~15% of market)

**Examples:** SVD-EM 7.5G/11P 4T, SVD-EM 0.75G 4T

### Wiring

```
RectaBot CN39                Lapond SVD-EM
─────────────                ──────────────────────
Pin 1  VFD_10V  ──────────►  AI1   (analog input 1)
Pin 2  VFD_EN   ──────────►  DI1   (digital input 1)
Pin 3  VFD_DIR  ──────────►  DI2   (digital input 2)
Pin 4  GND      ─────┬────►  GND   (analog ground)
                     └────►  COM   (digital common — jumper to GND)
```

### Parameter Configuration

| Parameter | Value | Description |
|---|---|---|
| `F1.00` | `1` | Run command source = terminal control |
| `F1.01` | `0` | Two-line mode (DI1=RUN, DI2=DIR) |
| `F1.06` | `2` | Speed source = analog AI1 |
| `F1.07` | `0` | DI1 function = Run/Stop |
| `F1.08` | `2` | DI2 function = Forward/Reverse direction |
| `F2.00` | `0` | AI1 input range = 0-10V |
| `F2.01` | `0.0` | AI1 minimum voltage |
| `F2.02` | `10.0` | AI1 maximum voltage |
| `F3.00` | `400` | Max output frequency (Hz) |
| `F3.05` | `5.0` | Acceleration time (s) |
| `F3.06` | `5.0` | Deceleration time (s) |

### Notes
- Lapond uses **F-group parameters** (F1.xx, F2.xx, F3.xx) instead of PD-style
- Default password to enter programming: `0000` or `1234`
- After config, save with **STORE** or **ENT** button

---

## 🔌 3. Delta VFD-M / VFD-EL (~10% of market)

**Examples:** VFD007M21A (0.75kW), VFD015M23A (1.5kW), VFD022EL43A (2.2kW)

### Wiring

```
RectaBot CN39                Delta VFD-M/EL
─────────────                ──────────────────────
Pin 1  VFD_10V  ──────────►  AVI   (analog voltage input)
Pin 2  VFD_EN   ──────────►  M0    (multi-function input 0)
Pin 3  VFD_DIR  ──────────►  M1    (multi-function input 1)
Pin 4  GND      ─────┬────►  ACM   (analog common)
                     └────►  DCM   (digital common — jumper to ACM)
```

### Parameter Configuration

| Parameter | Value | Description |
|---|---|---|
| `02-00` | `1` | Frequency source = external analog AVI |
| `02-01` | `1` | Operation command = external terminals |
| `02-05` | `0` | AVI input = 0-10V (not 4-20mA) |
| `04-04` | `1` | Two-wire / three-wire mode = **Two-wire 1** |
| `04-05` | `0` | Two-wire type 1: M0=RUN, M1=DIRECTION |
| `04-11` | `1` | M0 function = Forward/Reverse run |
| `04-12` | `0` | M1 function = Direction select |
| `01-00` | `400.0` | Max output frequency (Hz) |
| `01-09` | `5.0` | Acceleration time (s) |
| `01-10` | `5.0` | Deceleration time (s) |

### Notes
- Delta uses **two-digit group.member** parameter format (e.g., `02-00`)
- Access programming mode: **MODE** button → enter password if set
- After parameter change: **ENTER** to save, then **MODE** to exit

---

## 🔌 4. Generic NPN Sinking VFD (ATO, Yiqi, Tecorp, Powtran)

If your VFD is not listed above but supports **NPN sinking digital inputs**, use this generic config:

### Wiring (universal)

```
RectaBot CN39                Generic VFD
─────────────                ──────────────────────
Pin 1  VFD_10V  ──────────►  Analog voltage input (VI / AVI / AI1 / 10V)
Pin 2  VFD_EN   ──────────►  Run terminal (FWD / RUN / X1 / DI1)
Pin 3  VFD_DIR  ──────────►  Direction terminal (REV / DIR / X2 / DI2)
Pin 4  GND      ─────┬────►  Analog ground (GND / ACM / AGND)
                     └────►  Digital common (DCM / COM / SC — jumper to GND)
```

### Required VFD Settings (translate to your manual's parameter names)

1. **Run command source** = external terminals (not keypad)
2. **Frequency reference source** = external analog voltage 0-10V
3. **Control mode** = **Two-wire mode 1** (RUN + DIRECTION semantic)
   - NOT "Two-wire mode 2" (which uses FOR + REV semantic — incompatible without firmware patch)
4. **Digital input logic** = NPN sinking (negative logic)
5. **Analog input range** = 0-10V (not 4-20mA, not 2-10V)

---

## 🔌 5. PNP Sourcing / Industrial VFDs (Siemens, ABB, Schneider)

⚠️ **RectaBot's open-drain output cannot directly drive PNP sourcing inputs.** You need an **external 2-channel relay module** ($3-5 on AliExpress).

### Recommended Relay Module

Search AliExpress / Aliexpress for: **"2 channel 3.3V relay module optocoupler"**

Look for these features:
- 3.3V trigger compatible
- Optocoupler isolation
- 10A 250VAC contact rating (more than enough)
- LED indicators per channel

### Wiring with Relay Module

```
RectaBot CN39          Relay Module          PNP Sourcing VFD
─────────────          ────────────          ─────────────────
Pin 1  VFD_10V ──────────────────────────────► AI1 (0-10V analog)
Pin 2  VFD_EN  ────────► IN1 (3.3V trigger)
Pin 3  VFD_DIR ────────► IN2
Pin 4  GND     ────────► GND
                         +5V ←── from RectaBot +5V test point or USB
                                              ┌──────────────────┐
                         NO1 ─────────────────► DI1 (RUN)        │
                         COM1 ────────────────► +24V (VFD source)│
                         NO2 ─────────────────► DI2 (DIR)        │
                         COM2 ────────────────► +24V (VFD source)│
                                              └──────────────────┘
```

The relay's **dry contact (NO + COM)** bridges the VFD's +24V supply to its DI inputs when MCU asserts the corresponding signal — perfectly matching PNP sourcing behavior.

### Future RectaBot v2 "Pro" Tier

v2 will integrate **2× Omron G6K-2P-Y relays** directly on the board, eliminating the need for an external module. Industrial customers will get full plug-and-play compatibility with Siemens, ABB, Schneider VFDs out-of-the-box.

---

## 🛠️ Calibration Procedure (One-time Setup)

After connecting any VFD, calibrate the 0-10V converter:

1. Connect **multimeter** to CN39 pin 1 (VFD_10V) and pin 4 (GND)
2. Set multimeter to DC voltage, 20V range
3. In grblHAL console, send: `M3 S<MAX_RPM>` (e.g., `M3 S24000` for 24000 RPM max)
4. Read multimeter — should show close to 10.0V
5. **Adjust trim pot R118** on RectaBot (on top of PCB, labeled "SPIN_TRIM"):
   - Turn clockwise → higher voltage
   - Turn counter-clockwise → lower voltage
6. Repeat until multimeter reads exactly **10.00V at S<MAX_RPM>**
7. Send `M5` to stop, verify multimeter reads ~0V
8. Test mid-range: `M3 S<MAX_RPM/2>` should give ~5.0V

This calibration accounts for PWM source variation (3.2V vs 3.3V from MCU) and resistor tolerance. Do this **once** after first power-up; trim setting holds permanently.

---

## ⚡ Safety Notes

### E-Stop Behavior

When ESTOP is triggered (CN37 or grblHAL alarm):
1. MCU immediately sets GP27 PWM = 0%, GP30/31 = LOW
2. VFD_10V drops to 0V within 10ms
3. VFD_EN and VFD_DIR go to floating (inactive)
4. **VFD decelerates spindle per its programmed deceleration time** (typically 5s)

⚠️ For **HARD STOP** (no deceleration), you must wire an external safety relay that cuts motor power directly. RectaBot does not provide this — it's an industry-standard external safety device (e.g., Pilz PNOZ, Phoenix EMD, or simple contactor with auxiliary contact).

### Spindle Direction Change

Many VFDs **require spindle to be stopped before changing direction** to prevent mechanical damage. grblHAL handles this automatically — never send M3 → M4 without M5 in between.

### Cable Specifications

| Signal | Recommended Cable | Max Length |
|---|---|---|
| VFD_10V + GND | Shielded twisted pair, 22-24 AWG | 3 meters |
| VFD_EN + VFD_DIR | Standard hookup wire, 22-24 AWG | 3 meters |
| Combined cable | 4-core shielded (e.g., Belden 9534) | 3 meters |

For longer runs (>3m), use **shielded multi-conductor cable** and ground the shield only at the **VFD chassis** end (single-point grounding to prevent ground loops).

---

## 📋 Quick Reference Card

```
┌─────────────────────────────────────────────────────────┐
│  RectaBot CN39 → VFD Quick Wiring                       │
├─────────────────────────────────────────────────────────┤
│  Pin 1  VFD_10V  → Analog speed input (0-10V)           │
│  Pin 2  VFD_EN   → Run command (active LOW)             │
│  Pin 3  VFD_DIR  → Direction (active LOW)               │
│  Pin 4  GND      → Analog + digital ground              │
├─────────────────────────────────────────────────────────┤
│  VFD MUST be in "Two-line mode 1" (RUN/DIR semantic)    │
│  Calibrate trim pot R118 for 10.0V at max RPM           │
│  Use external 2-ch relay for PNP/industrial VFDs        │
└─────────────────────────────────────────────────────────┘
```

---

## 🆘 Support

For issues with specific VFD models not listed here, or custom configurations:

- **Documentation:** https://github.com/rectabot
- **Discord:** RectaBot Users community (planned)
- **Email:** hello@rectabot.org

**When asking for help, please include:**
1. VFD make and model
2. VFD parameter settings (PD001, PD002, etc.)
3. Multimeter readings at CN39 pins during M3/M5 commands
4. grblHAL `$$` output (settings dump)
5. Photo of wiring

---

*Document version 1.0 — last updated for RectaBot v1.0 hardware (firmware: grblHAL with my_machine_map.h).*
