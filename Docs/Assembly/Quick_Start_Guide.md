# RectaBot v1.0 — Quick Start Guide

**From an unboxed board to the first motor movement — step by step.**

Estimated time: **2-3 hours** (including soldering 28 TH components).

---

## 📦 What arrives

If you had the boards made from the files in [`Docs/Manufacturing/`](../Manufacturing/),
your parcels hold this. Two of them, because the two halves come from different places.

### From the PCB assembler
- ✅ **RectaBot v1.0 PCB** with the SMT components already mounted
- ✅ ENIG gold finish (durable, and pleasant to solder onto)
- ❌ **No through-hole components** — an assembler only fits what you selected when
  ordering, and the TH parts are not part of the SMT run. You solder those yourself
  (see [Hand_Solder_Components.md](Hand_Solder_Components.md))
- 📄 The invoice — keep it, customs may ask

### From the parts distributor
- ✅ **28 through-hole components** per board, plus whatever the minimum order
  quantities forced you to buy extra of
  - Detailed list: [Hand_Solder_Components.md](Hand_Solder_Components.md)
- ✅ **Male KEFA connectors** (KF2EDGK series) for the screw terminals
- 📄 The invoice — same reason

If you bought an assembled board instead, skip both lists and the soldering with them;
start at [First_Power_On_Procedure.md](First_Power_On_Procedure.md).

### What YOU need to provide (not in the package)

| Tool / Material | Details |
|---|---|
| **Soldering iron** | TS100 / Pinecil / Hakko FX-888D, set to 320-350°C |
| **Solder** | 60/40 Sn/Pb 0.6-0.8mm with rosin core |
| **Multimeter** | Any digital one with continuity and voltage modes |
| **Tweezers** | ESD-safe, for holding TH components during soldering |
| **Loupe / microscope** | Useful for visual QFN-80 RP2350B inspection before power-on |
| **24V DC power supply** | 24V / 2A min, e.g. Mean Well RS-50-24 or screw-terminal adapter |
| **USB-C cable** | Data + power, for firmware flash and debug |
| **Test pins / Dupont cables** | For probing rails |

---

## 🔍 Step 1: Visual inspection (10 min)

**Before you solder anything:**

### 1a. Top side check
Place the PCB under a loupe/microscope and verify:

- ✅ All QFN-80 (RP2350B) pads have a clean ENIG finish (no oxidation)
- ✅ The W5500 QFN and SP3485 SOIC-8 packages are correctly oriented (pin 1 marker = bottom-left)
- ✅ No **solder bridges** between adjacent RP2350B pins (the biggest risk)
- ✅ LM358 (U23), 2N7002 (Q1/Q2) are in place
- ✅ All LEDs (LED1-LED6, marked KT-0805G in BOM) are present and correctly oriented
- ✅ The TF (microSD) connector is well soldered and clicks via its push-push mechanism

### 1b. Bottom side check
- ✅ R36 and C33 are DNP (Do Not Populate) and are NOT present on the bottom — that's expected
- ✅ No visible PCB damage (scratches, exposed traces)

### 1c. What to do if you find a problem
- **Solder bridge** → rework with flux + braid
- **Cold joint** (matte, lumpy surface) → resolder with fresh flux
- **Misaligned IC** → reflow with hot air (300°C, 30s)
- **PCB damage / tear** → contact JLCPCB support with photos

---

## 🔧 Step 2: Hand-soldering TH components (60-90 min)

**Follow the order from [Hand_Solder_Components.md](Hand_Solder_Components.md):**

### Phase A: Lowest-profile components first
1. **D7** Zener 1N4742A — `cathode (band) must face the SPIN_10V net`
2. **R128** PV37W trim pot — orientation doesn't matter
3. **H1** SWD header — straight, 4-pin

### Phase B: Medium-height components
4. **C7** 100µF electrolytic — **POLARITY CRITICAL** — positive lead (longer) to +5V
5. **C61** 1nF/2kV ceramic chassis cap — Bob Smith capacitor

### Phase C: Taller components
6. **U3** B2424S-2WR3 — isolated DC/DC, 4-pin SIP
7. **RJ1** RJ45 J1B1211CCD — watch the shield pins (chassis ground)

### Phase D: Connectors (tallest, last)
8. **CN22-CN42** screw terminals (KEFA KF2EDGR)
   - Pin 1 of each connector aligns with the **PCB silkscreen marker** (a thicker line or square)
   - Short but strong solder joints — TH connectors take mechanical stress

### ⚠️ Polarities you MUST remember
| Component | Polarity info |
|---|---|
| **D7 1N4742A** | The cathode (black band) goes to **SPIN_10V**. Reversed = no overvoltage protection. |
| **C7 100µF** | Positive lead (longer) to the **+5V** side. Reversed = **explosion on first power-on!** |
| **U3 B2424S** | Pin 1 has a silkscreen dot — orient it to match the PCB silkscreen |
| **RJ1 RJ45** | Correctly oriented = LEDs face the PCB edge (outward) |

---

## 🧪 Step 3: Pre-power-on check (15 min)

**MANDATORY — do this BEFORE you connect 24V power!**

See [First_Power_On_Procedure.md](First_Power_On_Procedure.md) for the detailed procedure.

**Summary:**
1. Multimeter on continuity mode
2. Verify there are NO shorts:
   - 24V → GND (probe CN42 pin 1 and 2)
   - +5V → GND
   - +3.3V → GND
   - 24V_ISO → GND_ISO
3. Visually re-check the electrolytic polarity (C7) one more time

---

## ⚡ Step 4: First power-on (10 min)

**Detailed procedure with expected measurements:** [First_Power_On_Procedure.md](First_Power_On_Procedure.md)

**Summary:**

1. **Connect 24V** to CN42 (pin 1 = +24V, pin 2 = GND)
2. **Watch the LEDs:**
   - 24V power LED → lit ✅
   - 5V power LED → lit ✅
   - 3V3 power LED → lit ✅
   - 24V_ISO power LED → lit ✅
3. **Measure with the multimeter:**
   - +5V rail = 4.95 - 5.05V ✅
   - +3.3V rail = 3.25 - 3.35V ✅
   - +24V_ISO rail = 23.5 - 24.5V ✅
4. **NO SMOKE**, no crackling sounds, no overheating components

**If anything is off** → power down immediately and check the troubleshooting section in First_Power_On_Procedure.md.

---

## 💾 Step 5: Firmware flash (15 min)

### 5a. Enter BOOTSEL mode
1. Press and hold the **BOOT button** on the PCB
2. While holding BOOT, press and release the **RESET button**
3. Release the BOOT button
4. The RP2350B will enumerate as a **USB Mass Storage** device (drive `RPI-RP2`)

### 5b. Flash grblHAL firmware
1. Connect a USB-C cable PC ↔ RectaBot
2. Pick the `.uf2` that matches your machine's kinematics — use the [firmware configurator](https://rectabot.org/configurator/) or grab it from the firmware repo's `variants/` folder (e.g. `grblHAL_RectaBot_3axis_v1.0.uf2`, `..._4axis-a-ganged-y_v1.0.uf2`). RectaControl's Firmware panel can also flash it for you.
3. Open `RPI-RP2` in Explorer and drag-and-drop the `.uf2` into the drive
4. The drive disappears and the RP2350B reboots into grblHAL

### 5c. Verify the firmware
1. Open a serial monitor (PuTTY / Arduino IDE Serial Monitor / minicom)
2. Connect to the COM port (115200 baud, 8N1)
3. Send `$I` — you should see something like (a 4-axis build):
   ```
   [VER:1.1f.20260331:]
   [OPT:VNMPZTSL+2,100,1024,4,0]
   [AXS:4:XYZA]
   [BOARD:RectaBot v1.0]
   [DRIVER:RP2350@150MHz]
   ```
   `[AXS:4:XYZA]` confirms the axis count; the trailing `2` in `[OPT:...+2,...]` means a ganged/auto-square motor is active.

---

## ⚙️ Step 6: grblHAL configuration (20 min)

**Critical parameters for RectaBot v1.0:**

```
$0=5         ; Step pulse, microseconds
$1=25        ; Step idle delay, milliseconds
$2=0         ; Step invert OFF — Common Anode drivers need NO step invert (do NOT set 31)
$4=15        ; Enable invert — the 74HC14D inverts EN, so invert every axis: 7=3-axis, 15=4-axis, 31=5-axis
$5=0         ; Limit invert OFF — the opto inputs already read active-low
$6=1         ; Probe invert ON
; --- direction is PER MACHINE (depends on your wiring/mechanics), not a fixed value ---
$3=0         ; Direction invert — jog each axis; if it runs backwards, flip THAT axis's bit
$10=1        ; Status report options, mask
$11=0.010    ; Junction deviation, millimeters
$12=0.002    ; Arc tolerance, millimeters
$13=0        ; Report in inches, boolean
$20=0        ; Soft limits enable, boolean
$21=1        ; Hard limits enable, boolean
$22=1        ; Homing cycle enable, boolean
$23=3        ; Homing direction invert, mask
$24=25.000   ; Homing locate feed rate, mm/min
$25=500.000  ; Homing search seek rate, mm/min
$26=250      ; Homing switch debounce delay, milliseconds
$27=1.000    ; Homing switch pull-off distance, millimeters
$30=24000    ; Maximum spindle speed, RPM (for a 24kRPM VFD)
$31=0        ; Minimum spindle speed, RPM
$32=0        ; Laser-mode enable, boolean (leave 0 for spindle)
```

**Per-axis configuration** (X axis shown, same for Y/Z/A/B):
```
$100=80.000  ; X-axis steps per millimeter (depends on the driver)
$110=2000.000 ; X-axis maximum rate, mm/min
$120=200.000  ; X-axis acceleration, mm/sec²
$130=300.000  ; X-axis maximum travel, millimeters
```

---

## 🎮 Step 7: First axis test (10 min)

### 7a. Connect a DM556 driver to CN34 (X axis)

CN34 is a 4-pin **common-anode** stepper output. Silkscreen order is **+5V / STEP / DIR / EN** — there is **no GND pin**: the STEP/DIR/EN lines are actively pulled to GND by the on-board 74HC14D, and that is the return path.

- Pin 1 = **+5V** → DM556 **PUL+, DIR+, ENA+** (common all three `+` inputs to +5V)
- Pin 2 = **STEP** → DM556 **PUL−**
- Pin 3 = **DIR** → DM556 **DIR−**
- Pin 4 = **EN** → DM556 **ENA−**  *(optional — leave unconnected to keep the driver always enabled)*

**Common anode:** +5V feeds every `+` input on the driver, and the controller sinks each signal to GND to pulse it. Keep `$2=0` (step-invert off) — the 74HC14D idles HIGH and each step briefly pulls the opto low, which is correct for common anode. Setting `$2`≠0 holds the opto on at idle and can cause missed steps.

### 7b. Connect a motor to the DM556 outputs
- A+, A-, B+, B- (4-wire NEMA17/23 motor)
- Set the DM556 microstep DIP switches (e.g. 1600 steps/rev for 8 microsteps)

### 7c. Try a jog command in the serial monitor
```
$J=G91 X10 F500    ; Move 10mm to the right @ 500mm/min
$J=G91 X-10 F500   ; Move 10mm to the left
```

**What to expect:**
- ✅ The motor turns smoothly, without "skipping"
- ✅ Direction follows the DIR command
- ✅ It stops at the position
- ❌ If it jitters or doesn't move → see troubleshooting

---

## 🎯 Next steps

Once the X axis runs reliably:
1. **Repeat for Y, Z, A, B** (CN35-CN38)
2. **Connect limit switches** (CN27-CN31)
3. **Configure homing** (`$H` command)
4. **Connect the VFD** — see [VFD_Wiring_Guide.md](../Reference/VFD_Wiring_Guide.md)
5. **Connect coolant / vacuum relays to CN40** — read below before you buy or wire them
6. **Test G-code execution** with a sender (UGS, CNCjs, IO Sender)

### ⚠️ Wiring a relay to CN40 — which way round matters

`CN40` is `+5V · VAC · FLOOD · MIST · GND`. Wire each SSR control input between
**`+5V` (pin 1)** and its signal pin — the same common-anode arrangement as the stepper
drivers, which is why the `+5V` pin is on the connector at all.

| SSR control terminal | CN40 pin |
| :--- | :--- |
| `+` | **1** — `+5V` |
| `−` | **2** `VAC` (`M62`/`M63`) · **3** `FLOOD` (`M8`) · **4** `MIST` (`M7`) |

The signal passes through the 74HC14D inverter, so a coolant pin **idles at ~5 V and is
pulled to 0 V when the M-code turns the output on**. Wire the SSR to `GND` instead and
you get the exact opposite: the pump or vacuum runs **whenever the machine is idle**, and
switches **off** when the program calls for it. Nothing reports an error — it simply runs
until someone notices.

### Which relay to buy

Each pin is a 74HC14D channel behind a 330 Ω resistor — 25 mA absolute maximum, with `VOL`
guaranteed up to 4 mA. Both of these work:

**An optocoupler-input relay module** — the easy answer. Wire `VCC`→pin 1, `GND`→pin 5,
`IN`→the signal pin. It has a gain stage: a few mA from this pin lights an LED, whose
transistor pulls the 70–80 mA of coil current from the module's own `VCC`, so the board
never supplies the relay itself. Look for an input current of about 5 mA at 5 V.

**A solid-state relay wired straight on** — a **Fotek SSR-40 DA** was built and run this
way on the reference board:

| Fotek terminal | to |
| :--- | :--- |
| **1** | 230 V live in |
| **2** | 230 V out → load → neutral |
| **3** (control `+`) | `CN40` pin 1 — `+5V` |
| **4** (control `−`) | `CN40` pin 4 — `MIST` |

It switches from `M7`/`M9`, from the sender's coolant button, and survives repeated rapid
on/off. Measured: 5.07 V open circuit, **3.11 V across the control input**, ~5.9 mA, with
the output still pulling to ground. The Fotek's datasheet minimum is 3 V, so it clears.

What decides it is not the chip's current capability but how much voltage the 330 Ω leaves
for the relay. If a particular SSR needs more than roughly 3 V and will not trigger, put a
small P-channel MOSFET between — see [Pinout.md](../Reference/Pinout.md).

**All three outputs can drive one each, at the same time.** `MIST`, `FLOOD` and `VAC` come
off three *different* 74HC14D chips (`U7`, `U10`, `U15`), so a second and third SSR do not
pile onto the chip driving the first. Together they draw about **17.7 mA** from `CN40`
pin 1. Buy one relay per output and wire each control `−` to its own signal pin.

That margin belongs to the individual relay, though, not to how many of them you run — and
counterfeit Foteks are common. **Confirm each new one actually switches before you build it
into the machine.**

#### Industrial modules

DIN-rail interface relays (Phoenix Contact PLC-INTERFACE, Weidmüller TERMSERIES, Wago 859,
Omron G3RV-SR) are built to be driven by a PLC output that sources half an amp, so most of
them ask for 10–15 mA even on their 5 V models. Some low-current variants exist — check
that one number before ordering.

Their input stage is the same optocoupler the cheap module uses. What the money buys is the
output: contacts that survive, terminals that hold, and a housing that belongs on a
machine. For a mill that runs every day that is worth paying for; the electrical
requirement is identical either way.

A relay module's coils draw 70–80 mA each and kick when they release. If your module has a
**`JD-VCC` jumper**, pull it and feed `JD-VCC` from a separate 5 V supply — keeping grounds
common — so that kick stays off the same 5 V rail that carries your step pulses.

---

## 🆘 Troubleshooting

### Power rails out of spec
- **+5V < 4.9V** → TPS5430 buck may have a cold joint on L1 or the VSENSE divider
- **+3.3V < 3.2V** → AMS1117 LDO overloaded or poorly soldered
- **+24V_ISO < 23V** → B2424S DC/DC not working or 24V_IN too low

### LEDs don't light up
- **All LEDs off** → 24V isn't reaching the board (check CN42 polarity!)
- **Only the 24V LED is lit** → TPS5430 problem
- **24V + 5V LEDs lit, 3V3 off** → AMS1117 problem

### USB enumeration doesn't work
- **Drive does not appear** → wrong BOOT/RESET sequence, repeat it
- **Drive appears, then immediately disappears** → firmware file corrupted or wrong UF2

### Motor doesn't move
- **No motion / missed steps** → check `$2=0` (step invert must be OFF for the Common Anode drivers; `$2=31` holds the opto on at idle) and `$4` enable-invert matches your axis count (7/15/31)
- **Constant level on STEP** → 74HC14D not soldered properly
- **Motor "skips"** → microstep DIP switches don't match the $100 parameter
- **Axis runs the wrong way** → flip that axis's bit in `$3` (direction is per-machine)

---

## 📚 Further reading

- [Pinout.md](../Reference/Pinout.md) — complete GPIO map
- [Hardware_Design_Guidelines.md](../Reference/Hardware_Design_Guidelines.md) — electrical and mechanical design
- [VFD_Wiring_Guide.md](../Reference/VFD_Wiring_Guide.md) — how to wire various VFDs (analog 0-10V)
- [VFD_Modbus_Setup.md](../Reference/VFD_Modbus_Setup.md) — spindle control over RS-485/Modbus
- [Product_Specification_v1.0.md](../Reference/Product_Specification_v1.0.md) — complete datasheet

---

*Need help? Open an issue on GitHub or email hello@rectabot.org*
