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
- `Brand_Assets/favicon.svg`, `favicon.svg` — icons
- `CNAME` — custom domain (rectabot.org)
