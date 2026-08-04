# rectabot.org — RectaBot website

Static landing page + grblHAL firmware configurator for RectaBot. Part of the
`rectabot/RectaBot` repo; deployed to GitHub Pages (custom domain **rectabot.org**)
from this `website/` folder via `.github/workflows/deploy-pages.yml`.

- `index.html` — landing page (zero build; Tailwind + fonts via CDN)
- `configurator/` — firmware selector (kept in sync with the repo's `Docs/configurator/`)
- `board-photo.webp` / `board-photo-full.webp` — board photo (thumbnail + lightbox)
- `rectacontrol.png` — sender screenshot; copied from the RectaControl repo's
  `docs/images/rectacontrol.png`, re-copy it when the UI changes
- `Brand_Assets/favicon.svg`, `favicon.svg` — icons
- `CNAME` — custom domain (rectabot.org)
