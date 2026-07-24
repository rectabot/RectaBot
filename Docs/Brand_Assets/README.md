# RectaBot Brand Assets

Logo files for the RectaBot CNC controller project.

## License & brand usage

The RectaBot **hardware, firmware and documentation are open** (see `LICENSE`,
`LICENSE.hardware`, and the firmware repo). The **name "RectaBot" and the
logo/wordmark are NOT open-licensed** — they are the project's brand identity.

**Copyright (c) 2026 Filip Perić. All rights reserved** for the marks in this
folder (the `rectabot` wordmark, the `r` icon, the lockups, and the favicon).

### You MAY (no permission needed)
- Use the name and logo to **refer to** the genuine RectaBot project — "works with
  RectaBot", a link back to rectabot.org, reviews, tutorials, articles.
- Reproduce the logo **unmodified** to identify the official project.

### You MAY NOT (without written permission)
- Put the RectaBot name or logo on **your own hardware, firmware, or products** —
  including boards you manufacture from the open design files.
- Use the marks in a way that implies the RectaBot project **endorses, sponsors,
  or is affiliated** with your product.
- Use a **confusingly similar** name or logo (a modified wordmark, a similar `r`
  icon, the same cyan lockup) for a competing or derivative product.

### If you build on the open hardware design
The design files are yours to use under CERN-OHL-S v2 — **but the brand is not.**
**Rebrand your product:** use your own name and logo, and remove the RectaBot marks
from silkscreen, docs, and UI. This keeps the open design open while avoiding
confusion about who stands behind a given board.

Questions or permission requests: **hello@rectabot.org**

> *Why the brand is reserved while everything else is open: the design is meant to
> be shared and built on; the brand is how people know a board came from us and can
> trust its support and updates. Keeping the trademark while opening the tech is the
> standard open-project model — Arduino, Raspberry Pi and Linux all do exactly this.*

---

## Files

### Wordmark variants

| File | Use |
|---|---|
| `rectabot-wordmark-cyan.svg` | Primary wordmark for dark backgrounds (cyan-400 `#22d3ee`) |
| `rectabot-wordmark-white.svg` | PCB silkscreen, dark backgrounds |
| `rectabot-wordmark-dark.svg` | Light backgrounds, print, documents |

### Icon variants (square)

| File | Use |
|---|---|
| `rectabot-icon-cyan.svg` | Icon for dark backgrounds (cyan-400 square with dark `r`) |
| `rectabot-icon-light.svg` | Inverted icon |

### Combined lockup

| File | Use |
|---|---|
| `rectabot-lockup-horizontal.svg` | Icon + wordmark, for headers and banners |

### Other

| File | Use |
|---|---|
| `favicon.svg` | Favicon for web (64×64) |
| `rectabot-orbitron.svg` | Original wordmark master file |

---

## Typography

The wordmark uses **Orbitron** (Google Fonts, SIL Open Font License) at weight 900, letter-spacing `0.06em`, always lowercase.

## Color palette

| Token | Hex |
|---|---|
| Cyan 400 (primary) | `#22d3ee` |
| Slate 950 (background) | `#020617` |
| Slate 100 (text) | `#f1f5f9` |
| Cyan 300 (highlight) | `#67e8f9` |

---

## Usage examples

### HTML favicon

```html
<link rel="icon" type="image/svg+xml" href="favicon.svg">
```

### HTML wordmark

```html
<a href="#" style="display: flex; align-items: center; gap: 12px;">
  <img src="rectabot-icon-cyan.svg" width="36" height="36" alt="RectaBot">
  <img src="rectabot-wordmark-cyan.svg" height="24" alt="rectabot">
</a>
```

### Font loading (for inline text using Orbitron)

```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@400;700;900&display=swap"
      rel="stylesheet">
```

---

## Minimum sizes

- **Wordmark:** 80px wide minimum (for readability)
- **Icon:** 16px × 16px minimum (favicon size)
- **Lockup:** 200px wide minimum

## Clear space

- Around wordmark: 0.5× the height of the `r` glyph
- Around icon: 0.25× the icon side
