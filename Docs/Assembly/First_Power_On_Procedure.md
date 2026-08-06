# RectaBot v1.0 — First Power-On Procedure

**Safely powering up the board for the first time — with a multimeter in hand.**

Estimated time: **15-30 min** (carefully, no rush).

> ⚠️ **WARNING:** This is the riskiest moment of the entire process. A mistake here can destroy a $77 board. Go **slowly** and follow every step.

---

## 🎯 Purpose of this document

1. **Detect critical mistakes** (short circuits, wrong polarities) BEFORE current flows through the PCB
2. **Verify all voltages** after power-on
3. **Establish a baseline** for future troubleshooting

---

## 🛡️ Step 1: Pre-power continuity tests

**Multimeter in Continuity mode (with the buzzer).**

### 1a. Power rail short-circuit tests

Place the multimeter probes between the following points and verify that there is **NO continuity** (no beep):

| Test point 1 | Test point 2 | Expected | What a beep means |
|---|---|---|---|
| **CN42 pin 1** (+24V) | **CN42 pin 2** (GND) | NO continuity | 24V short → do not power on! |
| **+5V rail** (USB-C VBUS pin) | **GND** (USB-C shell) | NO continuity | 5V short → TPS5430 problem |
| **+3.3V rail** (RP2350 pin 3) | **GND** | NO continuity | 3.3V short → AMS1117 or MCU pad bridge |
| **24V_ISO** (U3 pin 3) | **GND_ISO** (U3 pin 4) | NO continuity | ISO short → opto LED rail problem |
| **CN39 pin 1** (VFD_10V) | **GND** | NO continuity | Spindle output short |

**What to do if any test beeps:**
1. **DO NOT POWER ON**
2. Inspect the PCB under a loupe — look for solder bridges on power pins
3. If the bridge is on the QFN-80 (RP2350) → rework with flux + braid + hot air
4. If the bridge is under a TH component → desolder, clean, resolder

### 1b. TH component polarities (visual check)

Before power-on, visually verify **one more time**:

- ✅ **D7 Zener** — the cathode (black band on the body) faces the **SPIN_10V** net (the silkscreen marks the cathode)
- ✅ **C7 100µF electrolytic** — positive lead (longer) on the **+5V** side (silkscreen: + symbol)
  - The **negative lead** has a white band on the can (cylindrical body)
- ✅ **U3 B2424S** — pin 1 marker (dot or "1") aligned with the PCB silkscreen

**A C7 polarity mistake = explosion at power-on.** Check TWICE.

### 1c. Connector seating
- ✅ All KEFA connectors (CN22-CN42) sit firmly with no loose pins
- ✅ RJ1 RJ45 correctly oriented, LEDs facing outward
- ✅ MicroSD slot (CARD1) clicks via the push-push mechanism

---

## ⚡ Step 2: First power-on (with a current-limited supply)

> 💡 **Pro tip:** if you have a lab power supply with a **current limit** feature, set the limit to **500mA** on the first try. If the board draws more than that → something is shorted, and the PSU will fold back before anything is destroyed.

### 2a. Prepare the power supply
- **24V DC, 2A min** (Mean Well RS-50-24 or similar)
- **Polarity:** red wire = **+24V**, black = **GND** (or blue/brown on industrial ones)
- 🔴 **WARNING: the 24V input has NO reverse-polarity protection.** Swapping +24V and GND on CN42 **will damage the board**. Triple-check polarity with a multimeter before every power-on.
- Connect to **CN42**:
  - **Pin 1 = +24V**
  - **Pin 2 = GND**

### 2b. Power ON

1. Keep the board away from metal surfaces (ESD safety)
2. **Turn on the power supply** (flip the switch or plug in the cable)
3. **Watch the LEDs** in the next 2 seconds:

### 2c. LED indicator sequence (expected)

> **Naming note:** the BOM uses `LED1-LED6` (KT-0805G) for the power LEDs.
> `D1` is the **SS54 ORing Schottky on the +5V rail** (TPS5430 buck → L3 → D1 → +5V; it blocks the USB-C 5V from back-feeding the buck). It is **not** a 24V reverse-polarity diode, and not a LED.

| LED | Location | When it lights | What it indicates |
|---|---|---|---|
| **LED1 - 24V power** | Near CN42 | Immediately at power-on | +24V rail OK |
| **LED2 - 5V power** | Near TPS5430 | ~10ms later | +5V buck working |
| **LED3 - 3V3 power** | Near AMS1117 | ~50ms later | +3.3V LDO working, MCU powered |
| **LED4 - 24V_ISO** | Near U3 | ~100ms later | ISO rail active (opto supply) |
| **LED5 - USB activity** | Near USB-C | OFF (no USB cable) | Off is OK with no PC connection |
| **LED6 - VFD output** | Near U23 (LM358) | Off at idle | Brightens with spindle PWM (0-10V output) |

**If any of LED1-LED4 does not light up:**
- **Power off immediately** (cut power)
- See the Troubleshooting section below

### 2d. Sensory check (smell + touch)

In the first 30 seconds:
- ❌ **NO burning smell** or smoke from any component
- ❌ **NO "spuckling" sounds** (sign of a short that arcs)
- ✅ **Components are COLD** or at most mildly warm:
  - TPS5430 (U1) and AMS1117 (U2) can be **pleasantly warm** (~40°C) under normal load
  - RP2350B (U4) should be **cold** (idle ~32°C)
  - LM358 (U23) should be **cold**

**If any component becomes too hot to touch (>60°C) within 30 seconds:**
- **CUT POWER IMMEDIATELY**
- A short circuit or misplaced component is the likely cause

---

## 🔬 Step 3: Voltage verification (multimeter)

**Multimeter in DC Voltage mode (0-50V range).**

### 3a. Power rail measurements

With **power on**, measure the following (black probe on GND, red on the test point):

| Test point | Expected | Tolerance | Action if out of spec |
|---|---|---|---|
| **CN42 pin 1 (24V_IN)** | 24.00V | 23.5 - 24.5V | Check the power supply |
| **+5V rail (LM358 pin 8)** | 5.00V | 4.95 - 5.05V | TPS5430 buck problem |
| **+3.3V rail (RP2350 pin 50)** | 3.30V | 3.25 - 3.35V | AMS1117 LDO problem |
| **+24V_ISO (U3 pin 3)** | 24.00V | 23.0 - 25.0V | B2424S isolated DC/DC problem |
| **CN39 pin 1 (VFD_10V)** | 0.00V | 0 - 0.1V | LM358 output stuck — at 0% PWM it should be 0V |

### 3b. Ground reference verification (input-side isolation)

**v1 uses optical isolation ONLY on the inputs** (CN22-CN31). Stepper outputs, VFD, AUX (SSR), and communication share the MCU ground. The test verifies the input-side barrier:

| Test point 1 | Test point 2 | Expected | What it means |
|---|---|---|---|
| **GND (USB shell)** | **GND_ISO (CN23 pin 3, isolated input side)** | **NO continuity** in DC | Input-side barrier works ✓ |
| **CGND (RJ45 shell)** | **GND** | ~0V through 1MΩ + 1nF/2kV | Bob Smith termination works |
| **GND (USB shell)** | **GND (CN40 pin 5, AUX)** | **HAS continuity** (≈0Ω) | Output side shares MCU GND (by design) ✓ |

⚠️ **Do not look for GND on a stepper connector.** CN34-CN38 are `+5V · STEP · DIR · EN` —
common anode, no ground pin (CN34 pin 4 is `EN_X_5V`). Probing pin 4 against USB GND will
**not** read 0 Ω, and that is correct behaviour, not a fault. CN40 pin 5 is a real GND on the
same non-isolated side, so it is the point that answers this test.

**If GND and GND_ISO have continuity (the buzzer beeps):**
- The input-side isolation barrier is breached
- Probably a solder bridge across the 2mm void barrier
- See Troubleshooting

**Note:** in v1 the stepper, VFD, and AUX output sides share the MCU ground (only the input side is isolated).

---

## 💾 Step 4: USB enumeration test

### 4a. Connect the USB-C cable
- PC side of the USB-C cable into the computer
- RectaBot side into the USB-C port on the board

### 4b. Observe
- **LED5 (USB activity LED)** should light up
- Windows/macOS should recognize a **new USB device**:
  - **Without firmware:** "USB Device" with unknown VID/PID (normal)
  - **With BOOTSEL active:** "RPI-RP2" Mass Storage Device

### 4c. BOOTSEL test
1. Press and hold the **BOOT button** (marked B on the PCB)
2. Press and release the **RESET button** (marked R on the PCB)
3. Release the BOOT button
4. **Expected:** Windows plays a "USB connected" sound, the `RPI-RP2` drive opens

**If BOOTSEL doesn't work:**
- The BOOT button is poorly soldered — check connectivity
- The RESET button does not make a clean contact — check continuity with the multimeter

---

## 🌐 Step 5: Ethernet PHY test (optional)

### 5a. Visual check
1. Connect an **RJ45 patch cable** (Cat5e or Cat6) between RJ1 and your router
2. **LINK LED on RJ1** should light up (green)
3. **ACT LED on RJ1** should blink (yellow)

### 5b. Without firmware
Without firmware, the W5500 doesn't initialize, but the RJ45 magnetics work at the physical level — the LINK LED should light up if the hardware is OK.

### 5c. With firmware (later)
RectaBot firmware comes up on a **fixed address, not DHCP**: `192.168.5.1 / 255.255.255.0`,
hostname `rectabot`. Give the PC an address on the same subnet (e.g. `192.168.5.20/24`) and:
```
ping 192.168.5.1
```
A router is not needed — a direct cable between the PC and RJ1 works. If the board answers
nowhere, it is on DHCP: either the address was changed by hand (`$300..$304`) or the image
predates 2026-07-29, when the static default first actually took effect. Connect over USB and
read `$$` to find out which.

---

## 🆘 Troubleshooting

### Problem: LED1 (24V power) does not light up
**Causes:**
- Wrong power-supply polarity (CN42 pin 1 and 2 swapped) — 🔴 **the 24V input has NO reverse-polarity protection; reversing it can damage the board.** Disconnect and check the board before retrying.
- Trace broken between CN42 and the power tree
- LED1 or its series resistor damaged

**Solution:**
1. Multimeter on the CN42 pins — verify 24V with correct polarity (+24V on pin 1, GND on pin 2)
2. ⚠️ If you applied reverse polarity, there is no protection — inspect the board for damage before powering again
3. Inspect the CN42 solder joints

---

### Problem: 24V LED is lit, but 5V LED is not
**Causes:**
- TPS5430 buck converter (U1) has a problem
- Inductor L1 (22µH) cold joint or soldered incorrectly
- VSENSE divider (resistors) wrong → buck does not enter switching mode

**Solution:**
1. Multimeter on U1 pin 7 (VIN) — should be 24V ✅
2. Multimeter on U1 pin 8 (PH) — should show switching ~50% duty at ~500kHz
3. Multimeter on the L1 output — should be 5V DC
4. If L1 is cold and there is no voltage → reflow L1 with hot air

---

### Problem: 5V lights up, 3.3V does not
**Causes:**
- AMS1117 LDO (U2) cold joint
- Output capacitor (C19 or similar) is shorted to GND
- RP2350B power pins draw too much current (short on the chip)

**Solution:**
1. Multimeter on U2 pin 3 (VIN) — should be 5V
2. Multimeter on U2 pin 2 (VOUT) — should be 3.3V
3. If VIN=5V and VOUT=0V → AMS1117 is damaged or the output is shorted
4. Cut power and run a continuity test between 3.3V and GND — if it beeps, look for a short on the MCU side

---

### Problem: 24V_ISO LED does not light up
**Causes:**
- B2424S-2WR3 (U3) cold joint on its 4 pins
- VCC side (24V_IN) has no voltage on pin 1
- Output side shorted to GND_ISO

**Solution:**
1. Multimeter on U3 pin 1 (VIN+) — should be 24V
2. Multimeter on U3 pin 2 (VIN-) — should be GND
3. Multimeter on U3 pin 3 (VOUT+) — should be 24V (no load) or 23V (with LED load)
4. If pin 3 = 0V → U3 is defective or soldered incorrectly

---

### Problem: Component gets hot
**Most common causes:**
- **RP2350B hot** → short on a 3.3V VDD pin, or a pad bridge on the QFN-80
- **TPS5430 hot (>60°C)** → 5V rail is overloaded (5V → GND short)
- **AMS1117 hot** → 3.3V rail is overloaded
- **B2424S hot** → 24V_ISO short

**Solution:**
1. **Cut power IMMEDIATELY**
2. Wait 1-2 minutes to let it cool down
3. Multimeter continuity test for all power rails (see Step 1a)
4. Visually look for solder bridges on the overheating chip
5. If you find no short → the chip is probably damaged (replace)

---

### Problem: BOOT/RESET doesn't enter BOOTSEL
**Causes:**
- Pull-up resistors missing on BOOT/RESET pins
- Side-tactile buttons whose mechanism doesn't click reliably
- 100nF debounce cap missing

**Solution:**
1. Multimeter continuity on the BOOT button — press and release, listen for the click
2. Verify the 10kΩ pull-up on the BOOT pin
3. If BOOTSEL doesn't work → flash via the **SWD interface** (H1 header) using **picoprobe** or **J-Link**

---

## ✅ Power-On Checklist (sign off when done)

- [ ] Pre-power continuity tests passed — no short circuits
- [ ] D7 cathode orientation verified
- [ ] C7 polarity verified
- [ ] 24V power-on with no sparks, smoke, or hot components
- [ ] LED1-LED4 are all active
- [ ] +5V rail = 5.00 ± 0.05V
- [ ] +3.3V rail = 3.30 ± 0.05V
- [ ] +24V_ISO rail = 24.0 ± 1.0V
- [ ] Input-side isolation OK (GND ≠ GND_ISO, no continuity)
- [ ] USB enumeration OK (PC sees the device)
- [ ] BOOTSEL enters Mass Storage mode (`RPI-RP2`)

**If all 10 boxes are ✅** → the board is ready for firmware flash and configuration.

Next step: [Quick_Start_Guide.md](Quick_Start_Guide.md) Step 5 (Firmware Flash).

---

## 📝 First Power-On Log Template

Fill this in as you power on your board for the first time — a handy record of the checks below:

```
RectaBot v1.0 — First Power-On Log
==================================
Date: ___________
Board #: ____
Inspector: ___________

Pre-power tests:
  [ ] 24V-GND short test: PASS / FAIL
  [ ] 5V-GND short test: PASS / FAIL
  [ ] 3.3V-GND short test: PASS / FAIL
  [ ] D7 polarity: VERIFIED
  [ ] C7 polarity: VERIFIED

Power-On:
  LED1 (24V):  ON / OFF
  LED2 (5V):   ON / OFF
  LED3 (3V3):  ON / OFF
  LED4 (ISO):  ON / OFF
  LED5 (USB):  ON / OFF
  LED6 (VFD):  ON / OFF

Voltage measurements:
  +5V rail:    _____ V (spec: 4.95-5.05)
  +3.3V rail:  _____ V (spec: 3.25-3.35)
  +24V_ISO:    _____ V (spec: 23-25)

USB Enumeration:  PASS / FAIL
BOOTSEL test:     PASS / FAIL
Ethernet LINK:    PASS / FAIL / N/A

Notes / Issues:
________________________________
________________________________
```

---

*Stay careful. Don't rush. The multimeter is your best friend.* 🔬⚡
