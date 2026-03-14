# HoloPath — CLAUDE.md

## Project Overview

HoloPath is a **free, browser-based hologram GIF generator** at [holopath.art](https://holopath.art). It transforms any image, animated GIF, or video into a styled holographic animation using a 13-stage Canvas 2D rendering pipeline. All processing is 100% client-side — no files are ever uploaded to a server.

**Tech stack:** TypeScript 5.5, Vite 5.4, Vitest 2.0, Canvas 2D API, no runtime npm dependencies.

---

## Directory Structure

```
holopath/
├── frontend/
│   ├── src/
│   │   ├── index.html        # Main app page (SPA entry point)
│   │   ├── app.ts            # Core application logic, event handlers
│   │   ├── hologram.ts       # 13-stage hologram rendering pipeline
│   │   ├── layout.ts         # 5 output layout composers
│   │   ├── presets.ts        # HoloPreset interface + 5 effect presets
│   │   ├── media-parser.ts   # File parsing: image/GIF/video → ImageData[]
│   │   ├── gif-encoder.ts    # Custom GIF encoder (LZW + median-cut)
│   │   ├── gif-decoder.ts    # Custom GIF decoder
│   │   ├── video-encoder.ts  # MP4/WebM export via MediaRecorder
│   │   └── styles.css        # Main app stylesheet
│   └── public/
│       ├── about.html        # About page
│       ├── faq.html          # FAQ page
│       ├── how-it-works.html # How It Works page
│       ├── privacy.html      # Privacy Policy page
│       ├── terms.html        # Terms of Use page
│       ├── contact.html      # Contact page
│       ├── articles/         # Articles section
│       │   ├── index.html    # Articles listing page
│       │   └── *.html        # 15 individual articles
│       ├── pages.css         # Stylesheet for all static subpages
│       ├── robots.txt        # Crawler instructions
│       ├── sitemap.xml       # Full sitemap
│       └── ads.txt           # AdSense ads.txt
├── nginx.conf                # Production Nginx config
├── CLAUDE.md                 # This file
├── README.md                 # User-facing documentation
└── SETUP.md                  # Development/deployment guide
```

---

## Development Commands

```bash
cd frontend
npm install           # Install dependencies (devDependencies only — no runtime deps)
npm run dev           # Dev server at localhost:5173
npm run build         # Production build → frontend/dist/
npm test              # Run Vitest tests
npm run test:watch    # Watch mode
```

---

## Architecture Notes

### Rendering Pipeline (`hologram.ts`)
`renderHoloFrame(source, frameIdx, totalFrames, preset)` applies 13 stages:
1. Wobble offset, 2. Luminance conversion, 3. Edge detection (Sobel),
4. Hue colourisation, 5. Flicker modulation, 6. Colour shift,
7. Glow boost, 8. Glitch displacement, 9. Scan lines, 10. Noise,
11. Chromatic aberration, 12. Grid overlay, 13. Scan beam + vignette

The `brightness` field in `HoloPreset` (0.5–2.0, default 1.0) applies a final multiplier to all RGB output channels.

### HoloPreset Interface (`presets.ts`)
All effect values are 0.0–1.0 (normalised). The `brightness` field uses a 0.5–2.0 range. Five presets: Classic, Cyberpunk, Ghost, Glitch, Matrix.

### Output Layouts (`layout.ts`)
- `pyramid4`: 4 rotated copies for 360° smartphone pyramid projectors
- `pyramid3`: 3 rotated copies for 270° showcase displays
- `fan`: Circular crop for LED POV fan displays
- `peppers_ghost`: Single image positioned for 45° glass reflectors
- `single`: Direct output, no compositing

### Static Pages
All subpages (`/about`, `/faq`, etc.) are plain HTML files served directly by Nginx. They load `/pages.css` for styling. No Vite processing — keep them self-contained.

The Nginx config uses `try_files $uri $uri.html $uri/ =404` for clean URLs.

Articles live at `/articles/` and follow the same static HTML pattern.

### AdSense
- Publisher ID: `ca-pub-5516736042033534`
- All pages include the AdSense script in `<head>`
- Bottom responsive ad slot: `7057676288`
- Rail ads (index only, visible at ≥1540px): left `8568993100`, right `2979965943`

---

## Key Conventions

- **TypeScript strict mode** — `noUnusedLocals` and `noUnusedParameters` are enforced. Fix compiler errors before committing.
- **Zero runtime dependencies** — Do not add npm packages that ship to the browser. Custom implementations only.
- **No server calls** — All processing must remain client-side. Do not add API calls or backend integrations.
- **Static subpages** — New pages go in `frontend/public/`. Follow the existing HTML template in `about.html`. Always include the AdSense script and `pages.css` link.
- **Sitemap** — Update `frontend/public/sitemap.xml` when adding new pages.
- **Navigation** — The nav in `index.html` and all static pages must stay in sync. Nav links: Generator, How It Works, FAQ, Articles, About, Contact.

---

## Common Tasks

### Add a new article
1. Create `frontend/public/articles/your-slug.html` using the article template
2. Add the URL to `frontend/public/sitemap.xml`
3. Add a card to `frontend/public/articles/index.html`

### Add a new effect slider
1. Add field to `HoloPreset` interface in `presets.ts`
2. Add HTML slider in `index.html` (copy an existing `.ctrl` block)
3. Read value in `getCurrentPreset()` in `app.ts`
4. Apply effect in `renderHoloFrame()` in `hologram.ts`

### Add a new output layout
1. Add the layout function in `layout.ts` and export it
2. Add the `LayoutMode` union type entry
3. Add `<option>` to `#layout-mode` select in `index.html`
4. Add an entry to `LAYOUT_NOTES` in `app.ts`

---

## Deployment

The production build outputs to `frontend/dist/`. Copy the entire `dist/` folder to `/var/www/holopath/dist/` on the server. Nginx serves it directly.

```bash
cd frontend && npm run build
rsync -av dist/ user@server:/var/www/holopath/dist/
```

Nginx config is at `nginx.conf` in the project root (not auto-deployed — update manually if server config changes).
