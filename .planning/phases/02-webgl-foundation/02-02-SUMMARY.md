---
phase: 02-webgl-foundation
plan: 02
subsystem: ui
tags: [gsap, scrambletext, loading-screen, css-animation, film-grain, scanline, progress-bar, gpu-tier]

# Dependency graph
requires:
  - phase: 02-webgl-foundation
    plan: 01
    provides: gpuTier variable, GSAP in import map, WebGLRenderer lifecycle, canvas below z-index 100

provides:
  - "#loader overlay — full-screen black loading screen at z-index 100"
  - "GSAP ScrambleText left-to-right character reveal of 'Tom Bernys' (~2.6s total)"
  - "Scanline band CSS animation (2s linear infinite) — tier 2+ only via .has-scanline"
  - "Film grain CSS animation (feTurbulence SVG, 0.3s steps) — tier 2+ only via .has-grain"
  - "Progress bar tween: 0% → 90% over 4.5s (last 10% for Plan 03 transition)"
  - "window._loaderTimeline and window._progressTween for Plan 03 safe cleanup"

affects: [03-hero-scene, 07-qa]

# Tech tracking
tech-stack:
  added:
    - gsap ScrambleTextPlugin — registered and used for character-by-character corruption reveal
  patterns:
    - Tier-aware loading: tier >= 2 enables scanline + film grain; tier 0/1 gets text-only on black
    - Fixed min-width per character slot (0.7em / 0.4em for space) prevents layout shift during scramble
    - GSAP timeline stored as window._loaderTimeline for safe kill() before overlay dissolve
    - Progress bar GSAP tween goes to 90% — final 10% jump triggered by Plan 03 transition

key-files:
  created: []
  modified:
    - index.html — loading overlay DOM + CSS (scanline, grain, progress bar) + GSAP ScrambleText logic

key-decisions:
  - "ScrambleText stagger 0.2s per character * 10 chars + 0.6s duration = ~2.6s total — matches ~2.5s target from research"
  - "Space character handled via gsap.set() with innerHTML '&nbsp;' rather than scramble — prevents artifact where ScrambleText tries to animate a space"
  - "Scanline/grain use CSS class toggle (.has-scanline, .has-grain) rather than inline style — clean separation of tier logic from CSS"
  - "Progress bar tween stops at 90% deliberately — Plan 03 completes to 100% on transition trigger for smooth loading-complete feel"
  - "window._loaderTimeline exposed so Plan 03 can call .kill() before dissolving overlay — prevents GSAP animation errors after DOM removal"

patterns-established:
  - "Tier-aware CSS activation: add class to #loader based on gpuTier >= threshold; CSS rules scoped to that class"
  - "Loading overlay cleanup protocol: kill GSAP timeline → kill progress tween → jump bar to 100% → fade overlay → remove from DOM (Plan 03 completes this)"

requirements-completed: [LOAD-01, LOAD-03]

# Metrics
duration: 6min
completed: 2026-03-22
---

# Phase 2 Plan 02: Loading Screen Summary

**GSAP ScrambleText decryption reveal of 'Tom Bernys' character-by-character with tier-aware CSS scanline sweep and film grain on black loading overlay**

## Performance

- **Duration:** ~6 min
- **Started:** 2026-03-22T19:31:31Z
- **Completed:** 2026-03-22T19:37:00Z
- **Tasks:** 2
- **Files modified:** 1

## Accomplishments

- Full-screen #loader overlay at z-index 100 (above canvas and main content) with 10 fixed-width character slots for "Tom Bernys"
- GSAP ScrambleTextPlugin drives left-to-right corruption glyph cycling — each character resolves sequentially with 0.2s stagger, total ~2.6s
- Tier 2+ devices get scanline band sweeping top-to-bottom every 2s and film grain noise via inline SVG feTurbulence; tier 0/1 shows text only on black
- Progress bar tween animates 0% → 90% over 4.5s; window._loaderTimeline and window._progressTween exposed for Plan 03 safe dissolve

## Task Commits

Each task was committed atomically:

1. **Task 1: Loading overlay DOM and CSS** - `dc73ba4` (feat)
2. **Task 2: GSAP ScrambleText reveal and tier-aware activation** - `9ead8ab` (feat)

**Plan metadata:** (pending docs commit)

## Files Created/Modified

- `index.html` — #loader DOM (10 .loader-char spans, .loader-progress bar), CSS (scanline/grain pseudo-elements with class guards, keyframes, JetBrains Mono text), GSAP ScrambleText logic and tier-aware initLoadingScreen()

## Decisions Made

- ScrambleText stagger 0.2s per character gives ~2.6s total reveal — matches the ~2.5s target from research
- Space character uses `gsap.set()` with innerHTML rather than scramble — avoids ScrambleText artifact with whitespace characters
- Scanline and grain activated via CSS class toggle on #loader (`.has-scanline`, `.has-grain`) rather than inline style — keeps tier logic in JS and visual rules in CSS cleanly
- Progress tween stops at 90% intentionally — Plan 03 completes it to 100% at transition moment for a more satisfying finish feel
- `window._loaderTimeline` exposed so Plan 03 can `tl.kill()` safely before dissolving the overlay (avoids GSAP "animation on removed element" errors)

## Deviations from Plan

None — plan executed exactly as written.

## Issues Encountered

None.

## User Setup Required

None — no external service configuration required. ScrambleTextPlugin loads from the existing GSAP CDN import map entry.

## Next Phase Readiness

- Plan 03 (loading screen transition / hero scene reveal) can begin immediately
- `window._loaderTimeline` and `window._progressTween` are available for kill() and completion
- The #loader overlay remains on screen — Plan 03 dissolves it after WebGL scene is ready
- startRAFLoop() is already called (canvas renders beneath overlay) so Plan 03's compileAsync will work without modification

---
*Phase: 02-webgl-foundation*
*Completed: 2026-03-22*
