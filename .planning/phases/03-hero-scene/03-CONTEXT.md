# Phase 3: Hero Scene - Context

**Gathered:** 2026-03-22
**Status:** Ready for planning

<domain>
## Phase Boundary

Immersive WebGL hero scene: "Tom Bernys" rendered as 3D glass text floating over an animated cylindrical tile background, responding to mouse and touch input. Includes bloom and vignette post-processing. The loading screen and WebGL infrastructure come from Phase 2; responsive/mobile adaptations belong to Phase 7.

</domain>

<decisions>
## Implementation Decisions

### Glass text appearance
- Crystal clear glass with strong refraction — background visible and distorted through letters
- Thick block extrusion — letters feel like solid glass sculptures with deep internal refraction
- Pure neutral color — no tint, color comes only from what's refracted behind the text
- Serif font — classic/editorial feel, serifs add refraction detail
- Single line layout — "Tom Bernys" on one line, wide cinematic feel
- ~60% viewport width — prominent but leaves breathing room for the background
- Sharp edge bevel — clean, precise cuts with light catching distinct angles
- Soft glow on edges — diffused light along edges, subtle but visible

### Interaction & idle motion
- Smooth follow — text gently rotates to face the cursor, fluid, magnetically attracted feel
- Medium rotation range (~25-30°) — noticeable 3D effect, text stays recognizable
- Idle state: float only — no auto-rotation, just calm vertical bob. Always alive but serene
- Slow drift back (~3-4s) when mouse stops — lazy, dreamy return to center position
- Text should have a floating effect and interact with the pointer (user-specified)

### Tile background style
- Muted color palette — low-saturation colors (navy, charcoal, deep teal), atmospheric but not monochrome
- Flowing motion animation — visible movement at comfortable pace, patterns ripple and evolve noticeably
- Medium density grid — enough tiles to feel immersive without being overwhelming
- Mixed patterns — tiles alternate between noise, gradient, and solid; each tile feels unique

### Post-processing & mood
- Subtle haze bloom — light glow around bright areas, adds softness without feeling heavy
- Medium vignette — clearly visible darkening that creates a natural spotlight on center/text
- No additional color grading — bloom and vignette are enough, colors stay true to tile palette
- Overall mood: atmospheric & immersive — feels like entering a space, not just viewing a page

### Claude's Discretion
- Exact serif font selection (must extrude and refract well in 3D)
- Crosshair styling at tile grid intersections
- Float animation amplitude and speed
- Exact muted color values for the tile palette
- Pattern distribution across tiles (noise vs gradient vs solid ratios)
- Bloom threshold and radius tuning
- Vignette falloff curve
- Lighting setup (number, position, intensity of lights)

</decisions>

<specifics>
## Specific Ideas

- The text should feel like a physical glass object suspended in space — solid, weighty, but floating
- The background should wrap around the viewer (cylindrical tiles), creating a sense of being inside an environment rather than looking at a flat page
- The slow drift-back transition should feel dreamy, like the text is settling in water
- Atmosphere is key — the combination of muted tiles, clear glass, and subtle bloom should create an enveloping, almost meditative first impression

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope

</deferred>

---

*Phase: 03-hero-scene*
*Context gathered: 2026-03-22*
