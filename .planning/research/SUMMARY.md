# Project Research Summary

**Project:** Tom Bernys Portfolio — WebGL Single-Page Site
**Domain:** Creative portfolio, freelance video editor / motion designer
**Researched:** 2026-03-22
**Confidence:** HIGH (stack, pitfalls) / MEDIUM (features, architecture specifics)

## Executive Summary

This is an immersive single-page creative portfolio built to the PromptHQ quality benchmark: a WebGL hero featuring 3D glass text (Three.js TextGeometry + refraction shader), a GPU-accelerated Navier-Stokes fluid simulation (ping-pong render targets), instanced cylindrical tiles with bloom post-processing, and a project showcase of Vimeo-hosted work below the fold. The architecture is intentionally constraint-driven — no bundler, no framework, no build step — just a single `index.html` with an ES module import map pointing to CDN-pinned libraries. This constraint keeps deployment trivially simple and forces clean module boundaries without framework overhead.

The recommended approach is Three.js r175 (not the project-spec r170 and not the latest r183) with GSAP ScrollTrigger for scroll-to-WebGL bridging and the Vimeo Player SDK for lazy-loaded embeds. The entire GPU fluid simulation must be hand-rolled in GLSL fragment shaders — no library exists that meets the quality bar in a no-bundler environment. Device tier detection (based on `MAX_TEXTURE_SIZE` + `hardwareConcurrency`) gates three quality levels from the first frame, and mobile is a first-class concern throughout, not an afterthought.

The dominant risk category is mobile GPU performance. The combination of fluid simulation, UnrealBloomPass, instanced geometry, and uncapped device pixel ratio will produce under 15fps on mid-range phones if not actively defended against. Every major system — fluid sim resolution, bloom mip levels, DPR cap — must be tiered by device capability from the outset. Three pitfalls carry retroactive costs so high they must be addressed before writing the first render loop: WebGL context loss recovery, float texture extension checking (iOS), and render target disposal on resize. If these are retrofitted, they require architectural refactors; if they are designed in from the start, they add under 30 lines of code.

---

## Key Findings

### Recommended Stack

No npm install, no bundler. All libraries load from CDN via `<script type="importmap">` in `index.html`. Three.js r175 is the correct version pin: it delivers a clean WebGL 2 baseline (r175 removed WebGL 1), avoids the r182/r183 `PostProcessing → RenderPipeline` rename that breaks all `EffectComposer` examples, and is newer than the project spec (r170). GLSL shaders are the second major technology — particularly the fluid simulation pipeline (advection, divergence, pressure Jacobi iteration, gradient subtraction) and the glass refraction shader for the name hero.

**Core technologies:**
- **Three.js r175** — 3D rendering, scene graph, post-processing — sweet spot between stability and WebGL 2 baseline
- **Vanilla ES Modules + import maps** — application logic with no bundler overhead; officially supported by all modern browsers
- **GLSL (WebGL 2)** — custom shaders for glass refraction, fluid simulation, instanced tile animation
- **GSAP 3.14.2 + ScrollTrigger** — glitch loader animation, scroll-to-WebGL uniform bridge (ScrollTrigger now free post-Webflow acquisition)
- **@vimeo/player 2.30.3** — programmatic iframe control for lazy-loaded project embeds
- **facetype.js (build-time)** — converts TTF fonts to Three.js typeface.json format; run once at design time

**Critical version constraints:**
- All Three.js postprocessing addons must come from the exact same CDN version as three.js core — mixing versions breaks compositing
- Three.js r183+ must be avoided until the EffectComposer/RenderPipeline rename stabilizes across the ecosystem
- Use Three.js built-in `three/addons/postprocessing/` — NOT the npm `postprocessing` package, which requires a bundler

### Expected Features

The hero WebGL experience is not a differentiator — it IS the product. The entire point of this portfolio is that a video editor / motion designer demonstrates extraordinary craft through the site itself. The project showcase (Vimeo embeds), contact section, and client logos are functional requirements. Everything else is polish.

**Must have (table stakes):**
- WebGL hero with glass name + GPU fluid simulation + instanced tile background — the core brand statement
- Glitch-style loading screen — masks WebGL init delay; prevents blank-screen first impression
- Project showcase with 6–10 Vimeo embeds — the actual evidence of skill
- Contact section (email link + Netlify Forms or Formspree) — the conversion goal of the entire site
- Client / brand logos section — social proof with minimal implementation cost
- Responsive layout with WebGL mobile fallback — >50% of views are mobile; broken mobile loses clients
- Open Graph meta tags + semantic HTML — every shared link needs a preview; Googlebot needs crawlable text

**Should have (competitive):**
- Scroll-triggered hero-to-content transition — cinematic handoff from WebGL world to project gallery
- Per-project hover reveal / thumbnail motion — gives the gallery life before video loads
- Post-processing pipeline: bloom + vignette — signature visual quality marker
- Scroll-triggered section entry animations — polish pass on content reveal

**Defer to v1.x / v2+:**
- Per-project custom hover animations (v1.x after core is validated)
- Analytics integration (Plausible/Fathom) — defer until there is traffic worth measuring
- Custom Vimeo player controls — only if default embed UI feels off-brand
- Dark/light mode toggle — conflicts with deliberate dark aesthetic; revisit only on feedback

**Hard anti-features (do not build):**
- CMS / admin panel — breaks single-HTML constraint; 6–10 projects rarely change
- Horizontal scroll gallery — accessibility and mobile-hostile; already decided vertical in PROJECT.md
- Custom video hosting — Vimeo provides HLS, analytics, privacy controls for free

### Architecture Approach

The architecture separates into two subsystems communicating through a single shared `AppState` object updated at the top of every `requestAnimationFrame` tick: a WebGL subsystem (Three.js renderer, fluid sim, glass text, instanced background, post-processing) and a DOM subsystem (scroll controller, section observer, Vimeo lazy loader). Neither subsystem calls the other directly — they read from and write to `AppState`. This single-source-of-truth pattern prevents the scroll jank that comes from independent scroll listeners in two subsystems.

**Major components:**
1. **AppState (state.js)** — shared plain object: scrollY, scrollVelocity, scrollProgress, isMobile, deviceTier, activeSection, isLoaded
2. **SceneManager** — Three.js renderer init, RAF loop, resize, EffectComposer chain; sole owner of `renderer.render()`
3. **FluidSimulation** — ping-pong WebGLRenderTarget pair; GLSL advection, divergence, Jacobi pressure solve, gradient subtract; mouse/touch velocity injection
4. **GlassText** — TextGeometry + MeshPhysicalMaterial (transmission, ior, thickness) or custom refraction ShaderMaterial with background render target
5. **TilesBackground** — InstancedMesh with animated vertex/fragment shaders; procedural noise
6. **PostProcessor** — EffectComposer: RenderPass → UnrealBloomPass (tiered) → SMAAPass → ShaderPass (vignette)
7. **LoadingScreen** — orchestrates asset preload, shows glitch text reveal, resolves Promise to ungate RAF
8. **ScrollController** — passive scroll listener writes to AppState; RAF tick reads from AppState
9. **SectionController** — IntersectionObserver per section; sets AppState.activeSection
10. **VideoManager** — IntersectionObserver lazy-loads Vimeo iframes when project cards approach viewport

**Build dependency order:** AppState → SceneManager → DeviceTier detection → FluidSim → GlassText → TilesBackground → HeroScene → PostProcessor → LoadingScreen → ScrollController → SectionController → VideoManager → main.js bootstrap

### Critical Pitfalls

1. **WebGL context loss without recovery** — iOS silently kills context when tab backgrounds. Gate with `webglcontextlost` / `webglcontextrestored` handlers from day one; structure `initRenderer()` as a callable function, not inline code.

2. **Float texture extensions absent on iOS** — `OES_texture_float_linear` is unreliable on iOS 14+. Check `getExtension('OES_texture_float_linear')` before initializing fluid sim; if absent, fall back to static animated gradient or RGBA8 encoding.

3. **Render targets not disposed on resize** — every `WebGLRenderTarget` allocates GPU VRAM that JavaScript GC never frees. Call `.dispose()` on every custom render target before creating new ones in the resize handler; monitor `renderer.info.memory.textures`.

4. **UnrealBloomPass kills mobile performance** — 5 mip-level bloom on mobile consumes 90% of GPU budget. Tier bloom: `nMips: 5` desktop, `nMips: 2` mid-range mobile, disabled on low-end. Detect via `navigator.maxTouchPoints` + device tier classification.

5. **Vimeo iframes blocking initial load** — 8 simultaneous Vimeo iframes at DOMContentLoaded means 80+ network requests competing with WebGL init. Use IntersectionObserver to inject iframes only when project cards approach the viewport (rootMargin: "200px").

6. **Uncapped device pixel ratio** — iPhone 13/14/15 are DPR 3, meaning 9× the fragment work vs DPR 1. Hard cap: `Math.min(window.devicePixelRatio, isMobile ? 1.5 : 2)` in the very first renderer setup line.

7. **Post-processing breaks MSAA** — `EffectComposer` renders to `WebGLRenderTarget`, which does not support hardware MSAA. Add `SMAAPass` at the end of the composer chain; do not rely on `antialias: true` in renderer options.

---

## Implications for Roadmap

### Phase 1: Project Scaffolding and HTML Skeleton
**Rationale:** The CDN import map must be validated against the actual deployment host before anything else is built — a MIME type error on GitHub Pages / Netlify will silently break the entire module system. Semantic HTML scaffolding must also be established before the WebGL layer is built on top, or SEO remediation becomes a structural refactor. This phase has zero WebGL content but unblocks everything downstream.
**Delivers:** Working `index.html` with import map, validated CDN resolution, semantic HTML skeleton (h1, section landmarks, visually-hidden text), Open Graph meta tags, Vimeo placeholder divs with `data-vimeo-id`, font typeface.json converted and committed, deployment host validated.
**Addresses:** Open Graph meta tags, SEO crawlable HTML (Pitfall 12), CDN import map MIME errors (Pitfall 9)
**Avoids:** SEO empty canvas pitfall; import map breakage discovered late

### Phase 2: WebGL Foundation — Renderer, Device Tier, Loading Screen
**Rationale:** All rendering infrastructure must be in place, and device tier detection must be wired before the first GPU object is created — retrofitting tier detection after fluid sim and bloom are built requires touching every system. The loading screen's glitch reveal must run before Three.js initializes (Pitfall 15), so it is built here as a CSS/minimal-JS system that kicks off WebGL init in parallel.
**Delivers:** Three.js renderer with DPR capped, WebGL context loss recovery handlers, device tier classification (high/medium/low), GSAP glitch text reveal loading screen, RAF loop scaffold, AppState singleton, ScrollController with passive listeners.
**Addresses:** WebGL context loss (Pitfall 1), uncapped DPR (Pitfall 8), loader timing (Pitfall 15), scroll jank (Pitfall 6)
**Avoids:** All "never retrofit" pitfalls established here as patterns for all subsequent phases

### Phase 3: WebGL Hero — Fluid Simulation
**Rationale:** The fluid sim is the highest-complexity, highest-GPU-cost, most mobile-problematic system. Build it in isolation (rendering to a fullscreen quad, not yet integrated with the full scene) so it can be tested and tiered before other systems add GPU pressure. Float texture extension check must gate initialization here.
**Delivers:** Ping-pong FBO fluid simulation, GLSL advection/divergence/pressure/gradient shaders, mouse/touch velocity injection, device-tiered resolution (256/128/64), iOS float texture fallback, integration as background texture.
**Addresses:** GPU fluid simulation (P1 feature), float texture iOS incompatibility (Pitfall 2), mobile performance (Pitfall 4 equivalent for fluid)
**Research flag:** Fluid shader implementation details may benefit from phase-level research — Navier-Stokes GLSL is niche and implementation-specific

### Phase 4: WebGL Hero — Glass Text and Instanced Tiles
**Rationale:** These depend on the renderer and device tier infrastructure from Phase 2 but are independent of the fluid sim. Glass text requires the font JSON (resolved in Phase 1). Building TilesBackground and GlassText together makes sense as they share the same scene and are composed into HeroScene at the end of this phase.
**Delivers:** FontLoader + TextGeometry "Tom Bernys" with MeshPhysicalMaterial transmission/refraction, InstancedMesh cylindrical tile background with animated vertex shader, HeroScene composition, basic camera setup.
**Addresses:** Glass name hero (P1 feature, primary differentiator), instanced tile background (differentiator)
**Avoids:** InstancedMesh resize leak (Pitfall 14), font load race condition (Pitfall 7)

### Phase 5: Post-Processing Pipeline
**Rationale:** Post-processing is added after all scene objects exist so it can be tuned against real scene content. Must include SMAA from the start (not as a polish pass) because adding it later can reveal antialiasing issues that cascade into shader revisions. Bloom tiering by device must mirror the fluid sim tiering established in Phase 3.
**Delivers:** EffectComposer chain (RenderPass → UnrealBloomPass → SMAAPass → vignette ShaderPass), selective bloom via layer mask technique, tiered bloom quality (5/2/0 mip levels by device tier), below-fold post-processing skip (Pitfall from ARCHITECTURE: anti-pattern 2).
**Addresses:** Post-processing pipeline (P2 feature), bloom mobile performance (Pitfall 4), MSAA breakage (Pitfall 10)
**Avoids:** Full pipeline running below the fold (architecture anti-pattern 2)

### Phase 6: Scroll Integration and Hero-to-Content Transition
**Rationale:** Scroll behavior depends on both the WebGL hero (Phase 3–5) and the DOM content sections (not yet built). This phase wires GSAP ScrollTrigger to hero opacity and WebGL uniforms, and establishes SectionController. It bridges the two subsystems through AppState.
**Delivers:** GSAP ScrollTrigger pinning hero canvas during hero phase, scroll-progress-driven hero fade-out, SectionController with IntersectionObserver per section, CSS transition for content reveal, hero visibility gating in RAF (heroAlpha < 0.01 → skip all GPU work), prefers-reduced-motion check.
**Addresses:** Scroll-triggered hero-to-content transition (P2 feature), reduced-motion accessibility (Pitfall 11)
**Avoids:** Two independent scroll listeners (architecture anti-pattern 1), below-fold GPU waste (architecture anti-pattern 2)

### Phase 7: Project Showcase — Vimeo Embeds and Gallery
**Rationale:** The project gallery is the highest-value DOM content (the actual evidence of skill). VideoManager with IntersectionObserver lazy loading must be the primary implementation strategy — never refactored in from direct iframes. Hover states are added here as part of gallery construction, not as a separate polish pass.
**Delivers:** VideoManager lazy-loading Vimeo iframes via IntersectionObserver, project card grid layout, per-project hover reveal animation, `data-vimeo-id` placeholder → iframe swap, Vimeo embed with `dnt=1, responsive=1` params.
**Addresses:** Project showcase (P1 feature), per-project hover reveal (P2 feature), Vimeo blocking page load (Pitfall 5), Vimeo autoplay mobile block (Pitfall 13)
**Avoids:** Architecture anti-pattern 3 (iframes on page load)

### Phase 8: Remaining Content Sections and Contact
**Rationale:** Client logos and contact section are DOM-only with no WebGL dependencies. They can be built in parallel with Phase 7 or immediately after. Contact form backend (Netlify Forms or Formspree) is independent and swappable without architectural impact.
**Delivers:** Client / brand logo grid (SVG logos, horizontal layout), contact section with email link + static form pointed at Netlify Forms/Formspree, footer with social links (Vimeo profile, LinkedIn, Instagram), section entry animations (GSAP or IntersectionObserver CSS class toggles).
**Addresses:** Contact section (P1), client logos (P1), social links (table stakes), section entry animations (P2)

### Phase 9: Polish, Performance Audit, and Launch Prep
**Rationale:** A dedicated final phase forces verification of all pitfall checklists, Lighthouse scoring, and real-device testing — tasks that get skipped when folded into feature phases.
**Delivers:** Verified "looks done but isn't" checklist (all 10 items from PITFALLS.md), Lighthouse performance audit on real deployment URL, tested on real iOS Safari (context loss, float textures, DPR cap), tested on mid-range Android (bloom performance, fluid at tier), passive listener audit, `renderer.info.memory` stability across 5 resize events, final Open Graph image screenshot committed.
**Addresses:** All critical and moderate pitfalls (verification pass), production deployment validation

### Phase Ordering Rationale

- **Phases 1-2 must come first:** Import map validation and renderer + device tier infrastructure have no dependencies and unblock everything else. Retrofitting either is expensive.
- **Phase 3 (fluid sim) before Phases 4-5:** The fluid sim is the highest-risk system (iOS compatibility, mobile GPU cost). Proving it works in isolation before adding glass text and post-processing makes debugging dramatically easier.
- **Phase 5 (post-processing) after Phase 4:** You cannot tune bloom without real scene objects to bloom.
- **Phase 6 (scroll) requires Phases 3-5 complete:** The hero-to-content transition needs a complete hero to fade out.
- **Phase 7 (Vimeo / project gallery) is independent of Phases 3-6:** Could theoretically start after Phase 1, but ordering it here ensures the page structure is stable before video content is added.
- **Phase 9 is always last:** Cannot verify "looks done but isn't" until it is done.

### Research Flags

Phases likely needing deeper research during planning:
- **Phase 3 (Fluid Simulation):** The Navier-Stokes GLSL implementation is niche, with multiple variant approaches (MAC grid, semi-Lagrangian advection, staggered grids). The PavelDoGreat reference implementation exists but integrating it with Three.js WebGLRenderTarget API has known gotchas. Recommend `/gsd:research-phase` targeting the fluid-Three.js integration specifically.
- **Phase 4 (Glass Text):** The MeshPhysicalMaterial transmission/refraction approach requires a separate WebGLRenderTarget for the background scene texture. The exact setup for "real-time refraction of a WebGL background" (not environment map cube) is sparsely documented. Worth a targeted research pass.

Phases with standard patterns (skip research-phase):
- **Phase 1 (Scaffolding):** HTML structure, import maps, meta tags — all standard and well-documented.
- **Phase 2 (Renderer Foundation):** Three.js renderer init, DPR capping, context loss — patterns are explicit in official docs and PITFALLS.md.
- **Phase 5 (Post-Processing):** EffectComposer + UnrealBloomPass + SMAA is the documented Three.js pattern; STACK.md has the exact pipeline.
- **Phase 6 (Scroll):** GSAP ScrollTrigger is extensively documented; the AppState bridge pattern is explicit in ARCHITECTURE.md.
- **Phase 7 (Vimeo):** IntersectionObserver lazy loading is a solved, well-documented pattern.
- **Phase 8 (Content Sections):** DOM-only HTML/CSS with no novel patterns.

---

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| Stack | HIGH | All library choices verified against official docs, CDN package pages, and Three.js migration guide. Version pin rationale is based on known breaking changes, not inference. |
| Features | MEDIUM | Grounded in multiple industry sources (Vimeo blog, Codrops, motion design portfolio guides) but feature prioritization is partly judgement-based. No direct user research available — inferred from category norms. |
| Architecture | MEDIUM-HIGH | Component boundaries and data flow patterns verified across multiple Codrops 2025-2026 articles and Three.js forum. Vanilla JS single-file variant is inferred from well-documented components; no single source describes this exact setup end-to-end. |
| Pitfalls | HIGH | Core WebGL pitfalls (context loss, float textures, render target disposal, DPR) verified against Three.js official docs, MDN, and active GitHub issues. Confidence is highest here because these are documented failure modes with reproducible conditions. |

**Overall confidence:** HIGH for execution decisions (what to build and in what order). MEDIUM for scope decisions (which features belong in v1 vs v1.x) — those are judgment calls that should be validated against the project owner's priorities.

### Gaps to Address

- **Font choice is unresolved:** TextGeometry requires a typeface.json-format font. The specific typeface for "Tom Bernys" has not been selected. Font choice is a Phase 1 blocker — must be decided and converted before glass text can be built. Risk: if the chosen font is not available in typeface.json format, conversion quality via facetype.js may require glyph review.

- **Fluid sim resolution numbers are estimates:** ARCHITECTURE.md recommends 256/128/64 by tier; STACK.md recommends 1024/512/256. These are conflicting. The correct numbers depend on the actual frame budget on target hardware. Resolve in Phase 3 via profiling on real mid-range Android.

- **Contact form backend not chosen:** Netlify Forms (zero-config for Netlify-hosted sites) vs Formspree (host-agnostic) depends on deployment host. No decision recorded in research. Low stakes — either works — but should be decided before Phase 8.

- **Number and content of project showcase items:** Research assumes 6–10 Vimeo project videos. Actual count and Vimeo IDs are not in scope for this research but are required before Phase 7 can be completed.

---

## Sources

### Primary (HIGH confidence)
- https://threejs.org/manual/en/installation.html — CDN import map pattern
- https://threejs.org/docs/#api/en/geometries/TextGeometry — FontLoader/TextGeometry import paths
- https://threejs.org/docs/#manual/en/introduction/How-to-dispose-of-objects — render target disposal
- https://github.com/mrdoob/three.js/wiki/Migration-Guide — r170→r183 breaking changes
- https://gsap.com/docs/v3/Plugins/ScrollTrigger/ — GSAP 3.14.2, ScrollTrigger free tier
- https://www.jsdelivr.com/package/npm/@vimeo/player — @vimeo/player 2.30.3
- https://www.khronos.org/webgl/wiki/HandlingContextLost — context loss recovery
- https://developer.mozilla.org/en-US/docs/Web/API/WebGL_API/WebGL_best_practices — MDN WebGL best practices
- https://web.dev/learn/performance/lazy-load-images-and-iframe-elements — Vimeo lazy loading

### Secondary (MEDIUM confidence)
- https://github.com/PavelDoGreat/WebGL-Fluid-Simulation — fluid sim reference architecture (MIT, verified exists)
- https://tympanus.net/codrops/2026/02/23/composite-rendering-the-brilliance-behind-inspiring-webgl-transitions/ — composite rendering pattern
- https://tympanus.net/codrops/2025/02/11/building-efficient-three-js-scenes-optimize-performance-while-maintaining-quality/ — Three.js performance patterns
- https://tympanus.net/codrops/2025/11/27/letting-the-creative-process-shape-a-webgl-portfolio/ — WebGL portfolio architecture
- https://tympanus.net/codrops/2025/06/05/how-to-create-responsive-and-seo-friendly-webgl-text/ — SEO + WebGL text pattern
- https://discourse.threejs.org/t/fluid-simulation-using-shaders-tutorial/85109 — Three.js fluid sim pattern
- https://vimeo.com/blog/post/video-editor-portfolio — portfolio feature norms
- https://vanschneider.com/blog/portfolio-tips/motion-design-portfolio-tips/ — motion design portfolio dos/don'ts

### Tertiary (MEDIUM-LOW confidence)
- https://www.javaspring.net/blog/using-javascript-to-detect-device-cpu-gpu-performance/ — device tier detection via maxTextureSize + hardwareConcurrency (pattern confirmed across multiple sources but no single authoritative reference)
- https://robertmarshall.dev/blog/lazy-load-vimeo-video-iframe-show-on-scroll/ — IntersectionObserver + Vimeo iframe injection (community article, HIGH confidence for pattern, MEDIUM for Vimeo-specific details)

---
*Research completed: 2026-03-22*
*Ready for roadmap: yes*
