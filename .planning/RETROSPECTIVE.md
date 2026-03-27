# Project Retrospective

*A living document updated after each milestone. Lessons feed forward into future planning.*

## Milestone: v1.0 — MVP

**Shipped:** 2026-03-27
**Phases:** 7 | **Plans:** 16 | **Timeline:** 5 days (2026-03-22 → 2026-03-27)

### What Was Built

- Full WebGL hero: glass "Tom Bernys" text with transmission/refraction, instanced cylindrical tile background, bloom+vignette post-processing, mouse/touch interaction
- Glitch-style loading screen: character scramble reveal, scanline sweep, film grain, shader warmup gate, clip-path dissolve transition
- Cinematic scroll: GSAP ScrollTrigger two-track (CSS parallax + Three.js mutations), content section entrance animations, glassmorphism CTA button, back-to-top orientation
- Project showcase: 3x3 Vimeo grid with oEmbed thumbnails, hover preview (Netflix effect), lightbox with @vimeo/player SDK, mobile double-tap, keyboard accessibility
- Content sections: client logo row, Formspree-backed contact form with inline feedback
- Mobile-responsive: single @media block + isMobile-gated WebGL tiering (DPR, FOV, tile count, antialias)
- Deployed on two static hosts: Vercel (production) + GitHub Pages

### What Worked

- **Single HTML file constraint** kept the architecture honest — no build system decisions to make, deployable anywhere
- **Phase sequencing** (scaffold → WebGL foundation → hero → scroll → content → responsive) matched actual dependency order exactly — zero re-work from wrong sequence
- **Summaries as living docs** — writing SUMMARY.md after each plan created a complete trail of decisions that made the PROJECT.md evolution trivial
- **Manual scramble fallback** — when GSAP Club plugins failed in ES module context, the setInterval approach was implemented immediately without debate and achieved identical visual result
- **Two-track ScrollTrigger pattern** (GSAP handles CSS, onUpdate handles Three.js) emerged organically and solved the mutation conflict cleanly — worth formalizing for future WebGL+GSAP work

### What Was Inefficient

- **ROADMAP.md was never updated during execution** — phases stayed "Not started" or "In Progress" even after completing, creating confusing pre-flight state at milestone close. Should update ROADMAP progress table after each plan completion
- **REQUIREMENTS.md checkboxes not maintained** — traceability table showed 10 pending items that were actually complete; adding `requirements-completed:` to SUMMARY frontmatter helps but the source document wasn't synced
- **Phase 03 summaries left with stale "PENDING" self-check** — review step at checkpoint wasn't finalized; self-check blocks in summaries should be completed before moving to next phase

### Patterns Established

- **importmap for CDN dependencies** — Three.js r175 addons resolved correctly from jsDelivr; no build step needed for complex WebGL dependency trees
- **fontReady chain** — async font load → buildTextMesh → setupPostProcessing → warmupShaders guarantees GPU assets compiled before first visible frame
- **Two-track ScrollTrigger** — GSAP timeline for CSS (canvas opacity/transform), ScrollTrigger.create onUpdate for Three.js object mutations (position, uniform values); never mix them
- **entranceComplete flag** — gates scroll-driven Three.js writes to prevent conflict with entry animation timeline; essential for any "enter then scroll" sequence
- **Canvas-scoped passive touch listeners** — prevents mobile scroll blocking; `canvas.addEventListener('touchstart', …, { passive: false })` scoped to canvas only
- **isMobile-gated WebGL tiering** — single boolean check gates FOV, scale, tile count, antialias; avoids MediaQuery listeners in RAF loop

### Key Lessons

1. **GSAP Club plugins don't work in browser ES module context** — any future plan using ClubGSAP plugins (SplitText, ScrambleText, MorphSVG) must plan a vanilla fallback upfront
2. **ROADMAP.md should be a living status board** — update it after every plan execution, not just at milestone boundaries
3. **Two deployment targets from the start** — having both Vercel and GitHub Pages as targets from Phase 1 caught host-specific issues early; worth keeping as constraint for future static sites
4. **Placeholder IDs in data arrays are tech debt that ages badly** — Vimeo IDs and Formspree FORM_ID should be flagged as immediate post-launch blockers in a TODO comment inside the code itself, not just in planning docs
5. **detect-gpu tier detection is a strong foundation** — building everything (antialias, tile count, bloom levels, DPR) on a single `gpuTier` value makes mobile optimization trivial later

### Cost Observations

- All phases executed with Claude Sonnet 4.6 (no Opus needed for this stack)
- Fastest plans: 1-3 min (targeted implementations with clear specs)
- Slowest plan: Phase 1-02 (~60 min, Comfortaa font conversion debugging)
- Most impactful single decision: choosing r175 over r170 (avoided downstream breaking changes at r182/r183)

---

## Cross-Milestone Trends

### Process Evolution

| Milestone | Phases | Plans | Key Change |
|-----------|--------|-------|------------|
| v1.0 MVP | 7 | 16 | Baseline established — vanilla WebGL portfolio |

### Cumulative Quality

| Milestone | LOC | Stack | Zero-Dep Additions |
|-----------|-----|-------|-------------------|
| v1.0 MVP | 3,099 | HTML/CSS/JS + Three.js r175 + GSAP | detect-gpu, @vimeo/player |
