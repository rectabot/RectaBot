# RectaBot Brand Guide

**Version 1.0**

Technical reference for using the RectaBot brand mark. This document covers logo files, typography, color tokens, and usage rules so anyone modifying, redistributing, or building on RectaBot hardware can identify their version correctly.

---

## 🔤 Typography

### Primary brand font: **Orbitron**

- **Source:** Google Fonts (https://fonts.google.com/specimen/Orbitron)
- **License:** SIL Open Font License (free for any use)
- **Weights used:** 400, 700, **900 (always for wordmark)**

### Supporting fonts (web/docs)
- **Inter** — body text
- **JetBrains Mono** — code, specs, terminal

---

## 🎨 Color palette

| Token | Hex | Usage |
|---|---|---|
| **Cyan 400** (primary) | `#22d3ee` | Wordmark, icons, accents |
| **Slate 950** (deep dark) | `#020617` | Background, body |
| **Slate 100** (text light) | `#f1f5f9` | Body text on dark |
| **Cyan 300** (highlight) | `#67e8f9` | Hover states, sub-accents |
| **White** | `#ffffff` | PCB silkscreen, prints |

**Background palette:**
- Dark surfaces: `#020617` (slate-950) or `#0f172a` (slate-900)
- Light surfaces: `#f1f5f9` (slate-100) or pure white
- Brand surfaces: cyan-400 (`#22d3ee`) — accent only, never large

---

## 📐 Logo lockups

### 1. Wordmark only

```
       rectabot
```

- Font: **Orbitron** weight 900
- Letter-spacing: `0.06em`
- Cyan-400 on dark, slate-950 on light, white on PCB

**SVG files:**
- `logos/rectabot-wordmark-cyan.svg` (primary, dark backgrounds)
- `logos/rectabot-wordmark-white.svg` (PCB silkscreen)
- `logos/rectabot-wordmark-dark.svg` (light backgrounds, print)

### 2. Icon only (square)

```
  ┌───┐
  │ r │
  └───┘
```

- Lowercase `r` in cyan-400 rounded square
- For favicons, small badges, app icons

**SVG files:**
- `logos/rectabot-icon-cyan.svg` (dark background)
- `logos/rectabot-icon-light.svg` (cyan background, dark r)

### 3. Horizontal lockup (icon + wordmark)

```
  ┌───┐
  │ r │  rectabot
  └───┘
```

For headers, banners, page tops.

**SVG file:** `logos/rectabot-lockup-horizontal.svg`

---

## ✅ Do's

- Use lowercase `rectabot` in the wordmark
- Maintain letter-spacing `0.06em`
- Keep generous whitespace around the logo (min: 0.5× character height)
- Use white wordmark on PCB silkscreen
- Use cyan-400 (`#22d3ee`) as the brand accent color
- Use the icon (square with `r`) when space is tight

---

## ❌ Don'ts

- Don't capitalize `RECTABOT` or write `Rectabot` in marks (in body text, "RectaBot" is fine)
- Don't substitute other fonts
- Don't rotate, skew, or distort the logo
- Don't recolor to red/green/purple/etc.
- Don't stretch or compress horizontally
- Don't apply drop shadows, gradients, or 3D effects

---

## 📏 Sizing & clear space

### Minimum sizes
- **Wordmark:** 80px wide minimum (readable)
- **Icon:** 16px × 16px minimum (favicon size)
- **Lockup:** 200px wide minimum

### Clear space
- Around the wordmark: 0.5× the height of the `r` glyph
- Around the icon: 0.25× the icon side

---

## 🖥️ Web implementation

### HTML for navigation

```html
<a href="#" class="flex items-center gap-3">
  <div class="w-9 h-9 bg-cyan-400 rounded-lg flex items-center justify-center">
    <span class="text-slate-950 text-2xl font-black leading-none"
          style="font-family: 'Orbitron', sans-serif;">r</span>
  </div>
  <span style="font-family: 'Orbitron', sans-serif;"
        class="text-xl font-black tracking-wider text-cyan-400">rectabot</span>
</a>
```

### Font loading

```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@400;700;900&display=swap"
      rel="stylesheet">
```

### Favicon

```html
<link rel="icon" type="image/svg+xml" href="favicon.svg">
```

---

## 🛠️ Source files

```
web/branding/
├── BRAND_GUIDE.md          ← This document
└── logos/
    ├── rectabot-wordmark-cyan.svg
    ├── rectabot-wordmark-white.svg
    ├── rectabot-wordmark-dark.svg
    ├── rectabot-icon-cyan.svg
    ├── rectabot-icon-light.svg
    ├── rectabot-lockup-horizontal.svg
    └── rectabot-orbitron.svg          ← Original final pick
```

**Favicon:** `web/favicon.svg`

---

## 📜 License

These brand assets are part of the RectaBot project, licensed under **CERN-OHL-S v2** (Strong reciprocal open hardware). You may use, modify, and redistribute the logo files when including, modifying, or redistributing the RectaBot hardware design, provided you comply with the CERN-OHL-S terms.

For derivative works, change the wordmark to your own to avoid confusion with the upstream RectaBot project.
