---
phase: 03-hero-scene
plan: 01
subsystem: hero-scene
tags: [webgl, three.js, glass-text, post-processing, bloom, vignette]
dependency_graph:
  requires: [02-03-SUMMARY.md]
  provides: [glass text mesh, env map, bloom+vignette pipeline]
  affects: [index.html]
tech_stack:
  added:
    - FontLoader (three/addons/loaders/FontLoader.js)
    - TextGeometry (three/addons/geometries/TextGeometry.js)
    - RoomEnvironment (three/addons/environments/RoomEnvironment.js)
    - EffectComposer (three/addons/postprocessing/EffectComposer.js)
    - RenderPass (three/addons/postprocessing/RenderPass.js)
    - UnrealBloomPass (three/addons/postprocessing/UnrealBloomPass.js)
    - OutputPass (three/addons/postprocessing/OutputPass.js)
    - ShaderPass + VignetteShader (three/addons/postprocessing + shaders)
  patterns:
    - PMREMGenerator + RoomEnvironment for glass refraction env map
    - MeshPhysicalMaterial with transmission=1/ior=1.5 for crystal glass look
    - Analytical viewport-width scaling for text (FOV + cameraZ trigonometry)
    - EffectComposer replacing direct renderer.render() in RAF loop
    - fontReady chain ensures buildTextMesh → setupPostProcessing → warmupShaders order
key_files:
  modified:
    - path: index.html
      role: Added glass text system and post-processing pipeline; updated RAF loop, resize handler, context restore, and main() bootstrap
decisions:
  - "Comfortaa Bold used for TextGeometry — only pre-converted typeface.json available in project"
  - "Font loaded via loadAsync (Option A) not inline JSON — avoids 580KB bloat in index.html"
  - "setupPostProcessing() called inside fontReady.then() chain to guarantee RenderPass sees the text mesh"
  - "warmupShaders() chained after fontReady so compileAsync covers MeshPhysicalMaterial (glass)"
  - "Lighting: warm DirectionalLight (key) + cool DirectionalLight (fill) + white PointLight (rim) for bevel highlights"
  - "Bloom: strength=0.4, radius=0.3, threshold=0.75 — subtle haze per CONTEXT.md decision"
  - "Vignette: darkness=1.5 — medium visible darkening per CONTEXT.md decision"
metrics:
  duration: ~10 min
  completed: 2026-03-22
  tasks_completed: 1
  files_modified: 1
---

# Phase 3 Plan 1: Glass Text and Post-Processing Pipeline Summary

**One-liner:** Crystal-clear glass text "Tom Bernys" with transmission/refraction material, RoomEnvironment env map, directional+point lighting rig, and EffectComposer bloom+vignette pipeline.

## What Was Built

### Task 1: Glass text mesh with environment map, lighting, and post-processing pipeline

All Phase 3 hero scene subsystems required by this plan were implemented in `index.html`:

**Imports added:**
- FontLoader, TextGeometry, RoomEnvironment
- EffectComposer, RenderPass, UnrealBloomPass, OutputPass, ShaderPass, VignetteShader

**Functions added:**
- `setupEnvMap()` — PMREMGenerator + RoomEnvironment, sets `scene.environment` before any glass material is created
- `setupLighting()` — warm key DirectionalLight + cool fill DirectionalLight + rim PointLight for bevel highlight catching
- `loadFont()` — async fetch of `fonts/comfortaa_bold.typeface.json`, stores result in `loadedFont` for context restore
- `buildTextMesh(font)` — TextGeometry with bevel, `geometry.center()`, analytical 60%-viewport-width scaling via FOV math, `MeshPhysicalMaterial` with `transmission=1, ior=1.5, thickness=0.5`
- `setupPostProcessing()` — EffectComposer with RenderPass → UnrealBloomPass → VignetteShader ShaderPass → OutputPass

**Wiring changes:**
- RAF loop: `composer.render()` replaces `renderer.render()`, with fallback for pre-init frames
- Resize handler: `composer.setSize(w, h)` added alongside renderer resize
- Context restore handler: rebuilds env map, lighting, text, and composer
- `main()`: font load parallelized with loading screen; text+post-processing built before `warmupShaders()` so `compileAsync` covers the glass material

## Task 2: Checkpoint — Human Verify

Status: AWAITING — checkpoint reached. Server running at http://localhost:4003.

## Deviations from Plan

None — plan executed exactly as written.

## Self-Check: PENDING

(Will be finalized after checkpoint approval)
