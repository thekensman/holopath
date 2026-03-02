# HoloPath — Changes (March 2026)

## Critical: GIF Encoder LZW Fix

**Problem:** Generated GIF files were corrupt — all frames decoded as solid black. PIL,
ffmpeg, and browser GIF decoders all failed to read the LZW compressed data.

**Root cause:** The LZW code-size bump used `>=` (early change) instead of `>` (deferred
change). Standard GIF decoders expect deferred change timing. When `nextCode` exactly
equalled `2^codeSize`, the encoder bumped to the larger code size one step too early.
The decoder, still reading at the old size, got the wrong bits. This single bit-alignment
error cascaded through the entire frame, producing garbage pixel indices that mapped to
black palette entries.

**Fix:** `gif-encoder.ts` line 216: `nextCode > (1 << codeSize)` — encoder uses deferred
change (`>`) which is what browsers and PIL expect for decoding. `gif-decoder.ts` line 177:
`dictSize >= (1 << codeSize)` — decoder uses early change (`>=`) which is what most
external GIF encoders use. These intentionally differ because they serve different roles:
the encoder creates output GIFs decoded by the browser's built-in decoder, while the
custom decoder only reads externally-produced GIFs uploaded by users. A browser-native
fallback in `media-parser.ts` catches any remaining edge cases.

**Verified:** The cartoon GIF input (800×600, 31 frames, disposal=1 with transparency)
decoded 0/480000 pixels with `>` but 480000/480000 pixels with `>=`. The `>` decoder
truncated at 32768 pixels because it fell out of sync with the external encoder's
bit-width timing.

**Files:** `gif-encoder.ts`, `gif-decoder.ts`, `media-parser.ts`

---

## Critical: Pyramid Layout Size Fix (Clipping Approach)

**Problem:** The overlap-prevention formula from the previous fix made subjects too small.
A 4:3 landscape input produced subjects at only 32% of canvas width (64% quadrant fill),
appearing as tiny dark rectangles inside each quadrant. The mathematical constraint
`sh × (1 + aspect/2) ≤ halfCanvas - inset - gap` is correct but overly conservative —
for landscape images, the rotated side copies' vertical span (`sw/2`) consumes too much of
the available space, shrinking all subjects.

**Fix:** Replaced the algebraic constraint with **canvas clipping regions**. Each subject is
drawn inside a `ctx.clip()` rectangle that masks it to its quadrant (top half, bottom half,
left half, right half). Subjects are now sized to fill ~88% of their half-canvas (minus a
small 3% inset), giving 83% quadrant fill for ALL aspect ratios. Any corner overflow
between adjacent quadrants is simply clipped away at the center cross (1px separator).

Before: 256×192 subject in 800×800 canvas (32% fill, 64% quadrant)
After:  331×248 subject in 800×800 canvas (41% fill, 83% quadrant)

This matches the visual density of the original fixed-fill code but works correctly for
every aspect ratio — portrait, landscape, square, and extreme ratios.

**Files:** `layout.ts` — rewrote `composePyramid4()`

---

## GIF Palette Quality

**Problem:** Multi-frame GIF output had poor color accuracy on later frames because the
256-color palette was built from only the first frame.

**Fix:** Palette now samples pixels round-robin from ALL frames, giving much better global
color coverage across the animation. Particularly noticeable on presets with shifting hues.

**Files:** `gif-encoder.ts` — `encodeGif()` palette sampling

---

## New Feature: Video Output (MP4/WebM)

Added video export as an alternative to GIF, using the browser's built-in MediaRecorder
API with `canvas.captureStream()`. Produces significantly smaller files than GIF for
longer animations.

- Auto-detects best available codec (H.264 → VP9 → VP8)
- Chrome/Edge: MP4 or WebM; Firefox: WebM; Safari: MP4
- Format selector in Output panel, disabled if browser lacks support
- Video preview uses `<video>` element with autoplay + loop

**Files:** `video-encoder.ts` (new), `app.ts`, `index.html`

---

## GIF Decoder Robustness

Added validation and browser-native fallback for external GIF decoding. After the custom
LZW decoder runs, the first frame is checked for all-black output (a sign of corrupt
decode). If detected, falls back to `<img>` + Canvas for browser-native rendering. Also
wraps the decoder in try/catch so malformed GIFs don't crash the app.

**Files:** `media-parser.ts` — `parseAnimatedGif()`

---

## Ad Layout: Rail Ads + Support Banner

- **Left/right rail ads** (160×600 sticky skyscrapers) on the generator page only,
  shown at `>= 1540px` viewport width. Removed from all subpages to keep content-to-ad
  ratio clean — the `<aside>` elements with inline AdSense styles were reserving 600px of
  vertical space before the CSS could hide them, causing a blank page on first load.
- **Support banner** below nav on all pages — dismissible, remembered per session
- **Donation buttons** moved from footer to above-the-fold banner; footer retains a subtle
  text link as secondary touchpoint
- Bottom ad (7057676288) retained on all pages

**Ad slot IDs:** Left rail: `8568993100`, Right rail: `2979965943`, Bottom: `7057676288`

**Files:** `index.html`, `styles.css`, `pages.css`, all 5 subpage HTML files

---

## Favicon & Social Assets

Generated and added:

- `apple-touch-icon.png` (180×180) — hexagonal hologram symbol with cyan glow
- `favicon-32.png` (32×32) — same design, optimized for small size
- `og-image.png` (1200×630) — social preview card with icon, title, tagline

Updated `<head>` on index.html and all subpages with proper `<link>` refs.

**Files:** `public/apple-touch-icon.png`, `public/favicon-32.png`, `public/og-image.png`,
all HTML files

---

## Build / TypeScript Fixes (from earlier this session)

- Removed unused variables flagged by `noUnusedLocals` across 4 source files
- Added `makeGifFrame` test helper to decoder test block
- Excluded `src/__tests__` from `tsconfig.json` (tests run via vitest's own TS pipeline)

---

## Nginx Fix (from earlier this session)

Changed SPA fallback `try_files $uri $uri/ /index.html` to static-site fallback
`try_files $uri $uri.html $uri/ =404`. Eliminated redirect loop caused by SPA routing
config on a site with no client-side router.

---

## Deployment

```bash
tar xzf holopath-project.tar.gz
cd holopath/frontend
npm install
npm run build
# Copy dist/ + public/ assets to web server
rsync -avz --delete dist/ server:/var/www/holopath/dist/
rsync -avz public/*.png public/*.css public/*.html server:/var/www/holopath/
```

---

## Rebrand: HoloGen → HoloPath

All references updated across 164 occurrences: source files, HTML pages, meta tags, OG
image, nginx conf, documentation, canonical URLs, and sessionStorage keys. Domain updated
from `hologen.app` to `holopath.art`. OG social preview image regenerated with
"HOLO::PATH" title.

**Files:** All HTML, CSS, TS, JSON, MD, conf, and PNG files.
