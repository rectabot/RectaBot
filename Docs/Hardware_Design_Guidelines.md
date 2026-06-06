# Smernice za dizajn RectaBot hardvera (KiCad / EasyEDA)

Ovaj dokument služi kao referenca pri kreiranju štampane pločice (PCB-a) na osnovu odobrene arhitekture.

## 1. Struktura slojeva (4-Layer PCB)
S obzirom na odluku o 4 sloja i potrebu za smanjenjem EMI smetnji (naročito zbog VFD-a), preporučeni *Layer Stackup* je:
- **Top Layer (Signal / Power routing):** Signali za komponente koje se nalaze na vrhu, rutiranje kratkih logičkih linija i napajanje.
- **Layer 2 (GND Plane):** Solidan (neprekinut) sloj uzemljenja. Ovo je krucijalno za kratke povratne struje (Return Paths) i izolaciju ispod W5500 SPI linija, kao i za hlađenje TPS5430DDAR.
- **Layer 3 (Power Plane / Signal):** Ostrva (polygons) sa +5V i +3.3V napajanjem. Eventualno rutiranje kontrolnih linija.
- **Bottom Layer (Signal):** Rutiranje dužih signalnih linija, npr. od mikrokontrolera do konektora.

## 2. Napajanje (Power Tree)
- **TPS5430DDAR (C9864) Buck Konverter (24V -> 5V):** Dizajnirati prema Texas Instruments preporukama. *Exposed Pad* ispod čipa **mora** biti jako isprošivan via (termalnim prelazima) ka Layer 2 i Bottom sloju kako bi se efikasno rasipala toplota i izbeglo pregrevanje. Induktor treba da bude oklopljen i smešten što bliže čipu.
- **LDO za 3.3V (npr. AMS1117-3.3 - C6186):** Očekuje pad napona sa 5V na 3.3V. S obzirom na mali naponski skok (1.7V razlike), termalni dizajn će biti izuzetno siguran.

## 3. Komunikacija (Izolacija i EMI)
- **RS485 Modbus:** RectaBot v1.0 koristi **SP3485EN-L/TR** transceiver (radi direktno na 3.3V — nema potrebe za level shift kao kod MAX485). TVS zaštita preko **PSM712-LF-T7** dioda na A/B izlazima — postavljene uz konektor, ne uz IC chip. Za dodatnu zaštitu od VFD buke mogu se koristiti digitalni izolatori (npr. SI8621) sa zasebnim izolovanim DC/DC napajanjem (B0505S serija) ako se pojavi noise problem.
- **RS422 (Pendant):** Koristi se SP3485EN-L/TR (dva chipa — jedan TX, jedan RX) za Full Duplex vezu. Vodi računa o prilagođenju i terminacionom otporniku (120Ω) posebno na ulaznoj strani (RX) dugog kabla.
- **W5500 (Ethernet):** Rutovati SPI linije (RX, CS, SCK, TX) držeći ih blizu jedna drugoj i uvek iznad solidnog GND nivoa bez prekida kako bi se sačuvao integritet signala. Smeštaj RJ45 porta preporučljivo što dalje od visokonaponskih izlaza/VFD konektora.

## 4. Ose (5V Common Anode ka DM556)
- GrblHAL RP2350 čip izdaje I/O signale na 3.3V. RectaBot v1.0 koristi **74HC14D** (Schmitt-trigger HEX **inverter**) za level shift 3.3V→5V. Bafere napajati sa +5V sa glavnog buck konvertera. Pažnja: 74HC14D **invertuje signal** — firmware mora imati postavljen `STEP_INVERT_MASK`, `DIR_INVERT_MASK` i `EN_INVERT_MASK` da bi DM556 dobio ispravnu polarnost.
- Kontrolni izlazi (Step, Dir, Enable) kod *Common anode* se obično ponašaju tako što mikrokontroler daje "logičku 0" na izlaz bafera (čime se obezbeđuje GND), dok je drugi izvod optokaplera na DM556 fiksno povezan na +5V.
- Po Schmitt invertovanju (74HC14D), kad MCU postavi GP_STEP na HIGH (3.3V), izlaz iz 74HC14D ide LOW → optokapler DM556 se uključuje → step puls. Zato je `STEP_INVERT_MASK=31` (svih 5 osa) obavezno.
- *Pažnja:* PIO konfiguracija grblHAL-a za step generisanje za RP2040/RP2350 iziskuje **uzastopne (consecutive)** pinove na mikrokontroleru! Za to smo odvojili `GP8` - `GP12` samo za Step pulseve, zatim `GP13` - `GP17` za Dir i `GP18` - `GP22` za Enable signale. Obavezno pratiti ovu numeraciju prilikom crtanja šeme!

### Alternativa za v2 (razmatrana, NE u v1):
- **74HCT245** (octal bus transceiver) — ne invertuje, ali troši više pinova i ne čisti šum kao Schmitt trigger.
- **74LVC07A** (open-drain buffer) — radi sa external pull-up otpornicima na 5V, dodaje 6 R-ova ali ne invertuje.
- Za v1 zadržana **74HC14D** zbog Basic statusa na JLCPCB (besplatna montaža) i čistog Schmitt input-a.

## 5. Spindle Interface (0-10V + EN/DIR)

RectaBot v1 implementira **univerzalni VFD interfejs** na konektoru CN39 (4-pin KF2EDGR-3.5):
- Pin 1: `VFD_10V` — analog speed reference 0-10V DC
- Pin 2: `VFD_EN` — Run/Enable, active LOW open-drain
- Pin 3: `VFD_DIR` — Direction, active LOW open-drain
- Pin 4: `GND` — signal ground

### 5.1 PWM → 0-10V Converter (LM358DR2G)

**Topologija:** Non-inverting op-amp sa dvostepenim RC filterom + Zener clamp + LED indikator.

**Komponente i raspored:**
- **U23 = LM358DR2G** (Basic, SOIC-8) — dual op-amp, VCC = 24V_IN, rail-to-rail nije problem na 24V supply.
- **Filter stage 1:** R116 (10k) + C81 (100nF) — cutoff 159Hz, attenuacija PWM nosioca.
- **Filter stage 2:** R117 (10k) + C83 (100nF) — drugostepeno filtriranje, ukupno 88dB na 25kHz PWM.
- **Gain network:** R122 (10k) + R128 (10k trim pot PV37W) + R120 (10k) → safe gain range **1.5-3.0** (max 9.9V iz 3.3V PWM).
- **Overvoltage clamp:** R123 (1kΩ) serijski + **D7 = 1N4742A 12V Zener (TH, ručno lemljen)** + C84 (10nF) — clamps na 12V max čak i pri kvaru op-ampa.
- **LED indikator:** U23B sekcija konfigurisana kao unity buffer za VFD_10V → R121 (2k) → LED6 (vizuelni indikator stvarne brzine).

**Kalibracija:** trim pot R128 podesi multimetrom tako da na 100% PWM (M3 S24000) dobiješ tačno **10.0V** na CN39 pin 1.

### 5.2 EN/DIR Open-Drain Buffers (2× 2N7002)

**Topologija:** N-MOSFET open-drain, gate pull-down failsafe.

**Komponente:**
- **Q1, Q2 = 2N7002** (Basic, SOT-23) — N-MOSFET, 60V Vds rating, dovoljno za bilo koji industrijski VFD koji ima do 24V interni pull-up.
- **R125, R126 (1kΩ)** — gate series resistor, limit struje pri tranzijent.
- **R124, R127 (10kΩ)** — gate pull-down, drži MOSFET OFF tokom MCU reset/boot (failsafe).

**Operativna logika:**
| MCU pin (GP30/GP31) | MOSFET stanje | CN39 pin napon | VFD vidi |
|---|---|---|---|
| HIGH (3.3V) | ON (zatvoren) | Pulled to GND (~0V) | **ACTIVE** (sinking input triggered) |
| LOW (0V) | OFF (otvoren) | Floating / high-Z | INACTIVE (VFD pull-up drži HIGH) |
| MCU reset/boot | OFF (pull-down failsafe) | Floating | INACTIVE ✓ failsafe |

**Zašto je ovo bezbedno:**
- 2N7002 Drain blokira do **60V** → ako VFD ima 24V interni pull-up, **napon NE dolazi nazad u RP2350B pin**.
- Pull-down R124/R127 garantuje da spindle **nikad ne pokreće sam od sebe** tokom MCU reset-a, BOOT-a, firmware update-a, ili crash-a.
- Open-drain topologija znači da možeš povezati više VFD-ova u paralelu (npr. dual spindle), gde svaki ima svoj pull-up.

### 5.3 PCB Layout napomene za Spindle sekciju

- **Routing PWM signala** (GP27 → R116): kratak trace, NE rutirati preko izolaciono praznine. Spindle PWM ostaje u GND_MCU domeni.
- **U23 LM358 decoupling:** 100nF (C82) jako blizu pina 8 (VCC). Bez ovoga, oscilacije pri visokoj struji LED-a.
- **D7 Zener placement:** ručno lemljen TH (DO-41) → smesti **tačno između R123 i CN39 pin 1**, sa katodom (traka) ka CN39 strani.
- **R128 trim pot:** smesti tako da je adjust screw pristupačan **bez demontaže ploče** — top adjust orijentacija.
- **Ground reference:** CN39 pin 4 (GND) mora biti **dedicated ground trace** ka U23 GND, ne deli sa noisy step/dir ground-om.
- **TVS na VFD_EN/VFD_DIR:** opciono u v1 (2N7002 60V Vds dovoljan), planiran TVS array u v2 za bolju EMC compliance.

### 5.4 VFD Kompatibilnost (kratko)

| VFD Family | Digital Input Type | Radi out-of-the-box? |
|---|---|---|
| Huanyang HY01/HY02 | Sinking (NPN) | ✅ Da |
| Lapond SVD-EM | Sinking (NPN) | ✅ Da |
| Delta VFD-M/EL | Sinking (NPN) | ✅ Da |
| Hitachi WJ200 | Sinking (NPN) | ✅ Da |
| ATO inverter | Sinking (NPN) | ✅ Da |
| Siemens G120 | Sourcing (PNP) | ⚠️ Treba external relay modul |
| ABB ACS580 | Sourcing (PNP) | ⚠️ Treba external relay modul |
| Schneider ATV | Sourcing (PNP) | ⚠️ Treba external relay modul |

**Detaljne instrukcije po VFD modelu:** [VFD_Wiring_Guide.md](VFD_Wiring_Guide.md).

### v2 Planirano poboljšanje
- Dodati **2× Omron G6K-2P-Y relay** opciono za PNP/sourcing VFD kompatibilnost iz kutije (integrisani relay tier).
