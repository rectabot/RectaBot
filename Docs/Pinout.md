# RP2350B Pinmap (RectaBot v1.0)

RP2350B (QFN-80) ima 48 GPIO pinova (GP0-GP47).

**Level shift 3.3V → 5V:** Stepper signali (STEP/DIR/EN) i Coolant izlazi (MIST/FLOOD/VAC) idu kroz **74HC14D** (Schmitt-trigger HEX inverter, Basic na JLCPCB). Tri 74HC14D čipa (U7, U10, U15) pokrivaju 18 signala.

**KRITIČNO: 74HC14D INVERTUJE signal.** U grblHAL firmware-u moraju biti postavljeni:
- `$2=31` — STEP_INVERT_MASK (X|Y|Z|A|B)
- `$3=31` — DIRECTION_INVERT_MASK
- `$4=1`  — STEPPER_ENABLE_INVERT_MASK
- Coolant invert kroz driver konfiguraciju

Stepper drajveri (DM556) rade u **Common Anode** modu: +5V na OPT+, 74HC14D povlači OPT- ka GND.

**Galvanska izolacija ulaza:** 10 izolovanih ulaza preko **LTV-217-B-G** optokaplera (U8-U20), napajanih sa **+24V_ISO** (B2424S-2WR3 izolovan DC/DC). GND_ISO razdvojen od GND preko 2mm copper void barijere.

| Funkcija | RP2350 Pin | Napomena |
| :--- | :--- | :--- |
| **Auxiliary izlazi** | | Kroz 74HC14D FREE kanale → SSR |
| MIST | `GP0` | Kroz 74HC14D (invertovano) → SSR, grblHAL M7/M9 |
| FLOOD | `GP1` | Kroz 74HC14D (invertovano) → SSR, grblHAL M8/M9 |
| VAC | `GP2` | Kroz 74HC14D (invertovano) → SSR, grblHAL M62/M63 |
| SD Card Detect | `GP3` | MicroSD socket CD switch (active LOW kad kartica ubačena) |
| **W5500 Ethernet (SPI0)** | | |
| MISO | `GP4` | SPI0 RX |
| SCSn | `GP5` | SPI0 CS |
| SCLK | `GP6` | SPI0 SCK |
| MOSI | `GP7` | SPI0 TX |
| **X Osa** | | Kroz 74HC14D |
| STEP_X → STEP_X_5V | `GP8` | 74HC14D ulaz/izlaz |
| DIR_X → DIR_X_5V | `GP13` | 74HC14D ulaz/izlaz |
| EN_X → EN_X_5V | `GP18` | 74HC14D ulaz/izlaz |
| **Y Osa** | | Kroz 74HC14D |
| STEP_Y → STEP_Y_5V | `GP9` | 74HC14D ulaz/izlaz |
| DIR_Y → DIR_Y_5V | `GP14` | 74HC14D ulaz/izlaz |
| EN_Y → EN_Y_5V | `GP19` | 74HC14D ulaz/izlaz |
| **Z Osa** | | Kroz 74HC14D |
| STEP_Z → STEP_Z_5V | `GP10` | 74HC14D ulaz/izlaz |
| DIR_Z → DIR_Z_5V | `GP15` | 74HC14D ulaz/izlaz |
| EN_Z → EN_Z_5V | `GP20` | 74HC14D ulaz/izlaz |
| **A Osa** | | Kroz 74HC14D |
| STEP_A → STEP_A_5V | `GP11` | 74HC14D ulaz/izlaz |
| DIR_A → DIR_A_5V | `GP16` | 74HC14D ulaz/izlaz |
| EN_A → EN_A_5V | `GP21` | 74HC14D ulaz/izlaz |
| **B Osa (Y2 tandem)** | | Kroz 74HC14D |
| STEP_B → STEP_B_5V | `GP12` | 74HC14D ulaz/izlaz |
| DIR_B → DIR_B_5V | `GP17` | 74HC14D ulaz/izlaz |
| EN_B → EN_B_5V | `GP22` | 74HC14D ulaz/izlaz |
| **W5500 Ethernet** | | |
| W5500_INT (INTn) | `GP23` | Interrupt |
| **RS485 Modbus** | UART1 | MAX485 ili ext. modul |
| RS485 TX | `GP24` | UART1 TX → MAX485 DI |
| RS485 RX | `GP25` | UART1 RX ← MAX485 RO |
| RS485 DE/RE | `GP26` | Smer komunikacije |
| **Spindle / Laser** | | |
| Spindle PWM / Laser PWM | `GP27` | PWM za VFD i diodni laser modul (3.3V TTL, deljen na CN39 + CN33). Za VFD ide kroz LM358 0-10V converter; za laser direktno na CN33. |
| RS422 TX (Pendant) | `GP28` | UART0 TX → SP3485 Chip1 (TX hardwired) |
| RS422 RX (Pendant) | `GP29` | UART0 RX ← SP3485 Chip2 (RX hardwired) |
| Spindle Enable (VFD_EN) | `GP30` | Enable VFD → 2N7002 open-drain Q1 → CN39 pin 2 (VFD_EN, active LOW open-drain) |
| Spindle Dir (VFD_DIR) | `GP31` | Direction VFD → 2N7002 open-drain Q2 → CN39 pin 3 (VFD_DIR, active LOW open-drain) |
| W5500_RST (RSTn) | `GP32` | Reset W5500, 4.7kΩ pull-up |
| **Limiti / Kontrolni ulazi** | | Via LTV-217-B-G opto, 24V_ISO |
| X_MIN | `GP33` | Limit switch X osa |
| Y_MIN | `GP34` | Limit switch Y osa |
| Z_MAX | `GP35` | Limit switch Z osa (homing gore) |
| A_MIN | `GP36` | Limit switch A osa |
| B_MIN | `GP37` | Limit switch B osa (Y2 auto-square) |
| ESTOP | `GP38` | Emergency Stop → grblHAL alarm |
| PROBE | `GP39` | Tool length probe |
| CYCLE_START | `GP40` | Pokretanje ciklusa |
| FEED_HOLD | `GP41` | Pauziranje |
| DOOR | `GP42` | NC serijski lanac mehaničkih MAX krajnika (X/Y/Z/A/B_MAX) → alarm ako osa izgubi korak. MIN pozicije koriste induktivne NPN senzore na individualnim kanalima. |
| **SD Kartica (SPI1)** | | |
| SD MOSI | `GP43` | SPI1 TX |
| SD MISO | `GP44` | SPI1 RX |
| SD CS | `GP45` | SPI1 CSN |
| SD CLK | `GP46` | SPI1 SCK |
| VBUS_DET | `GP47` | USB detekcija, razdelnik 10kΩ/10kΩ (R6/R8) → 2.5V (iznad VIH=2.15V) |
