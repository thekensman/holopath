# HoloGen — Hologram GIF Generator

Transform any image, GIF, or video into a stunning animated hologram.
**100% client-side** — all processing runs in your browser. No server needed.

## Features

- **13-stage hologram rendering pipeline** — scan lines, glow, glitch, chromatic aberration, noise, edge detection, and more
- **5 effect presets** — Classic, Cyberpunk, Ghost, Glitch, Matrix
- **5 output layouts** — Pyramid 360°, Pyramid 270°, Hologram Fan, Pepper's Ghost, Single
- **Real-time animated preview** — see changes instantly as you adjust effects
- **Client-side GIF encoding** — median-cut quantisation + LZW compression
- **Privacy-first** — your files never leave your device

## Quick Start

```bash
cd frontend
npm install
npm run dev
```

Open `http://localhost:5173`.

## Tech Stack

- **TypeScript / Vite** — build tooling
- **Canvas 2D / ImageData** — pixel-level hologram rendering
- **Custom GIF encoder/decoder** — no external dependencies
- **HTML5 Video API** — client-side video frame extraction
- **Static HTML subpages** — About, FAQ, How It Works, Privacy, Terms (for SEO/AdSense)

## Deploy

```bash
cd frontend && npm run build
# Serve dist/ with any static web server
```

See [SETUP.md](SETUP.md) for full deployment guide including Nginx, HTTPS, AdSense, DNS, and multi-project VPS configuration.
