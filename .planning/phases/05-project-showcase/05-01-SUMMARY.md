---
phase: 05-project-showcase
plan: 01
subsystem: ui
tags: [vimeo, css-grid, glassmorphism, intersection-observer, oembed, lightbox]

# Dependency graph
requires:
  - phase: 04-scroll-integration
    provides: Content section entrance animations, glassmorphism CTA pattern, ScrollTrigger, IntersectionObserver pattern

provides:
  - 3x3 project grid CSS with 16:9 aspect-ratio cards and edge-to-edge layout
  - Glassmorphism overlay CSS extending existing .cta-btn pattern
  - Lightbox DOM skeleton (fixed overlay with player wrap and metadata)
  - PROJECTS data array with 9 entries (placeholder IDs — Tom must replace)
  - initProjectsShowcase() generating 9 .project-card elements via grid.innerHTML
  - fetchThumbnails() fetching Vimeo oEmbed thumbnails in parallel
  - setupCardObserver() IntersectionObserver marking cards as near-viewport
  - @vimeo/player import map entry for Plan 02 lightbox wiring

affects:
  - 05-02 (hover interactions and lightbox behavior — all CSS classes ready)
  - 06-clients (section placeholder preserved)
  - 07-contact (section placeholder preserved)

# Tech tracking
tech-stack:
  added:
    - "@vimeo/player 2.30.3 (via jsDelivr CDN, added to import map)"
  patterns:
    - "oEmbed thumbnail fetch: Promise.all on PROJECTS map, graceful .catch(() => null) fallback"
    - "IntersectionObserver for lazy-load readiness: fires once per card, sets data-near-viewport=true"
    - "Glassmorphism overlay: extends .cta-btn pattern with rgba(5,5,8,0.65) + backdrop-filter: blur(10px)"

key-files:
  created: []
  modified:
    - "index.html — import map, CSS (grid/cards/overlay/lightbox), HTML structure, PROJECTS array, card DOM generation, oEmbed fetch, IntersectionObserver"

key-decisions:
  - "VIMEO_PAID = false by default — Tom sets to true if Vimeo Starter+ plan (determines background=1 vs autoplay params)"
  - "Placeholder Vimeo ID 76979871 used for all 9 projects — Tom must replace with real IDs before deploy"
  - "initProjectsShowcase() called before sectionObserver setup — ensures card DOM exists before entrance animation observer fires"
  - "ScrollTrigger.refresh() called after project content injection — recalculates positions now section has real content height"

patterns-established:
  - "CSS: #projects override pattern — split from combined placeholder rule, use display:block + min-height:unset to override placeholder styling"
  - "JS: DOM-first async pattern — synchronous innerHTML generation, then async oEmbed fetch, then observer setup"
  - "CSS: Lightbox hidden attribute + opacity transition via rAF trick (remove hidden, add .is-open on next frame)"

requirements-completed: [PROJ-01, PROJ-04]

# Metrics
duration: 3min
completed: 2026-03-25
---

# Phase 5 Plan 01: Project Showcase Grid Summary

**3x3 Vimeo project grid with glassmorphism overlays, oEmbed thumbnail fetch, IntersectionObserver lazy-load, and full lightbox DOM/CSS skeleton ready for Plan 02 interaction wiring**

## Performance

- **Duration:** 3 min
- **Started:** 2026-03-25T12:43:08Z
- **Completed:** 2026-03-25T12:45:56Z
- **Tasks:** 2
- **Files modified:** 1

## Accomplishments

- Built complete CSS foundation for project grid, cards, overlay, and lightbox — all classes ready for Plan 02 to wire interactions
- Implemented data-driven card generation (9 cards via PROJECTS.map → grid.innerHTML) with oEmbed thumbnail fetch in parallel
- Added @vimeo/player to import map and IntersectionObserver marking cards with data-near-viewport for hover preview activation

## Task Commits

Each task was committed atomically:

1. **Task 1: Add CSS for project grid, cards, overlay, and lightbox** - `b3776d9` (feat)
2. **Task 2: Add PROJECTS array, card DOM generation, oEmbed thumbnails, IntersectionObserver** - `19919a6` (feat)

**Plan metadata:** _(docs commit follows)_

## Files Created/Modified

- `/Users/tom/Desktop/ PORTFOLIO 2026/.claude/worktrees/agent-a4c42269/index.html` — Import map (@vimeo/player), CSS (projects section, grid, cards, overlay, lightbox), HTML structure (projects-grid div + lightbox skeleton), JS (PROJECTS array, initProjectsShowcase, fetchThumbnails, setupCardObserver, call in initScrollIntegration)

## Decisions Made

- `VIMEO_PAID = false` constant added — Tom sets to `true` if Vimeo Starter+ plan to enable `background=1` parameter for cleanest hover preview
- Placeholder Vimeo ID `76979871` used for all 9 projects — this is a well-known public Vimeo video for dev/testing; Tom must replace all 9 entries before launch
- `initProjectsShowcase()` called before `sectionObserver` setup inside `initScrollIntegration()` — ensures card DOM is in place before the section entrance observer fires and triggers `is-visible`
- `ScrollTrigger.refresh()` called immediately after section observer setup — recalculates scroll positions now that `#projects` has real content height instead of the former `min-height: 100vh` placeholder

## Deviations from Plan

None — plan executed exactly as written.

## Issues Encountered

None.

## Known Stubs

| File | Location | Content | Reason |
|------|----------|---------|--------|
| index.html | PROJECTS array (9 entries) | All 9 projects use `id: '76979871'` (same placeholder Vimeo ID), `title: 'Projet N'`, `client: 'Client X'` | Tom's real Vimeo IDs not yet provided. Plan explicitly uses public video `76979871` as stand-in for dev/testing per 05-RESEARCH.md |

These stubs do NOT prevent the plan's goal from being achieved — the grid renders with real oEmbed thumbnails from the placeholder ID, demonstrating the full visual/functional outcome. Tom must populate real IDs before launch.

## User Setup Required

**Tom must replace placeholder Vimeo IDs before launch.** In `index.html`, find the `PROJECTS` array and replace each `id: '76979871'` with the actual Vimeo video ID. Also update `title`, `role`, and `client` fields per project.

If Tom has a Vimeo Starter+ plan, set `const VIMEO_PAID = true` to enable `background=1` on hover preview iframes (hides Vimeo controls for cleaner look).

## Next Phase Readiness

- All CSS classes defined: `.project-card`, `.project-overlay`, `.lightbox`, `.lightbox-close`, `.lightbox-inner`, `.lightbox-player-wrap`, `.lightbox-meta`, `.touch-revealed`
- Card DOM generated with `data-vimeo-id`, `data-index`, `tabindex="0"`, `role="button"`, `aria-label` attributes
- IntersectionObserver sets `data-near-viewport="true"` on cards entering viewport
- Plan 02 can now wire: hover preview iframe injection, mouseleave cleanup, click → openLightbox(), mobile double-tap, Escape key, backdrop click

---
*Phase: 05-project-showcase*
*Completed: 2026-03-25*
