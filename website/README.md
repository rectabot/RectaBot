# rectabot.org — RectaBot website

Static landing page + grblHAL firmware configurator for RectaBot. Part of the
`rectabot/RectaBot` repo; deployed to GitHub Pages (custom domain **rectabot.org**)
from this `website/` folder via `.github/workflows/deploy-pages.yml`.

- `index.html` — landing page (zero build; Tailwind + fonts via CDN)
- `configurator/` — firmware selector; the only copy. Its `VARIANTS` map and the
  firmware repo's `variants/` folders share the same keys, so a new build means
  adding it in both places
- `board-photo.webp` / `board-photo-full.webp` — board photo (thumbnail + lightbox)
- `rectacontrol.png` — sender screenshot; copied from the RectaControl repo's
  `docs/images/rectacontrol.png`, re-copy it when the UI changes
- `Brand_Assets/favicon.svg`, `favicon.svg` — browser tab icon (identical files)
- `apple-touch-icon.png` — 180×180, for iOS home screens, which ignore SVG icons.
  Resized from the RectaControl app icon (`rectacontrol/build/icon.png`), so the
  two stay the same picture; regenerate from there if the brand mark changes
- `CNAME` — custom domain (rectabot.org)
