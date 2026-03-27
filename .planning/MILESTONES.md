# Milestones

## v1.0 MVP (Shipped: 2026-03-27)

**Phases completed:** 7 phases, 16 plans, 30 tasks
**Timeline:** 2026-03-22 → 2026-03-27 (5 days)
**Codebase:** 3,099 LOC — single index.html, vanilla JS + Three.js r175 via CDN

**Key accomplishments:**

- Static HTML scaffold with Three.js CDN importmap, semantic landmarks, OG meta tags, and Vercel zero-config deploy
- Comfortaa Bold converted to Three.js typeface.json (850 glyphs) — TextGeometry blocker resolved before Phase 3
- Production WebGL renderer with detect-gpu tier detection, visibility-aware RAF loop, iOS context loss recovery, and leak-free resize
- GSAP scramble-text loading screen: character-by-character "Tom Bernys" reveal with scanline sweep and film grain overlay
- Shader warmup gate + clip-path glitch dissolve transitions loading screen to hero WebGL canvas
- Crystal-clear glass text "Tom Bernys" with MeshPhysicalMaterial transmission/refraction, RoomEnvironment env map, and EffectComposer bloom+vignette pipeline
- Instanced cylindrical tile background with procedural patterns, mouse/touch quaternion rotation, idle float, and entry dezoom animation
- GSAP ScrollTrigger two-track hero scroll-out: canvas parallax+fade, glass text Z recession, tile spread uniform, and glassmorphism CTA button
- Film grain background on content sections with IntersectionObserver entrance animations completing the cinematic scroll experience
- 3x3 Vimeo project grid with glassmorphism overlays, oEmbed thumbnails, hover preview (Netflix effect), lightbox with @vimeo/player SDK, mobile double-tap, and keyboard accessibility
- Clients logo row (6 placeholder brands) and Formspree-backed contact form with inline success/error feedback and mailto link
- Single @media block for full layout reflow + isMobile-gated WebGL tuning (FOV 65, scale 0.62, 4x3 quad-tree, no antialias on low-tier)
- Deployed on Vercel (production) + GitHub Pages (gh-pages branch) — both hosts verified live

**Known gaps (post-launch user actions):**
- Formspree FORM_ID: replace `REPLACE_WITH_FORM_ID` in index.html once account created at formspree.io
- Vimeo IDs: replace placeholder `76979871` in PROJECTS array (9 entries) with real video IDs

---
