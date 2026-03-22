# Phase 1: Scaffolding - Context

**Gathered:** 2026-03-22
**Status:** Ready for planning

<domain>
## Phase Boundary

Deployable HTML shell with validated CDN import map for Three.js, semantic HTML structure, font conversion for 3D text, and meta tags. No WebGL rendering yet — just the skeleton that everything builds on.

</domain>

<decisions>
## Implementation Decisions

### Typography & Fonts
- **3D glass text font**: Display/creative style, arrondi/organique (e.g., Comfortaa, Quicksand, or similar rounded modern display font). Must be converted to typeface.json format via facetype.js for TextGeometry.
- **Loader text font**: Monospace/terminal style for the glitch character reveal (e.g., JetBrains Mono, Fira Code, or Space Mono via Google Fonts)
- **UI body font**: Inter (Google Fonts CDN) for all section titles, descriptions, navigation, metadata
- **Hierarchy**: Display font for "Tom Bernys" only (3D + loader), Inter for everything else

### HTML Structure (top to bottom)
- **Section order**: Hero → Clients → Projects → Contact
- **Hero content**: "Tom Bernys" as main element + "Monteur Vidéo & Motion Designer" as subtitle
- **Language**: French for v1. Structure should anticipate future English translation (data attributes or similar), but no i18n system needed now

### Identity & Color Palette
- **Background**: Pure black (#000000)
- **Primary text**: Blanc cassé (#F5F0EB) — warm off-white, not harsh pure white
- **Secondary/accent**: Beige sable (#D4C5B2) — warm, organic, sophisticated
- **Contrast level**: Subtil et clean — no excessive glow, no neon. Accents are warm and understated
- **Overall vibe**: Sobre, classé, chaleureux — the work is the star, the UI stays elegant and quiet

### Meta & SEO
- **Page title**: "Tom Bernys | Portfolio"
- **Meta description**: Claude to write an optimized French description for SEO
- **OG image**: Placeholder for now — will need a static screenshot once the site is built
- **Domain**: Not yet decided — use relative paths, no hardcoded domain
- **Favicon**: Inline SVG data URI with initials "TB"

### Claude's Discretion
- Exact display font choice (must be rounded/organic, available on Google Fonts, convertible to typeface.json)
- Exact mono font choice for loader
- Meta description copywriting
- Favicon design details
- HTML landmark structure and ARIA attributes

</decisions>

<specifics>
## Specific Ideas

- The warm blanc cassé + beige palette is inspired by the organic/artistic direction — should feel like a high-end creative studio, not a tech startup
- PromptHQ's single-file approach (PROMPT_REPRODUCTION.md) is the technical reference for CDN import maps and file structure
- Font conversion (display font → typeface.json) is a prerequisite for Phase 3 and must be done in this phase

</specifics>

<deferred>
## Deferred Ideas

- English translation — noted for post-v1. Structure should be translation-friendly but no i18n implementation now.

</deferred>

---

*Phase: 01-scaffolding*
*Context gathered: 2026-03-22*
