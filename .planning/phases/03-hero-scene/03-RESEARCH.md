# Phase 3: Hero Scene - Research

**Researched:** 2026-03-22
**Domain:** Three.js WebGL — 3D glass text, instanced tile background, post-processing, mouse/touch interaction
**Confidence:** HIGH (core APIs verified via official docs and official examples)

---

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions

**Glass text appearance**
- Crystal clear glass with strong refraction — background visible and distorted through letters
- Thick block extrusion — letters feel like solid glass sculptures with deep internal refraction
- Pure neutral color — no tint, color comes only from what's refracted behind the text
- Serif font — classic/editorial feel, serifs add refraction detail
- Single line layout — "Tom Bernys" on one line, wide cinematic feel
- ~60% viewport width — prominent but leaves breathing room for the background
- Sharp edge bevel — clean, precise cuts with light catching distinct angles
- Soft glow on edges — diffused light along edges, subtle but visible

**Interaction & idle motion**
- Smooth follow — text gently rotates to face the cursor, fluid, magnetically attracted feel
- Medium rotation range (~25-30°) — noticeable 3D effect, text stays recognizable
- Idle state: float only — no auto-rotation, just calm vertical bob. Always alive but serene
- Slow drift back (~3-4s) when mouse stops — lazy, dreamy return to center position
- Text should have a floating effect and interact with the pointer

**Tile background style**
- Muted color palette — low-saturation colors (navy, charcoal, deep teal), atmospheric but not monochrome
- Flowing motion animation — visible movement at comfortable pace, patterns ripple and evolve noticeably
- Medium density grid — enough tiles to feel immersive without being overwhelming
- Mixed patterns — tiles alternate between noise, gradient, and solid; each tile feels unique

**Post-processing & mood**
- Subtle haze bloom — light glow around bright areas, adds softness without feeling heavy
- Medium vignette — clearly visible darkening that creates a natural spotlight on center/text
- No additional color grading — bloom and vignette are enough, colors stay true to tile palette
- Overall mood: atmospheric & immersive — feels like entering a space, not just viewing a page

### Claude's Discretion
- Exact serif font selection (must extrude and refract well in 3D)
- Crosshair styling at tile grid intersections
- Float animation amplitude and speed
- Exact muted color values for the tile palette
- Pattern distribution across tiles (noise vs gradient vs solid ratios)
- Bloom threshold and radius tuning
- Vignette falloff curve
- Lighting setup (number, position, intensity of lights)

### Deferred Ideas (OUT OF SCOPE)
None — discussion stayed within phase scope
</user_constraints>

---

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|-----------------|
| HERO-01 | "Tom Bernys" rendered as 3D extruded glass text (TextGeometry + refraction/transmission shader) floating at center of viewport | TextGeometry from `three/addons/geometries/TextGeometry.js` + FontLoader.parse() with inline JSON. MeshPhysicalMaterial with transmission=1, ior=1.5, thickness drives the glass look. RoomEnvironment+PMREMGenerator supplies the env map required for realistic refraction. |
| HERO-02 | Glass text reacts to mouse/touch — rotation impulse on interaction, auto-spin, vertical float animation | Pointer/touch normalized coordinate tracking → lerp-based rotation on textMesh. Idle float via Math.sin(clock.elapsed). Touch: `e.touches[0].clientX / window.innerWidth * 2 - 1`. No external library needed. |
| HERO-03 | Background displays instanced cylindrical tiles with procedural patterns (noise, grid, gradient) wrapping around the viewer | InstancedMesh with CylinderGeometry. Per-tile ShaderMaterial using GLSL noise functions (inline, no external lib). Tiles arranged on a large radius sphere/cylinder surrounding the camera. instanceMatrix.needsUpdate in RAF loop. |
| HERO-04 | Crosshairs render at tile grid intersections | LineSegments or Points geometry placed at computed grid intersection positions. Can reuse the same instanced matrix data to position a second InstancedMesh of thin cross shapes. |
| HERO-05 | WebGL canvas is fixed behind scrollable DOM content | Already established in Phase 2: `#canvas { position: fixed; z-index: 0; }` and `main { position: relative; z-index: 1; pointer-events: none; }`. No work needed — just verify canvas stays fixed as DOM content is added. |
</phase_requirements>

---

## Summary

Phase 3 builds on the Phase 2 WebGL foundation (renderer, scene, camera, context loss handling, RAF loop, shader warmup gate) by adding the full hero scene. Three distinct technical subsystems need to be built and wired together: 3D glass text, an instanced cylindrical tile background, and a bloom+vignette post-processing pipeline.

The most technically sensitive part is the glass text. `MeshPhysicalMaterial` with `transmission: 1` requires an environment map to produce realistic refraction — without one, the glass appears black or flat. The standard approach is `RoomEnvironment` + `PMREMGenerator.fromScene()` which generates a procedural room environment without any external texture files, which fits the project's no-build-tools, single-file constraint perfectly. Font loading uses `FontLoader.parse()` with the pre-converted `fonts/comfortaa_bold.typeface.json` (already present in the repo), avoiding any async fetch request during initialization.

The instanced tile background uses a single `InstancedMesh` of `CylinderGeometry`, with per-tile transforms set via `setMatrixAt()` and a custom `ShaderMaterial` that reads a per-instance `instanceIndex` to generate unique noise/gradient/solid patterns per tile. Arranged in a large-radius hemisphere around the camera, this creates the "wrapped around the viewer" effect without needing a special cylindrical projection.

Post-processing is handled by Three.js's built-in `EffectComposer` with `RenderPass → UnrealBloomPass → ShaderPass (vignette) → OutputPass`. All passes import from `three/addons/postprocessing/` which is already mapped in the project's importmap via `"three/addons/"`. The EffectComposer must be resized in the existing `window.addEventListener('resize', ...)` handler alongside the renderer resize — failing to do this is the most common pitfall with EffectComposer.

**Primary recommendation:** Build in this order: (1) font+text geometry with glass material, (2) env map setup, (3) lighting, (4) post-processing pipeline, (5) instanced tiles, (6) crosshairs, (7) mouse/touch interaction. Each subsystem is independently testable and the order minimizes wasted iteration.

---

## Standard Stack

### Core
| Library | Version | Purpose | Why Standard |
|---------|---------|---------|--------------|
| Three.js | r175 (already in project) | 3D rendering engine | Already used; r175 is WebGL 2 baseline per project decision |
| Three.js TextGeometry | addons (r175) | Extruded 3D text from typeface.json | Only built-in Three.js solution for typeface.json text extrusion |
| Three.js FontLoader | addons (r175) | Parses typeface.json into Font object | Required by TextGeometry; `parse()` avoids fetch |
| Three.js MeshPhysicalMaterial | core (r175) | PBR glass/transmission material | Standard Three.js transmission; no external dependency |
| Three.js RoomEnvironment | addons (r175) | Procedural env map for glass reflections | No external texture file needed; generates in-browser |
| Three.js PMREMGenerator | core (r175) | Pre-filters env map for PBR materials | Required to make env map usable with MeshPhysicalMaterial |
| Three.js InstancedMesh | core (r175) | Render many tile instances in one draw call | Standard Three.js instancing; critical for tile count performance |
| Three.js EffectComposer | addons (r175) | Post-processing pipeline | Official Three.js solution; all passes available via importmap |
| Three.js UnrealBloomPass | addons (r175) | Bloom/glow post-processing | High-quality bloom; well-maintained official pass |
| Three.js ShaderPass | addons (r175) | Custom vignette pass | Generic pass for custom GLSL; tDiffuse uniform pattern |
| Three.js OutputPass | addons (r175) | Final output with color management | Required in r152+ to replace deprecated renderToScreen |
| GSAP | 3.14.2 (already in project) | Idle float animation, drift-back tween | Already imported; handles easing math; used in Phase 2 |

### Supporting
| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| Three.js RenderPass | addons (r175) | First pass: renders scene to texture | Always first pass in EffectComposer |
| Three.js CylinderGeometry | core (r175) | Tile geometry | Short wide cylinders (radiusTop ≈ radiusBottom, low height) for tile look |

### Alternatives Considered
| Instead of | Could Use | Tradeoff |
|------------|-----------|----------|
| MeshPhysicalMaterial (built-in) | MeshTransmissionMaterial (Drei) | Drei's version has better quality but requires React/R3F — not applicable here |
| RoomEnvironment (procedural) | External HDRI .exr/.hdr | Better lighting but requires a network request and EXRLoader — breaks single-file constraint |
| Custom ShaderPass (vignette) | VignetteShader from addons | Three.js addons includes VignetteShader at `three/addons/shaders/VignetteShader.js` — use this instead of hand-rolling |
| Inline GLSL noise | Simplex noise library | No CDN; inline Simplex or value noise in GLSL is ~20 lines and avoids any external dependency |

**Installation:** No new packages needed. All required modules are already in the importmap under `"three/addons/"`.

---

## Architecture Patterns

### Recommended Project Structure (within the single-file index.html)

The existing module script already has a clean section structure. Phase 3 adds:

```
<script type="module">
  // [existing Phase 2 code: initRenderer, initScene, RAF loop, context handlers, loader, transition]

  // ─── Phase 3 additions ───────────────────────────────────────────────────
  // 1. setupEnvMap()         — RoomEnvironment + PMREMGenerator
  // 2. buildTextMesh()       — FontLoader.parse + TextGeometry + MeshPhysicalMaterial
  // 3. buildTileBackground() — InstancedMesh + ShaderMaterial
  // 4. buildCrosshairs()     — LineSegments at grid intersections
  // 5. setupPostProcessing() — EffectComposer + passes
  // 6. initInteraction()     — pointermove/touchmove → target rotation
  // 7. updateInteraction()   — called in RAF loop: lerp toward target
  // 8. updateTiles()         — called in RAF loop: animate shader uniforms
  // 9. updateFloat()         — called in RAF loop: sinusoidal Y bob on textMesh
```

The RAF loop changes from `renderer.render(scene, camera)` to `composer.render()` once the composer is set up. All Phase 2 code continues to work; the composer replaces the direct render call.

### Pattern 1: Font Loading with Inline JSON (avoid fetch)

**What:** `FontLoader.parse()` accepts the raw JSON object from an inline `<script type="application/json">` tag or a JS object literal. Avoids a network request for the font file.
**When to use:** The font JSON is already on disk in `fonts/comfortaa_bold.typeface.json`. In a single-file constraint, embed the font inline OR fetch it once during the warmup gate.

```javascript
// Source: https://threejs.org/docs/#examples/en/loaders/FontLoader
import { FontLoader } from 'three/addons/loaders/FontLoader.js';
import { TextGeometry } from 'three/addons/geometries/TextGeometry.js';

// Option A: fetch during warmup (font file is on same origin, served by Vercel)
const loader = new FontLoader();
const font = await loader.loadAsync('fonts/comfortaa_bold.typeface.json');

// Option B: inline JSON (large but eliminates fetch)
// const fontData = { /* paste full typeface.json contents as a JS variable */ };
// const font = loader.parse(fontData);
```

For this project, Option A (fetch via `loadAsync`) is recommended. The file is served from the same Vercel origin, is cached after first load, and avoids bloating index.html by ~1.5MB.

### Pattern 2: MeshPhysicalMaterial Glass Setup

**What:** Transmission + env map for realistic refraction.
**When to use:** Glass text mesh.

```javascript
// Source: https://threejs.org/docs/#api/en/materials/MeshPhysicalMaterial
// Source: https://threejs.org/docs/#api/en/extras/PMREMGenerator

import { RoomEnvironment } from 'three/addons/environments/RoomEnvironment.js';

function setupEnvMap(renderer, scene) {
  const pmremGenerator = new THREE.PMREMGenerator(renderer);
  const envTexture = pmremGenerator.fromScene(new RoomEnvironment()).texture;
  scene.environment = envTexture;  // applies to all PBR materials automatically
  pmremGenerator.dispose();
}

const glassMaterial = new THREE.MeshPhysicalMaterial({
  transmission: 1.0,        // fully transmissive — glass
  ior: 1.5,                 // standard glass IOR
  thickness: 0.5,           // affects how deep light travels through
  roughness: 0.0,           // perfectly smooth glass surface
  metalness: 0.0,
  transparent: true,        // required alongside transmission
  envMapIntensity: 1.0,
});
```

**Critical:** `scene.environment` must be set BEFORE the glass material is added to the scene for it to pick up the env map. Set it in `setupEnvMap()` called early in `initScene()`.

### Pattern 3: TextGeometry with Bevel

**What:** Extrude + bevel for the sharp-edge glass sculpture look.
**When to use:** Building the "Tom Bernys" text mesh.

```javascript
// Source: https://threejs.org/docs/#examples/en/geometries/TextGeometry
import { TextGeometry } from 'three/addons/geometries/TextGeometry.js';

const geometry = new TextGeometry('Tom Bernys', {
  font: font,
  size: 1.0,           // tune to achieve ~60% viewport width
  depth: 0.4,          // thick block extrusion
  curveSegments: 6,    // lower = better perf; 6 is fine for glass (smooth by material)
  bevelEnabled: true,
  bevelThickness: 0.03,
  bevelSize: 0.02,
  bevelSegments: 4,
});
geometry.center();    // centers geometry around origin for easy rotation

const textMesh = new THREE.Mesh(geometry, glassMaterial);
```

**Note:** `depth` replaced `height` in older Three.js docs. The current parameter name is `depth` in r175.

### Pattern 4: InstancedMesh Tile Background

**What:** One draw call for all tiles; per-instance transform set at init, shader animated per frame via uniforms.
**When to use:** Building the cylindrical tile background.

```javascript
// Source: https://threejs.org/docs/#api/en/objects/InstancedMesh
const tileGeo = new THREE.CylinderGeometry(0.45, 0.45, 0.08, 8);
const tileMat = new THREE.ShaderMaterial({
  uniforms: {
    uTime: { value: 0.0 },
  },
  vertexShader: `/* pass instanceId via vInstanceId varying */`,
  fragmentShader: `/* per-tile pattern based on vInstanceId + uTime */`,
});
const tiles = new THREE.InstancedMesh(tileGeo, tileMat, TILE_COUNT);

const dummy = new THREE.Object3D();
// Arrange tiles on hemisphere surface at large radius
for (let i = 0; i < TILE_COUNT; i++) {
  // spherical coordinates → position
  dummy.position.set(x, y, z);
  dummy.lookAt(0, 0, 0);  // face inward toward camera
  dummy.updateMatrix();
  tiles.setMatrixAt(i, dummy.matrix);
}
tiles.instanceMatrix.needsUpdate = true;
scene.add(tiles);

// In RAF loop:
tileMat.uniforms.uTime.value = clock.getElapsedTime();
// instanceMatrix.needsUpdate NOT needed every frame if positions don't change
```

**Note:** ShaderMaterial does not receive `gl_InstanceID` by default in Three.js unless you pass it as a varying from a custom vertex shader. The standard approach is to write `attribute float instanceIndex;` — but Three.js r175 WebGL 2 supports `gl_InstanceID` natively in GLSL ES 3.0 (`#version 300 es` with `glslVersion: THREE.GLSL3`). This is the cleaner approach.

### Pattern 5: EffectComposer Pipeline

**What:** RenderPass → UnrealBloomPass → ShaderPass (vignette) → OutputPass.
**When to use:** Replace `renderer.render(scene, camera)` with `composer.render()` in RAF loop.

```javascript
// Source: https://threejs.org/docs/#examples/en/postprocessing/EffectComposer
// Source: https://threejs.org/examples/webgl_postprocessing_unreal_bloom.html
import { EffectComposer } from 'three/addons/postprocessing/EffectComposer.js';
import { RenderPass } from 'three/addons/postprocessing/RenderPass.js';
import { UnrealBloomPass } from 'three/addons/postprocessing/UnrealBloomPass.js';
import { OutputPass } from 'three/addons/postprocessing/OutputPass.js';
import { ShaderPass } from 'three/addons/postprocessing/ShaderPass.js';

let composer;

function setupPostProcessing() {
  composer = new EffectComposer(renderer);
  composer.addPass(new RenderPass(scene, camera));

  const bloomPass = new UnrealBloomPass(
    new THREE.Vector2(window.innerWidth, window.innerHeight),
    0.4,   // strength  — tune: subtle haze
    0.3,   // radius    — tune
    0.75   // threshold — only bright areas bloom
  );
  composer.addPass(bloomPass);

  // Vignette via VignetteShader from addons
  // import { VignetteShader } from 'three/addons/shaders/VignetteShader.js';
  // const vignettePass = new ShaderPass(VignetteShader);
  // vignettePass.uniforms['darkness'].value = 1.6;  // tune
  // composer.addPass(vignettePass);

  composer.addPass(new OutputPass());
}

// In resize handler:
composer.setSize(window.innerWidth, window.innerHeight);
```

### Pattern 6: Mouse/Touch Interaction with Lerp

**What:** Track normalized pointer position → target rotation → lerp textMesh rotation in RAF loop.
**When to use:** Building the "smooth follow" interaction.

```javascript
// Source: https://discourse.threejs.org/t/rotate-object-by-following-mouse-position/22359
const pointer = { x: 0, y: 0 };
const targetRot = { x: 0, y: 0 };
let isIdle = true;
let idleTimer = null;

function onPointerMove(e) {
  const clientX = e.touches ? e.touches[0].clientX : e.clientX;
  const clientY = e.touches ? e.touches[0].clientY : e.clientY;
  pointer.x = (clientX / window.innerWidth) * 2 - 1;
  pointer.y = -(clientY / window.innerHeight) * 2 + 1;

  const maxAngle = Math.PI / 6;  // 30°
  targetRot.y = pointer.x * maxAngle;
  targetRot.x = pointer.y * maxAngle * 0.5;

  isIdle = false;
  clearTimeout(idleTimer);
  idleTimer = setTimeout(() => { isIdle = true; }, 150);
}

window.addEventListener('pointermove', onPointerMove);
window.addEventListener('touchmove', e => onPointerMove(e), { passive: true });

// In RAF loop (alpha controls lerp speed):
const lerpSpeed = isIdle ? 0.015 : 0.08;  // slow drift back vs fast follow
textMesh.rotation.y += (targetRot.y - textMesh.rotation.y) * lerpSpeed;
textMesh.rotation.x += (targetRot.x - textMesh.rotation.x) * lerpSpeed;

// Idle float:
textMesh.position.y = Math.sin(clock.getElapsedTime() * 0.6) * 0.08;
```

**When idle=true**, targetRot drifts toward {0,0}, so lerpSpeed=0.015 gives the dreamy 3-4s drift-back at 60fps (0.985^(60*3) ≈ 0.01, close enough to center).

### Anti-Patterns to Avoid

- **Missing OutputPass:** In Three.js r152+, omitting OutputPass causes incorrect gamma/color output (scene looks washed out). Always add it as the final pass.
- **Forgetting `composer.setSize()` on resize:** EffectComposer maintains internal render targets. Resizing only the renderer leaves the composer at the old size, causing blurry/stretched output. Must call `composer.setSize(w, h)` in the same resize handler.
- **`transmission` without `scene.environment`:** Glass will appear black or show no refraction. RoomEnvironment must be set before the glass mesh is added.
- **`geometry.center()` missing:** TextGeometry origin is at bottom-left of first character by default. Without centering, rotation pivot is wrong and the text drifts off-center during interaction.
- **Using `transparent: false` with transmission:** Must set `transparent: true` alongside `transmission > 0` or the material falls back to opaque rendering.
- **Animating `instanceMatrix` every frame unnecessarily:** If tile positions don't change, only set `instanceMatrix.needsUpdate = true` once at init. For animated patterns, only update `uTime` uniform — no need to touch instanceMatrix.
- **High `curveSegments` on TextGeometry:** Default is 12. For a glass mesh the smoothness comes from the material's normal-based shading, not geometry density. Use 4–6 to keep vertex count manageable.
- **GLSL club plugin pattern repeated:** Phase 2 learned that GSAP Club plugins fail in importmap ES module context. For GLSL, use inline shader strings — they work perfectly in `<script type="module">`.

---

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Post-processing pipeline | Custom framebuffer swap | EffectComposer | Handles ping-pong buffers, render target management, pass ordering |
| Bloom | Gaussian blur shader | UnrealBloomPass | Separable blur with bright-pass extraction; complex to get right |
| Vignette | Inline GLSL in custom pass | VignetteShader (`three/addons/shaders/VignetteShader.js`) | Already written, exposes `darkness` and `offset` uniforms |
| Environment map | Manual CubeCamera | RoomEnvironment + PMREMGenerator | No HDR file needed; automatic prefiltering for PBR |
| Float animation | Custom sin tween | `Math.sin(clock.getElapsedTime())` directly in RAF | Trivial; no library needed |
| Touch normalization | Complex touch lib | 3 lines of inline math | Straightforward; see Pattern 6 |

**Key insight:** Everything in this phase is achievable with Three.js addons already in the importmap. The only external asset is the font JSON already in `fonts/comfortaa_bold.typeface.json`.

---

## Common Pitfalls

### Pitfall 1: Black Glass / No Refraction
**What goes wrong:** `MeshPhysicalMaterial` with `transmission: 1` renders as black or flat grey.
**Why it happens:** Transmission refraction requires `scene.environment` to be set. Without an env map, there is no light source to refract.
**How to avoid:** Call `setupEnvMap()` before building the text mesh. Set `scene.environment = pmremTexture` in `initScene()`.
**Warning signs:** Text appears black or dark on first render.

### Pitfall 2: EffectComposer Resize Not Called
**What goes wrong:** After window resize, post-processing output is blurry, stretched, or the wrong size.
**Why it happens:** EffectComposer maintains its own internal WebGLRenderTarget instances. Resizing the renderer does not resize these.
**How to avoid:** In the existing resize handler, add `composer.setSize(w, h)` immediately after `renderer.setSize(w, h)`.
**Warning signs:** Bloom looks pixelated or misaligned after resize.

### Pitfall 3: Text Not Centered / Wrong Rotation Pivot
**What goes wrong:** Text rotates around its left-bottom corner instead of its center.
**Why it happens:** TextGeometry's default origin is at the bottom-left of the text baseline.
**How to avoid:** Call `geometry.computeBoundingBox(); geometry.center();` after creating the geometry.
**Warning signs:** Text visibly orbits a point during mouse-follow interaction.

### Pitfall 4: Transmission Performance on Low-End Devices
**What goes wrong:** FPS drops on mobile or GPU tier 0/1 devices. Transmission triggers an extra render pass of the entire scene.
**Why it happens:** Three.js renders the scene a second time into a transmission buffer for each transmissive object.
**How to avoid:** For tier 0/1 devices (already detected in `gpuTier`), fall back to `MeshStandardMaterial` with low opacity instead of transmission. Or keep transmission but disable post-processing on low-tier devices.
**Warning signs:** GPU tier 0/1 with transmission enabled; mobile device overheating.

### Pitfall 5: `compileAsync` Must Include Hero Geometry
**What goes wrong:** Glass text stutters or flickers on the first visible frame after the loading transition.
**Why it happens:** The existing `warmupShaders()` calls `renderer.compileAsync(scene, camera)` — but if the text mesh is added to the scene AFTER the loading screen is dismissed, it compiles on first render.
**How to avoid:** Build all scene geometry (text, tiles, crosshairs) and add to scene BEFORE calling `warmupShaders()` in the bootstrap sequence. The loading screen already gates on `shadersReady`, so Phase 3 geometry just needs to be added during `initScene()`.
**Warning signs:** Visible frame stutter/flash right as the loading transition completes.

### Pitfall 6: Font Fetch 404 in Local Dev
**What goes wrong:** `FontLoader.loadAsync('fonts/comfortaa_bold.typeface.json')` fails with a 404 when running directly from the filesystem (`file://`).
**Why it happens:** Browser blocks cross-origin requests from `file://` URLs.
**How to avoid:** Always test via a local HTTP server (e.g., `npx serve .` or VS Code Live Server). This is already the workflow for this project since Phase 1.
**Warning signs:** Console shows `net::ERR_FILE_NOT_FOUND` or CORS error for the font.

### Pitfall 7: ShaderMaterial on InstancedMesh — gl_InstanceID Availability
**What goes wrong:** Tile patterns look identical on all tiles; `gl_InstanceID` is zero everywhere.
**Why it happens:** GLSL ES 1.0 (default in Three.js WebGL 2 mode) does not expose `gl_InstanceID`. Must opt into GLSL ES 3.0.
**How to avoid:** Set `glslVersion: THREE.GLSL3` on the `ShaderMaterial`. In GLSL ES 3.0, `gl_InstanceID` is available natively as a built-in.
**Warning signs:** All tiles show the same pattern regardless of position.

---

## Code Examples

Verified patterns from official sources:

### Full Post-Processing Setup (r175 correct pattern)
```javascript
// Source: https://threejs.org/examples/webgl_postprocessing_unreal_bloom.html
import { EffectComposer } from 'three/addons/postprocessing/EffectComposer.js';
import { RenderPass } from 'three/addons/postprocessing/RenderPass.js';
import { UnrealBloomPass } from 'three/addons/postprocessing/UnrealBloomPass.js';
import { OutputPass } from 'three/addons/postprocessing/OutputPass.js';
import { ShaderPass } from 'three/addons/postprocessing/ShaderPass.js';
import { VignetteShader } from 'three/addons/shaders/VignetteShader.js';

composer = new EffectComposer(renderer);
composer.addPass(new RenderPass(scene, camera));
composer.addPass(new UnrealBloomPass(
  new THREE.Vector2(window.innerWidth, window.innerHeight),
  0.4,   // strength
  0.3,   // radius
  0.75   // threshold
));
const vignettePass = new ShaderPass(VignetteShader);
vignettePass.uniforms['darkness'].value = 1.5;
composer.addPass(vignettePass);
composer.addPass(new OutputPass());
```

### RoomEnvironment (no external texture file)
```javascript
// Source: https://threejs.org/docs/#examples/en/environments/RoomEnvironment
import { RoomEnvironment } from 'three/addons/environments/RoomEnvironment.js';

const pmremGenerator = new THREE.PMREMGenerator(renderer);
scene.environment = pmremGenerator.fromScene(new RoomEnvironment()).texture;
pmremGenerator.dispose();
```

### TextGeometry — Centered, Beveled
```javascript
// Source: https://threejs.org/docs/#examples/en/geometries/TextGeometry
import { FontLoader } from 'three/addons/loaders/FontLoader.js';
import { TextGeometry } from 'three/addons/geometries/TextGeometry.js';

const font = await new FontLoader().loadAsync('fonts/comfortaa_bold.typeface.json');
const geometry = new TextGeometry('Tom Bernys', {
  font,
  size: 1.0,
  depth: 0.4,
  curveSegments: 6,
  bevelEnabled: true,
  bevelThickness: 0.03,
  bevelSize: 0.02,
  bevelSegments: 4,
});
geometry.center();
```

### InstancedMesh with GLSL3 per-instance ID
```javascript
// Source: https://threejs.org/docs/#api/en/objects/InstancedMesh
const tileMat = new THREE.ShaderMaterial({
  glslVersion: THREE.GLSL3,
  uniforms: { uTime: { value: 0 } },
  vertexShader: `
    out float vId;
    void main() {
      vId = float(gl_InstanceID);
      gl_Position = projectionMatrix * modelViewMatrix * instanceMatrix * vec4(position, 1.0);
    }
  `,
  fragmentShader: `
    in float vId;
    uniform float uTime;
    out vec4 fragColor;
    // ... noise/gradient/solid pattern based on vId
    void main() {
      fragColor = vec4(color, 1.0);
    }
  `,
});
```

---

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| `height` param in TextGeometry | `depth` param | Three.js r125+ | Old docs/examples using `height` will produce a warning |
| `renderToScreen = true` on last pass | `OutputPass` as final pass | Three.js r152 | Missing OutputPass causes wrong gamma/color output in r175 |
| `renderer.physicallyCorrectLights = true` | Default in all r155+ | Three.js r155 | No longer needed; physically correct lighting is always on |
| `antialias` in EffectComposer | None needed | — | EffectComposer renders to render target; antialias setting on renderer ignored; use MSAA render target if needed |
| Separate GLSL noise library | Inline GLSL value/simplex noise | N/A | No CDN needed; 15-20 line implementations are sufficient |

**Deprecated/outdated:**
- `THREE.FontLoader` from `three/examples/jsm/loaders/FontLoader.js`: Import path changed to `three/addons/loaders/FontLoader.js` — the importmap already uses `"three/addons/"` which maps correctly.
- Old `height` property on TextGeometry: Use `depth` in r175.

---

## Open Questions

1. **Font size scaling to ~60% viewport width**
   - What we know: TextGeometry `size` is in Three.js world units; camera is PerspectiveCamera at z=5, FOV=40.
   - What's unclear: Exact `size` value that maps to 60% viewport width depends on camera FOV and the bounding box of the Comfortaa font glyphs for "Tom Bernys".
   - Recommendation: Compute analytically after text is created: `geometry.computeBoundingBox()`, measure `bbox.max.x - bbox.min.x`, compare to visible width at camera z=0 (visible width = 2 * z * tan(FOV/2 * π/180)), and scale `textMesh.scale.x` accordingly. Or iterate during dev.

2. **Tile arrangement: sphere vs flat grid vs cylinder**
   - What we know: The goal is "wrapping around the viewer, creating a sense of being inside an environment". A sphere or hemisphere arrangement achieves this; a flat grid in front of the camera does not.
   - What's unclear: Exact radius and tile count for "medium density" that looks right.
   - Recommendation: Start with a hemisphere of radius=8, ~120 tiles distributed via Fibonacci sphere algorithm. Fibonacci gives even distribution without clustering at poles.

3. **Transmission performance on mobile (GPU tier 0/1)**
   - What we know: Transmission adds one extra full-scene render pass. FPS can drop 30–50% on low-end devices.
   - What's unclear: Whether gpuTier 0/1 can sustain 30fps with transmission enabled.
   - Recommendation: Plan a fallback path: if `gpuTier <= 1`, use `MeshStandardMaterial({ color: 0xffffff, roughness: 0.1, metalness: 0.1, opacity: 0.3, transparent: true })` instead of transmission. This is a Phase 7 concern but should be designed in from Phase 3.

4. **Font file size and load time**
   - What we know: `comfortaa_bold.typeface.json` was generated with 850 glyphs (full set). Typical full typeface.json files are 300-800KB.
   - What's unclear: Exact file size and whether it should be included in the shader warmup gate or loaded in parallel.
   - Recommendation: Load font via `FontLoader.loadAsync()` in parallel with other init tasks. Add font loading as a Promise in the `Promise.all([revealDone, shadersReady, fontReady])` gate.

---

## Sources

### Primary (HIGH confidence)
- [TextGeometry — Three.js Docs](https://threejs.org/docs/#examples/en/geometries/TextGeometry) — constructor params, depth vs height, bevel settings
- [FontLoader — Three.js Docs](https://threejs.org/docs/#examples/en/loaders/FontLoader) — import path, loadAsync, parse method
- [MeshPhysicalMaterial — Three.js Docs](https://threejs.org/docs/#api/en/materials/MeshPhysicalMaterial) — transmission, ior, thickness, roughness
- [EffectComposer — Three.js Docs](https://threejs.org/docs/#examples/en/postprocessing/EffectComposer) — setup, addPass, setSize
- [UnrealBloomPass — Three.js Docs](https://threejs.org/docs/#examples/en/postprocessing/UnrealBloomPass) — constructor params
- [InstancedMesh — Three.js Docs](https://threejs.org/docs/#api/en/objects/InstancedMesh) — setMatrixAt, instanceMatrix.needsUpdate
- [PMREMGenerator — Three.js Docs](https://threejs.org/docs/#api/en/extras/PMREMGenerator) — fromScene, dispose
- [RoomEnvironment — Three.js Docs](https://threejs.org/docs/#examples/en/environments/RoomEnvironment) — no external texture file needed
- [Official bloom example — threejs.org/examples](https://threejs.org/examples/webgl_postprocessing_unreal_bloom.html) — OutputPass import path verified

### Secondary (MEDIUM confidence)
- [Playing with Light and Refraction in Three.js — Codrops (March 2025)](https://tympanus.net/codrops/2025/03/13/warping-3d-text-inside-a-glass-torus/) — transmission+IOR glass text technique; R3F/Drei but technique applies to vanilla Three.js
- [Creating the Effect of Transparent Glass — Codrops (2021)](https://tympanus.net/codrops/2021/10/27/creating-the-effect-of-transparent-glass-and-plastic-in-three-js/) — MeshPhysicalMaterial glass recipe; still valid in r175
- [Three.js forum: transmission performance](https://discourse.threejs.org/t/transmission-effect-pbr-material-performance/59274) — FPS impact data points
- [Three.js forum: MeshPhysicalMaterial performance cost](https://discourse.threejs.org/t/meshphysicalmaterial-can-i-measure-how-much-more-expensive-it-is/60398) — cost measurement discussion

### Tertiary (LOW confidence)
- Three.js forum discussions on rotate-to-follow-mouse patterns — technique is well-established but exact lerp alpha values are empirical; needs tuning during implementation
- Fibonacci sphere distribution for tile arrangement — standard algorithm but not Three.js specific; needs validation during implementation

---

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH — all core libraries are official Three.js addons, already in importmap, API docs verified
- Architecture: HIGH — patterns match official Three.js examples and existing project structure
- Pitfalls: HIGH for structural pitfalls (OutputPass, resize, centering); MEDIUM for performance (transmission on mobile) — depends on actual GPU
- Interaction/animation: MEDIUM — lerp alpha values are empirical; the pattern is verified but tuning is required

**Research date:** 2026-03-22
**Valid until:** 2026-04-22 (Three.js addons APIs are stable; valid ~30 days)
