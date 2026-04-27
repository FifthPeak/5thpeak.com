# 5thPeak — Deploy Guide

This site is a static bundle. Drop the folder onto Netlify, point your Squarespace DNS at it, and you're live. Below is the full path.

---

## What's in this folder

```
/                               Static site root — publish this whole folder
├── index.html
├── approach.html
├── principals.html
├── contact.html
├── investors.html              (noindex'd — login stub; will move to portal subdomain)
├── assets/
│   └── css/
│       └── site.css            Shared stylesheet (was inlined per page)
├── favicon.svg                 Outlined Cormorant glyph paths — renders identically anywhere
├── favicon-32.png              Legacy fallback
├── apple-touch-icon.png        iOS home-screen
├── icon-192.png, icon-512.png  PWA / Android
├── og-image.png                1200×630 social preview, real Cormorant Garamond Light Italic
├── robots.txt
├── sitemap.xml
├── netlify.toml                Build & headers config
└── DEPLOY.md                   This file (safe to leave; Netlify will serve it but no one links to it)
```

The `5thPeak-Developer-Handoff.gdoc` shortcut and the `Cormorant_Garamond/` source-font folder
(if still present) are NOT needed for the deploy. The site loads Cormorant from Google Fonts
CDN at runtime. Move both up one level into `Cowork Handoff/` (or anywhere outside `site/`)
before publishing — they don't break anything if left, but they bloat the deploy unnecessarily.

---

## Step 1 — Push the site to GitHub, then connect Netlify

### 1a. The repo

GitHub repo: **`FifthPeak/5thpeak.com`** (private). Already created.

SSH remote: `git@github.com:FifthPeak/5thpeak.com.git`

### 1b. Initialize Git and push

Open Terminal and run:

```bash
cd "/Users/brandonbenson/Documents/Claude/Projects/FifthPeak Website and Rebrand/Cowork Handoff/site"

git init -b main
git add .
git commit -m "Initial commit — 5thPeak static site"
git remote add origin git@github.com:FifthPeak/5thpeak.com.git
git push -u origin main
```

The repo's `.gitignore` already excludes `*.gdoc`, `Cormorant_Garamond/`, `.DS_Store`, and `_wordmark-preview.html`, so nothing extraneous will get pushed.

### 1c. Connect Netlify to the repo

1. Sign up / log in at https://app.netlify.com (free tier covers this site forever).
2. **Add new site → Import an existing project → Deploy with GitHub**.
3. Authorize Netlify to access your GitHub account.
4. Pick the `5thpeak.com` repo.
5. Build settings — leave at defaults. `netlify.toml` already declares `publish = "."` and no build command, so Netlify will figure it out automatically.
6. Click **Deploy**. First build takes ~30 seconds. You'll get a temporary URL like `inspirational-pasteur-12abc.netlify.app`.

From here on, every `git push` to `main` triggers a redeploy in ~30 seconds. Pull requests get their own preview URLs automatically.

## Step 2 — Add your custom domain

1. In Netlify: **Site configuration → Domain management → Add a domain**. Enter `5thpeak.com`.
2. Netlify will tell you to either (a) move the domain's nameservers to Netlify, or (b) add specific DNS records at your current registrar.

The simpler path with Squarespace is **(b) DNS records at Squarespace**, because moving nameservers means moving everything — email forwarding, MX, etc. So in Squarespace:

- Log in to Squarespace → **Settings → Domains → 5thpeak.com → DNS Settings**.
- Add an `A` record: host `@` → value `75.2.60.5` (Netlify's load balancer IP — confirm the current value in Netlify's UI when you add the domain).
- Add a `CNAME` record: host `www` → value `<your-netlify-subdomain>.netlify.app`.
- Save. Propagation is usually 5–30 minutes, occasionally up to 24 hours.

3. Back in Netlify, click **Verify DNS configuration**. Once it's green, **enable HTTPS** — Netlify provisions a Let's Encrypt cert automatically.
4. In Netlify, set the primary domain to `www.5thpeak.com` (or the apex, your call — but pick one and let the other 301 to it). The `<link rel="canonical">` tags in every page point to `https://www.5thpeak.com/…`, so `www` is the cleanest match.

## Step 3 — Netlify Forms (contact form)

Forms are auto-detected on first deploy because the `<form>` has `data-netlify="true"`. After your first deploy:

1. **Submit one test message** through the form on the live site. Netlify uses this to confirm form processing.
2. Go to **Forms** in the Netlify dashboard. You'll see "contact" listed with the test submission.
3. Click the form → **Settings & usage → Form notifications → Add notification → Email notification**. Point it at `info@5thpeak.com` (or wherever).
4. Free tier gives you 100 form submissions per month. More than enough.

The form's `action="/?submitted=true"` redirects to the homepage with a query param after submission. If you want a dedicated thank-you page, create `thanks.html` and change the action to `/thanks.html`.

The honeypot (`netlify-honeypot="bot-field"`) and Akismet spam filter are on by default. If spam ever leaks through, enable reCAPTCHA in the form settings.

## Step 4 — Investor portal (later)

`investors.html` is a stub. The `<form action="#">` does nothing yet. When you're ready:

- Stand up the portal (Memberstack, Auth0, custom, or Squarespace Members Area) at `portal.5thpeak.com`.
- Either (a) update `investors.html` to redirect to the portal, or (b) leave the login UI in place and change `action="#"` to your portal's sign-in endpoint.
- The page is already `<meta name="robots" content="noindex,nofollow">` and excluded from `sitemap.xml`/`robots.txt`.

## Step 5 — Verify after deploy

A short post-deploy checklist:

1. **Mobile** — open the live URL on your phone. Hero, principals, contact form should all look right at iPhone width.
2. **Forms** — send yourself a test message and confirm the email notification arrives.
3. **Social previews** — paste your URL into LinkedIn's [Post Inspector](https://www.linkedin.com/post-inspector/) and Twitter/X's card validator. The OG image should render. (LinkedIn caches aggressively; force a refresh in Post Inspector if it shows the wrong thing.)
4. **Search Console** — submit `https://www.5thpeak.com/sitemap.xml` to [Google Search Console](https://search.google.com/search-console).
5. **Lighthouse** — run an audit in Chrome DevTools → Lighthouse. Targets: Performance ≥ 95, Accessibility ≥ 95, Best Practices = 100, SEO = 100.

---

## Confirmed by Brandon

- "Over fifty years of combined experience" — confirmed.
- $250M figure on `approach.html` — correct, leave as-is.
- Locke Liddell & Sapp — historical name, leave for now.
- OG image rendered with real Cormorant Garamond Light Italic; favicons and SVG favicon match.
- Wordmark `.th` `vertical-align: 0.45em` — locked.

## Open for later

- **Brandon and Carolyn both labeled "Founder"** — left as-is for now. If one is technically Co-Founder, edit `principals.html`.
- **Calligraphic peak / mountain mark** — being commissioned from a designer. When delivered, drop the SVG into `assets/img/` and add it to the `.brand` lockup in the nav and footer.
- **Investor portal** — moving to `portal.5thpeak.com`. Stub login at `investors.html` stays until then.

---

## Editing later

The site is plain HTML + one CSS file. Edit any file in this folder and re-deploy (drag-drop or `git push`). All five HTML pages share `/assets/css/site.css` — change a color or a font-size in one place and every page updates.
