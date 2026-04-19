# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Static website for Radio Expres — a live streaming radio station (classic Rock & Pop, Espinar, Cusco, Peru). No build process; everything runs directly in the browser via CDN.

## Development

Open `index.html` directly in a browser or serve it with any static file server:

```sh
python3 -m http.server 8080
# or
npx serve .
```

Deployed to GitHub Pages with custom domain `radioexpres.pe` (configured via `CNAME`). Pushing to `main` deploys automatically.

## Architecture

The entire application lives in a single `index.html` file (~1,150 lines) containing HTML, inline CSS, and JavaScript. There is no build step, no npm, no separate JS modules.

**Stack (all via CDN):**
- **Vue 3** — reactive state, UI rendering, player controls
- **Tailwind CSS** — utility styling with custom theme colors (primary red `#e01020`, dark bg `#0a0a0a`)
- **Google Fonts** — Bebas Neue (display), Inter (body)

**External services:**
- **Icecast2 stream**: `https://sonic.sistemahost.es:7118/live` — audio source and metadata endpoint (`/status-json.xsl`)
- **iTunes Search API** — album artwork lookups (100×100 scaled to 600×600)
- **All Origins CORS proxy** — fallback when direct metadata fetch fails due to CORS

## Player Internals

The player has non-obvious logic worth understanding before editing:

- **Metadata polling**: fetches `/status-json.xsl` every 5 seconds; tries direct fetch first, then All Origins proxy on failure
- **Artwork caching**: in-memory `Map` keyed by `"artist - title"` to avoid duplicate iTunes requests
- **Anti-buffer-bloat**: every 15s checks if buffer exceeds 8s ahead of playback; silently reloads stream if so (prevents live stream drift)
- **Reconnection**: exponential backoff on playback errors (2s → 4s → 6s → 8s → 10s → 10s), max 6 attempts
- **Visualizer**: 18 CSS bars using `transform` for GPU acceleration, with staggered `animation-delay` values

## Assets

All images in `assets/img/`: logo, favicon, player background (`bg-player.webp`), DJ profile photos (`team/`), and sponsor logos (`sponsors/`).
