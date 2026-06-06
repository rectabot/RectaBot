# RP2350B Pinmap (RectaBot v1.0)

The RP2350B (QFN-80) has 48 GPIO pins (GP0-GP47).

**Level shift 3.3V → 5V:** Stepper signals (STEP/DIR/EN) and coolant outputs (MIST/FLOOD/VAC) pass through the **74HC14D** (Schmitt-trigger HEX inverter, Basic part on JLCPCB). Three 74HC14D chips (U7, U10, U15) cover 18 signals.

**CRITICAL: the 74HC14D INVERTS the signal.** The following must be set in grblHAL firmware:
- `$2=31` — STEP_INVERT_MASK (X|Y|Z|A|B)
- `$3=31` — DIRECTION_INVERT_MASK
- `$4=1`  — STEPPER_ENABLE_INVERT_MASK
- Coolant invert via driver configuration

Stepper drivers (DM556) operate in **Common Anode** mode: +5V on OPT+, the 74HC14D pulls OPT- to GND.

**Input galvanic isolation:** 10 isolated inputs via **LTV-217-B-G** optocouplers (U8-U20), powered from **+24V_ISO** (B2424S-2WR3 isolated DC/DC). GND_ISO is separated from GND by a 2mm copper void barrier.

| Function | RP2350 Pin | Notes |
| :--- | :--- | :--- |
| **Auxiliary outputs** | | Through 74HC14D FREE channels → SSR |
| MIST | `GP0` | Through 74HC14D (inverted) → SSR, grblHAL M7/M9 |
| FLOOD | `GP1` | Through 74HC14D (inverted) → SSR, grblHAL M8/M9 |
| VAC | `GP2` | Through 74HC14D (inverted) → SSR, grblHAL M62/M63 |
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
| A_MIN | `GP36` | A-axis limit switch |
| B_MIN | `GP37` | B-axis limit switch (Y2 auto-square) |
| ESTOP | `GP38` | Emergency Stop → grblHAL alarm |
| PROBE | `GP39` | Tool length probe |
| CYCLE_START | `GP40` | Cycle start |
| FEED_HOLD | `GP41` | Pause |
| DOOR | `GP42` | NC serial chain of mechanical MAX switches (X/Y/Z/A/B_MAX) → alarm if an axis loses steps. MIN positions use inductive NPN sensors on individual channels. |
| **SD card (SPI1)** | | |
| SD MOSI | `GP43` | SPI1 TX |
| SD MISO | `GP44` | SPI1 RX |
| SD CS | `GP45` | SPI1 CSN |
| SD CLK | `GP46` | SPI1 SCK |
| VBUS_DET | `GP47` | USB VBUS detect, 10kΩ/10kΩ divider (R6/R8) → 2.5V (above VIH=2.15V) |
