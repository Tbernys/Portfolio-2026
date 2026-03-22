---
phase: 02-webgl-foundation
plan: 03
subsystem: ui
tags: [three.js, gsap, webgl, shader, loading-screen, dissolve, clip-path, promise]

# Dependency graph
requires:
  - phase: 02-01
    provides: WebGL renderer with compileAsync capability, RAF loop, scene/camera
  - phase: 02-02
    provides: Loading screen DOM, window._loaderTimeline, progress bar tween

provides:
  - warmupShaders() — pre-compiles all scene shaders via renderer.compileAsync during loading
  - Promise.race gate — 2s minimum display, 5s maximum fallback before transition fires
  - Glitch dissolve transition — clip-path fragment animation dissolves #loader overlay
  - loader DOM removal on transition complete — no residual compositing overhead
  - Complete Phase 2 loading-to-hero flow verified on production

affects: [03-hero-scene, phase-3-geometry, phase-7-polish]

# Tech tracking
tech-stack:
  added: []
  patterns:
    - Promise.race([Promise.all([minDisplay, shadersReady]), fallback]) gate pattern for loading-screen lifecycle
    - warmupShaders() wraps renderer.compileAsync — Phase 3 adds hero geometry before calling
    - GSAP timeline.kill() before DOM removal — prevents tween-on-removed-element errors
    - clip-path animation for fragmentation dissolve aesthetic (not opacity fade)
    - pointer-events: none applied immediately on transition start — hero interactive during dissolve

key-files:
  created: []
  modified:
    - index.html

key-decisions:
  - "ScrambleTextPlugin (GSAP Club) replaced with manual scramble implementation — plugin import broke in ES module context; manual implementation uses setInterval with random chars"
  - "Dissolve transition uses clip-path inset() with staggered rectangle reveals rather than polygon fragments — simpler, equally effective glitch feel"
  - "Progress bar repositioned under text (not above) for visual hierarchy — user feedback during verification"
  - "Dissolve slowed: 0.8s duration per fragment with overlap — user found original too fast"
  - "Text reveal must fully complete before dissolve starts — wait logic added to prevent overlap"
  - "Text shake removed from dissolve transition — user preferred cleaner separation of effects"
  - "Glitch effect accentuated: stronger scramble chars, more intense scanline, higher grain opacity"

patterns-established:
  - "Shader warmup pattern: call renderer.compileAsync(scene, camera) during loading, not on first visible frame"
  - "Loading gate: Promise.race([Promise.all([minDisplay, ready]), fallback]).then(transition) — reuse for any future loading-gated content"
  - "Loader cleanup: kill tweens → pointer-events none → animate out → remove from DOM"

requirements-completed: [TECH-06, LOAD-02]

# Metrics
duration: ~45min
completed: 2026-03-22
---

# Phase 2 Plan 03: Loading Screen Transition Summary

**Shader warmup gate via renderer.compileAsync + Promise.race(2s/5s) + clip-path glitch dissolve transitions the loading screen to the hero WebGL canvas**

## Performance

- **Duration:** ~45 min (including post-checkpoint fixes during human verification)
- **Started:** 2026-03-22
- **Completed:** 2026-03-22
- **Tasks:** 2 (1 auto + 1 checkpoint:human-verify)
- **Files modified:** 1 (index.html)

## Accomplishments

- warmupShaders() pre-compiles all Three.js scene shaders via renderer.compileAsync during the loading animation — first visible hero frame has zero shader stutter
- Promise.race gate ensures loading screen shows for at least 2 seconds and no more than 5 seconds regardless of compile time
- Glitch dissolve transition uses CSS clip-path inset() animation to fragment the overlay apart — intentional digital-corruption aesthetic, not a clean fade
- #loader DOM fully removed after transition completes — no residual compositing overhead
- All 5 Phase 2 success criteria verified on production by human reviewer (https://portfolio-2026-three-pi.vercel.app)
- Manual scramble implementation written to replace broken GSAP ScrambleTextPlugin ES module import

## Task Commits

Each task was committed atomically:

1. **Task 1: Implement shader warmup gate and glitch dissolve transition** - `2115613` (feat)

Post-checkpoint fix commits (applied during human verification):

- `0d482cc` fix(02-02): add .js extension to ScrambleTextPlugin import path
- `5f8ca8a` fix(02): move progress bar under text, slow down dissolve transition
- `2acca1d` fix(02-03): wait for text reveal to complete before dissolve transition
- `0598c0f` style(02): accentuate glitch effect
- `839312e` fix(02): replace broken ScrambleTextPlugin with manual scramble implementation
- `ef8f6b4` style(02): remove text shake from dissolve transition

## Files Created/Modified

- `index.html` — warmupShaders(), transition(), Promise.race gate wired into main(); manual scramble implementation replacing ScrambleTextPlugin; clip-path dissolve CSS; progress bar repositioned

## Decisions Made

- ScrambleTextPlugin (GSAP Club) abandoned — ES module import path resolution failed in the importmap context. Manual implementation uses setInterval with random character cycling, achieves identical visual result without the dependency.
- clip-path inset() fragment approach chosen over polygon fragment splitting — simpler to control timing and stagger, equally effective glitch aesthetic.
- Text reveal must fully complete before dissolve starts — overlapping the two animations felt visually confused. Added wait logic: dissolve fires only after scramble resolves.
- Text shake during dissolve removed — the separate effects (shake + dissolve) competed; clean dissolve alone reads as more intentional.
- Progress bar moved under text — above was too visually dominant during the character reveal sequence.

## Deviations from Plan

### Auto-fixed Issues

**1. [Rule 3 - Blocking] Replaced ScrambleTextPlugin with manual implementation**
- **Found during:** Task 1 (Implement shader warmup gate and glitch dissolve transition) — surfaced during human verification
- **Issue:** ScrambleTextPlugin is a GSAP Club plugin; the .js import path failed to resolve in the browser ES module importmap context, breaking the entire loading screen text reveal
- **Fix:** Wrote manual scramble implementation using setInterval — cycles through random chars then resolves to target character per position, same stagger timing as the original plan
- **Files modified:** index.html
- **Verification:** Loading screen text reveals correctly on production; human reviewer confirmed
- **Committed in:** `839312e`

**2. [Rule 1 - Bug] Dissolve fired before text reveal completed**
- **Found during:** Human verification of Task 2
- **Issue:** Promise.race 2s timer fired and started dissolve while the ~2.6s text scramble was still mid-reveal, causing both animations to run simultaneously and look broken
- **Fix:** Added wait for text reveal to fully complete before allowing transition() to run
- **Files modified:** index.html
- **Verification:** Dissolve now starts cleanly after text has resolved; confirmed by human reviewer
- **Committed in:** `2acca1d`

---

**Total deviations:** 2 auto-fixed (1 blocking, 1 bug)
**Impact on plan:** Both fixes essential for correct visual experience. The ScrambleTextPlugin replacement was a dependency issue uncovered at runtime; the dissolve timing fix was a sequencing bug in the Promise gate logic. No scope creep.

## Issues Encountered

- GSAP Club plugins (ScrambleText, SplitText) are not available as standard ES module imports — any future plan referencing GSAP Club features must verify CDN/importmap availability or plan a manual implementation fallback.
- The 2s minimum display timer and the ~2.6s text scramble duration were misaligned — the gate fired before the animation finished. For future loading screens, minimum display timer should be >= animation duration, or the gate should await the animation directly.

## User Setup Required

None - no external service configuration required.

## Next Phase Readiness

- Phase 2 fully complete — all 5 ROADMAP success criteria verified on production
- warmupShaders() is ready for Phase 3 to add hero geometry BEFORE calling it, so compileAsync covers all hero materials
- RAF loop, renderer, tier detection, loading screen, and dissolve transition are all production-ready
- fonts/comfortaa_bold.typeface.json exists (850 glyphs) — Phase 3 TextGeometry blocker already resolved
- Blocker: Fluid sim resolution conflict (ARCHITECTURE.md 256/128/64 vs STACK.md 1024/512/256) — resolve in Phase 3 via profiling

---
*Phase: 02-webgl-foundation*
*Completed: 2026-03-22*
