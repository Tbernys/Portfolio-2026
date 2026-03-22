# Pitfalls Research

**Domain:** WebGL single-page portfolio / creative portfolio site
**Researched:** 2026-03-22
**Confidence:** HIGH (core WebGL/Three.js pitfalls verified across official docs, Three.js forum, MDN, and multiple community sources)

---

## Critical Pitfalls

### Pitfall 1: WebGL Context Loss Without Recovery Handler

**What goes wrong:**
The browser silently kills the WebGL context — this happens on iOS when the tab goes to background, when the device runs low on memory, or when too many WebGL contexts exist across tabs. Three.js does not automatically recover. The canvas goes black and stays black. Users see a broken site and have no recourse except a full page reload.

**Why it happens:**
Developers test only on desktop under ideal conditions. Mobile Safari enforces aggressive memory limits (especially on older iPhones with unified CPU/GPU memory). The context loss event fires but nothing handles it, so the render loop stops forever.

**How to avoid:**
Listen for `webglcontextlost` and `webglcontextrestored` on the canvas element. In the `contextlost` handler, call `event.preventDefault()` — this signals to the browser that you intend to recover. In `contextrestored`, re-initialize the renderer, re-upload all textures, geometries, and render targets, and restart the animation loop. Structure initialization code as a function you can call twice: once on load, once on restore.

```js
canvas.addEventListener('webglcontextlost', (e) => {
  e.preventDefault();
  cancelAnimationFrame(rafId);
}, false);

canvas.addEventListener('webglcontextrestored', () => {
  initRenderer(); // re-run full setup
  startLoop();
}, false);
```

**Warning signs:**
- Canvas goes black on iOS after backgrounding the app
- No error in console on mobile (context loss is silent by default)
- Works fine on desktop Chrome but fails on real iOS devices

**Phase to address:** WebGL hero setup phase — build recovery into the renderer initialization from day one, not as a retrofit.

---

### Pitfall 2: GPU Fluid Sim Requires Float Texture Extensions Not Available on All iOS

**What goes wrong:**
The GPU Navier-Stokes fluid simulation requires `OES_texture_float` and `OES_texture_float_linear` WebGL extensions to ping-pong float framebuffers. On iOS 13 and earlier, WebKit falsely reported supporting `OES_texture_float_linear` but did not implement it. On iOS 14+, the extension is correctly reported as absent. The simulation silently renders garbage or throws shader errors on a significant percentage of real mobile devices.

**Why it happens:**
Desktop development never triggers this code path. Float texture extensions are available on virtually all desktop GPUs but are absent or broken on a large fraction of mobile GPUs (especially iOS and older Android).

**How to avoid:**
Always check extension availability before initializing the fluid simulation. Provide a graceful fallback: if float textures are unavailable, fall back to `RGBA8` encoding (pack 16-bit fixed-point values into two 8-bit channels) or simply disable the fluid simulation and show a static animated background instead. Do not let a missing extension crash the page.

```js
const ext = renderer.getContext().getExtension('OES_texture_float_linear');
if (!ext) {
  // Fall back: use NearestFilter instead of LinearFilter, or disable fluid
  fluidEnabled = false;
}
```

**Warning signs:**
- Fluid works on desktop but produces a black or artifact-filled canvas on iOS
- Console error: "No OES_texture_float support for float textures"
- GPGPU examples from Three.js repo fail on Safari iPadOS

**Phase to address:** Fluid simulation implementation phase — build the capability check and fallback path before writing the full simulation, not after.

---

### Pitfall 3: Render Targets Not Disposed — GPU Memory Leak

**What goes wrong:**
Every `WebGLRenderTarget` allocates GPU VRAM. The fluid simulation uses ping-pong double-buffered render targets. Post-processing uses an `EffectComposer` with multiple passes (bloom, vignette), each backed by render targets. If any of these are created dynamically, resized on window resize without disposing the old target first, or recreated without cleanup, GPU memory accumulates. On mobile, where VRAM may be as low as 512MB shared with system RAM, the tab crashes or the context is killed.

**Why it happens:**
Three.js does not garbage-collect GPU resources automatically. JavaScript GC frees the JS object wrapper but the GPU allocation persists until `.dispose()` is called explicitly. Developers resize the renderer on `window.resize` but forget to dispose and recreate render targets at the new resolution.

**How to avoid:**
On every window resize, dispose all render targets before creating new ones:

```js
function onResize() {
  const w = window.innerWidth, h = window.innerHeight;
  renderer.setSize(w, h);
  composer.setSize(w, h); // EffectComposer handles its own passes
  // For custom render targets:
  myRenderTarget.dispose();
  myRenderTarget = new THREE.WebGLRenderTarget(w, h);
  // Same for fluid sim ping-pong buffers
  fluidSimA.dispose();
  fluidSimB.dispose();
  [fluidSimA, fluidSimB] = createFluidTargets(w, h);
}
```

Monitor with `renderer.info.memory` (textures, geometries) — if counts grow over time, there is a leak.

**Warning signs:**
- `renderer.info.memory.textures` count increases over time
- Page crashes after a few minutes on mobile
- Chrome DevTools Memory tab shows growing GPU memory

**Phase to address:** WebGL setup and post-processing phase — establish the resize handler correctly before adding more render targets.

---

### Pitfall 4: UnrealBloomPass Kills Mobile Performance

**What goes wrong:**
`UnrealBloomPass` creates a mip-map chain of bloom textures and runs multiple blur passes — typically 5 render-to-texture operations per frame. On a desktop GPU this is cheap. On a mobile GPU it can consume 90% of GPU budget alone, leaving nothing for the scene geometry, fluid simulation, and instanced cylinder background. Result: 10-15fps on mobile instead of the target 30fps+.

**Why it happens:**
Developers build on desktop where bloom is imperceptible in cost. The performance budget on mobile is 10-20x tighter. A post-processing pipeline that costs 2ms on desktop costs 20-40ms on a mid-range phone.

**How to avoid:**
On mobile (detect via `navigator.maxTouchPoints > 0` or a pixel-ratio/GPU heuristic), reduce bloom to 2 mip levels instead of 5, halve the render target resolution, or replace UnrealBloomPass with a single-pass Gaussian blur. Better: disable bloom on mobile entirely and compensate with emissive material values. The `EffectComposer` supports conditional pass enablement:

```js
unrealBloomPass.enabled = !isMobile;
simpleMobileBloom.enabled = isMobile;
```

**Warning signs:**
- 60fps on desktop Chrome, 12fps on a real mid-range Android
- GPU frame time in Chrome DevTools spikes on bloom pass
- Battery drains visibly on iOS devices

**Phase to address:** Post-processing pipeline phase — implement mobile detection and tiered quality levels from the start, not after performance complaints.

---

### Pitfall 5: Vimeo iframes Block the Initial Page Load

**What goes wrong:**
Six to ten Vimeo iframes all initialized on page load make 60-100 network requests to `player.vimeo.com`, load Vimeo's player JS bundle, fetch video thumbnails, and run tracking scripts — all before the user has scrolled to see any of them. This blocks Time to Interactive, inflates Largest Contentful Paint, and competes with Three.js initialization for network bandwidth and main-thread time. The WebGL hero appears janky because JS is blocked.

**Why it happens:**
Developers paste the Vimeo embed code and move on. The default embed code has no lazy loading. The `loading="lazy"` attribute on iframes is a browser hint only — it does not prevent Vimeo's scripts from loading.

**How to avoid:**
Use the `lite-vimeo-embed` custom element or a manual Intersection Observer approach: render placeholder `<div>` elements with thumbnail images until the user scrolls them into view, then swap in the actual `<iframe>`. This defers 100% of Vimeo's network cost until the user reaches the project section.

```js
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      const div = entry.target;
      const iframe = document.createElement('iframe');
      iframe.src = div.dataset.vimeoSrc;
      div.replaceWith(iframe);
      observer.unobserve(div);
    }
  });
}, { rootMargin: '200px' });

document.querySelectorAll('.vimeo-placeholder').forEach(el => observer.observe(el));
```

**Warning signs:**
- Chrome Lighthouse shows large unused JS from `player.vimeo.com`
- Network waterfall shows 30+ requests firing immediately on page load
- WebGL initialization is delayed by 3-5 seconds on slow connections

**Phase to address:** Project showcase / video embed phase — implement lazy loading as the primary embed strategy, never as an optimization pass.

---

### Pitfall 6: Scroll Jank From Synchronous Scroll Event Handlers

**What goes wrong:**
Reading `window.scrollY` in a synchronous scroll event handler and immediately updating Three.js uniforms or camera position forces the browser to defer the scroll composite step, causing dropped frames and visible stutter as the user scrolls from the hero into the project grid. This is especially bad on mobile where the compositor thread is already under load.

**Why it happens:**
The intuitive pattern — `window.addEventListener('scroll', updateScene)` — is synchronous and fires hundreds of times per second on fast scroll. Any work inside the handler that touches layout or triggers a WebGL state change causes jank.

**How to avoid:**
Store the scroll position in the event handler (cheap), read it in the animation loop (correct):

```js
let scrollY = 0;
window.addEventListener('scroll', () => { scrollY = window.scrollY; }, { passive: true });

function animate() {
  requestAnimationFrame(animate);
  // Use scrollY here, not in the scroll handler
  updateSceneFromScroll(scrollY);
  renderer.render(scene, camera);
}
```

Mark all scroll, touch, and wheel event listeners as `{ passive: true }` — this tells the browser the handler will never call `preventDefault()`, allowing it to start scrolling immediately.

**Warning signs:**
- Chrome DevTools shows "Forced reflow" warnings during scroll
- Scroll handler appears in the "Long Tasks" flame chart
- iOS "rubber band" scroll feels sticky or laggy

**Phase to address:** Scroll integration phase — passive listeners and rAF-deferred reads must be the pattern from the first line of scroll code.

---

## Moderate Pitfalls

### Pitfall 7: TextGeometry Font Load Race Condition

**What goes wrong:**
`TextGeometry` requires a JSON font file loaded asynchronously via `FontLoader`. If the render loop starts before the font is loaded, the scene either throws an error or renders no text — resulting in a blank hero. Worse, if the font CDN is slow or blocked, the "Tom Bernys" hero text never appears at all, with no fallback.

**How to avoid:**
Gate scene initialization on font load completion. Use a `Promise`-based wrapper around `FontLoader.loadAsync()`. While the font is loading, show the glitch loader screen (which has no dependency on the font). Bundle the font JSON as a base64 data URI inline in the HTML to eliminate the network dependency entirely — this is the most robust approach for a single-file portfolio site. The font JSON for a subset of characters (Latin only) is typically 30-60KB.

**Phase to address:** WebGL hero setup phase.

---

### Pitfall 8: Device Pixel Ratio Set to `window.devicePixelRatio` Uncapped

**What goes wrong:**
Modern iPhones and high-end Androids have DPR of 3. Setting `renderer.setPixelRatio(window.devicePixelRatio)` on these devices means every pixel in CSS is rendered as 9 pixels in WebGL (3×3). The fluid simulation render targets, the post-processing passes, and the main scene framebuffer all scale to 3× resolution — producing 9× the GPU fragment workload. Frame rate collapses.

**How to avoid:**
Cap DPR at 1.5 for mobile, 2 for desktop:

```js
const dpr = Math.min(window.devicePixelRatio, isMobile ? 1.5 : 2);
renderer.setPixelRatio(dpr);
```

The visual difference between DPR 1.5 and 3 is imperceptible on a small phone screen. The performance difference is 4×.

**Phase to address:** WebGL renderer initialization — set this in the very first renderer setup, never allow uncapped DPR.

---

### Pitfall 9: Three.js r170 Via CDN Import Map — MIME Type Errors on Some Hosts

**What goes wrong:**
Import maps require the browser to resolve bare specifiers (`import * from 'three'`) against the map. If the static host (GitHub Pages, some Netlify configs) serves `.js` files with an incorrect MIME type, or if the CDN URL is unavailable, the entire page fails to load with an opaque network error. There is no fallback.

**How to avoid:**
Test the import map against the actual deployment host before any other work. Pin CDN URLs to specific versions (already done: r170) so upstream updates cannot break the site. Verify MIME types are `application/javascript` for all CDN-served modules. Keep a local copy of Three.js as a fallback in the repository for emergency use.

**Warning signs:**
- Console error: "Failed to resolve module specifier" or "MIME type mismatch"
- Works on `localhost` but breaks on GitHub Pages or Netlify
- No Three.js error — page simply shows nothing

**Phase to address:** Initial project setup / scaffolding phase.

---

### Pitfall 10: Post-Processing Breaks MSAA Antialiasing

**What goes wrong:**
`WebGLRenderer` with `antialias: true` uses hardware MSAA on the default framebuffer. As soon as you introduce `EffectComposer`, rendering moves to `WebGLRenderTarget` buffers, which do not support MSAA in WebGL. The bloom, vignette, and any other passes will look aliased — jagged edges on the 3D glass text and cylinder geometry.

**Why it happens:**
This is a documented Three.js behavior. The renderer's built-in antialiasing only applies to the final canvas output, not to render targets. Developers enable `antialias: true` and assume it covers the entire pipeline.

**How to avoid:**
Use SMAA (Subpixel Morphological Anti-Aliasing) or FXAA as a post-processing pass at the end of the `EffectComposer` chain. This is a shader-based approach that works on render targets. The `SMAAPass` from Three.js examples is the recommended choice — better quality than FXAA with reasonable cost.

**Phase to address:** Post-processing pipeline phase — add SMAA when setting up EffectComposer, before any visual polish work.

---

### Pitfall 11: No Reduced-Motion / Prefers-Reduced-Motion Support

**What goes wrong:**
Users with vestibular disorders or motion sensitivity who have set `prefers-reduced-motion: reduce` in their OS will see an aggressively animated WebGL scene with fluid simulation, bloom, and scroll-based camera movement — potentially causing nausea or headaches. This is both a UX failure and an accessibility compliance issue. Google's 2025 algorithm updates increasingly weight accessibility signals.

**How to avoid:**
Check `window.matchMedia('(prefers-reduced-motion: reduce)').matches` at startup. If true: pause the fluid simulation, disable camera shake/parallax, reduce bloom intensity to zero, and leave only static geometry visible. The page must still be fully usable without motion.

**Phase to address:** Accessibility pass — implement alongside mobile detection, before any motion-heavy effects are added.

---

## Minor Pitfalls

### Pitfall 12: SEO — Hero Content Is Canvas, Not HTML Text

**What goes wrong:**
Googlebot cannot read content rendered into a WebGL canvas. If "Tom Bernys" and all navigational text exists only as `TextGeometry` on the canvas, search engines index an empty page. The portfolio is invisible to Google search.

**How to avoid:**
Maintain semantic HTML in the DOM alongside the WebGL canvas. The hero `<h1>Tom Bernys</h1>` should exist as a visually-hidden (not `display:none`) HTML element. Use CSS to visually hide it (position it under the canvas with `aria-hidden` on the canvas and visible text for screen readers/crawlers). The HTML text is invisible to sighted users but real to crawlers and assistive tech.

**Phase to address:** HTML structure phase — establish the semantic skeleton before building the WebGL layer on top of it.

---

### Pitfall 13: Vimeo Autoplay Blocked on Mobile

**What goes wrong:**
Any Vimeo embed configured with `autoplay=1` will be silently blocked by iOS Safari and Android Chrome unless the video is also `muted=1`. If project showcase videos are set to autoplay with sound to create a richer demo experience, they will not play at all on mobile — leaving placeholder thumbnails visible instead of the video.

**How to avoid:**
Use `autoplay=1&muted=1` for any autoplay Vimeo embeds. For project showcase videos that should be user-initiated (not autoplay), ensure the click/tap target is large and clearly afforded. Do not rely on autoplay for critical content.

**Phase to address:** Video embed phase.

---

### Pitfall 14: Instanced Cylinder Background — Per-Instance Attribute Buffer Resize

**What goes wrong:**
If the number of instanced cylinders is computed based on viewport size and the `InstancedMesh` is recreated on every window resize, the old mesh's geometry and material are not disposed. Each resize creates a new GPU allocation. On mobile, users frequently trigger resize events by rotating the device or showing/hiding the browser chrome, causing repeated leak events.

**How to avoid:**
Size the `InstancedMesh` to the maximum needed instance count at init time. On resize, update instance matrices and visibility flags rather than recreating the mesh. If recreation is unavoidable, always call `mesh.geometry.dispose()` and `mesh.material.dispose()` before removing the old mesh from the scene.

**Phase to address:** Instanced background implementation phase.

---

### Pitfall 15: Loading Screen Timing — WebGL Init Blocks Glitch Animation

**What goes wrong:**
The glitch loader with character-by-character text reveal is intended to mask WebGL initialization time. If Three.js renderer initialization, shader compilation, and font loading happen on the main thread synchronously before the DOM is ready, the CSS animation for the loader never plays — users see a white flash then a fully loaded scene with no transition.

**How to avoid:**
Start the CSS glitch animation immediately in `DOMContentLoaded`. Defer all Three.js initialization behind a `setTimeout(init, 0)` or `requestIdleCallback` to yield the main thread to the loader animation first. Measure shader compilation time in development with `renderer.debug.checkShaderErrors = true` and with browser Performance profiling — it can be 500ms-2s on first load.

**Phase to address:** Loading screen / entry experience phase.

---

## Technical Debt Patterns

| Shortcut | Immediate Benefit | Long-term Cost | When Acceptable |
|----------|-------------------|----------------|-----------------|
| Hardcode DPR at 1 | Fast dev iteration | Blurry on retina screens; bad impression for a visual portfolio | Never — set capped DPR from day one |
| Skip dispose() on resize | Less code | GPU memory leak, tab crash on mobile after 5+ minutes | Never |
| Paste Vimeo iframes directly | Quick embed | Blocks page load, hurts WebGL init timing | Never for 6+ embeds |
| `antialias: true` without SMAA pass | Looks fine in isolation | Aliasing appears as soon as EffectComposer is added | Only before post-processing is introduced |
| No context loss handler | Works everywhere in dev | Silent breakage on iOS backgrounding | Never for a shipped portfolio |
| Uncapped `window.devicePixelRatio` | Accurate on all screens | Crashes performance on iPhone 13/14/15 | Never |

---

## Integration Gotchas

| Integration | Common Mistake | Correct Approach |
|-------------|----------------|------------------|
| Vimeo embed | Paste default iframe code | Use Intersection Observer lazy loading; defer until viewport |
| Vimeo autoplay | `autoplay=1` without `muted=1` | Always pair `autoplay=1&muted=1`; test on real iOS Safari |
| Google Fonts CDN | Load in `<head>` synchronously | Use `rel="preconnect"` + `display=swap`; font load must not block WebGL init |
| Three.js CDN import map | Assume CDN always up | Pin to specific version; test on deployment host for MIME type errors |
| `OES_texture_float` extension | Assume desktop support = mobile support | Check `getExtension()` result before building fluid sim; provide fallback |

---

## Performance Traps

| Trap | Symptoms | Prevention | When It Breaks |
|------|----------|------------|----------------|
| Bloom at full resolution on mobile | <15fps on mid-range phones | Reduce to 2 mip levels; halve pass resolution on mobile | Any real mobile device with DPR 2+ |
| Fluid sim at full resolution on mobile | 100% GPU usage, thermal throttling | Run physics at 128×128, visuals decoupled; or disable on mobile | Any phone with <3GB RAM |
| Uncapped DPR (3× on iPhone) | 9× fragment cost, frame drops | Cap at 1.5 mobile / 2 desktop | iPhone 12 and newer (all DPR 3) |
| Creating new objects in rAF loop | JS GC pauses every few seconds causing hitches | Pre-allocate Vector3/Color instances; reuse with `.set()` | Immediately visible as frame spikes |
| Resize without dispose | GPU memory grows indefinitely | Dispose → create pattern for all render targets | After 3-5 device rotations on mobile |
| All Vimeo iframes on DOMContentLoaded | Slow initial load, WebGL delayed | Intersection Observer lazy load | Always, from first page visit |

---

## UX Pitfalls

| Pitfall | User Impact | Better Approach |
|---------|-------------|-----------------|
| No fallback when WebGL unsupported | Blank page for users on old Android browsers or WebGL-disabled setups | Detect with `WebGLRenderingContext` check; show static image fallback |
| Fluid sim consuming touch input | User cannot scroll; interactions feel broken on mobile | Ensure fluid mouse/touch handlers do not call `preventDefault()` on scroll-axis touch events |
| Video autoplay competing with ambient WebGL audio (if any) | Unexpected sound clash | No audio in WebGL hero; Vimeo embeds muted by default |
| No skip/reduce-motion option | Motion-sensitive users suffer through full animation | Implement `prefers-reduced-motion` check; provide visible "reduce motion" toggle |
| Glitch loader text reveal too slow on fast connections | Users who load quickly still wait for the animation | Cap loader to 2-3 seconds maximum regardless of connection speed |

---

## "Looks Done But Isn't" Checklist

- [ ] **Context loss recovery:** Canvas goes black on iOS after backgrounding — verify `webglcontextrestored` handler re-initializes fully
- [ ] **Float texture fallback:** Open on real iOS Safari (not simulator) — fluid sim either works or gracefully degrades; it does not crash
- [ ] **Render target disposal:** Open DevTools Memory tab, resize window 5 times — `renderer.info.memory.textures` stays constant
- [ ] **Bloom on mobile:** Open on mid-range Android with Performance throttle 6× in DevTools — frame rate stays above 25fps
- [ ] **Vimeo lazy loading:** Load page on 3G throttle — Network tab shows zero requests to `player.vimeo.com` until scroll reaches video section
- [ ] **Passive scroll listeners:** Chrome DevTools shows no "non-passive event listener" warnings in the console
- [ ] **SEO HTML:** View page source — `<h1>Tom Bernys</h1>` and section headings exist in HTML, not only in canvas
- [ ] **SMAA active:** Post-processing pipeline running — glass text edges are smooth, not jagged
- [ ] **DPR capped:** On iPhone 15 (DPR 3), `renderer.getPixelRatio()` returns 1.5, not 3
- [ ] **Font bundled / load handled:** Throttle network to Slow 3G — font loads without breaking the hero; loader animation plays during wait

---

## Recovery Strategies

| Pitfall | Recovery Cost | Recovery Steps |
|---------|---------------|----------------|
| Context loss not handled | MEDIUM | Add `contextlost`/`contextrestored` handlers; refactor init into restartable function |
| Float texture crash on iOS | HIGH | Requires fallback code path for fluid sim; may need to replace sim with CSS animation on mobile |
| GPU memory leak from render targets | MEDIUM | Add disposal to resize handler; monitor `renderer.info.memory` during testing |
| Bloom killing mobile fps | LOW | Feature-flag bloom off for mobile; test and ship |
| Vimeo blocking page load | LOW | Replace direct iframes with Intersection Observer placeholders; 1-2 hour refactor |
| Scroll jank | LOW | Convert to passive listeners + rAF-deferred read; low-risk refactor |
| No SEO HTML structure | MEDIUM | Requires adding semantic DOM layer and ensuring it does not visually interfere with WebGL |

---

## Pitfall-to-Phase Mapping

| Pitfall | Prevention Phase | Verification |
|---------|------------------|--------------|
| WebGL context loss | Phase: WebGL renderer setup | Test on real iOS Safari: background and restore tab |
| Float texture iOS incompatibility | Phase: Fluid simulation implementation | Test on real iOS 14 Safari; check extension availability |
| Render target GPU memory leak | Phase: WebGL renderer setup + any render target creation | Monitor `renderer.info.memory` across 5 resize events |
| Bloom mobile performance | Phase: Post-processing pipeline | Profile with Chrome DevTools on 6× CPU throttle |
| Vimeo iframe blocking load | Phase: Video embed / project showcase | Lighthouse audit: zero Vimeo requests before scroll |
| Scroll jank | Phase: Scroll integration | DevTools: no "non-passive" warnings; scroll flame chart clean |
| Font load race condition | Phase: WebGL hero / loading screen | Slow 3G test: loader plays; hero appears after font load |
| Uncapped DPR | Phase: WebGL renderer setup | `renderer.getPixelRatio()` check on iPhone |
| CDN import map MIME errors | Phase: Project scaffolding / setup | Deploy to actual host (GitHub Pages / Netlify) before any other work |
| Antialiasing broken by post-processing | Phase: Post-processing pipeline | Visual inspection of glass text edges with EffectComposer active |
| Prefers-reduced-motion | Phase: Accessibility pass | Test with `prefers-reduced-motion: reduce` in OS settings |
| SEO empty canvas | Phase: HTML structure scaffolding | `curl` or View Source: heading text present in HTML |
| Vimeo autoplay blocked on mobile | Phase: Video embed phase | Test autoplay Vimeo on real iOS Safari in Low Power Mode |
| InstancedMesh resize leak | Phase: Instanced background implementation | Rotate device 5× on mobile; check memory |
| Loader blocking WebGL init | Phase: Loading screen / entry experience | Profile: loader CSS animation plays before first Three.js frame |

---

## Sources

- Three.js official disposal docs: https://threejs.org/docs/#manual/en/introduction/How-to-dispose-of-objects
- Three.js forum — context loss recovery: https://discourse.threejs.org/t/how-to-fix-three-webglrenderer-context-lost/66395
- Three.js forum — context loss on iOS: https://discourse.threejs.org/t/how-to-fix-context-lost-android-iphone-ios/56829
- Khronos WebGL Wiki — HandlingContextLost: https://www.khronos.org/webgl/wiki/HandlingContextLost
- Three.js issue — GPGPU examples on iPadOS: https://github.com/mrdoob/three.js/issues/19837
- Three.js issue — OES_texture_float iOS 14: https://github.com/mrdoob/three.js/issues/25741
- WebGL cross-platform issues: https://webglfundamentals.org/webgl/lessons/webgl-cross-platform-issues.html
- Codrops — Building Efficient Three.js Scenes (2025-02-11): https://tympanus.net/codrops/2025/02/11/building-efficient-three-js-scenes-optimize-performance-while-maintaining-quality/
- MDN — WebGL best practices: https://developer.mozilla.org/en-US/docs/Web/API/WebGL_API/WebGL_best_practices
- Discover Three.js tips and tricks: https://discoverthreejs.com/tips-and-tricks/
- Codrops — SEO-friendly WebGL text (2025-06-05): https://tympanus.net/codrops/2025/06/05/how-to-create-responsive-and-seo-friendly-webgl-text/
- web.dev — Lazy load images and iframes: https://web.dev/learn/performance/lazy-load-images-and-iframe-elements
- Vimeo Help — Autoplay restrictions: https://help.vimeo.com/hc/en-us/articles/29677068222737-Troubleshooting-Autoplay-restrictions
- Vimeo Help — Inline playback on mobile: https://help.vimeo.com/hc/en-us/articles/12425812053265-Inline-playback-on-mobile
- Scroll jank — passive listeners: https://copyprogramming.com/howto/scroll-events-requestanimationframe-vs-requestidlecallback-vs-passive-event-listeners
- Three.js forum — UnrealBloom optimize: https://discourse.threejs.org/t/unreal-bloom-optimize/35476
- PixelFreeStudio — WebGL in mobile development: https://blog.pixelfreestudio.com/webgl-in-mobile-development-challenges-and-solutions/
- Evil Martians — OffscreenCanvas Web Workers: https://evilmartians.com/chronicles/faster-webgl-three-js-3d-graphics-with-offscreencanvas-and-web-workers

---
*Pitfalls research for: WebGL creative portfolio (Tom Bernys — single-page, Three.js r170, GPU fluid sim, post-processing, Vimeo embeds)*
*Researched: 2026-03-22*
