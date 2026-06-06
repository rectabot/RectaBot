# RectaBot v2 Pro — Architecture Plan

**Dva glavna differentiator-a:**
1. 🛡️ **Full I/O Galvanic Isolation** — true industrial-grade noise immunity
2. 🎯 **Closed-Loop Encoder Feedback** — world's first hobby grblHAL controller with closed-loop verification

**MCU:** RP2350B (zadržano iz v1) — koristi 2. PIO banku za encoder feedback. **Zero MCU porting risk.**

**Target launch:** Q2 2027 (posle v1 validation + market feedback).

---

## 🧠 MCU Decision Log (05.06.2026)

**Razmatrano i odbačeno za v2:**

- ❌ **NXP i.MX RT1170** — research je pokazao:
  - Nema grblHAL port (3-6 meseci solo porting)
  - Samo BGA-289 paket → 6-layer HDI PCB potreban (JLCPCB Economic ne podržava)
  - $19/chip + ~$3 Extended fee + skuplji PCB = $30-40 BOM increase
  - LCSC stock: 1 komad pre-order
  - **Total: 9-12 meseci bez revenue → dead-end risk**
- ❌ **Renesas RA8M1** — sličan problem, nema grblHAL port
- ⚠️ **STM32H723VGT6** — solid kandidat (postoji grblHAL port, $5.76, LQFP-100), ali ne donosi unique angle koji opravdava jump

**ODABRANO: RP2350B + 2nd PIO bank za encoder feedback**

Razlozi:
- v1 platforma već validirana
- 12 PIO state machines dostupno (v1 koristi samo 5)
- **7 slobodnih PIO SM** → savršeno za 5 axis encoders + spare
- Niko nije pravio commercial grblHAL board sa closed-loop feedback
- Marketing pitch: *"World's first hobby CNC controller with closed-loop encoder feedback. Every move verified."*

---

## 🎯 Closed-Loop Encoder Feedback (NEW v2 Feature)

### Koncept:
Svaka osa dobija quadrature encoder (ABZ signali). RP2350B 2nd PIO bank obrađuje encoder pulses paralelno sa step generation. Firmware može:
- **Verifikuje stvarnu poziciju** vs commanded position posle svakog move-a
- **Detektuje izgubljene korake** (alarm pre nego što obrada propadne)
- **Backlash compensation real-time** (poznavajući stvarnu poziciju)
- **Optionally:** closed-loop PID correction (ako stvarna pozicija drift-uje)

### Hardware potrebno po osi:
- **Optical incremental encoder** sa kvadratnim signal output-om
  - Zajedničke opcije: Omron E6B2-CWZ6C, US Digital E2, generic Chinese 600 PPR encoders
  - Signali: A+, A-, B+, B-, Z+ (index), Z-, +5V, GND
  - Cena: $30-80 po enkoderu (zavisi od PPR-a)
- **Differential receiver IC** (npr. AM26C32 ili SN65LBC179) — pretvara differential signale u single-ended za MCU
- **2 GPIO pina po osi** (A i B kanali, kvadrat)
- **Optional 1 GPIO za index** (Z kanal — homing reference)

### Total dodatne komponente za 5 axes:
- 5× encoder receivers (3-channel ABZ × 5 = 15 channels) → 2× AM26C32 quad receivers
- 15 GPIO pinova (RP2350B ima 48, dovoljno!)
- 5× konektora (može deliti sa stepper konektorima CN34-CN38 ako se proširuje na 8-pin)

### PIO program za enkoder:
```
; Quadrature decoder PIO state machine
; Reads A and B pins, outputs incremental count
; Handles all 4 transition states
; ~120MHz sample rate (PIO clock) = 30M counts/sec — overkill
; Real-world: 1KHz encoder = 1000 transitions/sec, easy
```

grblHAL plugin za encoder feedback **postoji** (`encoder/` u grbl folderu) — ne treba sve od nule, samo prilagoditi PIO driver.

### Sistemska korist:
- **Lost-step detection** = alarm pre nego što G-code ide pogresno (kupcu skupo)
- **Higher feeds safely** = može se gurnuti mašina brže jer detektuje gubljenje
- **Better surface finish** = backlash compensation
- **Diagnostics** = stvarna pozicija u real-time dashboard-u (web/pendant)

### Marketing impact:
- "Closed-loop" je magic reč za professional CNC operatere
- Trenutno samo industrial controllers (Centroid Allin1, Mesa 7i80 + LinuxCNC) imaju
- **Mi bismo prvi sa grblHAL ploča koja podržava ovo nativno**

---

## 📊 v1 vs v2 — Isolation Scope

```
v1 (Current)                          v2 (Target)
─────────────────────                 ──────────────────────────
INPUTS:  ✅ Isolated (LTV-217)        INPUTS:  ✅ Isolated (kept)
                                                  + faster 6N137 for fast signals
OUTPUTS: ❌ Share MCU GND             OUTPUTS: ✅ Isolated (NEW)
  - Stepper (CN34-CN38)                  - Stepper via 6N137/PI3304
  - Spindle (CN39)                       - Spindle via SI8621 + B0505S
  - AUX/SSR (CN40)                       - AUX via opto
COMMS:   ❌ Share MCU GND             COMMS:   ✅ Isolated (NEW)
  - RS485 (CN22)                         - SI8621 isolated transceiver
  - RS422 Pendant (CN41)                 - SI8621 + B0505S
  - Ethernet (RJ45)                      - Kept (transformer-isolated already)
  - USB (USB-C)                          - Kept (no isolation needed for PC)
```

**Result:** v2 = **true industrial galvanic isolation** = differentiator vs konkurencija + zero noise propagation iz VFD/stepper rails-a u MCU.

---

## 🏗️ v2 Power Tree (planiran)

```
24V DC IN ──► SS54 (reverse protection) ──┬──► TPS5430 buck ──► +5V_MCU (MCU domain)
                                          │
                                          ├──► B2424S-2WR3 ──► +24V_ISO_IN (kept, opto LEDs)
                                          │
                                          ├──► B2424S-3WR3 NOVO ──► +24V_ISO_OUT (stepper, spindle)
                                          │     └─► MP1584 buck ──► +12V_ISO_OUT (LM358 supply)
                                          │     └─► AMS1117 LDO ──► +3.3V_ISO_OUT (digital iso side)
                                          │
                                          ├──► B0505S NOVO ──► +5V_ISO_COMM (RS-485/422 transceivers)
                                          │
                                          └──► LM358 supply (24V rail-to-rail, isolated side)

+5V_MCU ──► AMS1117-3.3 ──► +3.3V_MCU (MCU, W5500, USB CDC)
```

**Tri odvojena izolovana domena:**
1. `24V_ISO_IN` (postojeći) — 10 opto inputs
2. `24V_ISO_OUT` **NOVO** — 5 stepper outputs + spindle + AUX
3. `5V_ISO_COMM` **NOVO** — RS-485 i RS-422 transceivers

---

## 🔌 Komponente — Konkretan plan

### NOVE komponente za v2:

| Designator | Component | Function | Cost (LCSC, qty 10+) |
|---|---|---|---|
| **U_iso1-5** | PI3304-WUEX (×5) | Quad digital isolator za STEP/DIR/EN po osi | ~$1.80 × 5 = $9.00 |
| **U_iso6** | SI8621BD-B-IS | Dual digital isolator (RS-485 TX/RX) | ~$1.40 |
| **U_iso7-8** | SI8621BD-B-IS (×2) | RS-422 TX i RX kanali | ~$2.80 |
| **U_iso9** | SI8642BD-B-IS | Quad isolator za Spindle EN/DIR/PWM/STATUS | ~$2.10 |
| **U_iso10-12** | 6N137 (×3) | Coolant outputs M7/M8/M62 (slow opto, jeftino) | ~$0.45 × 3 = $1.35 |
| **U_dcdc2** | B2424S-3WR3 | NEW: 3W isolated DC/DC za output domain | ~$3.20 |
| **U_dcdc3** | B0505S-1WR3 | 1W isolated 24V→5V za comm transceivers | ~$2.10 |
| **U_buck** | MP1584EN-LF | 24V→12V buck za LM358 supply (isolated side) | ~$0.45 |
| **U_ldo2** | AMS1117-3.3 (×2) | 3.3V LDO za isolated digital side | ~$0.08 × 2 = $0.16 |

**Total nove komponente:** ~$22.56 per board (Basic prioritet gde moguće)

### POSTOJEĆE komponente — neke menjamo:

| v1 | → v2 | Zašto |
|---|---|---|
| 74HC14D (×3) | **74HCT08D** ili ostaje | 14D je inverter; HCT08 je buffer (ne invert), 5V output, lakša firmware logika |
| LM358 (24V supply iz MCU rail) | **LM358 (12V_ISO_OUT supply)** | Premešten na izolovani domain |
| 2N7002 (×2) | **Možda ostaje ili PI3304 integrise** | Verifikovati posle layout-a |
| LTV-217-B-G (×10) | **Ostaje + 3× 6N137 za brze signale** | Slow LTV-217 OK za mehaničke kontakte, ne za PWM |

---

## 💰 Cost Impact Analiza

### v1 BOM (procena):
- ~80 komponenti
- Per-board PCBA: **~$42** (5 ploča batch)
- Per-board sa TH parts: **~$77**

### v2 BOM (procena):
- ~95-100 komponenti
- Per-board cost increase: **+$25-30**
- Per-board sa TH parts: **~$105-110**

### Tržišna pozicija:

| Tier | Plan | Target price | Margin |
|---|---|---|---|
| **v1 Standard** | Optical inputs only | $180-200 | Healthy |
| **v2 Standard** | Same as v1 (continue selling) | $150-170 | Lower (matured) |
| **v2 Pro** | Full I/O isolation | **$280-320** | High |
| **v2 Ultimate** | Pro + SmartPendant | **$450-500** | Premium |

**Strategija:** v2 Standard → comodity, v2 Pro → premium. Differentiation kroz Full Isolation badge.

---

## 🗺️ v2 Layout Changes

### Mehaničke promene:
- PCB povećan: **150×100mm → 180×120mm** (više prostora za dodatne komponente i clearance)
- Layer count: **4 → 6** (možda) za bolji EMI control + odvojene power planes
- Solder mask: ostaje 0.030mm (validirano u v1)
- 4-mounting hole pattern (kako v1)

### Layout zone:
```
┌────────────────────────────────────────────────────────────────┐
│  CN42 (24V IN)                                    CN41 (RS422)  │
│  ┌─────────┐                                                    │
│  │ POWER   │  ┌─────────────────┐                               │
│  │ ZONE    │  │  MCU ZONE       │  ┌─────────────────┐         │
│  │ TPS5430 │  │  RP2350B + W5500│  │  COMM ISOLATION │  USB-C  │
│  │ B2424S  │  │  SD card        │  │  SI8621 ×2      │  RJ45   │
│  │ B0505S  │  │  USB CDC        │  │  B0505S         │         │
│  │ MP1584  │  │                 │  │                 │         │
│  └─────────┘  └─────────────────┘  └─────────────────┘         │
│       ═══════════════════════════════════════════                │
│       ⚡ 2mm COPPER VOID — ISOLATION BARRIER ⚡                  │
│       ═══════════════════════════════════════════                │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐           │
│  │ STEPPER  │ │ SPINDLE  │ │ AUX/SSR  │ │ INPUTS   │           │
│  │ ISOLATION│ │ ISOLATION│ │ ISOLATION│ │ ISOLATION│           │
│  │ PI3304×5 │ │ SI8642   │ │ 6N137 ×3 │ │ LTV-217  │           │
│  │ +24V_ISO_│ │ +12V_ISO │ │ +5V_ISO  │ │ ×10      │           │
│  │  OUT     │ │ LM358    │ │          │ │ +24V_ISO │           │
│  └──────────┘ └──────────┘ └──────────┘ │  IN      │           │
│  CN34-CN38   CN39         CN40          └──────────┘           │
│   (5×4P)     (4P VFD)     (5P AUX)      CN23-CN32              │
└────────────────────────────────────────────────────────────────┘
```

---

## 🔬 Tehnička pitanja za rešavanje

### 1. PIO compatibility sa digital isolators?
- PI3304 ima ~10ns propagation delay
- PIO step generation @ 300kHz = 3.3µs period → 10ns je OK (0.3%)
- **Verify:** signal integrity test na prototipu

### 2. Common GND between isolated stepper domains?
- Kupac može povezati DM556 driver-e na zajednički GND napajanja
- To bi mogao biti **soft short** kroz driver-e (bypass barriere)
- **Solution:** preporuka u dokumentaciji — odvojeni napajači po osi ako traže max isolation
- Alternativno: opto-isolated DM556 (već postoji), zadržava barrier

### 3. Spindle PWM signal speed?
- LM358 RC filter (159Hz) je spor — SI8642 ne mora biti najbrži
- **Datasheet check:** SI8642 do 150Mbps, daleko više nego što treba

### 4. USB isolation?
- USB ISO bi zahtevao USB hub IC sa isolation (npr. ADuM4160)
- **Cost:** dodatnih $5-7
- **Use case:** samo industrial customer sa PLC interfejs
- **Decision:** preskakanje u v2, dodavanje u "v2 Ultimate Industrial" varijanti

### 5. Backup za Modbus error tolerance?
- RS-485 SI8621 + B0505S = $4-5
- Worth it za industrial environment
- v2 default uključuje, ne opciono

---

## 📋 v2 Development Workflow

### Trigger:
v2 razvoj kreće **tek** nakon:
1. ✅ v1 hardware validiran (passes bring-up)
2. ✅ v1 firmware stabilan
3. ✅ Min 3 nedelje real-world use
4. ✅ Lista korisničkih problema iz v1 (svaki ozbiljni problem → v2 fix)
5. ✅ Min 5 prodatih v1 ploča (validacija market interest)

### Faze v2 razvoja:
| Faza | Trajanje | Output |
|---|---|---|
| **1. Architecture finalize** | 2 nedelje | v2 specs dokument |
| **2. EasyEDA šema** | 3 nedelje | Verified schematic + DRC |
| **3. PCB layout** | 4 nedelje | Verified Gerber + DRC + DRC |
| **4. Prototype order** (5-10 ploča) | 2-3 nedelje | Physical PCBs |
| **5. Bring-up + testing** | 2-3 nedelje | First-pass validation |
| **6. Rev 1 issues fix + reorder** | 2-3 nedelje | Final design |
| **7. Production batch** (50 ploča?) | 3-4 nedelje | Inventory |
| **8. Marketing launch** | continuous | Sales |

**Total v2 timeline:** ~5-6 meseci od odluke do prodaje.

---

## 🎯 Marketing Positioning

### v1 (current):
> *"Open-source 5-axis CNC controller with optically isolated inputs."*

### v2 Pro — **Dual Differentiator**:
> *"World's first hobby grblHAL controller with closed-loop encoder feedback AND true industrial galvanic isolation. Every move verified. Zero noise propagation."*

Bullet points za landing/marketing:
- ✅ **Closed-loop feedback** — encoder feedback on every axis (lost-step detection, real position)
- ✅ **Full I/O Isolation** — 1kV barrier between MCU and all field signals (inputs + outputs)
- ✅ **Same plug-and-play wizard** as v1 (configurator + WebSerial)
- ✅ **6× faster than RP2040 alternatives** (RP2350 PIO step gen)
- ✅ **Open hardware** (KiCad files + BOM published)

### v2 Ultimate (Pro + Pendant):
> *"The only CNC system you can configure end-to-end without a computer. RectaBot Ultimate Pro."*

### Pricing Strategy:
| Tier | Features | Target Price |
|---|---|---|
| **v2 Standard** | Like v1 — kept for cost-sensitive market | $180-200 |
| **v2 Pro** | + Full I/O Iso + Encoder Feedback | **$320-380** |
| **v2 Ultimate** | Pro + SmartPendant bundle | **$500-580** |

**Margin strategija:** Pro tier (~$320) ima dovoljno marže za:
- $150 sve nove komponente (iso + encoder receivers)
- $50 production overhead
- $120 net margin za scaling biznis

---

## 📚 Reference komponente

### Datasheets za review:
- **PI3304-WUEX** (Pericom Semiconductor): https://www.diodes.com/assets/Datasheets/PI3304.pdf
- **SI8621BD-B-IS** (Silicon Labs): https://www.silabs.com/documents/public/data-sheets/Si86xx.pdf
- **SI8642BD-B-IS** (Silicon Labs): quad version
- **B2424S-3WR3** (MORNSUN): existing supplier
- **B0505S-1WR3** (MORNSUN): existing supplier
- **MP1584EN-LF** (MPS): widely available, cheap buck
- **6N137** (multiple): generic fast opto

### Alternatives (po potrebi):
- **ADuM3210** (Analog Devices) — alternative za SI8621
- **HCPL-2630** — alternative za 6N137 (industrial grade)
- **R-7805** (Recom) — alternative za AMS1117 ako toplota bude problem

---

## 🔗 Veza sa drugim dokumentima

- [project_v2_ideas.md](../memory/project_v2_ideas.md) — original v2 ideja lista (memory)
- [Hardware_Design_Guidelines.md](Hardware_Design_Guidelines.md) — v1 design refs
- [Schematic_References.md](Schematic_References.md) — v1 IC schematics
- [Product_Specification_v1.0.md](Product_Specification_v1.0.md) — v1 spec

---

*Sledeći korak: Posle v1 validation, finalize v2 specs i krenuti sa EasyEDA šema.*
