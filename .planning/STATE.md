# Project State

## Project Reference

See: .planning/PROJECT.md (updated 2026-03-22)

**Core value:** Visitors instantly see the quality of Tom's work through an immersive visual experience and embedded video reels
**Current focus:** Phase 2 — WebGL Foundation

## Current Position

Phase: 2 of 7 (WebGL Foundation)
Plan: 02-01 complete — next: 02-02 (Loading Screen)
Status: Phase 2 in progress — WebGL lifecycle foundation complete
Last activity: 2026-03-22 — 02-01 complete: production WebGLRenderer with detect-gpu tier detection, visibility-aware RAF loop, iOS context loss fallback, and leak-free resize

**Deployed URL:** https://portfolio-2026-three-pi.vercel.app
**Vercel project:** toms-projects-56bd8057/portfolio-2026

Progress: [███░░░░░░░] 15%

## Performance Metrics

**Velocity:**
- Total plans completed: 3
- Average duration: 22 min
- Total execution time: 1.07 hours

**By Phase:**

| Phase | Plans | Total | Avg/Plan |
|-------|-------|-------|----------|
| 01-scaffolding | 2 | 62 min | 31 min |
| 02-webgl-foundation | 1 | 2 min | 2 min |

**Recent Trend:**
- Last 5 plans: 01-01 (2 min), 01-02 (60 min), 02-01 (2 min)
- Trend: Baseline established

*Updated after each plan completion*

## Accumulated Context

### Decisions

Decisions are logged in PROJECT.md Key Decisions table.
Recent decisions affecting current work:

- Three.js r175 preferred over project-spec r170 (research finding: r175 = WebGL 2 baseline, avoids r182/r183 rename breakage)
- GSAP 3.14.2 + ScrollTrigger for scroll-to-WebGL bridge (ScrollTrigger now free)
- @vimeo/player 2.30.3 for lazy-loaded embeds
- Comfortaa 700 chosen as display font — rounded organic terminals match brand vibe; TTF available on Google Fonts for facetype.js conversion
- JetBrains Mono chosen for loader monospace font — refined terminal aesthetic
- og:image uses relative path (og-image.jpg) — no domain hardcoded
- Subtitle "Monteur Vidéo & Motion Designer" rendered as visible hero text (not sr-only) in accent color #D4C5B2
- Comfortaa typeface.json uses 850 glyphs (full set, not Latin Basic subset) — user chose not to restrict in facetype.js
- Deployed to Vercel via npx vercel --yes — zero-config static deploy; Netlify/GitHub Pages remain for Phase 7 second-host requirement
- Meta description simplified to more sober/professional tone (user feedback on "Réalisations pour des marques exigeantes")
- antialias disabled on GPU tier 0/1 — reduces fill-rate pressure on low-end devices; tier >= 2 enables antialias
- iOS context loss fallback: 3-second window.location.reload() if webglcontextrestored never fires — covers iOS 17/18 backgrounding bug
- preventDefault NOT called on webglcontextlost — Three.js r175 handles internally
- GSAP added to import map at Plan 02-01 even though first used in Plan 02-02 — avoid separate commit later

### Pending Todos

None yet.

### Blockers/Concerns

- Font conversion COMPLETE: fonts/comfortaa_bold.typeface.json exists with 850 glyphs — Phase 3 TextGeometry blocker resolved
- Contact form backend not chosen: Netlify Forms vs Formspree. Low stakes — decide before Phase 6 planning.
- Fluid sim resolution numbers conflict between ARCHITECTURE.md (256/128/64) and STACK.md (1024/512/256) — resolve in Phase 3 via profiling.

## Session Continuity

Last session: 2026-03-22
Stopped at: 02-01 complete. WebGL lifecycle foundation done. Next: 02-02 (Loading Screen — GLSL shader warmup, progress bar, GSAP fade-out).
Resume file: None
