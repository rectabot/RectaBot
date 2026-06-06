# RectaBot Landing Page

Pre-launch landing page za RectaBot v1.0.

## 📂 Struktura

- `index.html` — Single-page landing (sve u jednom fajlu)
- Zero build process — Tailwind preko CDN

## 🔍 Pregled lokalno

Otvori `index.html` u browser-u:

**Windows:** Desni klik → Open with → Chrome/Firefox/Edge
**Ili:** `start "" "C:\Users\Filip\Desktop\RectaBot\web\index.html"`

## 🌐 Deploy na GitHub Pages

Kad budeš spreman:

1. Push repo na GitHub
2. Settings → Pages → Source: Deploy from branch
3. Branch: `main`, Folder: `/web`
4. Save → URL će biti `https://<username>.github.io/RectaBot/`

## 🔗 Sa custom domain (rectabot.org)

1. GitHub Pages settings → Custom domain: `rectabot.org`
2. Kod registrara (Namecheap/Porkbun) postavi:
   - CNAME `www` → `<username>.github.io`
   - A records `@` → 185.199.108.153, .109.153, .110.153, .111.153
3. Cloudflare proxy ON (free CDN + SSL)

## 📨 Formspree integracija (waitlist)

Trenutno waitlist forma ima placeholder URL:
```html
<form action="https://formspree.io/f/YOUR_FORM_ID" method="POST">
```

Setup:
1. Registruj se na [formspree.io](https://formspree.io) (free tier: 50 submissions/mo)
2. Kreiraj novu formu, kopiraj endpoint ID
3. Zameni `YOUR_FORM_ID` u `index.html` na linija ~360

## 🎨 Design choices

- **Dark theme** (slate-950 background) — industry standard za technical products
- **Cyan-400 accent** — communicates electronics/networking
- **Inter** body font, **JetBrains Mono** za code/specs
- **Mobile responsive** — single column ispod md (768px)
- **No JS frameworks** — vanilla scroll behavior + Tailwind utilities

## ✏️ Sledeća iteracija

Posle prvog bring-up-a:
- [ ] Dodati prave fotke ploče (zameniti ASCII diagram opciono)
- [ ] Video demonstracija (motion test)
- [ ] Customer testimonials (kad ih bude)
- [ ] Detaljnije pricing kad budeš definisao
- [ ] Documentation links ka hostovanim docs (Docusaurus/MkDocs?)
