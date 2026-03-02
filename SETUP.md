# HoloPath — Setup & Deployment Guide

**Architecture:** Fully client-side. All hologram rendering, GIF/video encoding, and media
parsing runs in the browser via JavaScript and Canvas 2D. No backend server, no Python,
no ffmpeg, no Docker required. Just static HTML/CSS/JS served by any web server.

---

## 1. Local Development

### Prerequisites

- **Node.js** ≥ 18
- **npm** (comes with Node)

### Run Locally

```bash
cd frontend
npm install
npm run dev
```

Open `http://localhost:5173`. That's it — no backend to start.

### Build for Production

```bash
cd frontend
npm run build
```

Output goes to `frontend/dist/`. This folder contains everything needed to deploy.

### Verify the Build

```bash
ls dist/
# Should contain: index.html, assets/, about.html, faq.html,
# how-it-works.html, privacy.html, terms.html, pages.css,
# sitemap.xml, robots.txt, ads.txt, favicon.svg
```

---

## 2. Production Deployment

HoloPath is a static site. Deploy it anywhere: Nginx, Caddy, Cloudflare Pages,
Netlify, Vercel, GitHub Pages, S3 + CloudFront, etc.

### Option A: VPS with Nginx

This is the recommended setup for co-hosting with other projects (e.g. SandPath).

```bash
# On your VPS
sudo mkdir -p /var/www/holopath

# From your local machine
cd frontend
npm run build
rsync -avz --delete dist/ you@your-server:/var/www/holopath/dist/

# Install nginx config
sudo cp nginx.conf /etc/nginx/sites-available/holopath
sudo ln -sf /etc/nginx/sites-available/holopath /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl reload nginx
```

### Option B: Cloudflare Pages / Netlify / Vercel

1. Connect your Git repo
2. Set build command: `cd frontend && npm install && npm run build`
3. Set output directory: `frontend/dist`
4. Deploy

No environment variables or server configuration needed.

### Option C: GitHub Pages

```bash
cd frontend && npm run build
# Copy dist/ contents to your gh-pages branch or docs/ folder
```

---

## 3. DNS Configuration

If using a custom domain (e.g. holopath.art):

1. Buy domain from any registrar
2. Create DNS records:
   - `A` → your server IP
   - `AAAA` → your server IPv6 (if available)
   - `CNAME www` → `holopath.art`
3. Wait for propagation (~5–30 minutes)

---

## 4. HTTPS (VPS only)

```bash
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d holopath.art -d www.holopath.art
# Verify auto-renewal
sudo certbot renew --dry-run
```

---

## 5. Ad Integration (AdSense)

### Apply

1. Go to [Google AdSense](https://www.google.com/adsense/)
2. Submit `https://holopath.art`
3. AdSense looks for: original content (5 subpages with 480–1,300 words each),
   privacy policy, terms of use, site navigation, and `ads.txt`

### After Approval

Replace `AUTO_SLOT_ID` in these files with your actual ad unit slot ID:
- `frontend/src/index.html`
- `frontend/public/about.html`
- `frontend/public/privacy.html`
- `frontend/public/terms.html`
- `frontend/public/faq.html`
- `frontend/public/how-it-works.html`

The AdSense publisher ID (`ca-pub-5516736042033534`) is already set in all pages.

### ads.txt

Already at `frontend/public/ads.txt`:
```
google.com, pub-5516736042033534, DIRECT, f08c47fec0942fa0
```

---

## 6. Donations

Support links are in the footer of every page:
- **Buy Me a Coffee:** https://buymeacoffee.com/stygnus
- **Ko-fi:** https://ko-fi.com/E1E21UH4DX

---

## 7. Multi-Project VPS (Co-hosting with SandPath)

| Project  | Type        | Port  | Domain          |
|----------|-------------|-------|-----------------|
| SandPath | Backend+FE  | 8000  | sandpath.app    |
| HoloPath  | Static only | —     | holopath.art     |

HoloPath needs no port — it's purely static files served by Nginx.

### Shared VPS Checklist

- [ ] Separate Nginx server blocks (sites-available/)
- [ ] Separate document roots (`/var/www/sandpath/`, `/var/www/holopath/`)
- [ ] Separate SSL certificates (certbot handles this)
- [ ] Separate `ads.txt` files (same pub ID is fine)
- [ ] Separate sitemaps

---

## 8. SEO

### Site Structure

| URL             | Page           | Priority | Content |
|-----------------|----------------|----------|---------|
| `/`             | Generator tool | 1.0      | App + SEO intro |
| `/how-it-works` | Technical guide| 0.8      | 1,132 words |
| `/faq`          | FAQ            | 0.7      | 1,313 words |
| `/about`        | About          | 0.6      | 606 words |
| `/privacy`      | Privacy Policy | 0.3      | 482 words |
| `/terms`        | Terms of Use   | 0.3      | 586 words |

### Built-in SEO Features

- `<title>` + `<meta description>` on every page
- `<link rel="canonical">` on every page
- Open Graph + Twitter Card meta tags
- JSON-LD WebApplication + FAQPage schemas
- `<noscript>` fallback content
- Semantic HTML with `<nav>`, `<main>`, `<article>`, `<footer>`
- `robots.txt` + `sitemap.xml`
- `ads.txt` for AdSense verification
- Favicon (SVG)
- HTTP → HTTPS + www → apex redirects
- Gzip compression
- Long-cache headers for static assets
- Security headers (HSTS, X-Frame-Options, etc.)

### Post-Deployment Checklist

1. Verify all pages return HTTP 200
2. Submit sitemap to Google Search Console
3. Submit to Bing Webmaster Tools
4. Verify `ads.txt` is accessible at `/ads.txt`
5. Test Open Graph with [Facebook Debugger](https://developers.facebook.com/tools/debug/)
6. Test structured data with [Rich Results Test](https://search.google.com/test/rich-results)

---

## 9. Updating in Production

```bash
# Build locally
cd frontend
npm run build

# Verify subpages are in dist/
ls dist/*.html
# about.html  faq.html  how-it-works.html  index.html  privacy.html  terms.html

# Deploy
rsync -avz --delete dist/ you@server:/var/www/holopath/dist/

# Verify
curl -sI https://holopath.art/ | head -3
curl -sI https://holopath.art/about | head -3
curl -sI https://holopath.art/faq | head -3
```

---

## 10. Cost Estimates

| Item           | Monthly Cost |
|----------------|-------------|
| VPS (shared)   | $4–6        |
| Domain         | ~$1.50      |
| SSL            | Free        |
| DNS            | Free        |
| **Total**      | **~$5–8**   |

If using Cloudflare Pages, Netlify, or Vercel free tier: **$0–1.50/month** (domain only).

---

## 11. Architecture Overview

```
Browser (client-side):
  ┌─────────────────────────────────────────────┐
  │  media-parser.ts                            │
  │  ├── Static images → Canvas ImageData       │
  │  ├── Animated GIFs → gif-decoder.ts (LZW)   │
  │  └── Videos → HTML5 <video> + Canvas        │
  │                                              │
  │  hologram.ts  (13-stage rendering pipeline)  │
  │  ├── Wobble → Luminance → Edge detect        │
  │  ├── Hue colorise → Flicker → Color shift    │
  │  ├── Glow → Glitch → Scan lines → Noise      │
  │  └── Chromatic aberration → Grid → Vignette   │
  │                                              │
  │  layout.ts  (5 display layouts)              │
  │  ├── Pyramid 360° / 270°                     │
  │  ├── Hologram Fan (circular crop)            │
  │  ├── Pepper's Ghost (16:9 lower-centre)       │
  │  └── Single (no compositing)                 │
  │                                              │
  │  gif-encoder.ts  (GIF89a + LZW + median cut) │
  │  └── Animated GIF with per-frame delays      │
  └─────────────────────────────────────────────┘

Server (static only):
  Nginx / CDN → serves HTML, CSS, JS, static pages
```

### Client-Side Modules

| Module            | Lines | Purpose |
|-------------------|-------|---------|
| `app.ts`          | 606   | UI state, file handling, animation loop, GIF generation |
| `hologram.ts`     | 337   | 13-stage hologram rendering pipeline (Canvas ImageData) |
| `layout.ts`       | 270   | 5 display layout compositors (Canvas 2D) |
| `gif-encoder.ts`  | 394   | GIF89a encoder with LZW compression + median-cut quantisation |
| `gif-decoder.ts`  | 386   | GIF89a decoder with LZW decompression + disposal methods |
| `media-parser.ts` | 286   | Unified frame extraction (images, GIFs, videos) |
| `presets.ts`      | 70    | 5 hologram effect presets |

### Browser Video Support

Since video decoding now uses the browser's HTML5 `<video>` element instead of ffmpeg,
supported video formats depend on the user's browser:

| Format | Chrome | Firefox | Safari | Edge |
|--------|--------|---------|--------|------|
| MP4 (H.264) | ✅ | ✅ | ✅ | ✅ |
| WebM (VP8/VP9) | ✅ | ✅ | ✅* | ✅ |
| OGG (Theora) | ✅ | ✅ | ❌ | ✅ |
| MOV (H.264) | ✅ | ❌ | ✅ | ✅ |

*Safari WebM support varies by version.

AVI, MKV, FLV, and WMV are **not** supported in browsers and have been removed
from the accepted file types.

---

## 12. Running Tests

### Frontend Tests

```bash
cd frontend
npm test
```

Expected: 20 tests across presets, hologram renderer, layout compositor,
GIF encoder, and media parser modules.

### CI Integration

```yaml
name: Test
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20' }
      - run: cd frontend && npm ci && npm test
      - run: cd frontend && npm run build
```
