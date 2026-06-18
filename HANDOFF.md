# Aleron MD landing page — production handoff

Single self-contained static page: **`index.html`** (inline CSS + JS, no build step).
Deploys via **GitHub Pages** from this repo. Nothing goes live until the work below
lands on the Pages source branch (`main`).

This file is the launch checklist. Work top to bottom.

---

## 🔴 Must clear before launch

1. **Login destination** — the hero "Login" button was *removed* (it pointed at a
   staging app). When the real login URL exists, re-add the button in the hero
   topline. The `.opening-hero-login` CSS is still in place (`index.html` ~line 379),
   so it's a one-line markup re-add inside `.opening-hero-topline` (~line 1001):
   `<a class="opening-hero-login" href="PROD_LOGIN_URL">Login</a>`

2. **Lead-capture endpoints** — `var ENDPOINTS` (`index.html` ~line 1749) is empty for
   both `membership` and `partner`. **Current behavior with no endpoint: the form shows
   "Application received" but only opens a `mailto:`** to `apply@AleronMD.com` /
   `partners@AleronMD.com` (~line 1753). If the visitor has no desktop mail client, the
   success message shows but **the lead is silently lost.** Before launch:
   - Paste real Formspree (or CRM) endpoint URLs into `ENDPOINTS`, and
   - Confirm both `@AleronMD.com` inboxes are live.

3. **License the runner hero image** — the runner slide background
   (`assets/hero-run/scene.jpg`) is a **GettyImages comp** (flagged at `index.html`
   ~line 971). License it or swap it before launch.

---

## 🔵 SEO / social — needs the production domain

4. **Canonical + social meta** — TODO at `index.html` ~line 15. Once the domain is set, add:
   - `<link rel="canonical" href="…">`
   - `<meta property="og:image" content="ABSOLUTE_URL">` and `og:url`
   - `<meta name="twitter:card" content="summary_large_image">`

   Without `og:image`, links shared to iMessage/social render with no preview image.

5. **robots.txt + sitemap.xml** — not present. Add both at the repo root.

---

## ⚙️ Decisions for the team

- **Analytics is currently zero.** The Cloudflare RUM beacon was removed (it only works
  when traffic is proxied through Cloudflare; on plain GitHub Pages it failed on every
  load). Pick a replacement before launch — GA4, Plausible, or Cloudflare Web Analytics
  *if* the domain is put behind Cloudflare.

- **Founder portraits** — `assets/team/jason-paper.png`, `hamed-paper.png` are wired in.
  Confirm these are the final approved images.

---

## ✅ Already done (this cleanup pass, branch `production-cleanup`)

- Promoted the landing page to `index.html`; pruned dead files/assets (working tree
  176M → ~26M) and optimized images.
- Hero carousel trimmed to two slides (surfer + runner); the couple slide was removed.
- Applied the responsive strategy (dedicated mobile hero, static mobile curve, stacked
  experience section); desktop ≥901px untouched, verified at 1280px.
- Continuum icons redrawn to the soft hand-drawn style.
- Removed the Cloudflare beacon; vendored the Apple / Fitbit / Eight Sleep wearable
  logos locally (`assets/logos/*.svg`) instead of a third-party CDN.
- Dropped the unused Caveat web font + dead CSS; trimmed Hanken Grotesk weights.

No console errors, no broken asset references.

---

## Deploy

1. Merge `production-cleanup` → `main` (Pages source).
2. `.nojekyll` is present (don't remove it).
3. Confirm the live URL renders the surfer hero and the "Connects with" logos.
