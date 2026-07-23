# rectabot.org — RectaBot website

Static landing page + grblHAL firmware configurator for RectaBot. Part of the
`rectabot/RectaBot` repo; deployed to GitHub Pages (custom domain **rectabot.org**)
from this `website/` folder via `.github/workflows/deploy-pages.yml`.

- `index.html` — landing page (zero build; Tailwind + fonts via CDN)
- `configurator/` — firmware selector (kept in sync with the repo's `Docs/configurator/`)
- `Brand_Assets/favicon.svg`, `favicon.svg` — icons
- `CNAME` — custom domain (rectabot.org)
