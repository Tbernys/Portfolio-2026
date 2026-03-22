This is a zero-build, static WebGL
  experience — no React, no Next.js, no
  bundler. Pure vanilla HTML/JS/CSS deployed
   on GitHub Pages.

  ---
  Core Technologies (most to least critical)

  
Three.js r170 — The backbone

  The entire site is a 3D WebGL scene.
  Three.js handles:
  
Scene, camera, renderer setup
glTF model loading (GLTFLoader for
logo_black.glb)
Render targets for multi-pass
post-processing
Instanced mesh rendering (background
tiles)
Environment lighting (RoomEnvironment)

  
Custom GLSL Shaders — The visual
identity

  Extensive hand-written shaders power every
   visual effect:
  
Glass shader — 8-sample refraction,
chromatic aberration, GGX specular,
Fresnel reflections
GPU Fluid Simulation — Full
Navier-Stokes solver (curl → velocity →
divergence → pressure → advection) at
128×128
Bloom — 3-pass Gaussian blur at multiple
resolutions (1/4, 1/8, 1/16)
Background tiles — Instanced quad-tree
with procedural patterns, cylindrical
wrapping, halftone dots
FXAA — Anti-aliasing post-process pass

  
WebGL 2 APIs — Rendering pipeline

  Multi-pass rendering pipeline:
  
Background render target (for glass
refraction sampling)
Main scene render
Bloom extraction → multi-level blur →
composite
Final output with tone mapping +
vignette

  
Canvas 2D API — Texture generation

  Used to dynamically generate:
  
Noise textures for shader roughness
Logo text overlay textures for
background tiles
All generated at runtime, no pre-baked
assets

  
ES6 Import Maps — Module loading

  Three.js loaded from jsDelivr CDN via
  import maps — no npm, no bundler:
  three → cdn.jsdelivr.net/npm/three@0.170.0

  
Discord REST API — Community widget

  Fetches live member/online counts from
  discord.com/api/v10/invites/... for the
  floating Discord widget with glitch hover
  effect.

  
CSS Animations — UI effects

  Keyframe animations for:
  
Loading screen glitch text resolver(character-by-character reveal with
scanlines)
Discord widget hover glitch effect