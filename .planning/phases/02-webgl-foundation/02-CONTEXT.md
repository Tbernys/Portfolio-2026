# Phase 2: WebGL Foundation - Context

**Gathered:** 2026-03-22
**Status:** Ready for planning

<domain>
## Phase Boundary

GPU infrastructure, device tier detection, and a glitch-style loading screen that initializes all WebGL resources and transitions visitors into the hero scene. Covers renderer setup, RAF loop, context loss recovery, render target disposal, and shader pre-compilation. The hero scene itself (3D text, tiles, interactions) belongs to Phase 3.

</domain>

<decisions>
## Implementation Decisions

### Glitch text reveal
- Digital corruption style: each character cycles through random unicode/symbols before resolving to the correct letter
- Characters resolve left to right (decryption/cracking feel)
- Full reveal takes ~2.5s (medium pace)
- White text on pure black background
- Uses the project's display font (converted typeface from Phase 1)

### Scanline overlay
- Animated sweep: a bright scanline band moves top-to-bottom repeatedly (~2s cycle)
- Band has noise/distortion — imperfect, analog feel, not a clean geometric line
- Subtle film grain texture on the black background adds life during loading
- Thin progress bar at bottom or top of screen in addition to the text reveal

### Loading-to-hero transition
- Glitch dissolve: loading screen fragments and breaks apart, revealing the hero scene underneath
- Transition duration ~1s (medium — readable but doesn't linger)
- Loading text and hero 3D text are separate elements — 2D text dissolves away, 3D glass text is already present in the hero behind it
- Transition triggers when GPU assets are ready, with a 2s minimum display time (so the animation is always seen) and the existing 5s fallback maximum

### Device tier behavior
- Low-end devices get simplified loading: skip film grain and scanline sweep, just the text reveal on black
- No visible tier indication to visitors — completely invisible to the user
- Tier detection and WebGL-unavailable fallback are at Claude's discretion

### Claude's Discretion
- Device tier detection strategy (GPU benchmark vs heuristics vs hybrid)
- WebGL-unavailable fallback approach (static HTML or CSS-only)
- Exact scramble glyph character set for the corruption effect
- Progress bar styling and position
- Film grain intensity and animation speed
- Scanline band width, opacity, and noise pattern
- RAF loop architecture and context recovery implementation

</decisions>

<specifics>
## Specific Ideas

- The decryption/cracking left-to-right resolve should feel like a password being cracked — each character scrambles independently then snaps into place
- The glitch dissolve transition should use the same visual language as the corruption text — fragmentation, not a clean effect
- Loading screen is the visitor's first impression — the 2s minimum ensures it's always experienced, not skipped on fast connections

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope

</deferred>

---

*Phase: 02-webgl-foundation*
*Context gathered: 2026-03-22*
