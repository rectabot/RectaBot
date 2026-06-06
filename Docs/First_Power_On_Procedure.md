# RectaBot v1.0 — First Power-On Procedure

**Bezbedno uključivanje ploče prvi put — sa multimetrom u ruci.**

Procenjeno vreme: **15-30 min** (pažljivo, bez žurbe).

> ⚠️ **UPOZORENJE:** Ovo je najrizičniji trenutak ceo proces. Greška ovde može uništiti $77 ploču. Idi **lagano**, prati svaki korak.

---

## 🎯 Cilj ovog dokumenta

1. **Detektovati kritične greške** (short circuits, pogrešne polaritete) PRE nego što struja pređe preko PCB-a
2. **Verifikovati sve napone** posle uključenja
3. **Postaviti baseline** za buduće troubleshooting

---

## 🛡️ Korak 1: Pre-Power Continuity Tests

**Multimeter u Continuity mode (sa zujalicom).**

### 1a. Power rail short circuit testovi

Postavi multimeter sonde između sledećih tačaka i proveri **nema continuity** (multimeter ne pišti):

| Test tačka 1 | Test tačka 2 | Očekivano | Šta znači ako pišti |
|---|---|---|---|
| **CN42 pin 1** (+24V) | **CN42 pin 2** (GND) | NO continuity | 24V short → ne uključuj! |
| **+5V rail** (USB-C VBUS pin) | **GND** (USB-C shell) | NO continuity | 5V short → TPS5430 problem |
| **+3.3V rail** (RP2350 pin 3) | **GND** | NO continuity | 3.3V short → AMS1117 ili MCU pad bridge |
| **24V_ISO** (U3 pin 3) | **GND_ISO** (U3 pin 4) | NO continuity | ISO short → opto LED rail problem |
| **CN39 pin 1** (VFD_10V) | **GND** | NO continuity | Spindle output short |

**Šta uraditi ako bilo koji test pišti:**
1. **NE UKLJUČUJ** napajanje
2. Inspektuj PCB pod lupom — traži solder bridge na power pinovima
3. Ako je bridge na QFN-80 (RP2350) → reword sa flux + braid + hot air
4. Ako je bridge ispod TH komponente → demontiraj, čistiti, vrati

### 1b. Polariteti TH komponenti (vizuelna provera)

Pre power-on-a, **još jednom** vizuelno proveri:

- ✅ **D7 Zener** — katoda (crna traka na telu) ide ka **SPIN_10V** net-u (silkscreen označava katodu)
- ✅ **C7 100µF elektrolit** — pozitivna noga (duža) na **+5V** strani (silkscreen: + simbol)
  - **Negativna noga** ima belu traku na kućištu (cilindrični deo)
- ✅ **U3 B2424S** — pin 1 marker (tačka ili broj 1) usklađen sa PCB silkscreen-om

**Greška na C7 polaritetu = eksplozija pri power-on-u.** Proveri DVAPUT.

### 1c. Connector seating
- ✅ Svi KEFA konektori (CN22-CN42) čvrsto sede, nema labavih pinova
- ✅ RJ1 RJ45 pravilno orijentisan, LED-ice spolja
- ✅ MicroSD slot (CARD1) klikne push-push mehanikom

---

## ⚡ Korak 2: First Power-On (sa current-limited napajanjem)

> 💡 **Pro tip:** Ako imaš laboratorijsko napajanje sa **current limit** funkcijom, postavi limit na **500mA** prvi put. Ako ploča vuče više od toga → nešto je short i power supply će se "fold-back-ovati" pre nego što išta uništiš.

### 2a. Pripremi napajanje
- **24V DC, 2A min** (Mean Well RS-50-24, ili slično)
- **Polaritet:** crveni žica = **+24V**, crni = **GND** (ili plavi/braon u industrijskim)
- Priključi na **CN42**:
  - **Pin 1 = +24V**
  - **Pin 2 = GND**

### 2b. Power ON

1. Drži ploču dalje od metalnih površina (ESD bezbednost)
2. **Uključi napajanje** (prebaci switch ili ukoči kabl)
3. **Posmatraj LED-ice** u sledećih 2 sekunde:

### 2c. LED indikator sekvenca (očekivano)

| LED | Lokacija | Kad svetli | Šta indicira |
|---|---|---|---|
| **D1 - 24V power** | Blizu CN42 | Odmah pri power-on | +24V rail OK |
| **D2 - 5V power** | Blizu TPS5430 | ~10ms posle | +5V buck radi |
| **D3 - 3V3 power** | Blizu AMS1117 | ~50ms posle | +3.3V LDO radi, MCU napaja |
| **D4 - 24V_ISO** | Blizu U3 | ~100ms posle | ISO rail aktivan (opto napajanje) |
| **D5 - USB activity** | Blizu USB-C | OFF (bez USB kabla) | Off je OK kad nema PC veze |

**Ako bilo koja od D1-D4 ne svetli:**
- **Odmah isključi napajanje** (cut power)
- Vidi Troubleshooting sekciju ispod

### 2d. Senzorni provera (njuh + dodir)

U prvih 30 sekundi:
- ❌ **NEMA zaspaljenja** ili dima iz bilo koje komponente
- ❌ **NEMA "spuckling" zvukova** (znak kratkog spoja koji vibrira)
- ✅ **Komponente su HLADNE** ili maksimalno blago tople:
  - TPS5430 (U1) i AMS1117 (U2) mogu biti **prijatno tople** (~40°C) pod normalnim load-om
  - RP2350B (U4) treba biti **hladan** (idle ~32°C)
  - LM358 (U23) treba biti **hladan**

**Ako ijedna komponenta postaje vruća za dodir (>60°C) u 30 sekundi:**
- **CUT POWER ODMAH**
- Verovatno postoji short circuit ili pogrešno lemljena komponenta

---

## 🔬 Korak 3: Verifikacija napona (multimeter)

**Multimeter u DC Voltage mode (0-50V range).**

### 3a. Power rails measurements

Sa **napajanjem uključenim**, izmeri sledeće (crna sonda na GND, crvena na test tačku):

| Test tačka | Očekivano | Tolerancija | Akcija ako out of spec |
|---|---|---|---|
| **CN42 pin 1 (24V_IN)** | 24.00V | 23.5 - 24.5V | Proveri napajanje |
| **+5V rail (LM358 pin 8)** | 5.00V | 4.95 - 5.05V | TPS5430 buck problem |
| **+3.3V rail (RP2350 pin 50)** | 3.30V | 3.25 - 3.35V | AMS1117 LDO problem |
| **+24V_ISO (U3 pin 3)** | 24.00V | 23.0 - 25.0V | B2424S izolovani DC/DC problem |
| **CN39 pin 1 (VFD_10V)** | 0.00V | 0 - 0.1V | LM358 output stuck — sa 0% PWM treba biti 0V |

### 3b. Ground reference verification (input-side isolation)

**v1 koristi optičku izolaciju SAMO na ulazima** (CN23-CN32). Stepper outputs, VFD, AUX (SSR) i komunikacija dele MCU ground. Test proverava input-side barijeru:

| Test tačka 1 | Test tačka 2 | Očekivano | Šta znači |
|---|---|---|---|
| **GND (USB shell)** | **GND_ISO (CN23 pin 3, isolated input strana)** | **NEMA continuity** u DC | Input-side barijera radi ✓ |
| **CGND (RJ45 shell)** | **GND** | ~0V kroz 1Mohm + 1nF/2kV | Bob Smith termination radi |
| **GND (USB shell)** | **GND (CN34 pin 4, stepper)** | **IMA continuity** (≈0Ω) | Stepper deli MCU GND (po dizajnu) ✓ |

**Ako GND i GND_ISO imaju continuity (zujalica pišti):**
- Input-side izolaciona barijera je probijena
- Verovatno solder bridge preko 2mm void barijere
- Vidi Troubleshooting

**Napomena za v2:** Stepper, VFD, i AUX strane planirane su za full izolaciju u v2 (zasebni 24V_ISO_OUT rail).

---

## 💾 Korak 4: USB Enumeration Test

### 4a. Priključi USB-C kabl
- PC stranu USB-C kabla u kompjuter
- RectaBot stranu u USB-C port na ploči

### 4b. Posmatraj
- **D5 (USB activity LED)** treba da svetli
- Windows/macOS treba da prepoznati **novi USB uređaj**:
  - **Bez firmware-a:** "USB Device" sa nepoznatim VID/PID (normalno)
  - **Sa BOOTSEL aktivnim:** "RPI-RP2" Mass Storage Device

### 4c. BOOTSEL test
1. Pritisni i drži **BOOT taster** (oznaka B na PCB-u)
2. Pritisni i pusti **RESET taster** (oznaka R na PCB-u)
3. Pusti BOOT taster
4. **Očekivano:** Windows pravi "USB connected" zvuk, otvara se `RPI-RP2` drive

**Ako BOOTSEL ne radi:**
- BOOT taster nije dobro lemljen — proveri konektivnost
- RESET taster ne pravi clean kontakt — proveri continuity multimetrom

---

## 🌐 Korak 5: Ethernet PHY Test (opciono)

### 5a. Vizuelni check
1. Priključi **RJ45 patch kabl** (Cat5e ili Cat6) između RJ1 i router-a
2. **LINK LED na RJ1** treba da svetli (zelena)
3. **ACT LED na RJ1** treba da trepće (žuta)

### 5b. Bez firmware-a
Ako nema firmware-a, W5500 se ne inicijalizuje, ali RJ45 magnetics rade na fizičkom nivou — LINK LED bi trebao da svetli ako je hardver OK.

### 5c. Sa firmware-om (kasnije)
Posle flash-a grblHAL-a, W5500 dobija IP preko DHCP, možeš pingovati:
```
ping rectabot.local       # mDNS (ako podržano)
ping <ip-from-router>     # direktno
```

---

## 🆘 Troubleshooting

### Problem: 24V LED ne svetli
**Uzroci:**
- Pogrešan polaritet napajanja (CN42 pin 1 i 2 zamenjeni)
- SS54 Schottky dioda (D1) je probijena ili lemljena obrnuto
- Prekid trace-a između CN42 i power tree-a

**Rešenje:**
1. Multimeter na CN42 pinovima — verifikuj 24V sa pravim polaritetom
2. Multimeter na D1 (SS54) — voltage drop ~0.4V u smeru napajanja
3. Inspektuj solder joints CN42

---

### Problem: 24V LED svetli, ali 5V LED ne
**Uzroci:**
- TPS5430 buck konverter (U1) ima problem
- Induktor L1 (22µH) cold joint ili pogrešno lemljen
- VSENSE divider (R-ovi) pogrešan → buck ne uključuje switching

**Rešenje:**
1. Multimeter na U1 pin 7 (VIN) — treba 24V ✅
2. Multimeter na U1 pin 8 (PH) — treba switching ~50% duty na ~500kHz
3. Multimeter na L1 izlaz — treba 5V DC
4. Ako je L1 hladan i nema napona → reflow L1 sa hot air

---

### Problem: 5V svetli, 3.3V ne
**Uzroci:**
- AMS1117 LDO (U2) cold joint
- Output kondenzator (C19 ili sl.) je short na GND
- RP2350B power pinovi vuku previše struje (short na chip-u)

**Rešenje:**
1. Multimeter na U2 pin 3 (VIN) — treba 5V
2. Multimeter na U2 pin 2 (VOUT) — treba 3.3V
3. Ako VIN=5V a VOUT=0V → AMS1117 je oštećen ili output short
4. Cut power, multimeter continuity između 3.3V i GND — ako pišti, traži short na MCU stranu

---

### Problem: 24V_ISO LED ne svetli
**Uzroci:**
- B2424S-2WR3 (U3) cold joint na 4 pina
- VCC strana (24V_IN) nema napon na pin 1
- Output strana short na GND_ISO

**Rešenje:**
1. Multimeter na U3 pin 1 (VIN+) — treba 24V
2. Multimeter na U3 pin 2 (VIN-) — treba GND
3. Multimeter na U3 pin 3 (VOUT+) — treba 24V (no load) ili 23V (sa LED load)
4. Ako pin 3 = 0V → U3 je defektan ili lemljen pogrešno

---

### Problem: Komponenta postaje vruća
**Najčešći uzroci:**
- **RP2350B vruć** → short na 3.3V VDD pinu, ili pad bridge na QFN-80
- **TPS5430 vruć (>60°C)** → 5V rail je preopterećen (short na 5V → GND)
- **AMS1117 vruć** → 3.3V rail je preopterećen
- **B2424S vruć** → 24V_ISO short

**Rešenje:**
1. **Cut power ODMAH**
2. Sačekaj 1-2 min da se ohladi
3. Multimeter continuity test svih power rails (vidi Korak 1a)
4. Vizuelno traži solder bridge na pregrejavajući chip
5. Ako ne nađeš short → chip je verovatno oštećen (replace)

---

### Problem: BOOT/RESET ne radi BOOTSEL
**Uzroci:**
- Pull-up otpornici na BOOT/RESET pinovima
- Tasteri su Side-Tactile i mehanika ne klikne dobro
- 100nF debounce kapica fali

**Rešenje:**
1. Multimeter continuity na BOOT taster — pritisni i otpusti, čuj klik
2. Verifikuj 10kΩ pull-up na BOOT pinu
3. Ako BOOTSEL ne radi → flashuj preko **SWD interfejsa** (H1 header) koristeći **picoprobe** ili **J-Link**

---

## ✅ Power-On Checklist (potpisni kad završiš)

- [ ] Pre-power continuity tests prošli — nema short circuit-a
- [ ] D7 katoda orijentacija verifikovana
- [ ] C7 polaritet verifikovan
- [ ] 24V power-on bez varnica, dima, ili dela koji greje
- [ ] D1-D4 LED-ice sve aktivne
- [ ] +5V rail = 5.00 ± 0.05V
- [ ] +3.3V rail = 3.30 ± 0.05V
- [ ] +24V_ISO rail = 24.0 ± 1.0V
- [ ] Input-side isolation OK (GND ≠ GND_ISO continuity-wise)
- [ ] USB enumeration OK (PC vidi uređaj)
- [ ] BOOTSEL ulazi u Mass Storage mode (`RPI-RP2`)

**Ako svih 10 box-ova ✅** → Ploča je spremna za firmware flash i konfigurisanje.

Sledeći korak: [Quick_Start_Guide.md](Quick_Start_Guide.md) Korak 5 (Firmware Flash).

---

## 📝 Bring-up Log Template

Predlažem da ti ovo popuniš za svaku od 5 ploča kao bring-up dokumentaciju:

```
RectaBot v1.0 — Bring-Up Log
============================
Datum: ___________
Broj ploče: 1 / 5
Inspektor: ___________

Pre-power tests:
  [ ] 24V-GND short test: PASS / FAIL
  [ ] 5V-GND short test: PASS / FAIL
  [ ] 3.3V-GND short test: PASS / FAIL
  [ ] D7 polaritet: VERIFIED
  [ ] C7 polaritet: VERIFIED

Power-On:
  D1 (24V):   ON / OFF
  D2 (5V):    ON / OFF
  D3 (3V3):   ON / OFF
  D4 (ISO):   ON / OFF

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

*Ostani pažljiv. Ne žuri. Multimeter je tvoj najbolji prijatelj.* 🔬⚡
