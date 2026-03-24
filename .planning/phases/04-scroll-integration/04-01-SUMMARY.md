---
phase: 04-scroll-integration
plan: 01
subsystem: ui
tags: [gsap, scrolltrigger, webgl, three.js, scroll, animation, mobile]

# Dependency graph
requires:
  - phase: 03-hero-scene
    provides: glassGroup, tilesMesh, fluid sim, RAF loop (startRAFLoop/stopRAFLoop), animateEntry, transition()

provides:
  - GSAP ScrollTrigger two-track scroll architecture wired to hero scene
  - Canvas parallax (-50vh at 0.5x scroll speed) + fade over 100vh via Track A CSS timeline
  - Glass text Z recession (3 to -2) via Track B onUpdate callback, gated on entranceComplete flag
  - Tile spread effect via uScrollProgress uniform in vertex shader
  - CTA button "Voir mon travail" with glassmorphism style at bottom-left of hero
  - Touch event fix: removed window-level preventDefault, canvas-scoped passive listeners
  - RAF loop optimization: pauses when scroll progress > 0.95 (canvas invisible)
  - Content section IntersectionObserver play-once fade/slide entrance animations
  - entranceComplete flag preventing scroll-driven Z conflicts during entry animation

affects: [05-content-sections, 06-contact, 07-deploy]

# Tech tracking
tech-stack:
  added: [gsap/ScrollTrigger 3.14.2 (imported from existing gsap import map entry)]
  patterns:
    - Two-track scroll architecture: Track A for CSS DOM props (GSAP timeline scrub), Track B for Three.js mutations (ScrollTrigger.create onUpdate)
    - IntersectionObserver play-once for content section entrances
    - Canvas-scoped passive touch listeners (no preventDefault)
    - entranceComplete flag to gate concurrent animation writes to same property
    - RAF pause optimization via scroll progress threshold

key-files:
  created: []
  modified:
    - index.html

key-decisions:
  - "Two-track scroll: GSAP timeline for CSS canvas parallax/fade, ScrollTrigger.create onUpdate for Three.js object mutations — cannot use gsap.to() on Three.js objects"
  - "entranceComplete flag gates glassGroup.position.z scroll writes — prevents conflict with 2.4s entry animation that also writes position.z"
  - "scrub: true (not scrub: false) — ensures symmetrical scroll reversal as required by user"
  - "No pin: true on hero — user explicitly rejected scroll hijacking; free scroll throughout"
  - "Canvas-scoped passive touch listeners replace window-level passive:false handlers — allows native mobile scroll while preserving glass text touch rotation"
  - "uScrollProgress uniform in tile vertex shader (not per-instance GSAP) — InstancedBufferGeometry requires uniform for per-shader-invocation animation"
  - "IntersectionObserver for section entrances (not ScrollTrigger once:true) — zero-dependency browser API, play-once pattern with unobserve"
  - "RAF pauses at progress > 0.95 — CSS opacity:0 does not stop GPU render pipeline; explicit stopRAFLoop() required"

patterns-established:
  - "Pattern: Two-track ScrollTrigger architecture for WebGL+CSS hybrid scroll animations"
  - "Pattern: entranceComplete flag for concurrent animation conflict prevention"
  - "Pattern: Mobile touch fix — canvas-scoped passive listeners, no window preventDefault"
  - "Pattern: Phase 5+ must call ScrollTrigger.refresh() after adding section content (comment left in code)"

requirements-completed: [SCRL-01, SCRL-02, SCRL-03]

# Metrics
duration: 18min
completed: 2026-03-24
---

# Phase 04 Plan 01: Scroll Integration Summary

**GSAP ScrollTrigger two-track hero scroll-out with canvas parallax+fade, glass text Z recession, tile spread uniform, glassmorphism CTA button, and mobile touch fix**

## Performance

- **Duration:** 18 min
- **Started:** 2026-03-24T22:25:00Z
- **Completed:** 2026-03-24T22:43:15Z
- **Tasks:** 1
- **Files modified:** 1

## Accomplishments
- Wired GSAP ScrollTrigger two-track architecture: Track A drives CSS canvas parallax (-50vh) and opacity fade, Track B drives Three.js glassGroup Z recession and tile spread uniform via onUpdate
- Added CTA button "Voir mon travail" with glassmorphism backdrop-filter style at bottom-left of hero, smooth-scrolls to first content section on click
- Fixed mobile touch event blocker: removed window-level preventDefault calls, replaced with canvas-scoped passive listeners — native mobile scroll now works
- Added uScrollProgress uniform to tile vertex shader with spreadX/spreadY dispersion logic — tiles spread outward from center as scroll progresses
- Implemented IntersectionObserver play-once entrance animations for all content sections (fade+slide, #projects gets spring cubic-bezier)
- Added entranceComplete flag gating scroll-driven Z writes against the 2.4s entry animation to prevent position.z conflicts

## Task Commits

Each task was committed atomically:

1. **Task 1: Touch event fix, ScrollTrigger hero transition, tile spread uniform, CTA button** - `1de54f1` (feat)

## Files Created/Modified
- `index.html` - Full Phase 3 implementation + scroll integration (copied from uncommitted Phase 3 state in main worktree, then modified with all Phase 4 scroll changes)

## Decisions Made
- Two-track ScrollTrigger architecture: GSAP timeline for CSS props (canvas, CTA), standalone ScrollTrigger.create for Three.js mutations (glassGroup Z, tile spread)
- entranceComplete flag (set at end of animateEntry RAF loop) gates scroll-driven Z writes to prevent conflict with the 2.4s entry zoom animation
- Canvas-scoped passive touch listeners (canvasEl.addEventListener) replace window-level passive:false handlers — preserves touch rotation while unblocking native scroll
- uScrollProgress uniform in tile vertex shader rather than per-instance GSAP animation — correct approach for InstancedBufferGeometry
- IntersectionObserver over ScrollTrigger once:true for section entrances — built-in API, zero additional weight
- Added comment `// Future phases: call ScrollTrigger.refresh() after adding section content` as a reminder for Phase 5 planner

## Deviations from Plan

**1. [Rule 2 - Missing Context] Copied uncommitted Phase 3 index.html from main worktree**

- **Found during:** Task 1 (reviewing working directory)
- **Issue:** The worktree had the Phase 3-01 partial version (736 lines, only basic glass text + post-processing). The full Phase 3 implementation (2094 lines, with fluid sim, tiles, custom glass shader, full RAF pipeline) existed only as an uncommitted modification in the main working directory.
- **Fix:** Copied the full Phase 3 index.html from `/Users/tom/Desktop/ PORTFOLIO 2026/index.html` to the worktree before applying scroll integration changes.
- **Files modified:** index.html
- **Verification:** File grew from 736 to 2241 lines; all Phase 3 features (tilesMesh, glassGroup, fluid sim, etc.) present.
- **Committed in:** 1de54f1

---

**Total deviations:** 1 (missing context — base file sync required)
**Impact on plan:** Required pre-work to ensure the correct Phase 3 base was in place before applying scroll integration. No scope creep.

## Issues Encountered
- Base file mismatch: worktree had Phase 3-01 partial; full Phase 3 was uncommitted in main directory. Resolved by copying the full version before applying Phase 4 changes.

## User Setup Required
None - no external service configuration required.

## Next Phase Readiness
- ScrollTrigger architecture in place — Phase 5 content sections can add content and must call `ScrollTrigger.refresh()` after DOM modifications (comment left in code at `initScrollIntegration()`)
- Content sections (#clients, #projects, #contact) have `content-section` class and IntersectionObserver fade-in wired — Phase 5 can add content directly
- CTA button wired and functional — visitors can orient to content sections
- Mobile scroll unblocked — Phase 5 responsive testing can proceed

---
*Phase: 04-scroll-integration*
*Completed: 2026-03-24*
