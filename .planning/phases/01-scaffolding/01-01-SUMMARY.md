---
phase: 01-scaffolding
plan: 01
subsystem: ui
tags: [three.js, html, importmap, cdn, webgl, og, meta, favicon, google-fonts]

# Dependency graph
requires: []
provides:
  - "Single-file index.html with inline CSS and JS (TECH-01 satisfied)"
  - "Three.js r175 loaded via CDN import map on jsDelivr (TECH-02 satisfied)"
  - "Rotating cube test proves import map resolves without errors"
  - "Semantic HTML shell: h1, four section landmarks, sr-only headings"
  - "Hero text visible: 'Tom Bernys' (Comfortaa 700) + subtitle (Inter, accent)"
  - "OG + Twitter Card meta tags with relative og-image.jpg reference"
  - "Placeholder og-image.jpg (1200x630 JPEG, black bg, brand palette text)"
  - "Inline SVG favicon with TB initials"
  - "CSS custom properties for brand palette (#000000/#F5F0EB/#D4C5B2)"
  - "Canvas-behind-DOM layering pattern (z-index: 0 / z-index: 1)"
affects:
  - "02-loader — reads canvas element, builds on Three.js import"
  - "03-glass-text — uses Comfortaa font, extends Three.js scene"
  - "04-scroll — targets section landmarks, extends Three.js scene"
  - "05-projects — targets #projects section"
  - "06-clients-contact — targets #clients and #contact sections"

# Tech tracking
tech-stack:
  added:
    - "Three.js 0.175.0 via CDN importmap (jsDelivr)"
    - "Comfortaa 700 (Google Fonts CDN)"
    - "Inter 100-900 variable (Google Fonts CDN)"
    - "JetBrains Mono 400/700 (Google Fonts CDN)"
  patterns:
    - "importmap-before-module: <script type=importmap> must appear before any <script type=module> to avoid parse errors"
    - "canvas-behind-dom: canvas position:fixed z-index:0; main position:relative z-index:1; pointer-events:none on main, auto re-enabled on interactive children"
    - "sr-only: position:absolute; width/height 1px; clip; clip-path:inset(50%); used for screen-reader-only headings"
    - "single-file: all CSS in <style>, all JS in <script type=module>, no external .css/.js files"

key-files:
  created:
    - "index.html — complete single-file HTML shell: importmap, fonts, favicon, meta, CSS, Three.js test scene"
    - "og-image.jpg — 1200x630 JPEG placeholder for OG social previews"
  modified: []

key-decisions:
  - "Three.js r175 (npm 0.175.0) pinned via importmap — avoids r182/r183 PostProcessing->RenderPipeline rename"
  - "Comfortaa 700 chosen as display font — rounded organic terminals match chaleureux/classe brand vibe, confirmed Google Fonts TTF available for Phase 3 typeface.json conversion"
  - "JetBrains Mono chosen for loader text — refined terminal aesthetic over Space Mono"
  - "og:image uses relative path (og-image.jpg) — no domain hardcoded per user decision"
  - "Subtitle 'Monteur Video & Motion Designer' is VISIBLE (not sr-only) in Phase 1 — serves as visible hero tagline in accent color"

patterns-established:
  - "importmap-before-module: importmap script tag placed as first script in <head>"
  - "canvas-behind-dom: layering via position:fixed + z-index for WebGL/DOM coexistence"
  - "sr-only: standard visually-hidden pattern for accessible section headings"
  - "data-lang on sections: anticipates future English translation without i18n system"
  - "CSS custom properties on :root for brand palette"

requirements-completed: [TECH-01, TECH-02]

# Metrics
duration: 2min
completed: 2026-03-22
---

# Phase 1 Plan 01: Scaffolding Summary

**Single index.html with Three.js r175 via CDN importmap, rotating cube test scene, semantic HTML shell, brand palette, and 1200x630 OG placeholder image**

## Performance

- **Duration:** 2 min
- **Started:** 2026-03-22T17:56:21Z
- **Completed:** 2026-03-22T17:57:41Z
- **Tasks:** 2
- **Files modified:** 2

## Accomplishments

- Proved CDN importmap pattern works: `import * as THREE from 'three'` resolves r175 from jsDelivr with zero console errors
- Established semantic HTML shell: visible hero text (h1 + subtitle), four section landmarks, sr-only headings for crawlers/screen readers
- Created complete meta layer: French SEO description, OG and Twitter Card tags, inline SVG TB favicon, CSS brand palette custom properties

## Task Commits

Each task was committed atomically:

1. **Task 1: Create index.html scaffold** - `2353ae7` (feat)
2. **Task 2: Create placeholder OG image** - `1cdac90` (feat)

## Files Created/Modified

- `/Users/tom/Desktop/ PORTFOLIO 2026/index.html` — Complete single-file HTML shell: importmap pinning Three.js r175, Google Fonts (Comfortaa/Inter/JetBrains Mono), TB favicon, French SEO meta, OG/Twitter tags, inline CSS with brand palette, semantic HTML body, inline Three.js test scene (rotating cube)
- `/Users/tom/Desktop/ PORTFOLIO 2026/og-image.jpg` — 1200x630 JPEG placeholder: black background, "Tom Bernys" in #F5F0EB, "Portfolio" in #D4C5B2

## Decisions Made

- Comfortaa 700 chosen as display font (over Quicksand) — stronger personality, more distinctive rounded terminals matching "chaleureux, classe" brief; Google Fonts TTF available for Phase 3 facetype.js conversion
- JetBrains Mono chosen for loader monospace font (over Space Mono) — more refined terminal aesthetic
- Subtitle "Monteur Vidéo & Motion Designer" rendered as visible hero text in accent color (#D4C5B2) — serves as tagline until Phase 2 3D text replaces visual treatment; HTML h1 stays for accessibility
- og:image set to relative path `og-image.jpg` — no domain hardcoded; will be replaced with site screenshot in later phase
- Pillow used for og-image.jpg generation (both Pillow and ImageMagick available; Pillow provides more precise color control)

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered

None.

## User Setup Required

None - no external service configuration required. Open `http://localhost:8080` after running `python3 -m http.server 8080` from project root to verify the rotating cube loads.

## Next Phase Readiness

- index.html is ready for Phase 2 (loader) to add a loading sequence before the Three.js scene initializes
- Canvas element `#canvas` and all section IDs (`#hero`, `#clients`, `#projects`, `#contact`) are in place for future phases to target
- Three.js importmap and module script structure is the pattern all subsequent phases extend
- Blocker still open: Comfortaa typeface.json conversion (facetype.js) must be done before Phase 3 can build TextGeometry glass text. This is a manual browser-based conversion step.

---

*Phase: 01-scaffolding*
*Completed: 2026-03-22*
