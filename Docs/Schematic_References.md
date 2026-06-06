# Referentne Šeme i Dizajn Smernice (RectaBot)

U ovom dokumentu nalaze se uputstva i referentne šeme vezane za ključne IC komponente iz našeg `BOM.csv`. Fokusiramo se na maksimalno korišćenje pasivnih SMD komponenti (pretežno 0603 i 0805) iz JLCPCB "Basic" kataloga, jer su one bukvalno besplatne za montažu.

## 1. Naponska grana (24V -> 5V): TPS5430DDAR
**Datasheet:** [TI TPS5430DDAR](https://www.ti.com/lit/ds/symlink/tps5430.pdf)
TPS5430 zahteva nekoliko ključnih pasivnih komponenti da bi radio kao stabilan "Buck" (step-down). 

**Referentna povezanost:**
- **VIN (Pin 7)**: Povezuje se na 24V. Potreban keramički kondenzator prema GND (C_in), obično `10uF / 50V` (0805).
- **EN (Pin 3)**: Može ostati otvoren (Floating) za automatsko paljenje, ili spojiti na VIN preko 100k otpornika.
- **PH (Pin 8)**: Power switch izlaz.
  - Odatle ide **Schottky dioda (npr. SS34 ili B340A)** usmerena *od* GND *prema* PH pinu (Katoda na PH).
  - Odatle ide **Induktor (L1)** od 33 µH do 47 µH (mora izdržati 3A+, pakovanje CD75 ili veće).
- **BOOT (Pin 1)**: Kondenzator između BOOT i PH pina. Standardna vrednost: `10 nF / 50V` (0603).
- **VOUT (iza Induktora L1)**: Potrebna izlazna filtracija, tj. kondenzator `220 uF` (elektrolitski) paralelno sa `10uF` keramikom.
- **VSENSE (Pin 4)**: Dovođenje izlaznog napona preko razdelnika. 
  - Formula: `Vout = 1.221V * (R1/R2 + 1)`. 
  - Za izlaz od 5.0V: **R1 = 10 kΩ**, **R2 = 3.24 kΩ** (Preciznost 1%). 

*Pasiva za BOM:* C_in i C_out komponente se lako pronalaze u JLCPCB Basic katalogu. Otpornici 10k (0603) i 3.24k (0603 1%) su takodje Basic.

## 2. LDO (5V -> 3.3V): AMS1117-3.3
**Datasheet:** [AMS1117](http://www.advanced-monolithic.com/pdf/ds1117.pdf)
Trivijalno i ultra jeftino!
- **IN**: +5V (dolazi sa TPS5430). Potreban `10uF` kondenzator ka GND.
- **OUT**: +3.3V. Potreban `22uF` tantalski ili `10uF` keramički kondenzator ka GND za stabilnost.
- **GND**: Uzemljenje.

## 3. Bufferi za ose (3.3V -> 5V): 74HC14D
RectaBot v1.0 koristi **74HC14D** (Schmitt-trigger HEX INVERTER) umesto 74HCT245. Razlozi: Basic na JLCPCB, čisti Schmitt input filtruje šum, manji paket SOIC-14.

**Datasheet:** [74HC14D](https://assets.nexperia.com/documents/data-sheet/74HC_HCT14.pdf)
- **VCC (Pin 14)**: +5V (dodati `100nF` decoupler jako blizu čipa).
- **GND (Pin 7)**: Uzemljenje.
- **Ulazi 1A, 2A, ..., 6A**: Povezani na MCU GP signale (3.3V TTL). 74HC14D ulazi prihvataju 3.3V kao logičku 1 jer Schmitt prag (VT+ ~3V na VCC=5V) može biti pređen sa 3.3V. **NAPOMENA:** strogo gledano 74HC14D treba VIH≥3.5V na VCC=5V; u praksi 3.3V radi pouzdano, ali za maksimalnu sigurnost može se koristiti 74**HCT**14D (TTL-kompatibilan, VIH=2V).
- **Izlazi 1Y, 2Y, ..., 6Y**: INVERTOVANI 5V izlaz → ide na STEP-/DIR-/EN- pinove DM556 (kroz serijski 330Ω). Zajednička žica DM556 ide na **+5V** rail (Common Anode topologija).
- **Tri 74HC14D čipa** (U7, U10, U15) pokrivaju 18 signala: 5×STEP + 5×DIR + 5×EN + 3×coolant (MIST, FLOOD, VAC).
- **KRITIČNO za firmware:** 74HC14D **invertuje** sve signale. U grblHAL-u postaviti:
  - `$2=31` — STEP_INVERT_MASK za sve 5 osa (X|Y|Z|A|B)
  - `$3=31` — DIRECTION_INVERT_MASK
  - `$4=1` — STEPPER_ENABLE_INVERT_MASK
  - Coolant invert kontroliše se kroz `MIST_INVERT` i `FLOOD_INVERT` u driver konfiguraciji

## 4. Ethernet kontroler: WIZnet W5500
**Datasheet:** [W5500](https://www.wiznet.io/wp-content/uploads/wiznethome/Chip/W5500/Document/W5500_ds_v109e.pdf)
Ethernet izviđa nešto složenije rutiranje za MDI/MDO linije.
- Standardno koristi 25 MHz kristal (npr. HC49/US, čest JLCPCB u Extended).
- **RJ45 konektor**: Najlakše je korisiti konektor sa integrisanom magnetikom (npr. HanRun HR911105A), što znatno olakšava posao tj. ne moraš da razvlačiš komponente po pločici za izolacioni transformator.
- **Napajanje**: Pokreće se sa 3.3V, obavezno posuti `100nF` decoupling kondenzatore oko svakog pina za VCC na čipu! W5500 povlači do 150mA pod punim opterećenjem.
- **Reference šema**: Prati Figuru 29 (strana 61) u zvaničnom W5500 datasheetu iznad ("W5500 Reference Schematic with RJ45"). Odatle prepisuješ pasivu: `49.9 Ohm` i `33 Ohm` otpornike koji su u JLCPCB `Basic` dostupnosti.

## 5. Komunikacija: SP3485EN-L/TR (RS485 za VFD i RS422 za Pendant)
RectaBot v1.0 koristi **SP3485EN-L/TR** (MaxLinear) — 3.3V RS-485/RS-422 transceiver. Prednost nad MAX485-om: radi direktno na 3.3V (nema potrebe za level shift), niska potrošnja, do 10 Mbps. **Datasheet:** [SP3485](https://www.maxlinear.com/document/2098)

**RS485 (Half Duplex za VFD/Modbus)** — 1× SP3485 (U19)
- Napajanje **3.3V** (i `100nF` decoupler blizu pina 8).
- RO (pin 1) na `GP25` (RP2350 RX).
- DI (pin 4) na `GP24` (RP2350 TX).
- RE (pin 2) i DE (pin 3) spojeni jedan za drugi i povezani na Modbus Direction pin `GP26`.
- A (pin 6) i B (pin 7) na konektor CN22; **120Ω** terminator preko jumper-a.
- **TVS zaštita:** `PSM712-LF-T7` (D4) na A/B liniji **uz konektor**, NE uz SP3485.
- Bez galvanske izolacije u v1; ako bude noise problem od VFD-a, razmotri SI8621 + B0505S za v2.

**RS422 (Full Duplex za Pendant ekran)** — 2× SP3485 (U21 TX, U22 RX)
- Iako SP3485 nije nativno RS-422, koristi se kao dva odvojena chip-a u jednosmernom modu:
  - **U21 (TX):** RE=GND (uvek receive disabled), DE=VCC (uvek transmit enabled) → samo šalje
  - **U22 (RX):** RE=GND (uvek receive enabled), DE=GND (uvek transmit disabled) → samo prima
- A/B izlaze U21 šalju ka pendant-u (RX strana pendanta) — diferencijalni par 1.
- Z/Y (A/B iz U22 perspective) prima sa pendant-a (TX strane pendanta) — diferencijalni par 2.
- **120Ω terminator 0603** na ulaznoj strani U22 (R111).
- **TVS zaštita:** `PSM712-LF-T7` (D5, D6) na svakom paru — **uz konektor CN32**.
- Pinovi: `GP28` (UART0 TX → U21 DI), `GP29` (UART0 RX ← U22 RO).

## 6. Spindle 0-10V Converter + EN/DIR (CN39)

RectaBot v1 integriše **0-10V analog speed reference** i **open-drain EN/DIR digital signale** za univerzalnu kompatibilnost sa većinom VFD-ova (Huanyang, Lapond, Delta, Hitachi, itd.).

### CN39 Pinout (4-pin KF2EDGR-3.5)
| Pin | Net | Funkcija | Napon |
|---|---|---|---|
| 1 | `VFD_10V` | Analog speed (0-10V) | 0-10V DC |
| 2 | `VFD_EN` | Run/Enable (active LOW open-drain) | 0V active / floating inactive |
| 3 | `VFD_DIR` | Direction (active LOW open-drain) | 0V active / floating inactive |
| 4 | `GND` | Signal/Analog ground | 0V |

### 6.1 PWM-to-0-10V Converter (LM358DR2G)

**Topologija (non-inverting amplifier sa 2-stage RC filter + Zener clamp):**

- **U23 = LM358DR2G** (Basic, SOIC-8). VCC = 24V_IN (rail-to-rail nije problem na 24V supply).
- **2-stage RC filter:** R116 (10k) + C81 (100nF), R117 (10k) + C83 (100nF) → cutoff 159Hz × 2, attenuacija 88dB na 25kHz PWM.
- **Gain network:** R122 (10k) + R118 (10k trim pot) + R120 (10k) → safe gain range **1.5-3.0** (max 9.9V iz 3.3V PWM).
- **Overvoltage protection:** R123 (1kΩ) serijski + **D7 = 1N4742A 12V Zener (TH, ručno lemljen)** + C84 (10nF) → clamps na 12V max bez obzira na kvar.
- **LED indikator:** U23B unity buffer prati VFD_10V net → R121 (2k) → LED6 (4mA na 10V).

**Kalibracija:** trim pot R118 podesi multimetrom tako da na 100% PWM dobiješ tačno 10.0V na CN39 pin 1.

### 6.2 EN/DIR Open-Drain Buffers (2× 2N7002)

**Topologija (open-drain MOSFET buffer):**

- **Q1, Q2 = 2N7002** (Basic, SOT-23) N-MOSFET, 60V Vds rating.
- **R125, R126 (1kΩ)** gate series — limit struje pri tranzijent.
- **R124, R127 (10kΩ)** gate pull-down — drži MOSFET OFF tokom MCU reset/boot (failsafe).
- **Source** pinovi na GND.
- **Drain** pinovi na CN39 pin 2 (VFD_EN) i pin 3 (VFD_DIR).

**Operativna logika:**
| MCU GP30/GP31 | MOSFET | CN39 pin | VFD vidi |
|---|---|---|---|
| HIGH (3.3V) | ON | Pulled to GND (~0V) | **ACTIVE** (sinking input triggered) |
| LOW (0V) | OFF | Floating (high-Z) | INACTIVE (VFD pull-up drži HIGH) |
| Reset/boot | OFF (pull-down failsafe) | Floating | INACTIVE ✓ |

**Zaštita MCU:** 2N7002 Drain blokira do 60V → ako VFD ima 24V interni pull-up na FOR/REV terminalu, **napon NE dolazi nazad u RP2350B pin**.

### 6.3 VFD Kompatibilnost

Sinking digital inputs (~85% tržišta): Huanyang HY01/HY02, Lapond SVD-EM, ATO, Delta VFD-M/EL, Hitachi WJ200 — **rade direktno** sa 2N7002 open-drain.

Sourcing/PNP digital inputs (industrijski, ~15%): Siemens G120, ABB ACS580, Schneider ATV — treba **eksterni 2-channel relay modul** ($3 na AliExpress). Za v2 planiran **integrisani relay tier** (Omron G6K-2P-Y).

**Detaljne instrukcije po VFD modelu:** vidi [VFD_Wiring_Guide.md](VFD_Wiring_Guide.md).
