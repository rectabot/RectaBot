# RectaBot Brand Guide

**Version 1.0 — Locked 07.06.2026**

This is the definitive guide for using the RectaBot brand mark. Read before placing the logo anywhere.

---

## 🎯 Brand identity

| | |
|---|---|
| **Wordmark** | `rectabot` (always lowercase) |
| **Reference** | Audi Quattro-inspired geometric typography |
| **Personality** | Industrial · Approachable · Tech-forward · Honest |

---

## 🔤 Typography

### Primary brand font: **Orbitron**

- **Source:** Google Fonts (https://fonts.google.com/specimen/Orbitron)
- **License:** SIL Open Font License (free for any use)
- **Weights used:** 400 (rare), 700 (rarely), **900 (always for wordmark)**
- **Why:** Geometric, rounded sans-serif with a tech feel. Echoes the Audi Quattro vibe Filip identified as the right brand reference.

### Supporting fonts:
- **Inter** — body text in marketing/web
- **JetBrains Mono** — technical contexts (code, specs, terminal)

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

### 1. Wordmark only (most-used)

```
       rectabot
```

- Font: **Orbitron** weight 900
- Letter-spacing: `0.06em`
- Cyan-400 on dark, slate-950 on light, white on PCB
- **Use for:** landing nav, documents, signature, business cards

**SVG files:**
- `logos/rectabot-wordmark-cyan.svg` (primary)
- `logos/rectabot-wordmark-white.svg` (PCB silkscreen, dark backgrounds)
- `logos/rectabot-wordmark-dark.svg` (light backgrounds, print)

### 2. Icon only (square)

```
  ┌───┐
  │ r │
  └───┘
```

- Lowercase `r` in cyan-400 rounded square
- **Use for:** favicons, GitHub avatar, app icon, small badges

**SVG files:**
- `logos/rectabot-icon-cyan.svg` (dark background)
- `logos/rectabot-icon-light.svg` (cyan background, dark r)

### 3. Horizontal lockup (icon + wordmark)

```
  ┌───┐
  │ r │  rectabot
  └───┘
```

- **Use for:** landing page nav, large headers, promotional banners

**SVG files:**
- `logos/rectabot-lockup-horizontal.svg`

---

## ✅ Do's

- ✅ Always use lowercase `rectabot` in the wordmark
- ✅ Maintain letter-spacing `0.06em` for the wordmark
- ✅ Keep generous whitespace around the logo (minimum: 0.5× character height)
- ✅ Use white wordmark on PCB silkscreen
- ✅ Use cyan-400 (`#22d3ee`) as the only brand accent color
- ✅ Use the icon (square with `r`) when space is tight or you need a recognizable mark

---

## ❌ Don'ts

- ❌ Don't capitalize `RECTABOT` or write `Rectabot` (in marks; in body text "RectaBot" is OK)
- ❌ Don't use other fonts (Helvetica, Arial, Comic Sans, etc.)
- ❌ Don't rotate, skew, or distort the logo
- ❌ Don't change the color to red, green, purple, etc.
- ❌ Don't stretch or compress the wordmark horizontally
- ❌ Don't use the ▣ unicode character anymore (replaced by Orbitron `r` icon)
- ❌ Don't apply drop shadows, gradients, or 3D effects

---

## 📏 Sizing & clear space

### Minimum sizes:
- **Wordmark:** 80px wide minimum (readable)
- **Icon:** 16px × 16px minimum (favicon size)
- **Lockup:** 200px wide minimum

### Clear space:
- Minimum padding around the wordmark = 0.5× the height of the `r` glyph
- For the icon, minimum padding = 0.25× the icon side

---

## 🖥️ Web implementation

### HTML for navigation:

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

### Font loading:

```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@400;700;900&display=swap"
      rel="stylesheet">
```

### Favicon:

```html
<link rel="icon" type="image/svg+xml" href="favicon.svg">
```

---

## 🏷️ Brand applications

### Where the logo appears:

| Context | Variant | File |
|---|---|---|
| Website nav | Lockup (small) | inline HTML |
| Website footer | Lockup (small) | inline HTML |
| Favicon | Icon | `favicon.svg` |
| GitHub org avatar | Icon (256×256) | `rectabot-icon-cyan.svg` (export to PNG) |
| GitHub social card | Lockup | `rectabot-lockup-horizontal.svg` |
| Business card | Wordmark | `rectabot-wordmark-dark.svg` for white card |
| PCB silkscreen | Wordmark white | `rectabot-wordmark-white.svg` (v2 onwards) |
| Email signature | Lockup small | inline embedded SVG |
| Invoice header | Wordmark dark | `rectabot-wordmark-dark.svg` |

---

## 🎬 Verbal brand voice

When writing about RectaBot:
- ✅ "rectabot" in the mark, "RectaBot" or "rectabot" in body text (both fine)
- ✅ Honest, technical, no-marketing-bs tone
- ✅ Use phrases like "open-source", "industrial-grade", "no vendor lock-in"
- ❌ Never claim features that aren't shipped yet (see `feedback-marketing-honesty` memory)

---

## 🛠️ Source files

All branding assets live in:

```
web/branding/
├── BRAND_GUIDE.md        ← This document
├── index.html            ← Logo concepts preview (will be cleaned up post-launch)
└── logos/
    ├── rectabot-wordmark-cyan.svg
    ├── rectabot-wordmark-white.svg
    ├── rectabot-wordmark-dark.svg
    ├── rectabot-icon-cyan.svg
    ├── rectabot-icon-light.svg
    ├── rectabot-lockup-horizontal.svg
    └── rectabot-orbitron.svg          ← Original final pick
```

**Favicon:** `web/favicon.svg` (root for HTML `<link>` reference)

---

## 📝 Change log

| Date | Change |
|---|---|
| 2026-06-07 | Initial brand identity locked. Orbitron font (lowercase wordmark) chosen over Michroma. Replaces previous ▣ unicode mark. |

---

*Questions? See landing/preview at `web/branding/index.html` or contact Filip.*
