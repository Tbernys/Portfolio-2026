---
phase: 02-webgl-foundation
plan: 01
subsystem: webgl
tags: [three.js, webgl, detect-gpu, gsap, raf-loop, context-recovery, device-tier]

# Dependency graph
requires:
  - phase: 01-scaffolding
    provides: index.html with canvas element, import map for three.js, CSS layout
provides:
  - detectTier() — async GPU tier detection via detect-gpu (tier 0-3)
  - initRenderer() — tier-aware WebGLRenderer with DPR clamping
  - initScene() — re-callable scene/camera initialization for context restore
  - startRAFLoop() / stopRAFLoop() — visibility-aware animation loop
  - webglcontextlost / webglcontextrestored handlers with iOS fallback
  - resize handler pattern (no render target leaks)
  - GSAP and detect-gpu added to import map
affects: [03-hero-scene, 04-loading-screen, 05-post-processing, 07-qa]

# Tech tracking
tech-stack:
  added:
    - detect-gpu@5.0.70 — GPU tier detection via CDN import map
    - gsap@3.14.2 — Added to import map (used starting Plan 02)
  patterns:
    - Tier-aware renderer config (antialias disabled on tier 0/1)
    - DPR clamped to 2 via Math.min(window.devicePixelRatio, 2)
    - re-callable initScene() pattern for context restore
    - visibilitychange guard on RAF loop
    - iOS context loss fallback: 3-second reload if webglcontextrestored never fires
    - Debug helpers commented-out for Phase 7 testing (simulateContextLoss, checkResizeLeaks)
    - Resize handler avoids render target allocation (pattern established for Phase 3)

key-files:
  created: []
  modified:
    - index.html — replaced Phase 1 test cube with production WebGL lifecycle infrastructure

key-decisions:
  - "antialias enabled only for GPU tier >= 2, disabled on low-tier devices (tier 0/1) to reduce fill-rate pressure"
  - "iOS context loss fallback: 3-second window.location.reload() started on webglcontextlost, clearTimeout'd if webglcontextrestored fires — covers iOS 17/18 backgrounding bug where restore never fires"
  - "Do NOT call preventDefault on webglcontextlost — Three.js r175 handles it internally"
  - "registerContextHandlers() called before startRAFLoop() in bootstrap sequence to ensure handlers are in place before first frame"
  - "GSAP added to import map now even though it's used in Plan 02 — establishes module resolution path"

patterns-established:
  - "Tier-aware renderer: gpuTier >= 2 enables antialias; affects all future renderer config decisions"
  - "re-callable initScene(): scene/camera re-created from scratch on context restore; Phase 3 hero scene must maintain this pattern"
  - "Resize handler shape: update camera.aspect + updateProjectionMatrix + renderer.setPixelRatio + renderer.setSize; Phase 3 adds renderTarget.setSize() here"

requirements-completed: [TECH-04, TECH-05]

# Metrics
duration: 2min
completed: 2026-03-22
---

# Phase 2 Plan 01: WebGL Foundation Bootstrap Summary

**Production WebGLRenderer with detect-gpu tier detection, visibility-aware RAF loop, iOS context loss recovery, and leak-free resize — replaces Phase 1 test cube with production lifecycle infrastructure**

## Performance

- **Duration:** ~2 min
- **Started:** 2026-03-22T19:26:56Z
- **Completed:** 2026-03-22T19:28:31Z
- **Tasks:** 2 (implemented together — both modify index.html sequentially)
- **Files modified:** 1

## Accomplishments
- Replaced Phase 1 rotating test cube with blank production-ready WebGL scene
- detect-gpu tier detection gates antialias (disabled on tier 0/1) and stored for future loading screen complexity decisions
- RAF loop pauses on visibilitychange hidden, resumes on visible — zero wasted GPU time in background tabs
- Context loss stops the loop; context restore re-initializes scene and restarts; 3-second reload fallback covers iOS Safari edge case
- Resize handler establishes the Phase 3 extension point (renderTarget.setSize calls go here)
- Debug helpers (simulateContextLoss, checkResizeLeaks) left commented-out for Phase 7 QA

## Task Commits

Each task was committed atomically:

1. **Task 1+2: Import map additions, production renderer, context handlers, debug helpers** - `f0bc7ec` (feat)

**Plan metadata:** (pending docs commit)

## Files Created/Modified
- `index.html` — Import map gains gsap, gsap/, detect-gpu; entire module script replaced with production WebGL lifecycle infrastructure

## Decisions Made
- antialias disabled on tier 0/1 to reduce fill-rate pressure on low-end devices
- iOS context loss 3-second reload fallback included because iOS 17/18 backgrounding can prevent webglcontextrestored from ever firing
- `preventDefault` NOT called on webglcontextlost — Three.js r175 handles this internally
- `registerContextHandlers()` is called before `startRAFLoop()` in `main()` to ensure no frames are rendered before event handlers are in place
- GSAP added to import map now (will be used starting Plan 02) to avoid a separate import map change commit

## Deviations from Plan

None — plan executed exactly as written. Both tasks were implemented in a single editing pass since they target the same file sequentially with no intervening steps. This is faithful to the plan's intent (Task 2 hardens Task 1's output).

## Issues Encountered

None. The indexOf-based test for "registerContextHandlers before startRAFLoop" produced a false negative because it found the function *definition* of startRAFLoop() rather than its *invocation* — visual inspection and the actual main() bootstrap confirm correct order.

## User Setup Required

None — no external service configuration required. CDN imports are configured in the import map.

## Next Phase Readiness
- Plan 02 (Loading Screen) can begin immediately — the black canvas is the correct starting state
- `window.renderer`, `window.scene`, `window.camera`, `window.gpuTier` are exposed for development debugging
- GSAP is in the import map and ready to import in Plan 02
- Phase 3 resize handler extension point is documented in code comments

---
*Phase: 02-webgl-foundation*
*Completed: 2026-03-22*
