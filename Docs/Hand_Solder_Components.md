# RectaBot v1.0 — Hand-Soldered Components Reference

**Komponente koje TI ručno lemiš nakon prijema ploče od JLCPCB-a.**

⚠️ **VAŽNO:** JLCPCB **NE ŠALJE** ove komponente — sve TH parts moraš zasebno naručiti sa LCSC-a.
Potvrđeno preko JLCPCB chata (Desmond, 04.06.2026):
> "If you didn't select them to be assembled at the ordering page, we will not charge them and hence we will not send them to you."

Vidi: [LCSC_Additional_Order.csv](LCSC_Additional_Order.csv) za kompletnu listu sa LCSC kodovima.

---

## 📦 Lista komponenti (po designator-u)

| # | Designator | Vrednost | Footprint | LCSC | Mounting |
|---|---|---|---|---|---|
| 1 | **C7** | 100µF elektrolit | TH BD6.0-P2.50 | C2873969 | Vertikalan, pazi polaritet (-/+ na silkscreen-u) |
| 2 | **C61** | 1nF / 2kV ceramic | TH L6.8-W2.6-P5.08 | C2976624 | Bob Smith chassis coupling |
| 3 | **CN22** | KF2EDGR-3.5-3P | TH 3-pin | C441172 | RS485 Modbus |
| 4 | **CN23** | KF2EDGR-3.5-3P | TH 3-pin | C441172 | FEED HOLD input |
| 5 | **CN24** | KF2EDGR-3.5-3P | TH 3-pin | C441172 | CYCLE START input |
| 6 | **CN25** | KF2EDGR-3.5-3P | TH 3-pin | C441172 | PROBE input |
| 7 | **CN26** | KF2EDGR-3.5-3P | TH 3-pin | C441172 | ESTOP input |
| 8 | **CN27** | KF2EDGR-3.5-3P | TH 3-pin | C441172 | B_LIMIT input |
| 9 | **CN28** | KF2EDGR-3.5-3P | TH 3-pin | C441172 | A_LIMIT input |
| 10 | **CN29** | KF2EDGR-3.5-3P | TH 3-pin | C441172 | Z_LIMIT input |
| 11 | **CN30** | KF2EDGR-3.5-3P | TH 3-pin | C441172 | Y_LIMIT input |
| 12 | **CN31** | KF2EDGR-3.5-3P | TH 3-pin | C441172 | X_LIMIT input |
| 13 | **CN32** | KF2EDGR-3.5-3P | TH 3-pin | C441172 | DOOR input |
| 14 | **CN33** | KF2EDGR-3.5-2P | TH 2-pin | C441171 | Laser PWM / GND |
| 15 | **CN42** | KF2EDGR-3.5-2P | TH 2-pin | C441171 | 24V DC INPUT |
| 16 | **CN34** | KF2EDGR-3.5-4P | TH 4-pin | C441173 | X Stepper (STEP/DIR/EN/GND) |
| 17 | **CN35** | KF2EDGR-3.5-4P | TH 4-pin | C441173 | Y Stepper |
| 18 | **CN36** | KF2EDGR-3.5-4P | TH 4-pin | C441173 | Z Stepper |
| 19 | **CN37** | KF2EDGR-3.5-4P | TH 4-pin | C441173 | A Stepper |
| 20 | **CN38** | KF2EDGR-3.5-4P | TH 4-pin | C441173 | B Stepper (Y2 tandem) |
| 21 | **CN39** | KF2EDGR-3.5-4P | TH 4-pin | C441173 | Spindle VFD (10V/EN/DIR/GND) |
| 22 | **CN40** | KF2EDGR-3.5-5P | TH 5-pin | C441174 | AUX (5V/VAC/FLOOD/MIST/GND) |
| 23 | **CN41** | KF2EDGR-3.5-6P | TH 6-pin | C441175 | RS422 Pendant (24V/TX+/TX-/RX+/RX-/GND) |
| 24 | **D7** | 1N4742A Zener 12V | TH DO-41 | C140853 | Pazi katoda (traka) ide ka SPIN_10V net-u |
| 25 | **H1** | PZ254V-11-04P | TH 4-pin 2.54mm | C2691448 | SWD debug header |
| 26 | **R128** | PV37W103C01B00 trim pot 10k | TH PV37W | C6630214 | Spindle 0-10V gain calibration |
| 27 | **RJ1** | J1B1211CCD | TH RJ45 | C910371 | Ethernet sa integrisanim magnetics |
| 28 | **U3** | B2424S-2WR3 | TH 4-pin DC/DC | C5369487 | Isolated 24V→24V_ISO za opto LEDs |

**Ukupno: 28 komponenti za ručno lemljenje (sve dolaze sa JLCPCB ploče)**

---

## 🛠️ Redosled lemljenja (preporuka)

### Faza 1: Najniže komponente prvo
1. **D7** Zener (savij noge u "P" oblik za stress relief)
2. **R128** trim pot
3. **H1** SWD header

### Faza 2: Srednje visine
4. **C7** elektrolit (polaritet! kraća noga = minus)
5. **C61** ceramic chassis cap

### Faza 3: Više komponente
6. **U3** B2424S DC/DC modul
7. **RJ1** RJ45 konektor (oprezno sa shield pinovima)

### Faza 4: Konektori (najviši)
8. Sve **CN22-CN42** screw terminals (KEFA)

### Faza 5: Provera
- Vizuelno pregled lemljenja
- Kontinuitet test multimetrom (GND ka glavnim pinovima)
- Power-on test bez glavnih komponenti spojenih

---

## ⚠️ Specifične napomene

### **D7 1N4742A Zener — polaritet KRITIČAN**
- Katoda (traka na telu) ide ka **SPIN_10V** (izlazni signal)
- Anoda (bez trake) ide ka **GND**
- Ako obrnuto → Zener nikad ne provodi → nema zaštite za VFD

### **C7 100µF elektrolit — polaritet KRITIČAN**
- Pozitivna noga (duža) ide ka **+5V** strani
- Negativna noga (kraća, sa belom trakom na kućištu) ide ka **GND**
- Ako obrnuto → eksplozija pri prvom power-on-u!

### **RJ1 RJ45 — pazi shield pinove**
- Ima 2 dodatna pina za chassis ground/shield
- Mora biti zalemljeno za EMI performance
- Bob Smith termination radi samo ako je shield povezan

### **R128 trim pot — kalibracija**
- Posle lemljenja: priključi VFD i podesi tako da na max PWM dobiješ tačno 10.0V
- Vidi [VFD_Wiring_Guide.md](VFD_Wiring_Guide.md) za detalje

---

## 📋 Šta TI moraš da kupiš zasebno (NIJE u JLCPCB paketu)

**SVE komponente sa ove liste (28 TH parts) + muški KEFA konektori za mate.**

Vidi: [LCSC_Additional_Order.csv](LCSC_Additional_Order.csv) — kompletan BOM za 5 ploča (~$50 sa muškim konektorima).
