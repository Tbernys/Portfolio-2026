# Requirements: Tom Bernys Portfolio

**Defined:** 2026-03-22
**Core Value:** Visitors instantly see the quality of Tom's work through an immersive visual experience and embedded video reels

## v1 Requirements

### Loading

- [ ] **LOAD-01**: Visitor sees a glitch-style loading screen with "Tom Bernys" revealed character by character while WebGL initializes
- [ ] **LOAD-02**: Loading screen transitions smoothly to the hero scene once 3D assets are ready (or fallback after 5s)
- [ ] **LOAD-03**: Scanline overlay animates during loading and fades out on transition

### Hero WebGL

- [ ] **HERO-01**: Visitor sees "Tom Bernys" rendered as 3D extruded glass text (TextGeometry + refraction/transmission shader) floating at center of viewport
- [ ] **HERO-02**: Glass text reacts to mouse/touch — rotation impulse on interaction, auto-spin on Z-axis, vertical float animation
- [ ] **HERO-03**: Background displays instanced cylindrical tiles with procedural patterns (noise, grid, gradient) wrapping around the viewer
- [ ] **HERO-04**: Crosshairs render at tile grid intersections for visual detail
- [ ] **HERO-05**: WebGL canvas is fixed behind scrollable DOM content

### Responsive / Mobile

- [ ] **RESP-01**: Site layout adapts gracefully to desktop (1024px+) and mobile (<768px) viewports
- [ ] **RESP-02**: WebGL renders at reduced DPR (max 1.5) on mobile devices
- [ ] **RESP-03**: Instanced tile count and shader complexity are reduced on mobile for 30fps+ target
- [ ] **RESP-04**: 3D text scales appropriately for mobile viewport (smaller size, adjusted FOV)
- [ ] **RESP-05**: All content sections reflow to single-column layout on mobile

### Scroll & Navigation

- [ ] **SCRL-01**: Visitor scrolls vertically from hero into content sections
- [ ] **SCRL-02**: Hero WebGL canvas fades/scales as visitor scrolls down, revealing content beneath
- [ ] **SCRL-03**: Fixed navigation or scroll indicator helps visitor orient on the page

### Projects

- [ ] **PROJ-01**: Visitor sees a showcase section displaying 6-10 projects with titles and thumbnails
- [ ] **PROJ-02**: Each project embeds a Vimeo player that loads lazily (IntersectionObserver)
- [ ] **PROJ-03**: Project thumbnails/cards have hover animation (title reveal, scale, or color shift)
- [ ] **PROJ-04**: Each project displays role/credits metadata (direction, motion, editing, etc.)

### Clients

- [ ] **CLNT-01**: Visitor sees a section displaying client/brand logos
- [ ] **CLNT-02**: Logos are SVG or high-quality images, displayed in a grid or horizontal row

### Contact

- [ ] **CTCT-01**: Visitor can submit a contact form (name, email, message) via Formspree or Netlify Forms
- [ ] **CTCT-02**: Visitor sees a direct email link as alternative to the form
- [ ] **CTCT-03**: Form provides clear feedback on submission (success/error state)

### Technical Foundation

- [x] **TECH-01**: Entire site is a single index.html file with inline CSS and JS (no build tools, no framework)
- [x] **TECH-02**: Three.js loaded via CDN import maps (jsDelivr)
- [x] **TECH-03**: Site deploys on any static host (GitHub Pages, Vercel, Netlify)
- [x] **TECH-04**: WebGL context loss is handled gracefully (re-initialization on `webglcontextrestored`)
- [x] **TECH-05**: All render targets are properly disposed on resize to prevent GPU memory leaks
- [ ] **TECH-06**: Shaders compile during loading screen, not on first visible frame

## v2 Requirements

### Enhanced WebGL

- **WGLX-01**: GPU fluid simulation (Navier-Stokes) interactive with mouse/touch
- **WGLX-02**: Post-processing bloom pipeline (bright extraction, multi-level Gaussian blur, composite)
- **WGLX-03**: Vignette post-processing pass
- **WGLX-04**: Bloom reduced to 2 levels on mobile (vs 3 desktop)

### Content

- **CONT-01**: Social/professional links (Vimeo profile, Instagram, LinkedIn)
- **CONT-02**: Open Graph and Twitter Card meta tags for link previews
- **CONT-03**: Favicon (inline SVG data URI)

### Polish

- **PLSH-01**: Per-project hover reveals with WebGL-driven distortion effects
- **PLSH-02**: Section entry animations triggered by scroll
- **PLSH-03**: Custom cursor trail effect on desktop

## Out of Scope

| Feature | Reason |
|---------|--------|
| Multi-page routing | Single page by design — simplicity and performance |
| CMS / admin panel | 6-10 projects rarely change; hardcoded HTML edits take <10 min |
| Blog / articles | Motion design work speaks louder; stale blog hurts credibility |
| Custom video player | Vimeo provides HLS streaming, analytics, privacy — no ROI to replace |
| Chat widget | Interrupts visual experience; solo freelancer can't maintain real-time |
| Horizontal scroll | Harder on mobile, scroll hijacking frustrates trackpad users |
| Password-protected pages | NDA work shown as "available on request" thumbnails instead |
| OAuth / user accounts | No login needed for a portfolio site |

## Traceability

| Requirement | Phase | Status |
|-------------|-------|--------|
| LOAD-01 | Phase 2 | Pending |
| LOAD-02 | Phase 2 | Pending |
| LOAD-03 | Phase 2 | Pending |
| HERO-01 | Phase 3 | Pending |
| HERO-02 | Phase 3 | Pending |
| HERO-03 | Phase 3 | Pending |
| HERO-04 | Phase 3 | Pending |
| HERO-05 | Phase 3 | Pending |
| RESP-01 | Phase 7 | Pending |
| RESP-02 | Phase 7 | Pending |
| RESP-03 | Phase 7 | Pending |
| RESP-04 | Phase 7 | Pending |
| RESP-05 | Phase 7 | Pending |
| SCRL-01 | Phase 4 | Pending |
| SCRL-02 | Phase 4 | Pending |
| SCRL-03 | Phase 4 | Pending |
| PROJ-01 | Phase 5 | Pending |
| PROJ-02 | Phase 5 | Pending |
| PROJ-03 | Phase 5 | Pending |
| PROJ-04 | Phase 5 | Pending |
| CLNT-01 | Phase 6 | Pending |
| CLNT-02 | Phase 6 | Pending |
| CTCT-01 | Phase 6 | Pending |
| CTCT-02 | Phase 6 | Pending |
| CTCT-03 | Phase 6 | Pending |
| TECH-01 | Phase 1 | Complete |
| TECH-02 | Phase 1 | Complete |
| TECH-03 | Phase 1 | Complete |
| TECH-04 | Phase 2 | Complete |
| TECH-05 | Phase 2 | Complete |
| TECH-06 | Phase 2 | Pending |

**Coverage:**
- v1 requirements: 31 total
- Mapped to phases: 31
- Unmapped: 0

---
*Requirements defined: 2026-03-22*
*Last updated: 2026-03-22 — traceability mapped after roadmap creation*
