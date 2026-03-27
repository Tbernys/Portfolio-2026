# Tom Bernys — Portfolio

## What This Is

A single-page portfolio website for Tom Bernys, freelance video editor and motion designer. The site combines an immersive dark WebGL hero (3D glass "Tom Bernys" text, instanced cylindrical tile background, bloom+vignette post-processing) with a vertical-scrolling showcase of Vimeo project cards, client logos, and a contact form. Built as a static site (single HTML file, vanilla JS + Three.js r175 via CDN importmap) — deployable anywhere, zero build step.

## Core Value

Visitors instantly see the quality of Tom's work through an immersive visual experience and embedded video reels — every interaction should reinforce "this person makes things look incredible."

## Requirements

### Validated

- ✓ Static site — single HTML file, vanilla JS + Three.js via CDN importmap — v1.0
- ✓ Glitch-style loading screen with character-by-character text reveal — v1.0
- ✓ Loading screen transitions smoothly to hero once GPU assets ready — v1.0
- ✓ Scanline overlay animates during loading and fades out on transition — v1.0
- ✓ "Tom Bernys" rendered as 3D extruded glass text (TextGeometry + MeshPhysicalMaterial transmission/refraction) — v1.0
- ✓ Glass text reacts to mouse/touch — rotation impulse, auto-spin, vertical float — v1.0
- ✓ Instanced cylindrical tile background with procedural patterns wrapping viewer — v1.0
- ✓ Crosshairs at tile grid intersections — v1.0
- ✓ WebGL canvas fixed behind scrollable DOM — v1.0
- ✓ Bloom + vignette post-processing pipeline — v1.0
- ✓ Vertical scroll from hero into content sections — v1.0
- ✓ Hero canvas fades/scales on scroll, revealing content beneath — v1.0
- ✓ Fixed back-to-top button for persistent page orientation — v1.0
- ✓ Project showcase: 3x3 grid with titles, thumbnails, role metadata — v1.0
- ✓ Vimeo lazy-load (IntersectionObserver) + hover preview + lightbox with SDK — v1.0
- ✓ Client logos section (placeholder brands) — v1.0
- ✓ Contact form (Formspree) + direct mailto link + inline feedback — v1.0
- ✓ Fully responsive — single @media block, single-column mobile layout — v1.0
- ✓ WebGL mobile tuning — DPR max 1.5, reduced FOV/scale/tiles, no antialias on low-tier — v1.0
- ✓ Deployed on Vercel + GitHub Pages — v1.0
- ✓ WebGL context loss handled (iOS backgrounding recovery) — v1.0
- ✓ Render targets disposed on resize (no GPU memory leaks) — v1.0
- ✓ Shaders compiled during loading screen, not on first visible frame — v1.0

### Active

- [ ] Replace placeholder Vimeo IDs in PROJECTS array with real video IDs (9 entries, all currently `76979871`)
- [ ] Create Formspree account and replace `REPLACE_WITH_FORM_ID` in index.html contact form action
- [ ] Replace client logo placeholders (Nike, Canal+, Dior, Red Bull, Ubisoft, Netflix) with real SVG/image assets

### Out of Scope

- Multi-page site or routing — single page by design
- CMS or admin panel — content hardcoded, edits take <10 min
- Blog or articles section — portfolio focus only
- E-commerce / pricing page — contact for quotes instead
- OAuth or user accounts — no login needed
- Real-time chat — contact form/email sufficient
- Custom video player — Vimeo provides HLS, analytics, privacy

## Context

- **v1.0 shipped:** 2026-03-27 — 5 days, 7 phases, 16 plans, 3,099 LOC
- **Stack confirmed:** Three.js r175 (not r170 as originally spec'd — r175 = WebGL 2 baseline, avoids r182/r183 rename breakage), GSAP 3.14.2 + ScrollTrigger, @vimeo/player 2.30.3, Formspree for contact
- **Both hosts live:** Vercel (production) + GitHub Pages (gh-pages branch)
- **Known post-launch actions:** Vimeo IDs + Formspree FORM_ID (user must complete)
- **Inspiration confirmed:** PromptHQ-level WebGL hero + Justin Buisson-style project gallery — hybrid approach validated

## Constraints

- **Stack**: Vanilla HTML/CSS/JS + Three.js r175 via CDN import maps. No React, no bundler, no npm. Single index.html file.
- **Hosting**: Portable — works on GitHub Pages, Vercel, Netlify. Served via HTTP (not file://).
- **Performance**: WebGL reduces on mobile — DPR max 1.5, FOV 65, scale 0.62, 4x3 tile quad-tree, no antialias on tier 0/1. Target 60fps desktop, 30fps+ mobile.
- **Video**: All project videos hosted on Vimeo, embedded via iframe/player API.
- **Assets**: Procedurally generated textures/noise. Font via Google Fonts CDN + converted typeface.json.

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Single HTML file, no framework | Matches PromptHQ approach, simplest deployment, no build step | ✓ Validated — 3,099 LOC manageable in one file |
| Three.js r175 (not r170) | r175 = WebGL 2 baseline; r182/r183 introduced breaking renames | ✓ Good — zero import errors in production |
| GSAP ScrambleText replaced with manual setInterval | GSAP Club plugins can't be imported via browser ES module importmap | ✓ Good — identical visual result, no Club dependency |
| clip-path inset() for dissolve transition | Simpler than polygon splitting, effective glitch aesthetic | ✓ Good |
| Two-track ScrollTrigger (GSAP CSS + onUpdate for Three.js) | Direct Three.js mutations inside GSAP timeline caused conflicts | ✓ Good — clean separation of concerns |
| Canvas-scoped passive touch listeners (not window-level) | Window passive:false blocked mobile scroll entirely | ✓ Good — touch rotation preserved, scroll unblocked |
| Raw iframe for hover preview (not SDK) | Avoids 9 concurrent Player SDK instances in memory | ✓ Good — SDK only for lightbox where pause() is required |
| Vimeo for video hosting | Pro quality, no ads, clean embed, industry standard for motion designers | — Pending (real IDs not yet in code) |
| Dark theme throughout | Matches video/motion design aesthetic, makes content pop | ✓ Validated |
| Vertical scroll (not horizontal) | More natural on mobile, easier responsive handling | ✓ Validated |

---
*Last updated: 2026-03-27 after v1.0 milestone — full portfolio deployed on Vercel + GitHub Pages*
