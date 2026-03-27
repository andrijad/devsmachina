# DevsMachina — Landing Page

Static landing page for [devsmachina.app](https://devsmachina.app).

## Stack

- Pure HTML/CSS/JS — no framework, no dependencies
- Hosted on Cloudflare Pages (via Wrangler / Workers Assets)
- Forms powered by [Formspree](https://formspree.io) + [Cloudflare Turnstile](https://www.cloudflare.com/products/turnstile/) for bot protection

## Structure

```
index.html       # Single-file landing page
privacy.html     # Privacy policy page
favicon.svg      # Site favicon
og-image.webp    # Open Graph / social share image
robots.txt       # Search engine directives
sitemap.xml      # XML sitemap
wrangler.jsonc   # Cloudflare Workers/Pages config
README.md
```

## Sections

- **Hero** — headline, value prop, early access CTA
- **How it works** — 4-step process (ingest → analyze → extract → dispatch)
- **The Agents** — 6 specialized AI agents (Pricing, SEO, Conversion, Ops Reporting, Inventory Signals, Localization)
- **Who it's for** — target audience (WooCommerce stores, DTC brands)
- **Waitlist** — early access signup form with Turnstile bot protection

## Local Development

No build step required. Open `index.html` directly in a browser.

## Deployment

Deployed via Cloudflare Pages connected to this repository. Every push to `main` triggers an automatic redeploy.

## Roadmap

- [x] v1.1 — Formspree integration for waitlist forms
- [x] v1.2 — Open Graph image
- [x] v1.3 — Favicon, robots.txt, sitemap.xml, privacy policy
- [ ] v2.0 — Agent detail pages

---

© 2026 DevsMachina
