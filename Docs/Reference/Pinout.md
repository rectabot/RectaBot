# RP2350B Pinmap (RectaBot v1.0)

The RP2350B (QFN-80) has 48 GPIO pins (GP0-GP47).

**Level shift 3.3V → 5V:** Stepper signals (STEP/DIR/EN) and coolant outputs (VAC/FLOOD/MIST) pass through the **74HC14D** (Schmitt-trigger HEX inverter, Basic part on JLCPCB). Three 74HC14D chips (U7, U10, U15) cover 18 signals.

**The 74HC14D inverts the signal, but in Common Anode mode this is already accounted for** — no firmware STEP invert is needed. Validated on bench (TB6600 + NEMA17, smooth both directions, no missed steps up to F2000):
- `$2=0` — STEP_INVERT_MASK. With `$2=0` the 74HC14D output idles HIGH (OPT− high → opto OFF) and each step pulse briefly pulls OPT− low (opto ON) — correct for Common Anode. Setting `$2`≠0 would hold the input opto continuously ON at idle and likely cause missed steps at speed.
- `$3` — DIRECTION_INVERT_MASK is **per-machine** (depends on wiring/mechanics). Jog each axis and flip its bit so X+/Y+/Z+ move the right way — do not treat it as a fixed value.
- `$4=15` — STEPPER_ENABLE_INVERT_MASK. The 74HC14D inverts EN, so enable is inverted on every axis: **7 = 3-axis, 15 = 4-axis, 31 = 5-axis** (reference board is 4-axis → 15).
- `$5=0` — LIMIT invert OFF (the LTV-217 opto inputs already read active-low). `$6=1` — PROBE invert ON.
- Coolant invert via driver configuration

Stepper drivers operate in **Common Anode** mode: +5V on OPT+, the 74HC14D pulls OPT− to GND. Validated with TB6600; verified-under-load values (and any driver-specific differences, e.g. DM-series) to be confirmed when machining.

**Input galvanic isolation:** 10 isolated inputs via **LTV-217-B-G** optocouplers (U8-U20), powered from **+24V_ISO** (B2424S-2WR3 isolated DC/DC). GND_ISO is separated from GND by a 2mm copper void barrier.

**Input wiring (each connector: `24V · GND · SIG`):** the inputs are **sinking (active LOW)** — same convention as the VFD outputs.
- **Mechanical switch** (limit / E-stop): wire between **GND** and **SIG**. The 24V pin is not needed.
- **Inductive sensor — NPN (sinking)**: use all three pins — **24V** (power) + **GND** + **SIG**. The sensor switches SIG to GND.
- **PNP (sourcing) sensors are not directly compatible** (they drive SIG to 24V, so no current flows through the opto LED and the input never sees them) — use an external relay module, as with PNP VFDs.

**AUX output wiring — SSR (`CN40`: `+5V · VAC · FLOOD · MIST · GND`):** the coolant
outputs are **Common Anode, exactly like the stepper drivers**. Wire the SSR control
input between **`+5V` (pin 1)** and the signal pin — **never between the signal pin and
`GND`**.

| SSR control terminal | CN40 pin |
| :--- | :--- |
| `+` | **1** — `+5V` |
| `−` | **2** `VAC` · **3** `FLOOD` · **4** `MIST` |

The 74HC14D inverts, so each coolant pin **idles HIGH (~5 V) and is pulled LOW when the
M-code switches the output on** — measured on the reference board: ~5 V at idle, ~0 V
under `M7`, back to ~5 V after `M9`. Wiring the SSR to `GND` instead inverts the whole
thing: the relay is **energised whenever the machine sits idle** and switches **off** the
moment `M7`/`M8`/`M62` runs. Nothing warns you — the flood pump simply runs all night.

#### ⚠️ Which relays these outputs can actually drive

Each output is a 74HC14D channel behind a **330 Ω** series resistor (`R45`, `R63`, `R91`),
the same arrangement that feeds the optocoupler in a DM556. Two figures from the datasheet
matter and they are not the same thing: **4 mA** is where `VOL` is *guaranteed* — the DC
test point — and **25 mA per pin** is the absolute maximum. Drawing more than 4 mA is not
an overload; it only means the datasheet stops promising a number.

**Measured on the reference board, with a Fotek SSR-40 DA switching a 230 V lamp:**

| | |
| :--- | :--- |
| Pin, open circuit | **5.07 V** |
| Across the relay's control input, with it connected | **3.11 V** |
| Remainder across the 330 Ω → current | 1.96 V → **~5.9 mA** |
| Implied `VOL` (5.07 − 3.11 − 1.96) | **≈ 0 V** — the output is not straining |
| Result | **It switches the lamp.** |

So a Fotek — a "control 3–32 VDC" input, meaning an internal current regulator that needs
3 V at its own terminals — gets what it needs, clearing its 3 V minimum. At 5.9 mA the
output still pulls essentially to ground, and there is roughly 4× headroom to the 25 mA
limit. What the 330 Ω sets is how much voltage is left for the relay, not whether the chip
can cope.

**The load this was drawn for** is anything whose input is an optocoupler — a relay/SSR
module with
a logic-level `IN` pin drawing ≤ 5 mA. Those have a gain stage: a few mA from this pin
lights an LED, whose transistor then pulls the 70–80 mA of coil current from the module's
own `VCC`. Note that DIN-rail interface relays (Phoenix Contact PLC-INTERFACE, Weidmüller
TERMSERIES, Wago 859, Omron G3RV-SR) use that same optocoupler input but are specified for
PLC outputs sourcing half an amp, so many of their 5 V models ask 10–15 mA. Check the
figure, not the brand.

**For a relay that will not trigger** — one needing more voltage than the 330 Ω leaves —
interpose a driver: a P-channel
MOSFET (e.g. AO3401) high side: source to `+5V`, gate to the signal pin, drain to the SSR
input, SSR return to `GND`. The gate draws no current, so the 330 Ω costs nothing and the
relay sees the full 5 V. The active-low signal suits a P-channel part directly.
*(Reasoned from the numbers above, not built and measured.)*

### NC or NO — and why NC is the one to ship

With **`$5=0`** an input reads **triggered when the circuit is OPEN**. So on the
`GND · SIG` wiring above:

| Device | `$5` | Broken wire behaves as |
|---|---|---|
| **NC** (normally closed) — recommended | `0` | **triggered → machine stops** ✅ |
| NO (normally open) | invert that axis | nothing — the fault is invisible ⚠️ |

That is the whole argument for NC: a cut wire, a loose terminal or a corroded
contact looks exactly like a tripped limit, so the machine refuses to move instead
of running blind past its own end stops. A NO switch fails silent — you find out
when the gantry does.

For inductive sensors the same rule picks the suffix: **`LJ12A3-4-Z/AX` (NPN NC)**
matches `$5=0`; `/BX` (NPN NO) needs `$5` inverted for that axis. `/AY` and `/BY`
are PNP and need a relay module either way.

`$5` is a per-axis mask, so a machine may mix types if it must — but a mixed
machine has mixed failure modes, which is its own kind of expensive.

| Function | RP2350 Pin | Notes |
| :--- | :--- | :--- |
| **Auxiliary outputs** | | Through 74HC14D FREE channels + 330 Ω — see *AUX output wiring* above |
| VAC | `GP0` | Through 74HC14D (inverted) → relay module, grblHAL M62/M63 |
| FLOOD | `GP1` | Through 74HC14D (inverted) → relay module, grblHAL M8/M9 |
| MIST | `GP2` | Through 74HC14D (inverted) → relay module, grblHAL M7/M9 |
| SD Card Detect | `GP3` | MicroSD socket CD switch (active LOW when card inserted) |
| **W5500 Ethernet (SPI0)** | | |
| MISO | `GP4` | SPI0 RX |
| SCSn | `GP5` | SPI0 CS |
| SCLK | `GP6` | SPI0 SCK |
| MOSI | `GP7` | SPI0 TX |
| **X axis** | | Through 74HC14D |
| STEP_X → STEP_X_5V | `GP8` | 74HC14D input/output |
| DIR_X → DIR_X_5V | `GP13` | 74HC14D input/output |
| EN_X → EN_X_5V | `GP18` | 74HC14D input/output |
| **Y axis** | | Through 74HC14D |
| STEP_Y → STEP_Y_5V | `GP9` | 74HC14D input/output |
| DIR_Y → DIR_Y_5V | `GP14` | 74HC14D input/output |
| EN_Y → EN_Y_5V | `GP19` | 74HC14D input/output |
| **Z axis** | | Through 74HC14D |
| STEP_Z → STEP_Z_5V | `GP10` | 74HC14D input/output |
| DIR_Z → DIR_Z_5V | `GP15` | 74HC14D input/output |
| EN_Z → EN_Z_5V | `GP20` | 74HC14D input/output |
| **A axis** | | Through 74HC14D |
| STEP_A → STEP_A_5V | `GP11` | 74HC14D input/output |
| DIR_A → DIR_A_5V | `GP16` | 74HC14D input/output |
| EN_A → EN_A_5V | `GP21` | 74HC14D input/output |
| **B axis (Y2 tandem)** | | Through 74HC14D |
| STEP_B → STEP_B_5V | `GP12` | 74HC14D input/output |
| DIR_B → DIR_B_5V | `GP17` | 74HC14D input/output |
| EN_B → EN_B_5V | `GP22` | 74HC14D input/output |
| **W5500 Ethernet** | | |
| W5500_INT (INTn) | `GP23` | Interrupt |
| **RS485 Modbus** | UART1 | MAX485 or external module |
| RS485 TX | `GP24` | UART1 TX → MAX485 DI |
| RS485 RX | `GP25` | UART1 RX ← MAX485 RO |
| RS485 DE/RE | `GP26` | Direction control |
| **Spindle / Laser** | | |
| Spindle PWM / Laser PWM | `GP27` | PWM for VFD and laser diode module (3.3V TTL, shared with CN39 + CN33). For VFD it goes through the LM358 0-10V converter; for laser, directly to CN33. |
| RS422 TX (Pendant) | `GP28` | UART0 TX → SP3485 Chip1 (TX hardwired) |
| RS422 RX (Pendant) | `GP29` | UART0 RX ← SP3485 Chip2 (RX hardwired) |
| Spindle Enable (VFD_EN) | `GP30` | Enable VFD → 2N7002 open-drain Q1 → CN39 pin 2 (VFD_EN, active LOW open-drain) |
| Spindle Dir (VFD_DIR) | `GP31` | Direction VFD → 2N7002 open-drain Q2 → CN39 pin 3 (VFD_DIR, active LOW open-drain) |
| W5500_RST (RSTn) | `GP32` | W5500 reset, 4.7kΩ pull-up |
| **Limits / Control inputs** | | Via LTV-217-B-G opto, 24V_ISO |
| X_MIN | `GP33` | X-axis limit switch |
| Y_MIN | `GP34` | Y-axis limit switch |
| Z_MAX | `GP35` | Z-axis limit switch (homing up) |
| A_MIN | `GP36` | A-axis limit switch — free on 4/5-axis builds, see below |
| B_MIN | `GP37` | B-axis limit switch (Y2 auto-square) |
| ESTOP | `GP38` | Emergency Stop → grblHAL alarm |
| PROBE | `GP39` | Tool length probe |
| CYCLE_START | `GP40` | Cycle start |
| FEED_HOLD | `GP41` | Pause |
| DOOR | `GP42` | Safety door input (grblHAL `SAFETY_DOOR`): opening the door holds the machine until closed. Neutral default (no forced invert) — set `$14` DOOR bit to match the switch (NO/NC). |
| **SD card (SPI1)** | | |
| SD MOSI | `GP43` | SPI1 TX |
| SD MISO | `GP44` | SPI1 RX |
| SD CS | `GP45` | SPI1 CSN |
| SD CLK | `GP46` | SPI1 SCK |
| VBUS_DET | `GP47` | USB VBUS detect, 10kΩ/10kΩ divider (R6/R8) → 2.5V (above VIH=2.15V) |

## A_MIN (`GP36`) is a spare isolated input on a 4-axis machine

A rotary A axis is not homed, so it has no limit switch and its input sits unused. That
makes `GP36` the one 24 V isolated input still free on a fully populated board — every
`AUXINPUT` is spoken for (E-stop, probe, cycle start, feed hold, door) — and it goes
through the same LTV-217 optocoupler as the other limits. A mechanical **tool setter** is
what it is worth spending on: a fixed pad the machine touches off against, which is what
tool-length offsets need and what the probe input alone cannot give you, since the probe
travels with the spindle.

When it is free, and when it is not:

| build | `GP36` |
|---|---|
| 4- and 5-axis, any Y arrangement | **free** — A takes motor 3, so an auto-squared Y2 lands on `GP37` |
| 3-axis, single or plain ganged Y | **free** — nothing claims it |
| 3-axis, auto-squared Y | **taken** — the second Y home switch is here |

grblHAL assigns a ganged motor to the highest channel, which is why the same wire lands on
a different pad depending on axis count. The board map does this for you; the table is
just so you can check the pad before wiring to it.

**The pad is wired; the firmware images do not read it.** None of the published builds
enable a tool setter, so this is a build you make yourself — the hardware is here for it,
which is the part you cannot add later.

Declare `GP36` as an aux input in the board map and define `TOOLSETTER_PIN 36` /
`TOOLSETTER_ENABLE 1`, guarded so an auto-squared 3-axis build does not claim the pad twice,
and grblHAL registers a second probe of its own, separate from `PROBE`. Settings survive the
flash — the tool setter's invert / pull-up / auto-select flags are bits in a word that
already exists, so the settings structure does not grow. See `variants/CONFIG.md` for how a
variant is built, and for the options where that is *not* true.
