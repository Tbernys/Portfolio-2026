# Tom Bernys — Portfolio

## What This Is

A single-page portfolio website for Tom Bernys, freelance video editor and motion designer. The site combines an immersive dark WebGL hero (3D glass name, fluid simulation) with a vertical-scrolling showcase of 6-10 embedded Vimeo projects, client logos, and a contact CTA. Built as a static site (single HTML file, no framework) deployable anywhere.

## Core Value

Visitors instantly see the quality of Tom's work through an immersive visual experience and embedded video reels — every interaction should reinforce "this person makes things look incredible."

## Requirements

### Validated

- [x] Immersive WebGL hero section with "Tom Bernys" in 3D glass text — Validated in Phase 2: Hero Scene
- [x] GPU fluid simulation interactive with mouse/touch — Validated in Phase 2: Hero Scene
- [x] Animated background (cylindrical tiles, bloom, procedural patterns) — Validated in Phase 2: Hero Scene
- [x] Glitch-style loading screen with character-by-character text reveal — Validated in Phase 3: Loading & Entrance
- [x] Post-processing pipeline (bloom, vignette) — Validated in Phase 2: Hero Scene
- [x] Vertical scroll from hero into content sections — Validated in Phase 4: Scroll Integration
- [x] WebGL effects degrade on mobile (reduced DPR, fewer bloom passes, simplified fluid) — Validated in Phase 2: Hero Scene

### Active

- [ ] Project showcase section with 6-10 embedded Vimeo players
- [x] Client/brand logos section — Validated in Phase 6: Content Sections (placeholder logos, real assets pending)
- [x] Contact CTA section (email, form) — Validated in Phase 6: Content Sections (Formspree FORM_ID pending)
- [x] Fully responsive — desktop and mobile optimized — Validated in Phase 7: Responsive & Launch (single @media block, WebGL mobile tuning)
- [x] Static site — single HTML file, no framework, vanilla JS + Three.js via CDN — Validated in Phase 7: Responsive & Launch (deployed on Vercel + GitHub Pages)

### Out of Scope

- Multi-page site or routing — single page only
- CMS or admin panel — content is hardcoded
- Blog or articles section — portfolio focus only
- E-commerce / pricing page — contact for quotes instead
- OAuth or user accounts — no login needed
- Real-time chat — contact form/email sufficient

## Context

- **Inspiration 1 — Justin Buisson** (justinbuisson.com): Horizontal scroll, minimalist white, full-bleed project pages, custom typography (Durango Kid + SANSPLOMB), subtle CSS interactions. Built on ReadyMag. Key takeaway: each project as a cinematic full-page moment.
- **Inspiration 2 — PromptHQ** (prompthq.fr): Full WebGL immersion, dark background, 3D glass logo with refraction shader, GPU Navier-Stokes fluid sim, instanced cylindrical tile background, bloom pipeline, glitch loader. Single HTML file. Key takeaway: technical spectacle as brand statement.
- **Direction chosen**: Hybride dark — PromptHQ-level WebGL hero transitioning into a scrollable project gallery on dark background. Best of both worlds.
- Full technical documentation of PromptHQ available in PROMPT_REPRODUCTION.md and PROMPT_DOC.md for reference during implementation.

## Constraints

- **Stack**: Vanilla HTML/CSS/JS + Three.js r170 via CDN import maps. No React, no bundler, no npm. Single index.html file.
- **Hosting**: Portable — must work on GitHub Pages, Vercel, Netlify, or any static host. Served via HTTP (not file://).
- **Performance**: WebGL allege on mobile — reduced DPR (max 1.5), fewer bloom levels (2 vs 3), simplified fluid resolution. Target 60fps desktop, 30fps+ mobile.
- **Video**: All project videos hosted on Vimeo, embedded via iframe/player API.
- **Assets**: Procedurally generated where possible (textures, noise). Minimal external assets. Font via Google Fonts CDN.

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Single HTML file, no framework | Matches PromptHQ approach, simplest deployment, no build step | — Pending |
| Three.js r170 via CDN | Proven stack from PromptHQ reference, no npm needed | — Pending |
| Vertical scroll (not horizontal) | More natural on mobile, easier responsive handling | — Pending |
| Vimeo for video hosting | Pro quality, no ads, clean embed, industry standard for motion designers | — Pending |
| WebGL allege on mobile | Performance over fidelity on constrained devices | — Pending |
| Dark theme throughout | Matches video/motion design aesthetic, makes content pop | — Pending |

---
*Last updated: 2026-03-25 after Phase 4 (Scroll Integration) complete — cinematic hero-to-content scroll transition with parallax, entrance animations, and back-to-top orientation*
