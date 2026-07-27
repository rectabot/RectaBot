# RectaBot — VFD Spindle Control over Modbus (RS-485)

**Version:** 1.0 | **Hardware:** RectaBot v1.0 | **Firmware:** grblHAL

This guide covers **digital** spindle control: the controller talks to the VFD over
RS-485 / Modbus RTU, so speed, run/stop and direction travel as messages instead of
as an analog voltage and two dry contacts.

> For **analog** control (0-10 V on CN39 + enable/direction lines) see
> [VFD_Wiring_Guide.md](VFD_Wiring_Guide.md). Both methods work; this one is the
> better of the two where the VFD supports it — see below.

---

## Why Modbus instead of 0-10 V

| | 0-10 V analog | Modbus RS-485 |
|---|---|---|
| Speed accuracy | analog chain (PWM → filter → op-amp), a few % error | exact — a number, not a voltage |
| Feedback | none; the sender displays what it *asked for* | real spindle RPM, current, fault state |
| Wiring | 3 signal wires + GND | 2 wires (A/B) + GND |
| Noise immunity | analog line next to stepper cables | differential pair, far more robust |
| Diagnostics | none | VFD faults surface as `ALARM:19` |

The trade-off is setup: the VFD must be told to take its commands from the bus, and
its parameters must describe the motor honestly, because **grblHAL reads the RPM
range out of the VFD** (see the register table below).

---

## 1. Wiring

RS-485 lives on **UART1** and needs a transceiver (MAX485 or a ready-made module):

| Signal | RP2350 pin | To transceiver |
|---|---|---|
| RS485 TX | `GP24` | DI |
| RS485 RX | `GP25` | RO |
| RS485 DE/RE | `GP26` | DE + RE (tied) |

From the transceiver to the VFD:

```
A  ────────────► A+  (or 485+ / RS+ / D+)
B  ────────────► B−  (or 485- / RS- / D-)
GND ───────────► GND (or COM / ACM)   ← do NOT skip this
```

Three rules that account for most "it does not talk" cases:

1. **A/B are a pair, not interchangeable with a swap of convenience.** If nothing
   answers, swapping A and B is the first thing to try — it costs nothing.
2. **Share the ground.** A differential pair still needs a common reference. A
   missing GND gives intermittent, weather-dependent behaviour, which is worse
   than a clean failure.
3. **One twisted pair, away from motor cables.** If the run is long (> 10 m) or the
   bus has several devices, terminate the far end with 120 Ω.

---

## 2. Pick the driver in the firmware

The shipped firmware has all VFD protocols compiled in; you select one at runtime:

| Setting | Meaning |
|---|---|
| `$395` | Spindle type — pick the driver that matches your VFD |
| `$460` | VFD Modbus address (must match the address set in the VFD) |
| `$461` | RPM ↔ Hz factor (only used by drivers that need it) |

`$395` takes effect **after a board reset**. Two drivers cover the Huanyang family:

- **`Huanyang v1`** — the original HY protocol (HY01/HY02/HY03 series, `PD` parameters)
- **`Huanyang P2A`** — the newer P2A generation

Ask the board itself rather than trusting a table — **`$SPINDLESH`** enumerates what
this firmware actually carries, with the id `$395` expects. On the shipped build:

| `$395` | Driver |
|---|---|
| `0` | PWM (analog 0-10 V, the default) |
| `1` | Huanyang v1 |
| `2` | Huanyang P2A |
| `3` | Durapulse GS20 |
| `4` | Yalang YL620A |
| `5` | MODVFD (generic Modbus) |
| `6` | H-100 |
| `7` | Nowforever |

In RectaControl: **Settings → Spindle & laser → Spindle type**, then reset the board.
`$I` lists every registered spindle, so you can confirm the driver came up.

---

## 3. The four registers grblHAL reads from the VFD

This is the part that surprises people. With the Huanyang v1 driver, the controller
**does not** take the speed range from its own settings — it interrogates the VFD at
startup (`spindle/vfd/huanyang.c`):

| Register | Read as | Used for |
|---|---|---|
| **PD144** | rated RPM at 50 Hz | converting every RPM command into Hz |
| **PD005** | max output frequency | the machine's maximum RPM |
| **PD011** | frequency lower limit | the machine's minimum RPM |
| **PD142** | rated motor current | spindle load calculation |

The conversion is `Hz × 10 = RPM × 5000 / PD144`.

> ### ⚠️ The PD144 trap
> If PD144 is left at its factory value (often `1440`, from a 4-pole industrial
> motor), **every commanded speed is wrong**. On a 24 000 RPM spindle, asking for
> 24 000 computes 833 Hz, PD005 clips that to 400 Hz, and the spindle simply runs
> flat out no matter what you command. If that happens, the sender is not at fault
> — PD144 is. (If the read fails entirely the driver falls back to 3000.)

---

## 4. Parameter set — worked example

Machine used for this example: **Huanyang HY03D023B** VFD (3.0 kW, 230 V input)
driving a **2.2 kW / 8 A / 24 000 RPM / 400 Hz** spindle.

### Motor block — from the spindle's nameplate, not the VFD's rating

| Register | Value | Why |
|---|---|---|
| `PD141` | `220` | rated motor voltage |
| `PD142` | `8.0` | rated motor **current** |
| `PD143` | `2` | poles — 24 000 RPM ÷ 400 Hz × 60 → 2-pole |
| `PD144` | `3000` | RPM at 50 Hz = 24 000 ÷ 400 × 50 |

> **Oversized VFD?** Fine, and often sensible — a 3.0 kW drive on a 2.2 kW spindle
> runs cooler and has headroom. But `PD142` must be the **motor's** 8 A, never the
> drive's rating: that number is what protects the spindle from overload, and what
> the load display is scaled against.

### Frequency and voltage block

| Register | Value | Why |
|---|---|---|
| `PD004` | `400` | base frequency |
| `PD005` | `400` | max output frequency → 24 000 RPM |
| `PD011` | `100` | lower limit → 6 000 RPM (see note) |
| `PD008` | `220` | max output voltage |
| `PD014` | `10`–`15` | acceleration time, seconds |
| `PD015` | `10`–`15` | deceleration time, seconds |

> **Why a 6 000 RPM floor:** these spindles are cooled and lubricated by their own
> rotation. Running one at a few hundred RPM to "take it easy" does the opposite.
> Set the floor in the VFD and grblHAL inherits it.

### Communication block

| Register | Meaning |
|---|---|
| `PD001` | run command source → RS-485 |
| `PD002` | frequency source → RS-485 |
| `PD163` | slave address (match `$460`) |
| `PD164` | baud rate |
| `PD165` | data format (8N1, RTU) |

The numeric codes for `PD164`/`PD165` differ between Huanyang revisions and even
between manuals for the same model. **If you already run a working Huanyang on this
bus, read its values off the panel and copy them** — a proven set beats a table.
Otherwise follow the manual that came with your unit.

---

## 5. Commissioning order

Do it in this order; each step is checkable on its own.

1. **Power off.** Wire A/B/GND. Leave the spindle unmounted or the collet empty.
2. Program the VFD: motor block → frequency block → communication block last.
   (Communication last, so you are not fighting a half-configured drive over a bus.)
3. Set `$395` to the matching driver and `$460` to the VFD's address, then **reset
   the board**.
4. Send `$I` — the driver must be listed. No entry means `$395` did not take, or the
   board was not reset.
5. Read `$$`. If min/max RPM now reflect your spindle, the controller is talking to
   the VFD and reading its registers. That is the moment the bus is proven.
6. Command a low speed (e.g. `M3 S6000`), watch the reported RPM settle at the
   commanded value, then `M5`. Only then fit a tool.

### Doing steps 2-6 with no motor connected

Recommended, in fact — settle the parameters and the bus while nothing can spin.

Everything above works with the VFD's **U/V/W empty**: parameters are set from the
panel, and the RPM range grblHAL reads comes from the drive's own registers, not from
the motor. Even a brief `M3 S6000` is meaningful — the RPM the sender displays is the
drive's output frequency, so the whole command path is proven. Only the current
reading stays at zero.

Two things to respect:

- **Insulate the U/V/W leads** if they are already in the terminals. They carry full
  output voltage the moment the drive runs.
- **Before wiring the motor afterwards: kill mains and wait for the DC bus to
  discharge.** The bus capacitors hold a dangerous voltage after the drive is
  switched off — wait for the display to go dark, then several minutes more (if the
  unit has a DC-bus LED, wait for it to go out). This applies every time the spindle
  is rewired, not just during commissioning.

Some drives detect **output phase loss** and fault when run with no motor. On the
cheap V/f Huanyang units this is usually absent, but if a `PHL`/`OPL`-type code
appears, it is expected behaviour — skip the spin test and connect the motor.

If the panel refuses to accept a parameter edit, the drive's parameter lock is set;
check which register that is in the manual for your revision.

---

## 6. Troubleshooting

| Symptom | Cause worth checking first |
|---|---|
| `ALARM:19` (Modbus exception) | wiring: A/B swapped, no shared GND, VFD not powered; then address / baud mismatch |
| Spindle always runs at max | `PD144` (see the trap above) |
| Reported RPM is a fixed multiple of the commanded one | `PD144` again — the ratio tells you the factor |
| Driver not listed in `$I` | `$395` not set, or the board was not reset afterwards |
| Works, then drops out under load | ground/shielding: route the RS-485 pair away from motor cables, terminate 120 Ω |
| Spindle will not go below some RPM | that is `PD011`, working as intended |

**Bus timing note:** Huanyang drives have been measured failing to answer when the
silent period between messages is under 6 ms. The firmware already applies longer
silence timeouts per baud rate for this family, so a Huanyang that does not respond
is a wiring or parameter problem, not a timing one.

---

## 7. Safety

- **E-stop must cut the spindle in hardware.** A Modbus stop is a message; a message
  can be lost. The E-stop chain has to drop the VFD's own enable or its supply, not
  rely on the bus. This is not a redundancy nicety — it is the only stop that works
  when the software is the thing that failed.
- **Direction changes only at standstill.** Reversing a 24 000 RPM spindle through
  zero stresses bearings and the drive; command `M5`, let it coast down, then start
  the other way.
- The spindle's own cooling (air or water) must be running before you cut. A Modbus
  link tells you the VFD is happy, not that the water pump is.
