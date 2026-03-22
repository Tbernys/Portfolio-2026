# Architecture Research

**Domain:** Immersive WebGL single-page portfolio (Three.js hero + scrollable DOM content)
**Researched:** 2026-03-22
**Confidence:** MEDIUM-HIGH (patterns verified across multiple current sources; specific vanilla JS single-file variants inferred from well-documented components)

## Standard Architecture

### System Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                        USER AGENT (Browser)                      │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                   index.html (entry point)               │    │
│  │  ┌──────────────────┐    ┌──────────────────────────┐   │    │
│  │  │  <canvas>        │    │  <div id="content">      │   │    │
│  │  │  (WebGL layer)   │    │  (DOM scroll layer)      │   │    │
│  │  └────────┬─────────┘    └────────────┬─────────────┘   │    │
│  └───────────┼─────────────────────────── ┼ ───────────────┘    │
├──────────────┼─────────────────────────── ┼ ───────────────────┤
│              │     JS Module Layer         │                     │
│  ┌───────────▼──────────────┐  ┌──────────▼─────────────────┐  │
│  │   WebGL Subsystem        │  │   DOM / Scroll Subsystem    │  │
│  │  ┌────────────────────┐  │  │  ┌──────────────────────┐  │  │
│  │  │  SceneManager      │  │  │  │  ScrollController     │  │  │
│  │  │  ├─ HeroScene      │  │  │  │  (scroll position,    │  │  │
│  │  │  │  ├─ FluidSim    │  │  │  │   velocity, progress) │  │  │
│  │  │  │  ├─ GlassText   │  │  │  └──────────┬───────────┘  │  │
│  │  │  │  └─ TilesBG     │  │  │  ┌──────────▼───────────┐  │  │
│  │  │  └─ PostProcessor  │  │  │  │  SectionController   │  │  │
│  │  │     ├─ BloomPass   │  │  │  │  (hero, projects,    │  │  │
│  │  │     └─ VignettePass│  │  │  │   logos, contact)    │  │  │
│  │  └────────────────────┘  │  └──────────────────────────┘  │  │
│  └──────────────────────────┘  └──────────────────────────────┘  │
├─────────────────────────────────────────────────────────────────┤
│                     Shared Bus                                    │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  AppState: { scrollY, scrollVelocity, isMobile,          │    │
│  │             deviceTier, activeSection, isLoaded }        │    │
│  └─────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
```

### Component Responsibilities

| Component | Responsibility | Typical Implementation |
|-----------|----------------|------------------------|
| SceneManager | Three.js renderer init, RAF loop, resize handling | Class with `init()`, `tick()`, `resize()` methods |
| HeroScene | Contains all hero 3D objects; owns camera | Scene + PerspectiveCamera, groups for logical layers |
| FluidSim | GPU Navier-Stokes simulation, writes velocity/density to textures | Ping-pong render targets, custom GLSL shaders |
| GlassText | TextGeometry + custom refraction shader for "Tom Bernys" | MeshPhysicalMaterial or custom shader with envMap |
| TilesBackground | Instanced cylindrical tile meshes, procedural noise animation | InstancedMesh, custom vertex/fragment shaders |
| PostProcessor | EffectComposer chain: Bloom → Vignette → tonemap | `three/addons/postprocessing/` pipeline |
| ScrollController | Single source of scroll truth for RAF tick | Plain scroll listener or smooth scroll proxy |
| SectionController | Tracks which DOM section is active, fires transition callbacks | IntersectionObserver per section |
| VideoManager | Lazy instantiates Vimeo iframes on scroll-into-view | IntersectionObserver + dynamic iframe injection |
| AppState | Shared mutable object — scroll values, device tier, flags | Singleton plain object; both subsystems read/write |
| LoadingScreen | Orchestrates asset preload, shows glitch text reveal | Async sequence with Three.js LoadingManager |

## Recommended Project Structure

Because this is a single HTML file with no bundler, modules are loaded via `<script type="importmap">` + ES module `<script type="module">`. The logical separation lives in the module files referenced by the HTML.

```
index.html                      # Entry point — markup + importmap + <script type="module" src="./js/main.js">
js/
├── main.js                     # App bootstrap: init order, RAF loop, resize listener
├── state.js                    # AppState singleton — shared scroll/device values
├── LoadingScreen.js            # Preload orchestration, glitch text reveal, fade-out
├── ScrollController.js         # Scroll position, velocity, progress (0-1 over page)
├── SectionController.js        # IntersectionObserver per section, fires enter/leave events
├── webgl/
│   ├── SceneManager.js         # Renderer, camera, resize, EffectComposer, RAF hook
│   ├── HeroScene.js            # Composes all hero 3D objects, exposes tick(dt, state)
│   ├── FluidSimulation.js      # Ping-pong FBO fluid, mouse input → velocity injection
│   ├── GlassText.js            # TextGeometry + refraction shader
│   ├── TilesBackground.js      # InstancedMesh tiles + animated shader
│   └── shaders/
│       ├── glass.vert.glsl
│       ├── glass.frag.glsl
│       ├── fluid-advect.frag.glsl
│       ├── fluid-divergence.frag.glsl
│       ├── fluid-pressure.frag.glsl
│       └── fluid-gradient.frag.glsl
└── dom/
    ├── VideoManager.js         # Vimeo lazy-load via IntersectionObserver
    └── ContactSection.js       # Email link, social links, simple reveal animation
assets/
├── fonts/                      # JSON font for TextGeometry (Typeface.js format)
└── (no textures — procedural)
```

### Structure Rationale

- **js/webgl/**: All GPU concerns isolated here. SceneManager is the only entry point that touches the renderer — nothing else calls `renderer.render()`.
- **js/dom/**: DOM-only logic. No Three.js imports allowed. Communicates through AppState only.
- **js/state.js**: The coupling seam. Both subsystems read/write this object each frame, avoiding direct references between WebGL and DOM code.
- **shaders/ as .glsl files**: Loaded via `fetch()` or inlined as tagged template literals. Keeping them separate enables syntax highlighting and easier iteration.
- **Single index.html**: Matches the PromptHQ reference pattern. No build step. Importmap maps `"three"` and `"three/addons/"` to CDN URLs (jsDelivr/esm.sh).

## Architectural Patterns

### Pattern 1: Shared AppState as RAF-tick bus

**What:** A single plain object (`AppState`) is updated at the top of every `requestAnimationFrame` callback. Both the WebGL subsystem and DOM subsystem read from it — neither calls the other directly.

**When to use:** Whenever two systems need the same scroll position, velocity, or device flags without coupling their implementations.

**Trade-offs:** Simple and zero-dependency. Scales to ~10 shared values before becoming unwieldy. Does not need reactive frameworks.

**Example:**
```javascript
// state.js
export const AppState = {
  scrollY: 0,
  scrollVelocity: 0,
  scrollProgress: 0, // 0-1 over full page height
  isMobile: false,
  deviceTier: 'high', // 'high' | 'medium' | 'low'
  activeSection: 'hero',
  isLoaded: false,
};

// main.js — RAF loop
function tick(timestamp) {
  requestAnimationFrame(tick);
  AppState.scrollY = ScrollController.current;
  AppState.scrollVelocity = ScrollController.velocity;
  SceneManager.tick(timestamp, AppState); // WebGL reads AppState
}
```

### Pattern 2: Ping-Pong Render Targets for Fluid Simulation

**What:** The fluid simulation uses two `WebGLRenderTarget` objects that swap roles each frame: one is read (input texture), one is written (output texture). This is the standard GPU-side stateful computation pattern.

**When to use:** Any simulation that needs to read previous frame state — fluid, particles, reaction-diffusion.

**Trade-offs:** Requires HALF_FLOAT textures for precision. Each simulation step (advect, diverge, pressure, gradient subtract) is a full-screen quad render. 5-6 passes per frame; significant GPU cost on mobile — reduce simulation resolution.

**Example:**
```javascript
// FluidSimulation.js
class FluidSimulation {
  constructor(renderer, resolution = 128) {
    this.res = resolution;
    this.read = createRenderTarget(resolution);
    this.write = createRenderTarget(resolution);
  }
  swap() {
    [this.read, this.write] = [this.write, this.read];
  }
  step(dt) {
    this.advect(dt);   this.swap();
    this.diverge();    this.swap();
    this.pressure();   this.swap(); // iterate 20-30x
    this.gradient();   this.swap();
    // this.read now holds current velocity field
  }
}
```

### Pattern 3: Hero-to-Content Scroll Transition via Opacity + Pointer-Events

**What:** The WebGL canvas sits `position: fixed` behind everything. The DOM content layer scrolls normally above it. As the user scrolls past the hero threshold (e.g. `scrollProgress > 0.15`), the hero 3D objects fade out (uniforms or scene opacity) and DOM content fades in. The canvas stays alive but renders a simplified or paused scene below the fold.

**When to use:** When WebGL and DOM content must coexist without a hard cut. The canvas is always present; what changes is what it draws.

**Trade-offs:** Canvas burns GPU even when invisible. Mitigate by reducing render quality below fold (skip post-processing, pause fluid sim when `scrollProgress > 0.3`). Simpler than routing to separate pages.

**Example:**
```javascript
// HeroScene.js tick
tick(dt, state) {
  const heroAlpha = 1 - smoothstep(0, 0.2, state.scrollProgress);
  this.heroGroup.visible = heroAlpha > 0.01;
  if (heroAlpha < 0.01) return; // skip all work when hero is fully scrolled past
  this.updateUniforms(state);
  this.fluidSim.step(dt);
}
```

### Pattern 4: Device Tier Detection at Init

**What:** Before starting the render loop, query the WebGL context for GPU info and run a brief CPU benchmark to classify the device as 'high', 'medium', or 'low'. Store in `AppState.deviceTier`. All subsystems read this to gate effects.

**When to use:** Always. Mobile devices cannot run the full fluid sim + 3-level bloom at native DPR.

**Trade-offs:** GPU vendor strings are unreliable; `maxTextureSize` and `hardwareConcurrency` are more stable signals. Provide a manual override as a fallback.

**Example:**
```javascript
function detectDeviceTier(gl) {
  const ext = gl.getExtension('WEBGL_debug_renderer_info');
  const renderer = ext
    ? gl.getParameter(ext.UNMASKED_RENDERER_WEBGL)
    : '';
  const maxTex = gl.getParameter(gl.MAX_TEXTURE_SIZE);
  const cores = navigator.hardwareConcurrency ?? 4;
  if (maxTex >= 8192 && cores >= 8) return 'high';
  if (maxTex >= 4096 && cores >= 4) return 'medium';
  return 'low';
}
// Apply tier
const tier = detectDeviceTier(renderer.getContext());
AppState.deviceTier = tier;
const fluidRes    = { high: 256, medium: 128, low: 64 }[tier];
const bloomLevels = { high: 3,   medium: 2,   low: 0  }[tier];
const dpr         = Math.min(window.devicePixelRatio, { high: 2, medium: 1.5, low: 1 }[tier]);
```

## Data Flow

### Primary Render Flow (each frame)

```
requestAnimationFrame(tick)
    │
    ▼
ScrollController.update()       ← reads window.scrollY (or smooth value)
    │
    ▼
AppState update                 ← scrollY, velocity, progress written here
    │
    ├──► HeroScene.tick(dt, AppState)
    │        │
    │        ├── FluidSim.step(dt)        ← ping-pong render targets
    │        │     └── mouse/touch → velocity field → density texture
    │        ├── GlassText uniforms       ← envMap, refraction strength
    │        ├── TilesBackground uniforms ← noise time, scroll offset
    │        └── heroAlpha from scrollProgress
    │
    └──► EffectComposer.render()          ← Bloom → Vignette → screen
```

### Mouse/Touch Input Flow

```
window mousemove / touchmove
    │
    ▼
InputManager.onPointer(e)
    │
    ▼
NDC coordinates (-1 to 1)
    │
    ▼
FluidSim.injectVelocity(x, y, dx, dy)   ← adds impulse to velocity field
```

### Scroll-to-Section Transition Flow

```
User scrolls
    │
    ▼
ScrollController (scroll listener)
    ├── AppState.scrollY updated
    └── AppState.scrollProgress = scrollY / (pageHeight - viewportHeight)
    │
    ▼
SectionController (IntersectionObserver callbacks)
    └── AppState.activeSection = 'hero' | 'projects' | 'logos' | 'contact'
    │
    ▼
Per-frame: HeroScene reads scrollProgress
    └── heroGroup fades out, post-processing skipped at threshold
    │
    ▼
Per-frame: DOM content reveals (CSS class toggle driven by activeSection)
```

### Vimeo Lazy Load Flow

```
Page load: iframes NOT in DOM
    │
    ▼
IntersectionObserver watching each .project-card
    │
    ▼
Card enters viewport (rootMargin ~200px ahead)
    │
    ▼
VideoManager.load(card)
    ├── Creates <iframe src="https://player.vimeo.com/video/ID?autoplay=0">
    └── Appends to card, marks card as loaded (prevents double-load)
```

## Scaling Considerations

This is a static portfolio. "Scaling" means device/browser range, not server load.

| Scale | Architecture Adjustments |
|-------|--------------------------|
| Desktop high-end | Full pipeline: 256-res fluid, 3-level bloom, DPR up to 2, all effects |
| Desktop mid-range | 128-res fluid, 2-level bloom, DPR capped at 1.5 |
| Mobile (deviceTier medium) | 128-res fluid at half-rate (step every other frame), 1-level bloom, DPR 1.5 |
| Mobile (deviceTier low) | No fluid sim (freeze or disable), no bloom, DPR 1, static glass text |
| WebGL unavailable | CSS fallback: gradient background + static text + Vimeo iframes load normally |

### Scaling Priorities

1. **First bottleneck — fluid simulation:** Most GPU-expensive component. Reduce `fluidRes` before touching anything else. On low-end mobile, disable entirely and freeze the last frame.
2. **Second bottleneck — bloom passes:** Each UnrealBloomPass level adds a full render pass. Drop from 3 → 2 → 1 → 0 levels based on tier.
3. **Third bottleneck — DPR:** Rendering at native DPR 3.0 (many modern phones) quadruples pixel count vs DPR 1.5. Always cap.

## Anti-Patterns

### Anti-Pattern 1: Two Separate Scroll Listeners (WebGL and DOM)

**What people do:** Wire scroll events in SceneManager for WebGL updates AND separately in DOM components for reveal animations. Each listener reads `window.scrollY` independently.

**Why it's wrong:** Scroll events are not guaranteed to fire in sync with `requestAnimationFrame`. Reading scroll in two places produces subtle misalignment — the DOM and WebGL can appear to be at different scroll positions for one frame. At 60fps this is invisible; at 30fps or during fast flings it produces visible jitter.

**Do this instead:** One `scroll` listener writes to `AppState.scrollY`. The RAF tick reads from `AppState` for both WebGL and DOM updates. Single source of truth, single update point per frame.

### Anti-Pattern 2: Running the Full Pipeline Below the Fold

**What people do:** Keep `EffectComposer.render()` executing at full quality even when the hero is not visible (user has scrolled to the projects section).

**Why it's wrong:** Bloom post-processing is expensive. Running 3 bloom passes to render a canvas the user cannot see wastes 10-20ms per frame, causes thermal throttling on mobile, and burns battery.

**Do this instead:** Gate expensive effects on `AppState.scrollProgress`. Once the hero is fully scrolled out (`progress > 0.25`), skip the post-processing passes entirely or switch to a `renderer.render()` call with no composer.

### Anti-Pattern 3: Creating Vimeo Iframes on Page Load

**What people do:** Put 6-10 `<iframe src="https://player.vimeo.com/...">` tags directly in the HTML.

**Why it's wrong:** Each Vimeo iframe triggers its own network request (player JS, thumbnail, metadata). 8 iframes = 8 concurrent external fetches at startup, significantly delaying the page's interactive time and competing with WebGL asset loading.

**Do this instead:** Render placeholder `.project-card` divs with a `data-vimeo-id` attribute. Use `IntersectionObserver` to inject the iframe only when the card is about to enter the viewport (rootMargin: "200px 0px"). This defers Vimeo network cost until the user actually scrolls to the projects section.

### Anti-Pattern 4: Shader Compilation During the RAF Loop

**What people do:** Compile or create new `ShaderMaterial` instances inside the render loop (e.g., on first visibility, in response to a state change).

**Why it's wrong:** GLSL compilation is synchronous and blocks the main thread. Compiling even a simple shader mid-frame drops to <10fps and produces a visible freeze. This is especially bad on mobile where compilation is slower.

**Do this instead:** Compile all shaders during the loading screen phase. Pre-warm the shader cache by rendering objects off-screen with a 1px viewport during `LoadingScreen.init()`. By the time the user sees the hero, all shaders are compiled.

### Anti-Pattern 5: Storing Per-Frame Data in DOM Attributes

**What people do:** Write scroll position or animation state into `element.dataset.scroll` or CSS custom properties on every frame to communicate between systems.

**Why it's wrong:** DOM writes trigger style recalculations. Writing 10 custom properties per frame forces the browser to mark elements dirty. At 60fps this can add 2-5ms of layout work per frame.

**Do this instead:** Use `AppState` (a plain JS object) for inter-system communication. Only write to the DOM when state actually changes meaningfully (e.g., switching `activeSection` from 'hero' to 'projects' — not continuously every frame).

## Integration Points

### External Services

| Service | Integration Pattern | Notes |
|---------|---------------------|-------|
| Vimeo Player | Lazy iframe injection via IntersectionObserver | Use `?autoplay=0&title=0&byline=0&portrait=0` params for clean embed. Do not use Vimeo Player SDK unless you need programmatic play/pause — it adds 50KB JS. |
| Google Fonts | Single `<link>` preconnect + stylesheet in `<head>` | Load one weight only. Font only used for DOM content, not TextGeometry (that uses Typeface.js JSON). |
| Three.js CDN | Import map pointing to jsDelivr ESM build | Pin exact version (`r170` per project decision). Do not use `@latest` in production — Three.js has breaking changes between minors. |

### Internal Boundaries

| Boundary | Communication | Notes |
|----------|---------------|-------|
| ScrollController ↔ SceneManager | AppState (one-way: scroll → state → WebGL) | SceneManager never calls ScrollController directly |
| FluidSim ↔ HeroScene | Direct method call (`fluidSim.step()`, `fluidSim.velocityTexture`) | Both are WebGL subsystem — direct reference is fine |
| SectionController ↔ VideoManager | Custom event or direct callback on section enter | VideoManager listens for 'projects' section active, triggers load sweep |
| HeroScene ↔ PostProcessor | Render target texture (HeroScene renders to FBO, PostProcessor reads it) | Enables post-process without re-rendering the scene |
| LoadingScreen ↔ SceneManager | Promise / callback: LoadingScreen resolves, main.js starts RAF | LoadingScreen controls the start gate; SceneManager does not self-start |

## Build Order (Dependencies Between Components)

Build in this order — each layer depends only on components already complete:

```
1. AppState (state.js)              ← no dependencies
   └── All other modules import this

2. SceneManager (renderer, camera)  ← depends on: AppState
   └── Three.js renderer init, resize, basic RAF structure

3. DeviceTier detection             ← depends on: SceneManager (needs GL context)
   └── Writes deviceTier into AppState

4. FluidSimulation                  ← depends on: SceneManager (renderer)
   └── Ping-pong FBOs, shader passes

5. GlassText                        ← depends on: SceneManager (scene, loader)
   └── Font JSON load + TextGeometry + glass shader

6. TilesBackground                  ← depends on: SceneManager (scene)
   └── InstancedMesh + animated shader

7. HeroScene                        ← depends on: FluidSim, GlassText, Tiles
   └── Composes all hero 3D objects into one scene

8. PostProcessor (EffectComposer)   ← depends on: SceneManager, HeroScene
   └── Bloom + Vignette chain

9. LoadingScreen                    ← depends on: GlassText (font load), shaders (compile)
   └── Preloads assets, shows glitch reveal, resolves Promise to ungate RAF

10. ScrollController                ← depends on: AppState
    └── Scroll listener, velocity calculation

11. SectionController               ← depends on: AppState, DOM sections exist
    └── IntersectionObserver per section

12. VideoManager                    ← depends on: SectionController (projects section signal)
    └── Vimeo lazy loader

13. main.js (bootstrap)             ← depends on: everything above
    └── Init order, RAF loop, wires AppState updates per tick
```

## Sources

- Composite rendering pipeline pattern: [Codrops — Composite Rendering: The Brilliance Behind Inspiring WebGL Transitions](https://tympanus.net/codrops/2026/02/23/composite-rendering-the-brilliance-behind-inspiring-webgl-transitions/) — MEDIUM confidence (verified pattern, article Feb 2026)
- Scroll-DOM synchronization: [Codrops — Building a Scroll-Revealed WebGL Gallery](https://tympanus.net/codrops/2026/02/02/building-a-scroll-revealed-webgl-gallery-with-gsap-three-js-astro-and-barba-js/) — MEDIUM confidence (uses GSAP ScrollSmoother, extrapolated to vanilla pattern)
- Portal/FBO pattern: [Codrops — Letting the Creative Process Shape a WebGL Portfolio](https://tympanus.net/codrops/2025/11/27/letting-the-creative-process-shape-a-webgl-portfolio/) — MEDIUM confidence
- Performance optimization patterns: [Codrops — Building Efficient Three.js Scenes](https://tympanus.net/codrops/2025/02/11/building-efficient-three-js-scenes-optimize-performance-while-maintaining-quality/) — MEDIUM confidence
- Device tier detection: [javaspring.net — JavaScript GPU/CPU Performance Detection](https://www.javaspring.net/blog/using-javascript-to-detect-device-cpu-gpu-performance/) — MEDIUM confidence (multiple sources agree on `maxTextureSize` + `hardwareConcurrency` approach)
- Vimeo lazy loading: [robertmarshall.dev — Lazy Load Vimeo Video Iframe](https://robertmarshall.dev/blog/lazy-load-vimeo-video-iframe-show-on-scroll/) — HIGH confidence (well-established IntersectionObserver + iframe injection pattern)
- Three.js import maps (CDN): [Three.js forum — CDN ES6 module import requires importmap](https://discourse.threejs.org/t/cdn-es6-module-import-requires-importmap-vanilla-js/68353) — HIGH confidence (official forum, documented Three.js pattern)
- Fluid simulation GPU pattern: [Three.js forum — Fluid Simulation using Shaders Tutorial](https://discourse.threejs.org/t/fluid-simulation-using-shaders-tutorial/85109) — MEDIUM confidence

---
*Architecture research for: Immersive WebGL single-page portfolio, Three.js r170, vanilla JS, no bundler*
*Researched: 2026-03-22*
