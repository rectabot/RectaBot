# RectaBot v1.0 — Quick Start Guide

**Od neraspakovane ploče do prvog kretanja motora — korak po korak.**

Procenjen vreme: **2-3 sata** (uključujući soldering 28 TH komponenti).

---

## 📦 Šta dobijaš u paketu

### Iz JLCPCB paketa (DHL Express DDP)
- ✅ **5× RectaBot v1.0 PCB** sa SMT montiranim komponentama
- ✅ ENIG zlatni finish (dugotrajan, dobro za lemljenje)
- ❌ **BEZ Through-Hole komponenti** (carinski razlozi)
- 📄 JLCPCB pakovanje + invoice

### Iz LCSC paketa (FedEx International Priority)
- ✅ **28 TH komponenti × 5 setova** + MOQ viškovi
  - Detaljna lista: [Hand_Solder_Components.md](Hand_Solder_Components.md)
- ✅ **Muški KEFA konektori** (KF2EDGK serija) za screw terminals
- 📄 LCSC invoice (čuvati za srpsku carinu!)

### Šta TI treba da imaš (ne dolazi u paketu)

| Alat / Materijal | Detalji |
|---|---|
| **Lemilica** | TS100 / Pinecil / Hakko FX-888D, podešena na 320-350°C |
| **Lemilo** | 60/40 Sn/Pb 0.6-0.8mm sa kalafonijumom (rosin core) |
| **Multimeter** | Bilo koji digitalni, sa continuity i voltage modom |
| **Pinceta** | ESD-safe, za držanje TH komponenti tokom lemljenja |
| **Lupa / mikroskop** | Korisno za QFN-80 RP2350B vizuelnu proveru pre power-on |
| **24V DC napajanje** | 24V / 2A min, npr. Mean Well RS-50-24 ili adapter sa screw terminal |
| **USB-C kabl** | Data + power, za firmware flash i debug |
| **Tester pinovi / Dupont kablovi** | Za probe testiranje rails-a |

---

## 🔍 Korak 1: Vizuelna inspekcija (10 min)

**Pre nego što počneš išta da lemiš:**

### 1a. Top side check
Stavi PCB pod lupu/mikroskop i proveri:

- ✅ Svi QFN-80 (RP2350B) padovi su čistog ENIG finish-a (nema oxidacije)
- ✅ W5500 QFN i SP3485 SOIC-8 paketi pravilno orijentisani (pin 1 marker = donji-levi)
- ✅ Nema **solder bridge-ova** između susednih pinova RP2350B (najveći rizik)
- ✅ LM358 (U23), 2N7002 (Q1/Q2) na svom mestu
- ✅ Sve LED-ice (D1-D6) su na svom mestu i pravilno orijentisane
- ✅ Konektor za TF (microSD) je dobro lemljen, push-push mehanika klikne

### 1b. Bottom side check
- ✅ R36 i C33 su DNP (Do Not Populate) i NEMA ih na donjoj strani — to je očekivano
- ✅ Nema vidnih oštećenja PCB-a (scratches, exposed traces)

### 1c. Šta uraditi ako nađeš problem
- **Solder bridge** → reword sa flux + braid
- **Cold joint** (mat, kvrgava površina) → ponovi lemljenje sa novim flux-om
- **Pomereni IC** → reflow toplim vazduhom (300°C, 30s)
- **Tear / damage na PCB-u** → kontaktiraj JLCPCB support sa fotografijama

---

## 🔧 Korak 2: Hand-Soldering TH komponenti (60-90 min)

**Slijedi redosled iz [Hand_Solder_Components.md](Hand_Solder_Components.md):**

### Faza A: Najniže komponente prvo
1. **D7** Zener 1N4742A — `pazi katoda (traka) ide ka SPIN_10V net-u`
2. **R128** trim pot PV37W — orijentacija nije bitna
3. **H1** SWD header — straight, 4-pin

### Faza B: Srednje visine
4. **C7** elektrolit 100µF — **POLARITET KRITIČAN** — pozitivna noga (duža) ka +5V
5. **C61** ceramic chassis cap 1nF/2kV — Bob Smith capacitor

### Faza C: Više komponente
6. **U3** B2424S-2WR3 — izolovani DC/DC, 4-pin SIP
7. **RJ1** RJ45 J1B1211CCD — pazi shield pinove (chassis ground)

### Faza D: Konektori (najviši, na kraju)
8. **CN22-CN42** screw terminals (KEFA KF2EDGR)
   - Pin 1 svakog konektora **na PCB silkscreen markeru** (debelja crta ili kvadrat)
   - Kratko ali snažno lemljenje — TH connectors trpe mehaničku silu

### ⚠️ Polariteti koje TI MORAŠ pamtiti
| Komponenta | Polaritet info |
|---|---|
| **D7 1N4742A** | Katoda (crna traka) ide ka **SPIN_10V net-u**. Obrnuto = nema zaštite od overvoltage. |
| **C7 100µF** | Pozitivna noga (duža) ka **+5V** strani. Obrnuto = **eksplozija pri prvom power-on-u!** |
| **U3 B2424S** | Pin 1 ima silkscreen tačku — orijentiši po PCB silkscreen-u |
| **RJ1 RJ45** | Pravilno orijentisan = LED-ice gledaju ka ivici PCB-a (spolja) |

---

## 🧪 Korak 3: Pre-Power-On Provera (15 min)

**OBAVEZNO uradi PRE nego što priključiš 24V napajanje!**

Vidi [First_Power_On_Procedure.md](First_Power_On_Procedure.md) za detaljnu proceduru.

**Skraćeno:**
1. Multimeter na continuity mode
2. Proveri da NEMA short:
   - 24V → GND (sondiraj CN42 pin 1 i 2)
   - +5V → GND
   - +3.3V → GND
   - 24V_ISO → GND_ISO
3. Proveri polaritete elektrolita (C7) još jednom vizuelno

---

## ⚡ Korak 4: Prvi Power-On (10 min)

**Detaljna procedura sa očekivanim merama:** [First_Power_On_Procedure.md](First_Power_On_Procedure.md)

**Skraćeno:**

1. **Priključi 24V** na CN42 (pin 1 = +24V, pin 2 = GND)
2. **Posmatraj LED-ice:**
   - 24V power LED → svetli ✅
   - 5V power LED → svetli ✅
   - 3V3 power LED → svetli ✅
   - 24V_ISO power LED → svetli ✅
3. **Multimeter measure:**
   - +5V rail = 4.95 - 5.05V ✅
   - +3.3V rail = 3.25 - 3.35V ✅
   - +24V_ISO rail = 23.5 - 24.5V ✅
4. **NEMA DIM**, NEMA pucketanja, NEMA pregrevanja komponenti

**Ako bilo šta nije OK** → odmah isključi i konsultuj First_Power_On_Procedure.md troubleshooting sekciju.

---

## 💾 Korak 5: Firmware Flash (15 min)

### 5a. Pripremi BOOTSEL mod
1. Pritisni i drži **BOOT taster** na PCB-u
2. Dok držiš BOOT, pritisni i otpusti **RESET taster**
3. Otpusti BOOT taster
4. RP2350B će se enumirovati kao **USB Mass Storage** (drive `RPI-RP2`)

### 5b. Flash grblHAL firmware
1. Priključi USB-C kabl PC ↔ RectaBot
2. Otvori `RPI-RP2` u Explorer-u
3. Drag-and-drop **`grblHAL_RectaBot_v1.0.uf2`** fajl u taj drive
4. Drive nestaje, RP2350B se reboot-uje sa grblHAL-om

### 5c. Verifikuj firmware
1. Otvori serial monitor (PuTTY / Arduino IDE Serial Monitor / minicom)
2. Connect na COM port (115200 baud, 8N1)
3. Trebao bi videti:
   ```
   GrblHAL 1.1f ['$' for help]
   [VER:1.1f.20260101:RectaBot v1.0]
   [OPT:VNMSL,35,1024,16,7]
   ```

---

## ⚙️ Korak 6: grblHAL konfiguracija (20 min)

**Kritični parametri za RectaBot v1.0:**

```
$0=10        ; Step pulse, microseconds
$1=25        ; Step idle delay, milliseconds
$2=31        ; Step pulse invert, mask (KRITIČNO za 74HC14D)
$3=31        ; Step direction invert, mask (KRITIČNO za 74HC14D)
$4=1         ; Invert step enable pin (KRITIČNO za 74HC14D)
$5=0         ; Invert limit pins, boolean
$6=0         ; Invert probe pin
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
$30=24000    ; Maximum spindle speed, RPM (za 24kRPM VFD)
$31=0        ; Minimum spindle speed, RPM
$32=0        ; Laser-mode enable, boolean (ostavi 0 za spindle)
```

**Per-axis konfiguracija** (X osa primer, isto za Y/Z/A/B):
```
$100=80.000  ; X-axis steps per millimeter (zavisi od driver-a)
$110=2000.000 ; X-axis maximum rate, mm/min
$120=200.000  ; X-axis acceleration, mm/sec²
$130=300.000  ; X-axis maximum travel, millimeters
```

---

## 🎮 Korak 7: Prvi Axis Test (10 min)

### 7a. Priključi DM556 driver na CN34 (X osa)
- Pin 1 = STEP_X_5V → DM556 PUL+
- Pin 2 = DIR_X_5V → DM556 DIR+
- Pin 3 = EN_X_5V → DM556 ENA+
- Pin 4 = GND → DM556 PUL-, DIR-, ENA- (common cathode, sve na GND)

**Napomena:** DM556 +5V (zajedničko PUL+, DIR+, ENA+) ide na **+5V rail**, NE na pin 4 GND!

### 7b. Priključi motor na DM556 izlaze
- A+, A-, B+, B- (4 žice NEMA17/23 motor)
- Postavi DM556 micro-step DIP switches (npr. 1600 steps/rev za 8 microsteps)

### 7c. Probaj jog komanda u serial monitoru
```
$J=G91 X10 F500    ; Pomeri 10mm udesno @ 500mm/min
$J=G91 X-10 F500   ; Pomeri 10mm ulevo
```

**Šta očekivati:**
- ✅ Motor se okreće glatko, bez "skipping"
- ✅ Smer prati DIR komandu
- ✅ Zaustavlja se na poziciji
- ❌ Ako trza ili ne kreće → vidi troubleshooting

---

## 🎯 Sledeći koraci

Kad ti X osa radi pouzdano:
1. **Ponovi za Y, Z, A, B** (CN35-CN38)
2. **Priključi limit switch-eve** (CN27-CN31)
3. **Konfiguriši homing** (`$H` komanda)
4. **Priključi VFD** — vidi [VFD_Wiring_Guide.md](VFD_Wiring_Guide.md)
5. **Test G-code execution** sa sender-om (UGS, CNCjs, IO sender)

---

## 🆘 Troubleshooting

### Power rails out of spec
- **+5V < 4.9V** → TPS5430 buck može imati cold joint na L1 ili VSENSE divider
- **+3.3V < 3.2V** → AMS1117 LDO opterećen ili nije dobro lemljen
- **+24V_ISO < 23V** → B2424S DC/DC ne radi ili 24V_IN prenisko

### LED-ice ne svetle
- **Sve LED-ice off** → 24V uopšte ne stiže (proveri polaritet CN42!)
- **Samo 24V LED svetli** → TPS5430 problem
- **24V + 5V LED svetle, 3V3 off** → AMS1117 problem

### USB enumeration ne radi
- **Drive ne nastaje** → BOOT/RESET sekvenca pogrešna, ponovi
- **Drive nastaje pa odmah nestane** → firmware fajl je korumpiran ili pogrešan UF2

### Motor ne kreće
- **0 puls na STEP pin** → grblHAL $2 nije postavljen na 31 (INVERT_MASK)
- **Konstantan HIGH na STEP** → 74HC14D nije dobro lemljen
- **Motor "skipping"** → mikrostep DIP switches ne odgovaraju $100 parametru

---

## 📚 Dalje čitanje

- [Pinout.md](Pinout.md) — kompletna GPIO mapa
- [Hardware_Design_Guidelines.md](Hardware_Design_Guidelines.md) — električni i mehanički dizajn
- [VFD_Wiring_Guide.md](VFD_Wiring_Guide.md) — kako spojiti različite VFD-ove
- [Product_Specification_v1.0.md](Product_Specification_v1.0.md) — kompletan datasheet

---

*Treba ti pomoć? Otvori issue na GitHub-u ili kontaktiraj kingort@gmail.com*
