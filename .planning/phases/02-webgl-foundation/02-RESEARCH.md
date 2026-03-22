# Phase 2: WebGL Foundation - Research

**Researched:** 2026-03-22
**Domain:** Three.js WebGL renderer setup, device tier detection, glitch loading screen, context recovery, shader precompilation
**Confidence:** HIGH (core stack verified against CDN source); MEDIUM (iOS context recovery nuances)

---

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions

**Glitch text reveal**
- Digital corruption style: each character cycles through random unicode/symbols before resolving to the correct letter
- Characters resolve left to right (decryption/cracking feel)
- Full reveal takes ~2.5s (medium pace)
- White text on pure black background
- Uses the project's display font (converted typeface from Phase 1)

**Scanline overlay**
- Animated sweep: a bright scanline band moves top-to-bottom repeatedly (~2s cycle)
- Band has noise/distortion — imperfect, analog feel, not a clean geometric line
- Subtle film grain texture on the black background adds life during loading
- Thin progress bar at bottom or top of screen in addition to the text reveal

**Loading-to-hero transition**
- Glitch dissolve: loading screen fragments and breaks apart, revealing the hero scene underneath
- Transition duration ~1s (medium — readable but doesn't linger)
- Loading text and hero 3D text are separate elements — 2D text dissolves away, 3D glass text is already present in the hero behind it
- Transition triggers when GPU assets are ready, with a 2s minimum display time (so the animation is always seen) and the existing 5s fallback maximum

**Device tier behavior**
- Low-end devices get simplified loading: skip film grain and scanline sweep, just the text reveal on black
- No visible tier indication to visitors — completely invisible to the user
- Tier detection and WebGL-unavailable fallback are at Claude's discretion

### Claude's Discretion
- Device tier detection strategy (GPU benchmark vs heuristics vs hybrid)
- WebGL-unavailable fallback approach (static HTML or CSS-only)
- Exact scramble glyph character set for the corruption effect
- Progress bar styling and position
- Film grain intensity and animation speed
- Scanline band width, opacity, and noise pattern
- RAF loop architecture and context recovery implementation

### Deferred Ideas (OUT OF SCOPE)
None — discussion stayed within phase scope
</user_constraints>

---

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|-----------------|
| TECH-04 | WebGL context loss is handled gracefully (re-initialization on `webglcontextrestored`) | Three.js r175 `onContextRestore` calls `initGLContext()` internally; app layer must re-upload render targets and re-attach scene objects after restore event |
| TECH-05 | All render targets are properly disposed on resize to prevent GPU memory leaks | `WebGLRenderTarget.setSize()` calls `dispose()` internally — no manual dispose needed on resize; `renderer.info.memory.textures` is the verification metric |
| TECH-06 | Shaders compile during loading screen, not on first visible frame | `renderer.compileAsync(scene, camera)` returns a Promise that resolves only when KHR_parallel_shader_compile finishes — use this to gate the loading-to-hero transition |
| LOAD-01 | Visitor sees a glitch-style loading screen with "Tom Bernys" revealed character by character while WebGL initializes | GSAP 3.14.2 ScrambleTextPlugin (now fully free, CDN-accessible) handles character scramble; Comfortaa 700 via Google Fonts already loaded in Phase 1 |
| LOAD-02 | Loading screen transitions smoothly to the hero scene once 3D assets are ready (or fallback after 5s) | `compileAsync()` Promise + 2s minimum display guard + 5s fallback `setTimeout`; transition: CSS opacity fade + JS-driven glitch dissolve fragment effect |
| LOAD-03 | Scanline overlay animates during loading and fades out on transition | Pure CSS `@keyframes` scanline sweep + film grain via CSS noise SVG data URI; fades out via opacity transition on hero reveal |
</phase_requirements>

---

## Summary

This phase bootstraps the Three.js WebGL renderer, implements GPU tier detection, and builds the glitch loading screen that all subsequent 3D work lives inside. The project constraint (single `index.html`, no build tools, CDN only) shapes every decision — all libraries must be importable via ES module CDN URLs.

Three.js r175 already handles `webglcontextlost`/`webglcontextrestored` at the renderer level by calling `initGLContext()` on restore, which resets the full GL state. Application code still needs to listen for the restore event to re-attach scene content and render targets. The iOS Safari context loss bug (backgrounding a tab) is a recurring issue across Safari versions (confirmed iOS 17 and M4 iPad reports in 2024-2025); the pattern is to listen for `webglcontextrestored` on the canvas and call the scene init function again.

Shader precompilation is solved by `renderer.compileAsync(scene, camera)` — it uses the `KHR_parallel_shader_compile` WebGL extension and returns a Promise, making it the correct async gate for the loading-to-hero transition. For the glitch text effect, GSAP ScrambleTextPlugin 3.14.2 is the right tool: it is now 100% free (Webflow acquisition, May 2025), available as an ESM module on jsDelivr CDN, and directly supports custom character sets for the corruption glyph style. Device tier detection uses `detect-gpu` (v5.0.70), which is also CDN-accessible as an ESM module.

**Primary recommendation:** Wire the loading screen around `renderer.compileAsync()` as the central readiness signal, use GSAP ScrambleTextPlugin for the character reveal (CDN ESM), detect-gpu for tier classification, and implement the scanline/film grain purely in CSS to avoid any additional JS payload.

---

## Standard Stack

### Core
| Library | Version | Purpose | Why Standard |
|---------|---------|---------|--------------|
| Three.js | r175 (0.175.0) | WebGL renderer, scene, camera | Already in project import map; chosen over r182/r183 to avoid rename breakage |
| GSAP | 3.14.2 | ScrambleTextPlugin for glitch text reveal; timeline orchestration | Now 100% free; ESM CDN available; ScrambleTextPlugin handles left-to-right character resolve and custom char sets natively |
| detect-gpu | 5.0.70 | GPU tier classification (0-3) without running a benchmark | ESM CDN-accessible; uses GPU renderer string + benchmark database; returns tier synchronously from UNMASKED_RENDERER_WEBGL |

### Supporting
| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| CSS `@keyframes` (no library) | — | Scanline sweep animation, film grain, progress bar | All scanline/grain effects are CSS-only — no JS or library needed |
| Page Visibility API (`document.visibilitychange`) | browser built-in | Pause RAF loop when tab is hidden; critical for iOS context recovery | Use instead of `pagehide`/`pageshow`; pairs with context restore handler |

### Alternatives Considered
| Instead of | Could Use | Tradeoff |
|------------|-----------|----------|
| GSAP ScrambleTextPlugin | Custom `setInterval` scramble loop | Custom code is ~40 lines and viable, but GSAP handles left-to-right timing, speed ramping, and custom char sets with zero bugs; free anyway |
| detect-gpu | `navigator.hardwareConcurrency` + `deviceMemory` heuristics | Heuristics are unreliable across GPU generations; detect-gpu uses actual GPU renderer string matched to benchmark database |
| CSS scanlines | Three.js post-processing pass | Post-processing adds a render target and a fullscreen quad; CSS costs nothing and the effect is identical for this purpose |
| `renderer.compileAsync()` | Render one hidden frame to warm shaders | compileAsync is the official API and non-blocking; hidden frame render still blocks the main thread briefly |

**Installation (CDN — no npm):**
All via import map additions to `index.html`. See Code Examples section for exact URLs.

---

## Architecture Patterns

### Recommended Structure (all in `index.html` `<script type="module">`)

```
<script type="module">
  ├── Phase 2 bootstrap (this phase)
  │   ├── detectTier()           — runs detect-gpu, returns 0-3
  │   ├── initRenderer()         — WebGLRenderer + resize + context events
  │   ├── initScene()            — scene + camera + placeholder objects for shader warmup
  │   ├── initLoadingScreen()    — DOM overlay creation
  │   ├── runGlitchReveal()      — GSAP ScrambleText timeline, left-to-right
  │   ├── runScanline()          — CSS class toggle (tier >= 1 only)
  │   ├── runFilmGrain()         — CSS class toggle (tier >= 1 only)
  │   ├── warmupShaders()        — renderer.compileAsync(scene, camera)
  │   └── transition()           — dissolve overlay out, start RAF loop
  └── RAF loop
      ├── visibilitychange guard
      └── renderer.render(scene, camera)
</script>
```

### Pattern 1: Loading Screen State Machine

**What:** A simple Promise-based gate with two conditions: (A) compileAsync resolves AND (B) 2s minimum elapsed. Whichever resolves last triggers the transition. A `setTimeout(transition, 5000)` serves as the maximum fallback.

**When to use:** Any time animation visibility must be guaranteed regardless of device speed.

```javascript
// Source: Three.js r175 source (cdn.jsdelivr.net/npm/three@0.175.0/build/three.module.js)
const minDisplay = new Promise(resolve => setTimeout(resolve, 2000));
const shadersReady = renderer.compileAsync(heroScene, camera);
const fallback = new Promise(resolve => setTimeout(resolve, 5000));

Promise.race([
  Promise.all([minDisplay, shadersReady]),
  fallback
]).then(() => transition());
```

### Pattern 2: WebGL Context Recovery

**What:** Three.js r175 internally handles `webglcontextlost` (calls `event.preventDefault()`, sets `_isContextLost = true`) and `webglcontextrestored` (calls `initGLContext()`, resets `_isContextLost = false`). The app layer only needs to re-bind application state that Three.js cannot know about.

**When to use:** Required for TECH-04 and the iOS backgrounding success criterion.

```javascript
// Three.js handles the GL-level restore. App code handles scene re-init.
canvas.addEventListener('webglcontextrestored', () => {
  // Three.js has already re-initialized the GL context at this point.
  // Re-create any WebGLRenderTarget instances (they're invalidated).
  // Re-add scene objects if they were removed during loss handling.
  // Restart the RAF loop if it was stopped.
  initScene();
  startRAFLoop();
});

canvas.addEventListener('webglcontextlost', () => {
  // Stop RAF loop to avoid errors while context is invalid.
  stopRAFLoop();
});
```

### Pattern 3: RAF Loop with Visibility Guard

**What:** Pause rendering when the tab is hidden to prevent wasted GPU cycles and to minimize the iOS context-loss window.

```javascript
// Source: MDN Page Visibility API
let rafId = null;

function startRAFLoop() {
  if (rafId) return;
  function loop() {
    rafId = requestAnimationFrame(loop);
    renderer.render(scene, camera);
  }
  rafId = requestAnimationFrame(loop);
}

function stopRAFLoop() {
  if (rafId) cancelAnimationFrame(rafId);
  rafId = null;
}

document.addEventListener('visibilitychange', () => {
  if (document.hidden) stopRAFLoop();
  else startRAFLoop();
});
```

### Pattern 4: Render Target Disposal on Resize

**What:** `WebGLRenderTarget.setSize()` internally calls `dispose()` before reallocating the framebuffer (verified in Three.js source). Do not call `dispose()` manually before `setSize()` — it is redundant and may cause double-dispose warnings.

```javascript
// Correct resize pattern — setSize handles dispose internally
window.addEventListener('resize', () => {
  const w = window.innerWidth, h = window.innerHeight;
  renderer.setSize(w, h);
  camera.aspect = w / h;
  camera.updateProjectionMatrix();
  // If you have render targets:
  myRenderTarget.setSize(w, h); // dispose() called internally
});
```

Verification: `renderer.info.memory.textures` must not increase after 5 resize cycles (success criterion TECH-05).

### Pattern 5: Scanline + Film Grain (CSS only)

**What:** The scanline sweep is a `::after` pseudo-element on the overlay `div` animated with `@keyframes`. Film grain is a CSS `url()` SVG data URI applied as a `background-image` with `animation: grain` shifting `background-position`.

```css
/* Scanline sweep — bright band moving top to bottom */
#loader::after {
  content: '';
  position: absolute;
  inset: 0;
  background: linear-gradient(
    to bottom,
    transparent 45%,
    rgba(255,255,255,0.04) 49%,
    rgba(255,255,255,0.10) 50%,
    rgba(255,255,255,0.04) 51%,
    transparent 55%
  );
  animation: scanline 2s linear infinite;
}

@keyframes scanline {
  from { transform: translateY(-100%); }
  to   { transform: translateY(200%); }
}
```

For film grain, use an inline SVG `<feTurbulence>` filter as a data URI on the overlay background — no external image needed, zero network request.

### Anti-Patterns to Avoid

- **Creating a new `WebGLRenderer` on context restore:** Three.js reinitializes the existing renderer in-place. Creating a new one leaks the old canvas and causes double-rendering.
- **Calling `renderer.compile()` (sync) instead of `compileAsync()`:** The sync version blocks the main thread during compilation, defeating the purpose of a loading screen on slow GPUs.
- **Adding `type="module"` script after body content:** Import maps must appear before any `<script type="module">` tags — the existing structure in `index.html` already correctly places the import map in `<head>`.
- **Using `window.devicePixelRatio` directly without clamping:** Phase 1 already clamps to 2. On resize, re-call `renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))` before `setSize()`.
- **Destroying the loading overlay before the opacity transition completes:** Set `pointer-events: none` immediately on transition start, remove from DOM only after `transitionend` event fires.

---

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Character scramble left-to-right with custom glyphs | `setInterval` loop with `Math.random()` char picking | GSAP ScrambleTextPlugin | Handles per-character timing, speed ramping, left-to-right stagger, custom char sets, cancellation — ~40 lines replaced by 2-line config |
| GPU tier classification | `navigator.hardwareConcurrency` branching | `detect-gpu` v5.0.70 | GPU renderer string matched against benchmark database covering thousands of devices; heuristics fail on same-CPU devices with different GPUs |
| Shader warmup frame | Invisible render pass with color mask | `renderer.compileAsync()` | Official Three.js API using `KHR_parallel_shader_compile` extension; non-blocking; correct Promise-based integration |
| CRT scanlines | Canvas 2D overlay drawing loop | CSS `@keyframes` pseudo-element | CSS GPU-composited, zero JS, same visual result |
| Film grain | JS noise function drawing to canvas | CSS SVG `feTurbulence` filter via data URI | No runtime JS, no extra canvas element, browser applies on GPU compositor |

**Key insight:** The loading screen is entirely DOM/CSS — it sits above the Three.js canvas. Three.js is only needed below the overlay. This separation means the scanline/grain effects never touch WebGL, removing an entire class of bugs.

---

## Common Pitfalls

### Pitfall 1: iOS Context Loss on Tab Background
**What goes wrong:** When a visitor backgrounds Safari on iPhone/iPad and returns, the WebGL context is lost. Three.js logs "Context Lost" and canvas renders black. This is confirmed broken across iOS 17 Developer Beta, iOS 18.7.2, and M4 iPad (2024-2025 reports).
**Why it happens:** iOS aggressively reclaims GPU memory from backgrounded tabs. Safari fires `webglcontextlost` without a guaranteed `webglcontextrestored`.
**How to avoid:**
1. Three.js r175 calls `event.preventDefault()` in its internal `webglcontextlost` handler — this tells the browser the app intends to restore. Do not add a second `webglcontextlost` listener that fails to call `preventDefault()`.
2. Add an app-level `webglcontextrestored` listener (see Pattern 2) to re-init scene objects after Three.js rebuilds the GL context.
3. Pair with `visibilitychange` to stop RAF while hidden — reducing the window during which context loss occurs.
4. The Page Visibility API (`document.visibilitychange`) fires before the context is lost in most cases, giving a clean stop-then-restore cycle.
**Warning signs:** Black canvas after returning from another app or tab on iOS.

### Pitfall 2: Shader Stutter on First Real Frame
**What goes wrong:** The hero scene appears but stutters or freezes for 50-200ms on first render because shaders compile on-demand.
**Why it happens:** WebGL compiles GLSL on first draw call for each unique material. Three.js does not pre-compile unless explicitly asked.
**How to avoid:** Call `renderer.compileAsync(heroScene, camera)` during the loading screen. Do NOT call `renderer.compile()` (sync) — it blocks the main thread.
**Warning signs:** Visible hitch immediately after the loading overlay fades.

### Pitfall 3: Render Target Texture Count Increases on Resize
**What goes wrong:** `renderer.info.memory.textures` increases by 1+ on every resize if render targets are not handled correctly.
**Why it happens:** If code calls `renderTarget.dispose()` manually then `renderTarget.setSize()`, the `setSize` call allocates a new framebuffer without decrementing the disposed count. Or, if new `WebGLRenderTarget` instances are created on each resize without disposing the old ones.
**How to avoid:** For render targets created once at init: call only `renderTarget.setSize(w, h)` on resize — `setSize` calls `dispose()` internally (verified in Three.js r175 source). Never call `dispose()` before `setSize()`. Never `new WebGLRenderTarget()` inside the resize handler.
**Warning signs:** `renderer.info.memory.textures` count grows monotonically across 5 resize events (the explicit TECH-05 success criterion).

### Pitfall 4: detect-gpu Fetches Benchmark JSON Asynchronously
**What goes wrong:** `getGPUTier()` is async (returns a Promise). If tier-dependent code runs synchronously before the Promise resolves, all devices get the default (no tier) behavior.
**Why it happens:** `detect-gpu` fetches a benchmark JSON file from UNPKG CDN to classify the GPU; this is a network request.
**How to avoid:** `await getGPUTier()` before initializing the renderer or starting the loading screen. The fetch is small and cached after first visit. If it fails (network error, old GPU not in DB), `detect-gpu` returns `tier: 1` as fallback — design the default rendering path for tier 1.
**Warning signs:** Film grain and scanline showing on low-end devices that should get simplified loading.

### Pitfall 5: GSAP ScrambleText Targeting DOM Text During Three.js Init
**What goes wrong:** The scramble animation updates DOM text content every ~16ms. If the loading overlay is removed from the DOM while the GSAP timeline is still running, GSAP throws errors and the transition breaks.
**Why it happens:** Race between GSAP timeline completion and the transition trigger.
**How to avoid:** Use `gsap.killTweensOf(element)` before removing the overlay, or structure the timeline so transition only fires after the scramble tween `onComplete` callback.

### Pitfall 6: Import Map Order with GSAP Plugins
**What goes wrong:** `import { ScrambleTextPlugin } from 'gsap/ScrambleTextPlugin'` fails if the import map entry for `gsap/` (with trailing slash) is not present.
**Why it happens:** GSAP plugins use sub-path imports relative to the `gsap` package root (`./utils/strings.js` etc.). Import maps must cover the `gsap/` prefix with a trailing-slash entry.
**How to avoid:** Add both a `"gsap"` entry and a `"gsap/"` entry to the import map. See Code Examples.

---

## Code Examples

Verified patterns from CDN source inspection and official docs:

### Import Map Addition for GSAP
```html
<!-- Source: cdn.jsdelivr.net/npm/gsap@3.14.2/ScrambleTextPlugin.js (verified accessible, HTTP 200) -->
<script type="importmap">
{
  "imports": {
    "three": "https://cdn.jsdelivr.net/npm/three@0.175.0/build/three.module.js",
    "three/addons/": "https://cdn.jsdelivr.net/npm/three@0.175.0/examples/jsm/",
    "gsap": "https://cdn.jsdelivr.net/npm/gsap@3.14.2/index.js",
    "gsap/": "https://cdn.jsdelivr.net/npm/gsap@3.14.2/"
  }
}
</script>
```

### GSAP ScrambleText — Left-to-Right Reveal
```javascript
// Source: gsap.com/docs/v3/Plugins/ScrambleTextPlugin/
import { gsap } from 'gsap';
import { ScrambleTextPlugin } from 'gsap/ScrambleTextPlugin';

gsap.registerPlugin(ScrambleTextPlugin);

// Characters resolve left to right: stagger each letter element individually
const letters = [...document.querySelectorAll('.loader-char')];
const tl = gsap.timeline();

letters.forEach((el, i) => {
  tl.to(el, {
    duration: 0.6,
    scrambleText: {
      text: el.dataset.char,          // target character
      chars: '!@#$%^&*<>{}[]|\\/?',  // corruption glyph set (Claude's discretion)
      revealDelay: 0.3,
      speed: 0.4,
    }
  }, i * 0.18); // stagger: each char starts 0.18s after previous
});
// Total timeline ~2.5s for "Tom Bernys" (10 chars × 0.18s + 0.6s reveal)
```

### detect-gpu Tier Detection
```javascript
// Source: cdn.jsdelivr.net/npm/detect-gpu@5.0.70/dist/detect-gpu.esm.js (verified ESM, HTTP 200)
import { getGPUTier } from 'https://cdn.jsdelivr.net/npm/detect-gpu@5.0.70/dist/detect-gpu.esm.js';

async function detectTier() {
  try {
    const { tier } = await getGPUTier();
    // tier 0 = no WebGL / blocklisted
    // tier 1 = ≥15 fps (low-end)
    // tier 2 = ≥30 fps (mid-range)
    // tier 3 = ≥60 fps (high-end)
    return tier;
  } catch {
    return 1; // safe default
  }
}
```

Note: detect-gpu fetches benchmark data from `https://unpkg.com/detect-gpu@5.0.70/dist/benchmarks` by default. This is a separate network request (JSON files, cached). For a no-external-dependency fallback, pass a `glContext` option to skip benchmark fetch and use renderer string heuristics only — acceptable since the project only needs low vs. not-low distinction.

### Shader Warmup Gate
```javascript
// Source: Three.js r175 source — compileAsync verified in three.module.js
const minDisplay  = new Promise(r => setTimeout(r, 2000));
const shadersReady = renderer.compileAsync(heroScene, camera);
const hardTimeout  = new Promise(r => setTimeout(r, 5000));

Promise.race([
  Promise.all([minDisplay, shadersReady]),
  hardTimeout
]).then(startTransition);
```

### Context Recovery
```javascript
// Source: Three.js r175 source — onContextRestore calls initGLContext() internally
// App-layer listener fires AFTER Three.js has rebuilt its GL state.
canvas.addEventListener('webglcontextlost', (e) => {
  stopRAFLoop();
  // Three.js already called e.preventDefault() in its own listener.
  // Do NOT add another call — it is already registered in WebGLRenderer constructor.
});

canvas.addEventListener('webglcontextrestored', () => {
  // Three.js has called initGLContext() — all internal GL resources rebuilt.
  // App must re-create any WebGLRenderTarget instances (they're invalidated).
  // Then restart the RAF loop.
  initScene();  // re-creates scene objects and render targets
  startRAFLoop();
});
```

**Important:** Three.js r175 registers its own `webglcontextlost` and `webglcontextrestored` listeners internally. Adding a duplicate `webglcontextlost` listener that does NOT call `preventDefault()` would cancel the restore attempt. Only add the app-level listener for `webglcontextrestored`.

### WebGL-Unavailable Fallback (Claude's Discretion: Recommended)
```javascript
// Before creating WebGLRenderer, test for WebGL support
function hasWebGL() {
  try {
    const canvas = document.createElement('canvas');
    return !!(canvas.getContext('webgl2') || canvas.getContext('webgl'));
  } catch { return false; }
}

if (!hasWebGL()) {
  // Show static fallback: hide canvas, show centered name in CSS
  document.getElementById('canvas').style.display = 'none';
  document.getElementById('hero').classList.add('no-webgl');
  // No loading screen — go straight to visible content
}
```

---

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| `renderer.compile(scene, camera)` (sync) | `renderer.compileAsync(scene, camera)` (Promise) | Three.js r158 | Non-blocking shader warmup; use compileAsync exclusively |
| GSAP ScrambleTextPlugin required Club GSAP paid membership | Fully free under standard license | May 2025 (Webflow acquisition) | Use freely in portfolio; no membership needed |
| `window.pageshow`/`pagehide` for tab backgrounding | `document.visibilitychange` + Page Visibility API | Established standard | More reliable cross-platform; pagehide unreliable in bfcache scenarios |
| gfxbench.com benchmark data in detect-gpu | Frozen gfxbench data (stopped Dec 2025) + alternative sources in progress | Dec 2025 | detect-gpu v5.0.70 benchmark data is still accurate for existing GPUs; no new GPUs added after Dec 2025 |

**Deprecated/outdated:**
- `renderer.compile()` (sync): Still works but blocks main thread — use `compileAsync()` instead.
- `WEBGL_lose_context.forceContextRestore()`: Not always available (extension may be unsupported, especially on iOS) — do not rely on it for recovery testing.

---

## Open Questions

1. **detect-gpu benchmark fetch and CSP**
   - What we know: `detect-gpu` fetches JSON from `unpkg.com` by default. This project has no CSP headers currently.
   - What's unclear: Whether Vercel adds any default CSP that would block the unpkg.com fetch.
   - Recommendation: Test in Vercel preview. If blocked, pass `glContext` to skip benchmark fetch and accept tier 1 heuristics only.

2. **GSAP ScrambleText + Comfortaa font rendering**
   - What we know: ScrambleTextPlugin targets DOM text elements. Comfortaa 700 is loaded via Google Fonts.
   - What's unclear: Whether scrambling individual `<span>` elements (one per character) with monospace-width slots in Comfortaa (proportional font) will cause layout shift during scramble.
   - Recommendation: Wrap each character in a fixed-width `<span>` with a min-width set to the widest glyph in the character set, or switch to JetBrains Mono (already loaded, monospace) for the loading screen — consistent with the terminal/decryption aesthetic.

3. **glitch dissolve transition implementation**
   - What we know: The dissolve should "fragment and break apart" the loading overlay — using the same visual language as the corruption effect.
   - What's unclear: The exact CSS/JS technique for fragmentation — options include CSS `clip-path` animation, canvas-drawn fragment tiles, or a Three.js shader pass on the loading screen.
   - Recommendation: CSS `clip-path` polygon animation driven by GSAP for ~4-6 rectangular fragments tearing away in staggered sequence — achieves the look without adding a second canvas or post-processing pass. This is at Claude's discretion.

4. **iOS `webglcontextrestored` reliability**
   - What we know: iOS aggressively loses context on backgrounding. The Three.js r175 `preventDefault()` call signals intent to restore. Safari 17.1+ has a partial fix but M4 and iOS 18.7.2 still show issues.
   - What's unclear: Whether `webglcontextrestored` always fires after `webglcontextlost` on modern iOS, or if a page reload is sometimes the only option.
   - Recommendation: Implement the graceful restore path (Pattern 2) as primary. Add a fallback: if `webglcontextrestored` has not fired within 3 seconds of `webglcontextlost`, reload the page (`window.location.reload()`). This covers the Safari edge case without a blank canvas.

---

## Sources

### Primary (HIGH confidence)
- Three.js r175 source (`cdn.jsdelivr.net/npm/three@0.175.0/build/three.module.js`) — `onContextLost`, `onContextRestore`, `initGLContext`, `compileAsync` implementation verified by direct inspection
- detect-gpu v5.0.70 source (`cdn.jsdelivr.net/npm/detect-gpu@5.0.70/dist/detect-gpu.esm.js`) — tier thresholds (0-3), async behavior, benchmark fetch URL verified by direct inspection
- GSAP ScrambleTextPlugin 3.14.2 (`cdn.jsdelivr.net/npm/gsap@3.14.2/ScrambleTextPlugin.js`) — ESM module, custom `chars` support verified by direct inspection; HTTP 200 confirmed
- Three.js forum: "Why does WebGLRenderTarget.setSize call this.dispose()?" — `setSize()` dispose-then-reallocate behavior confirmed
- MDN Page Visibility API — `visibilitychange` event behavior confirmed

### Secondary (MEDIUM confidence)
- Three.js discourse: "Context Lost when backgrounding Safari on iOS 17" — iOS context loss pattern and `webglcontextrestored` re-init recommendation
- CSS-Tricks / Webflow blog: "GSAP is Now Completely Free" — ScrambleTextPlugin free status confirmed (Webflow acquisition, May 2025)
- Three.js discourse: "Reducing shader compile time on scene initialization" — `compileAsync` use confirmed
- Three.js discourse: "Understanding relationship with texture.dispose() and renderer.info.textures" — texture count metric behavior

### Tertiary (LOW confidence — flag for validation)
- iOS 18.7.2 WebGL context loss reports (GitHub: google/model-viewer, mrdoob/three.js) — behavior specific to recent Safari versions, may be resolved in a future point release

---

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH — all libraries verified via CDN source inspection and HTTP status checks
- Architecture: HIGH — Three.js context handlers verified from source; compileAsync verified from source
- Pitfalls: MEDIUM — iOS context loss documented across multiple sources but behavior varies by Safari version (some issues may be resolved)
- GSAP license: HIGH — CSS-Tricks and Webflow blog confirm free status post-May 2025

**Research date:** 2026-03-22
**Valid until:** 2026-04-22 (30 days — stable libraries; detect-gpu benchmark data frozen Dec 2025 but still accurate for existing devices)
