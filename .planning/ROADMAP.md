# Roadmap: Tom Bernys Portfolio

## Overview

Seven phases build from static scaffolding upward through increasingly GPU-intensive WebGL systems, then close with DOM content and a responsive polish pass. Phase 1 validates the CDN import map before any GPU code is written. Phases 2-4 construct the immersive hero. Phases 5-6 add the evidence of skill (projects, logos, contact). Phase 7 hardens everything for mobile and real-device production.

## Phases

**Phase Numbering:**
- Integer phases (1, 2, 3): Planned milestone work
- Decimal phases (2.1, 2.2): Urgent insertions (marked with INSERTED)

Decimal phases appear between their surrounding integers in numeric order.

- [x] **Phase 1: Scaffolding** - Static HTML skeleton, CDN import map, deployment validation, font conversion
- [x] **Phase 2: WebGL Foundation** - Renderer, device tier detection, loading screen, RAF loop, context recovery (completed 2026-03-22)
- [ ] **Phase 3: Hero Scene** - Glass text, instanced tile background, mouse/touch interaction, post-processing pipeline
- [ ] **Phase 4: Scroll Integration** - GSAP ScrollTrigger hero fade, section controller, navigation indicator
- [ ] **Phase 5: Project Showcase** - Vimeo lazy-load gallery, project cards, hover animations, metadata
- [ ] **Phase 6: Content Sections** - Client logos, contact form, social links
- [ ] **Phase 7: Responsive and Launch** - Mobile WebGL tiering, layout reflow, performance audit, real-device verification

## Phase Details

### Phase 1: Scaffolding
**Goal**: A deployable, crawlable HTML shell that proves the CDN import map works before any WebGL code is written
**Depends on**: Nothing (first phase)
**Requirements**: TECH-01, TECH-02, TECH-03
**Success Criteria** (what must be TRUE):
  1. Visiting the deployed URL serves the page without module import errors in the browser console
  2. Three.js can be imported via the import map and a test scene renders (cube or sphere) on the production host
  3. A search engine crawler (or validator) sees semantic HTML landmarks (h1 "Tom Bernys", section elements, visually-hidden descriptive text) — not a blank canvas
  4. Open Graph meta tags render a correct link preview when the URL is pasted into Slack or iMessage
**Plans:** 2/2 plans complete

Plans:
- [x] 01-01-PLAN.md — Build complete index.html scaffold with import map, semantic HTML, meta tags, and Three.js test cube
- [x] 01-02-PLAN.md — Convert display font to typeface.json, deploy to static host, verify production

### Phase 2: WebGL Foundation
**Goal**: Users see a glitch-style loading screen while all GPU infrastructure initializes cleanly — on any device tier
**Depends on**: Phase 1
**Requirements**: TECH-04, TECH-05, TECH-06, LOAD-01, LOAD-02, LOAD-03
**Success Criteria** (what must be TRUE):
  1. Visitor sees "Tom Bernys" revealed character by character (glitch style) during the loading period — not a blank screen
  2. A scanline overlay animates during loading and fades out when the hero becomes visible
  3. The loading screen transitions smoothly to the hero scene once GPU assets are ready, or after the 5-second fallback
  4. On an iOS device, backgrounding and returning to the tab does not produce a black canvas — the WebGL context recovers
  5. Resizing the browser window five times in a row does not increase `renderer.info.memory.textures` (render targets are disposed correctly)
**Plans:** 3/3 plans complete

Plans:
- [ ] 02-01-PLAN.md — WebGL renderer, device tier detection, RAF loop, context recovery, resize handling
- [ ] 02-02-PLAN.md — Loading screen DOM/CSS, GSAP ScrambleText glitch reveal, scanline, film grain, progress bar
- [ ] 02-03-PLAN.md — Shader warmup gate, glitch dissolve transition, end-to-end verification

### Phase 3: Hero Scene
**Goal**: Visitors see the full immersive WebGL hero — "Tom Bernys" in 3D glass text floating over an animated cylindrical tile background, responding to mouse and touch
**Depends on**: Phase 2
**Requirements**: HERO-01, HERO-02, HERO-03, HERO-04, HERO-05
**Success Criteria** (what must be TRUE):
  1. "Tom Bernys" appears as extruded, refractive 3D glass text centered in the viewport
  2. Moving the mouse (or dragging on touch) causes the glass text to rotate in response; the text auto-spins and vertically floats when idle
  3. The background displays instanced cylindrical tiles with procedural animated patterns (noise, gradient) wrapping around the viewer, with crosshairs at grid intersections
  4. The WebGL canvas sits fixed behind scrollable DOM content and does not scroll with the page
  5. Bloom and vignette post-processing visibly enhance the scene — bright text/tiles glow; edges darken
**Plans:** 2 plans

Plans:
- [ ] 03-01-PLAN.md — Glass text mesh with env map, post-processing pipeline (bloom + vignette)
- [ ] 03-02-PLAN.md — Instanced tile background, crosshairs, mouse/touch interaction, idle float

### Phase 4: Scroll Integration
**Goal**: Scrolling down from the hero feels cinematic — the WebGL world fades away and content sections emerge naturally, with orientation cues present throughout
**Depends on**: Phase 3
**Requirements**: SCRL-01, SCRL-02, SCRL-03
**Success Criteria** (what must be TRUE):
  1. Visitor can scroll vertically from the hero into content sections without the page feeling "stuck" or hijacked
  2. As the visitor scrolls down, the hero WebGL canvas visibly fades or scales away, revealing content beneath — the transition feels intentional, not abrupt
  3. A fixed navigation element or scroll indicator is visible throughout the page, helping the visitor understand their position
**Plans**: TBD

### Phase 5: Project Showcase
**Goal**: Visitors can browse Tom's work — each project displays a title, role metadata, and a Vimeo player that loads only when the card approaches the viewport
**Depends on**: Phase 4
**Requirements**: PROJ-01, PROJ-02, PROJ-03, PROJ-04
**Success Criteria** (what must be TRUE):
  1. A section shows 6-10 project cards, each with a title and role/credits metadata (direction, motion, editing, etc.)
  2. Vimeo players load only when a project card scrolls near the viewport — the initial page load does not initiate Vimeo network requests for off-screen videos
  3. Hovering over a project card triggers a visible animation (title reveal, scale, or color shift)
  4. Vimeo videos play correctly on desktop; on mobile the player is present (autoplay is not expected)
**Plans**: TBD

### Phase 6: Content Sections
**Goal**: Visitors see who Tom has worked with and can reach him — logos communicate credibility, contact options are clear and functional
**Depends on**: Phase 5
**Requirements**: CLNT-01, CLNT-02, CTCT-01, CTCT-02, CTCT-03
**Success Criteria** (what must be TRUE):
  1. A client/brand logos section displays recognizable logos (SVG or high-quality images) in a grid or horizontal row
  2. Visitor can submit the contact form (name, email, message) and receives a clear success or error message after submission
  3. A direct email link is visible as an alternative to the form — clicking it opens the visitor's mail client
**Plans**: TBD

### Phase 7: Responsive and Launch
**Goal**: The site works correctly and performs at target frame rate on real mobile devices — every v1 requirement is met on both desktop and mobile before the URL goes live
**Depends on**: Phase 6
**Requirements**: RESP-01, RESP-02, RESP-03, RESP-04, RESP-05
**Success Criteria** (what must be TRUE):
  1. On a real mid-range Android device, the site maintains 30fps or better — verified by browser DevTools performance trace
  2. On a real iPhone, the WebGL canvas renders at max DPR 1.5 and the instanced tile count and shader complexity are visibly reduced compared to desktop
  3. The 3D glass text is legible and appropriately sized on a 375px-wide mobile viewport
  4. All content sections — hero, projects, logos, contact — reflow to a single-column layout on mobile with no horizontal overflow or clipped text
  5. The site deploys and loads correctly on at least two static hosts (e.g., GitHub Pages and Netlify) with no console errors
**Plans**: TBD

## Progress

**Execution Order:**
Phases execute in numeric order: 1 -> 2 -> 3 -> 4 -> 5 -> 6 -> 7

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Scaffolding | 1/2 | Complete    | 2026-03-22 |
| 2. WebGL Foundation | 3/3 | Complete    | 2026-03-22 |
| 3. Hero Scene | 0/TBD | Not started | - |
| 4. Scroll Integration | 0/TBD | Not started | - |
| 5. Project Showcase | 0/TBD | Not started | - |
| 6. Content Sections | 0/TBD | Not started | - |
| 7. Responsive and Launch | 0/TBD | Not started | - |
