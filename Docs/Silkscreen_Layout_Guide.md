# RectaBot v1.0 — Silkscreen Layout Guide

**Verzija:** 1.0 | **Datum:** 2026-05-10 | **Ploča:** 150×100mm, 4-layer

---

## TOP SILKSCREEN — KOMPLETAN LAYOUT

```
                                  Y = 100mm
   ┌───────────────────────────────────────────────────────────────┐
   │ ●NPTH                                              ●NPTH     │
   │  M3                                                  M3      │  Y=95
   │                                                                │
   │ [CN22] [CN23] [CN24] [CN25] [CN26] [CN27] [CN28] [CN29] [CN30][CN31]│
   │ X-LIM  Y-LIM  Z-MAX  A-LIM  B-LIM  ESTOP  PROBE  CYC-S F-HOLD DOOR  │  Y=88
   │                                                                │
   │  (LTV-217 opto čipovi ispod konektora)                       │
   │                                                                │
   │ ─── ─── ─── ─── ─── ─── ─── ─── ─── ─── ─── ─── ─── ─── ─── │  Y=83.5
   │              2mm ISOLATION BARRIER                            │  Y=85.5
   │ ─── ─── ─── ─── ─── ─── ─── ─── ─── ─── ─── ─── ─── ─── ─── │
   │                                                                │
   │  ┌──────────────────────────────────────────────────────────┐ │
   │  │             RectaBot v1.0                                │ │  Y=78
   │  │         5-Axis CNC Controller                            │ │  Y=74
   │  └──────────────────────────────────────────────────────────┘ │
   │                                                                │
   │  [USB-C]                                          [RJ45 ETH] │  Y=60
   │   USB                                              ETHERNET   │
   │                                                                │
   │                                                                │
   │           [RP2350B]    [W25Q128]   [W5500]                    │  Y=45
   │            MCU         FLASH        ETH                        │
   │                                                                │
   │                          ●                                     │
   │                       (5th hole)                                │  Y=50
   │                       NPTH M3 (centar)                          │
   │                                                                │
   │           [microSD]                                            │  Y=30
   │                                                                │
   │                                                                │
   │ [CN34]  [CN35]  [CN36]  [CN37]  [CN38]  [CN41]  [CN32]  [CN42]│
   │ X-STEP  Y-STEP  Z-STEP  A-STEP  B-STEP  AUX     RS485   24V IN│  Y=10
   │ STEP/DIR/EN/+5V                                               │
   │                                                                │
   │ S/N: __________                       © Filip Perić 2026      │  Y=4
   │                                       RP2350B + grblHAL        │  Y=2
   │ ●PTH GND                                            ●PTH GND  │  Y=0
   │  M3                                                  M3        │
   └───────────────────────────────────────────────────────────────┘
   X=0                                                       X=150mm
```

---

## BOTTOM SILKSCREEN — KOMPLETAN LAYOUT

```
                                  Y = 100mm
   ┌───────────────────────────────────────────────────────────────┐
   │ ●         (mounting hole positions, mirror)            ●      │
   │                                                                │
   │                                                                │
   │   (B2424S info)                                                │
   │   ⚠ 24V_ISO                                                   │
   │                                                                │
   │ ─── ─── ─── ─── ─── ─── ─── ─── ─── ─── ─── ─── ─── ─── ─── │
   │              ISOLATION BARRIER (2mm)                          │
   │ ─── ─── ─── ─── ─── ─── ─── ─── ─── ─── ─── ─── ─── ─── ─── │
   │                                                                │
   │                                                                │
   │      RectaBot v1.0 | RP2350B + grblHAL                        │  Y=70
   │      Made in JLCPCB | 4-layer | 1.6mm                         │  Y=66
   │                                                                │
   │                                                                │
   │           ┌────────────┐                                       │
   │           │  QR CODE   │                                       │  Y=50
   │           │  (10×10mm) │  → github.com/filip/rectabot         │
   │           └────────────┘                                       │
   │                                                                │
   │                                                                │
   │  ⚠ HIGH VOLTAGE                                                │  Y=15
   │     24V DC                                                     │
   │  near CN42                                                     │
   │                                                                │
   │      Pin 1 markers for ICs (●)                                 │  Y=8
   │      Decoupling caps (DNP: C33 for backup)                     │
   │                                                                │
   │ ●                                                       ●      │  Y=0
   └───────────────────────────────────────────────────────────────┘
   X=0                                                       X=150mm
```

---

## ELEMENTI PO PRIORITETU

### KRITIČNO (mora postojati)

#### 1. Title — "RectaBot v1.0"
- **Layer:** Top Silkscreen
- **Pozicija:** Centar gornje polovine, ispod izolacione barijere (~Y=78)
- **Visina teksta:** 3.5mm
- **Stroke Width:** 0.2mm
- **Font:** SansSerif (default JLCPCB)

#### 2. Subtitle — "5-Axis CNC Controller"
- **Layer:** Top Silkscreen
- **Pozicija:** Ispod Title (~Y=74)
- **Visina:** 2mm
- **Stroke:** 0.15mm

#### 3. Konektor labele — sve gornje opto (CN22-CN31)
- **Layer:** Top Silkscreen
- **Pozicija:** Iznad svakog opto konektora
- **Visina:** 1.2mm
- **Format:**

```
CN22    X-LIM
CN23    Y-LIM
CN24    Z-MAX
CN25    A-LIM
CN26    B-LIM
CN27    ESTOP
CN28    PROBE
CN29    CYC-START
CN30    FEED-HOLD
CN31    DOOR
```

#### 4. Konektor labele — donje stepere/komunikacija
- **Layer:** Top Silkscreen
- **Pozicija:** Iznad svakog konektora
- **Visina:** 1.2mm
- **Format:**

```
CN34    X-STEPPER (STEP/DIR/EN/+5V)
CN35    Y-STEPPER
CN36    Z-STEPPER
CN37    A-STEPPER
CN38    B-STEPPER
CN41    AUX (FLOOD/MIST/VAC/+5V/GND)
CN32    RS485 MODBUS (A+/B-/GND)
CN42    +24V IN
```

#### 5. RS422 Pendant + USB-C + RJ45 + microSD
```
CN20    RS422 PENDANT (24V/TX±/RX±/GND)
USBC1   USB-C (program/debug)
RJ1     ETHERNET 100BASE-TX
CARD1   microSD
```

#### 6. Pin 1 markeri (sve IC)
- **Layer:** Top Silkscreen
- **Pozicija:** Pored pin 1 svakog IC
- **Format:** mali kruzic (●) ili strelica (→)
- **Komponente:**
  - U4 (RP2350B)
  - U5 (W5500)
  - U6 (W25Q128)
  - U7, U10, U15 (74HC14D × 3)
  - U8-U20 (LTV-217 × 10)
  - U19, U21, U22 (SP3485 × 3)

#### 7. Polarity markeri (polarizovane komponente)
```
+ marker za:
  - Tantal capacitori (C7171)
  - Diode (D1, D2, D3, ...)
  - LED-ovi (LED1-LED5)
```

### VAŽNO (preporučujem)

#### 8. Footer info
```
© Filip Perić 2026
RP2350B + grblHAL
```
- **Pozicija:** Donja desna ivica (~Y=4)
- **Visina:** 1mm
- **Layer:** Top Silkscreen

#### 9. Serial Number area
```
S/N: __________
```
- **Pozicija:** Donja leva ivica (~Y=4)
- **Visina:** 1.2mm
- **Layer:** Top Silkscreen
- **Svrha:** prazan box za marker pen, numerisanje proizvedenih jedinica

#### 10. Warning za 24V
```
⚠ 24V DC
```
- **Pozicija:** Pored CN42 (24V IN konektor)
- **Visina:** 1.5mm
- **Layer:** Top Silkscreen
- **Svrha:** korisnik odmah vidi povišen napon

### OPCIONO (profesionalni touch)

#### 11. Bottom info
```
RectaBot v1.0 | RP2350B + grblHAL
Made in JLCPCB | 4-layer | 1.6mm
```
- **Pozicija:** Centar bottom layer-a
- **Visina:** 1.5mm
- **Layer:** Bottom Silkscreen

#### 12. QR kod
- **Pozicija:** Bottom layer, centar
- **Veličina:** 10×10mm
- **Sadržaj:** URL ka GitHub-u ili dokumentaciji
- **Generator:** qrcode-monkey.com (free, SVG export)
- **Import u EasyEDA:** Place → Image (SVG/PNG)

#### 13. ISO barrier markeri (input-side only)
```
─── ─── ─── ─── INPUT ISOLATION (10× OPTO) ─── ─── ─── ───
```
- **Pozicija:** Duž 2mm gap-a (Y=83.5-85.5) između MCU domena i 24V_ISO opto ulaza
- **Layer:** Top Silkscreen
- **Visina:** 1mm
- **Svrha:** vizuelno pokazuje gde je input-side optička izolacija (10× LTV-217). Outputs/komunikacija dele MCU GND u v1.

---

## DIMENZIJE TEKSTA — CHEAT SHEET

| Element | Visina | Stroke | Layer |
|---------|--------|--------|-------|
| **Title** (RectaBot v1.0) | 3.5mm | 0.2mm | Top Silkscreen |
| **Subtitle** | 2mm | 0.15mm | Top Silkscreen |
| **Konektor labele** | 1.2-1.5mm | 0.15mm | Top Silkscreen |
| **Reference designators** | 0.8-1.0mm | 0.15mm | Top Silkscreen |
| **Footer info** | 1mm | 0.15mm | Top Silkscreen |
| **Warnings** | 1.5mm | 0.2mm | Top Silkscreen |
| **Serial number** | 1.2mm | 0.15mm | Top Silkscreen |
| **Bottom info** | 1.5mm | 0.15mm | Bottom Silkscreen |
| **Pin 1 markeri** | 1mm circle | — | Top Silkscreen |

---

## JLCPCB SILKSCREEN PRAVILA

| Pravilo | Vrednost | Tvoja vrednost |
|---------|----------|----------------|
| **Min text height** | 1mm | OK (najmanja 1mm) |
| **Min stroke width** | 0.15mm | OK (0.15mm) |
| **Min text-to-pad gap** | 0.15mm | OK |
| **Min text-to-board edge** | 0.3mm | OK |
| **Color** | White (default) | OK |
| **Bottom silkscreen** | Yes (free) | Iskorišćeno |

---

## EASYEDA POSTUPAK

### Za tekst (text label):
1. **Place → Text** (ili shortcut **T**)
2. Properties:
   - **Layer:** Top Silkscreen
   - **Text:** "RectaBot v1.0"
   - **Height:** 3.5mm
   - **Stroke Width:** 0.2mm
3. Klikni gde želiš da postaviš
4. Ponovi za sve labele

### Za pin 1 marker:
1. **Place → Circle** (ili tačka)
2. Properties:
   - **Layer:** Top Silkscreen
   - **Diameter:** 0.8-1mm
   - **Fill:** Solid
3. Postavi pored pin 1 IC-a

### Za polarity (+/-):
1. **Place → Text**
2. **Text:** "+"
3. **Height:** 1mm
4. Postavi pored polarized component pad-a

### Za Warning trougao:
- Najlakše: **Place → Text** sa unicode "⚠"
- Ili: import SVG ikonice → Place → Image

### Za QR kod:
1. Generate QR online (qrcode-monkey.com, qr-code-generator.com)
2. Download kao PNG/SVG (10×10mm or 15×15mm)
3. **Place → Image** u EasyEDA
4. Layer: Bottom Silkscreen

---

## CHECKLIST — PRE GERBER EXPORT

### Top Silkscreen
- [ ] Title "RectaBot v1.0" jasno čitljiv
- [ ] Subtitle dodan
- [ ] Sve konektor labele (CN22-CN42)
- [ ] Pin 1 markeri za sve IC (RP2350, W5500, Flash, opto, SP3485, 74HC14D)
- [ ] Polarity markeri za polarizovane (tantal, dioda, LED)
- [ ] Warning ⚠ za 24V_IN (CN42)
- [ ] Footer (autor + tehnologija)
- [ ] Serial Number prazan box
- [ ] ISO barrier marker (linija duž 2mm gap-a)

### Bottom Silkscreen
- [ ] Bottom info text (model + tech)
- [ ] QR kod (opciono)
- [ ] Made in JLCPCB
- [ ] Mirror pin 1 markeri (opciono, za debug)

### Općenita provera
- [ ] Sve labele iznad pad-ova auto-uklonjene (JLCPCB pravilo)
- [ ] Min text height ≥1mm
- [ ] Stroke width ≥0.15mm
- [ ] Distance text-to-pad ≥0.15mm
- [ ] Vizuelna provera 3D Top + Bottom

---

## VIZUELNI PRIMERI (mockup)

### Title block (top)
```
   ╔══════════════════════════╗
   ║                          ║
   ║   RectaBot v1.0          ║   ← 3.5mm visina
   ║   5-Axis CNC Controller  ║   ← 2mm visina
   ║                          ║
   ╚══════════════════════════╝
```

### Konektor sa labelom
```
       X-LIMIT          ← 1.2mm tekst
       CN22             ← 1mm tekst (manji ref)
   ┌──┬──┬──┐
   │ ●│ ●│ ●│           ← 3-pin konektor
   └──┴──┴──┘
   1  2  3              ← pin labele 0.8mm (opciono)
```

### Pin 1 marker (IC)
```
   ●1                   ← 1mm circle marker
   ┌──────────┐
   │   U4     │
   │  RP2350B │         ← reference + part name
   │          │
   └──────────┘
```

### Warning label
```
   ┌──────┐
   │ ⚠    │             ← 2mm trougao (ako Unicode radi)
   │ 24V  │             ← 1.5mm tekst
   │ DC   │
   └──────┘
   ━━━━━━━━━━━━━━━━ pin za CN42
```

### S/N area (top, donji deo)
```
   ┌──────────────────┐
   │ S/N: __________  │   ← 1.2mm
   └──────────────────┘
```

---

## KAKO KONVERTOVATI U PDF

Tri opcije:

### Opcija A: VS Code (najlakše)
1. Instaliraj **"Markdown PDF"** extension
2. Otvori ovaj fajl u VS Code
3. Right-click → "Markdown PDF: Export (pdf)"
4. PDF se čuva pored MD fajla

### Opcija B: Online konvertor
1. Idi na **dillinger.io** ili **markdown-to-pdf.com**
2. Copy-paste sadržaj ovog fajla
3. Export → PDF

### Opcija C: Pandoc (command line)
```bash
pandoc Silkscreen_Layout_Guide.md -o Silkscreen_Layout_Guide.pdf
```

---

## NAPOMENE ZA RECTABOT v1

1. **Konektor labele su NAJVAŽNIJE** — korisnik mora znati gde da poveže šta
2. **Title + Author** = profesionalan touch
3. **S/N box** = ako planiraš da prodaješ jedinice (numerisanje)
4. **QR kod** = vredan ako imaš online dokumentaciju (GitHub repo)
5. **CE/FCC marks** = ostavi za v2 ako planiraš sertifikaciju

---

**Sa ovim Silkscreen layout-om, RectaBot v1.0 izgleda kao production-ready proizvod, ne hobi prototip.**
