# Stack Research

**Domain:** Creative WebGL portfolio site (single-page, no bundler, vanilla JS)
**Researched:** 2026-03-22
**Confidence:** HIGH (core decisions) / MEDIUM (fluid simulation specifics)

---

## Recommended Stack

### Core Technologies

| Technology | Version | Purpose | Why Recommended |
|------------|---------|---------|-----------------|
| Three.js | 0.175.0 (r175) | 3D rendering, scene graph, shaders | The only mature WebGL abstraction for this scope. r175 is the sweet spot: newer than the project's r170 spec, ships WebGL 1.0 removal cleanly, avoids r182/r183 PostProcessing→RenderPipeline rename that would break addons pattern. CDN-first design with import maps is officially supported. |
| Vanilla ES Modules | ES2022 | Application logic, scene orchestration | No framework overhead. Import maps let you use bare specifiers (`import from 'three'`) in a single HTML file without a bundler. All modern browsers support this natively. |
| GLSL (inline/ShaderMaterial) | WebGL 2 | Custom shaders — glass refraction, fluid, bloom | Three.js ShaderMaterial accepts raw GLSL. WebGL 2 is the baseline from r175 (WebGL 1 support removed). Write shaders as template literal strings in JS or embed as `<script type="x-shader">` tags. |

### Supporting Libraries

| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| GSAP | 3.14.2 | Scroll-linked animations, loading sequence, glitch text reveal | Use for the glitch loader text animation and scroll-based hero-to-content transitions. ScrollTrigger plugin (now free post-Webflow acquisition) handles pinning the WebGL canvas during hero phase and scrubbing effects on scroll. Load via CDN script tag — no import map needed. |
| @vimeo/player | 2.30.3 | Programmatic control of embedded Vimeo iframes | Use for lazy-loading players (only init when in viewport), play/pause on scroll, and disabling default Vimeo controls for custom UI. Load via jsDelivr CDN as ES module or classic script. |
| facetype.js (build-time tool) | web tool | Convert TTF/OTF fonts to Three.js typeface.json | Not a runtime library — use at design time to convert the chosen font (e.g. a Google Fonts TTF) to `.json` for TextGeometry. Run once, commit the JSON alongside index.html. |

### Development Tools

| Tool | Purpose | Notes |
|------|---------|-------|
| Live Server (VS Code) or `npx serve` | Local HTTP server | Import maps and ES modules require `http://` not `file://`. `npx serve .` requires no install. VS Code Live Server extension is zero-config. |
| Chrome DevTools WebGL Inspector | GPU draw call debugging | Built into Chrome. Use Spector.js extension for frame capture when debugging shader issues. |
| stats.js (optional) | FPS/ms overlay during dev | Load via CDN only in dev, remove before ship. Three.js examples use it: `https://cdn.jsdelivr.net/npm/stats.js@0.17.0/build/stats.min.js` |

---

## Installation

This project has no npm install step. All libraries load via CDN from the single `index.html`.

```html
<!-- Import map — defines bare specifiers for Three.js + addons -->
<script type="importmap">
{
  "imports": {
    "three": "https://cdn.jsdelivr.net/npm/three@0.175.0/build/three.module.js",
    "three/addons/": "https://cdn.jsdelivr.net/npm/three@0.175.0/examples/jsm/"
  }
}
</script>

<!-- GSAP + ScrollTrigger (classic script, loads before modules) -->
<script src="https://cdn.jsdelivr.net/npm/gsap@3.14.2/dist/gsap.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/gsap@3.14.2/dist/ScrollTrigger.min.js"></script>

<!-- Vimeo Player SDK -->
<script src="https://cdn.jsdelivr.net/npm/@vimeo/player@2.30.3/dist/player.min.js"></script>

<!-- Main app entry point — ES module -->
<script type="module" src="main.js"></script>
```

Then in `main.js`:

```javascript
import * as THREE from 'three';
import { FontLoader } from 'three/addons/loaders/FontLoader.js';
import { TextGeometry } from 'three/addons/geometries/TextGeometry.js';
import { EffectComposer } from 'three/addons/postprocessing/EffectComposer.js';
import { RenderPass } from 'three/addons/postprocessing/RenderPass.js';
import { UnrealBloomPass } from 'three/addons/postprocessing/UnrealBloomPass.js';
import { ShaderPass } from 'three/addons/postprocessing/ShaderPass.js';
```

---

## Key Subsystem Decisions

### 3D Text (Glass "Tom Bernys" name)

**Use:** `FontLoader` + `TextGeometry` from `three/addons/`.

Font must be in Three.js typeface.json format — not TTF directly. Steps:
1. Download the TTF for the chosen typeface (Google Fonts or licensed)
2. Convert at https://gero3.github.io/facetype.js/ or https://vextrude.com/font_converter
3. Save as `assets/fonts/yourfont.typeface.json` alongside index.html
4. Load with `new FontLoader().load('assets/fonts/...')`

Glass shader is a custom `ShaderMaterial` with transmission/refraction simulation in GLSL — Three.js's `MeshPhysicalMaterial` with `transmission: 1` is an alternative but requires a separate `WebGLRenderTarget` for the background texture, which is the right approach for real-time refraction without a custom shader.

### Post-Processing Pipeline

**Use:** Three.js built-in `EffectComposer` from `three/addons/postprocessing/` — NOT the npm `postprocessing` package.

The npm `postprocessing` library requires a bundler and its CDN path is not clean for import maps. The built-in Three.js addons path (`three/addons/postprocessing/`) works directly via the import map with no extra configuration.

Pipeline order:
```
RenderPass → UnrealBloomPass → ShaderPass (vignette/custom) → output
```

For selective bloom (glow only on specific objects like the glass text, not the background):
- Use the layer-mask technique: assign bloom objects to `layers.enable(1)`, render bloom pass with dark material swap, composite with second RenderPass
- This is the standard Three.js approach — documented in the official example `webgl_postprocessing_unreal_bloom_selective.html`

Mobile allege: reduce `UnrealBloomPass` resolution and passes. On mobile use `bloomPass.nMips = 3` (vs 5 on desktop).

### GPU Fluid Simulation

**Use:** Custom WebGL fragment shader pipeline — NOT a library.

The reference implementation (Pavel Dobryakov's WebGL-Fluid-Simulation) and PromptHQ both run the Navier-Stokes solver entirely in fragment shaders using ping-pong render targets. The architecture:

```
[velocity texture] → advection shader → divergence shader
→ pressure solver (jacobi iteration, N passes) → gradient subtract
→ [updated velocity texture] → splat on mouse input
→ [dye/color texture] → render to screen
```

This does NOT use Three.js scene graph — it runs on raw `WebGLRenderTarget` pairs managed directly. Integrate with Three.js by rendering the fluid result as a texture on a fullscreen `PlaneGeometry` or as a background `THREE.WebGLRenderTarget` passed to the scene.

Key implementation reference: https://github.com/PavelDoGreat/WebGL-Fluid-Simulation (MIT license, plain WebGL, extractable pattern)

Mobile allege: halve the fluid resolution (e.g. 512 → 256 simulation grid).

### Scroll Architecture

**Use:** GSAP ScrollTrigger for all scroll-driven behavior.

Pin the hero canvas with `ScrollTrigger.pin()` while user scrolls through a trigger distance, then unpin and transition to the project showcase. GSAP's `scrub: true` links WebGL uniform values (e.g. fluid dissipation, bloom strength) directly to scroll position for smooth transitions.

Do NOT use CSS `position: sticky` for the WebGL canvas — it creates compositing layer issues on mobile Safari.

### Vimeo Embeds

**Use:** `@vimeo/player` JS SDK for programmatic control.

Embed strategy: render iframe elements in HTML with `data-vimeo-id` attributes but do NOT set `src` initially. Use `IntersectionObserver` to detect when a project card enters the viewport, then initialize `new Vimeo.Player(element, { id: videoId, autopause: true })` at that point. This avoids loading 6-10 Vimeo iframes simultaneously on page load.

Key embed options:
- `background: 0` — show controls (set `1` for silent autoplay loops if used in hero)
- `dnt: 1` — do not track (GDPR-friendly)
- `responsive: 1` — iframe scales to container
- `autopause: 1` — pause when out of viewport (default behavior)

---

## Alternatives Considered

| Recommended | Alternative | When to Use Alternative |
|-------------|-------------|-------------------------|
| Three.js 0.175.0 | Three.js 0.170.0 (project spec) | Only if you have an existing codebase already on r170 with known-working shader code. For greenfield, r175 is safer: cleaner WebGL 2 baseline, no hidden deprecation warnings. |
| Three.js 0.175.0 | Three.js 0.183.2 (latest) | Avoid for now: r182/r183 renamed PostProcessing to RenderPipeline, which means forum examples and the PromptHQ codebase patterns use the old API. r175 is the last stable release before that churn. |
| Three.js EffectComposer | npm `postprocessing` package | If you had a bundler (Vite/webpack). The pmndrs `postprocessing` library has better performance (single geometry pass) but requires npm. Not usable in a no-bundler setup via clean CDN import. |
| GSAP ScrollTrigger | Native Scroll-driven Animations (CSS) | CSS scroll-driven animations are browser-native but don't bridge to JS/WebGL uniform values. GSAP ScrollTrigger's `onUpdate` callback is the correct bridge between scroll position and Three.js uniforms. |
| Custom fluid shaders | A-Frame or Babylon.js fluid | No mature CDN-ready fluid plugin exists for either. Custom shader is the only viable no-bundler approach matching PromptHQ quality. |
| Vimeo Player SDK | YouTube IFrame API | Vimeo is already chosen for video hosting. YouTube API has similar capability but different embed URLs and requires a different SDK. Only switch if migrating video hosting. |
| typeface.json (FontLoader) | troika-three-text | `troika-three-text` renders SDF fonts at any size without pre-conversion and has excellent quality, but its CDN path is not clean for import maps (requires bundler for tree-shaking). For a small number of text objects, FontLoader + typeface.json is simpler and reliable. |

---

## What NOT to Use

| Avoid | Why | Use Instead |
|-------|-----|-------------|
| React / Vue / Svelte | Framework overhead adds a bundler requirement, breaks the single-HTML-file constraint, and adds no value for a site with no reactive state | Vanilla ES modules with import maps |
| Vite / webpack / Rollup | The entire point of this architecture is zero build step. Bundlers add CI complexity, break CDN caching strategy, and are unnecessary here | `<script type="importmap">` in index.html |
| `postprocessing` npm package (pmndrs) | Designed for bundler environments. CDN UMD build is available but not reliably tree-shaken; the documented import path (`postprocessing`) doesn't resolve in an import map without manual remapping | Three.js built-in `three/addons/postprocessing/` |
| Three.js r183 (latest) | PostProcessing renamed to RenderPipeline in r182 — breaks the `EffectComposer` pattern shown in all PromptHQ-style examples and Three.js docs | Three.js 0.175.0 until the ecosystem catches up |
| `MeshStandardMaterial` for glass text | No real-time refraction. Will look like a flat shiny surface, not glass | `MeshPhysicalMaterial` with `transmission`, `ior`, `thickness` + a background render target, or custom ShaderMaterial with environment map sampling |
| CSS animations for the loading sequence | Can't synchronize precisely with WebGL canvas readiness events | GSAP timeline triggered from Three.js `renderer.compile()` callback |
| `window.devicePixelRatio` unclamped | Retina phones at DPR 3-4 will 9-16x the GPU workload. Must clamp | `Math.min(window.devicePixelRatio, 1.5)` on mobile via UA sniff or media query |
| Inline `<script>` for Three.js module code | ES module `import` statements inside inline scripts don't have access to the document's import map in all browsers (Safari 15 edge case) | Always use `<script type="module" src="main.js">` with an external file |

---

## Stack Patterns by Variant

**Desktop (DPR > 1.5, no touch):**
- Pixel ratio: `Math.min(window.devicePixelRatio, 2.0)`
- Fluid sim resolution: 1024 grid
- Bloom: `UnrealBloomPass` with `nMips: 5`, strength 0.8, radius 0.3
- Shadow map: enabled on key lights

**Mobile (touch, DPR > 1.5):**
- Pixel ratio: `Math.min(window.devicePixelRatio, 1.5)` — hard cap
- Fluid sim resolution: 256 grid, reduced jacobi iterations (20 → 10)
- Bloom: `UnrealBloomPass` with `nMips: 3`, lower strength
- Instanced tile count: reduce by 50%
- Detection: `('ontouchstart' in window) || navigator.maxTouchPoints > 0`

**Low-end mobile (< 2GB RAM proxy: canvas size test):**
- Disable fluid simulation entirely, show static gradient background
- Detection: `navigator.deviceMemory < 2` (Chrome only) or `renderer.getContext().MAX_TEXTURE_SIZE < 4096`

---

## Version Compatibility

| Package | Compatible With | Notes |
|---------|-----------------|-------|
| three@0.175.0 | EffectComposer, UnrealBloomPass, RenderPass (all from same version) | CRITICAL: all postprocessing addons must come from the same CDN version as three.js core. Mixing versions causes duplicate THREE internals and broken compositing. |
| three@0.175.0 | FontLoader, TextGeometry (from same version addons path) | Same rule — always pin everything to `0.175.0`. |
| gsap@3.14.2 | Three.js (any version) | GSAP has no Three.js dependency — they communicate only via callbacks and uniform mutation. No compatibility concern. |
| @vimeo/player@2.30.3 | Three.js (any version) | Isolated iframe communication via postMessage. No Three.js coupling. |

---

## Sources

- https://threejs.org/manual/en/installation.html — Official CDN import map pattern (HIGH confidence)
- https://www.jsdelivr.com/package/npm/three — Version 0.183.2 latest, 0.175.0 confirmed (HIGH confidence)
- https://github.com/mrdoob/three.js/wiki/Migration-Guide — r170→r183 breaking changes (HIGH confidence)
- https://threejs.org/docs/#api/en/geometries/TextGeometry — FontLoader/TextGeometry import paths (HIGH confidence)
- https://www.jsdelivr.com/package/npm/@vimeo/player — @vimeo/player 2.30.3 (March 2026) (HIGH confidence)
- https://gsap.com/docs/v3/Plugins/ScrollTrigger/ — GSAP 3.14.2, free ScrollTrigger (HIGH confidence)
- https://github.com/PavelDoGreat/WebGL-Fluid-Simulation — Fluid sim reference implementation, MIT license (MEDIUM confidence — verified repo exists and is MIT, shader architecture confirmed)
- https://gero3.github.io/facetype.js/ — Font conversion tool for typeface.json (MEDIUM confidence — tool exists, actively referenced in Three.js forum)
- https://threejs.org/examples/webgl_postprocessing_unreal_bloom.html — Official bloom pipeline example (HIGH confidence)
- https://webglfundamentals.org/webgl/lessons/webgl-resizing-the-canvas.html — Canvas resize / DPR best practices (HIGH confidence)

---

*Stack research for: Tom Bernys Portfolio — WebGL single-page site*
*Researched: 2026-03-22*
