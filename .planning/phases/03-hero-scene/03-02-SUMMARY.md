---
phase: 03-hero-scene
plan: 02
subsystem: hero-scene
tags: [webgl, tiles, interaction, mouse, touch, float, shader, lighting]
dependency_graph:
  requires: [03-01-SUMMARY.md]
  provides: [instanced tile background, mouse/touch interaction, idle float, entry animation]
  affects: [index.html]
tech_stack:
  patterns:
    - Cylindrical instanced tile background with quad-tree layout for varied tile sizes
    - Custom ShaderMaterial with procedural patterns (noise, checkerboard, gradient, scanlines, vertical bars)
    - Deterministic seeded tile pattern assignment (no randomness on reload)
    - Quaternion-based mouse/touch text rotation with velocity damping
    - Idle vertical float animation (sin wave bob)
    - Entry dezoom animation from scale 1.6 to 1.0 over 2.4s with easeOutCubic
    - Edge glow on tile borders responding to mouse proximity
    - Custom glass shader with GGX specular, fresnel, organic scratches
key_files:
  modified:
    - path: index.html
      role: Added tile background system, interaction handlers, idle float, entry animation, lighting and shader tuning
decisions:
  - "Quad-tree tile layout with 4 size categories for visual variety"
  - "6 deterministic pattern types with seeded assignment — no randomness on reload"
  - "Balanced color distribution: reduced violet dominance, more blue/green/teal alternation"
  - "Crosshairs removed per user request"
  - "Corner links removed per user request — clean gaps between tiles"
  - "Logo text overlay on tiles removed per user request"
  - "Edge glow on tile borders: thin, short, subtle — only near mouse pointer"
  - "Quaternion rotation with 0.06 lerp speed, 0.92 velocity damping, ±25° max amplitude"
  - "Entry dezoom: scale 1.6→1.0, z offset +2, duration 2.4s easeOutCubic"
  - "Glass shader specular reduced: light1=0.4, light2=0.2, clamp=0.5"
  - "Glass shader composition: refracted*1.9, fresnel blend 0.35, self-illumination 0.3"
  - "Scene lighting reduced: ambient 0.3, p1=1.8, p2=1.2, p3=1.5, directional=0.4"
metrics:
  completed: 2026-03-23
  tasks_completed: 3
  files_modified: 1
---

# Phase 3 Plan 2: Tile Background, Interaction & Polish Summary

**One-liner:** Instanced cylindrical tile background with procedural patterns, mouse/touch quaternion rotation, idle float, entry dezoom animation, and extensive visual tuning of lighting and shader parameters.

## What Was Built

### Task 1: Instanced tile background with procedural shader patterns
- Quad-tree layout generating tiles of 4 different sizes for visual variety
- 6 deterministic pattern types: noise, checkerboard, gradient, scanlines, vertical bars, diamond
- Seeded pattern assignment ensuring consistent look across page reloads
- Balanced color distribution avoiding violet dominance
- Muted color palette: navy, charcoal, deep teal, slate, dark violet

### Task 2: Mouse/touch interaction with idle float
- Quaternion-based rotation responding to pointer movement
- Smooth lerp (0.06) with velocity damping (0.92)
- ±25° max rotation amplitude
- Idle vertical float: gentle sin-wave bob
- Slow drift back to center when mouse stops

### Task 3: Visual polish and tuning
- Entry dezoom animation: text arrives from scale 1.6 over 2.4s
- Edge glow on tile borders near mouse pointer (thin, subtle)
- Glass shader specular and fresnel values tuned down for less aggressive reflections
- Scene lighting intensities reduced across all light sources
- Removed: crosshairs, corner links, logo text overlay on tiles

## Deviations from Plan
- Crosshairs (planned) were removed per user preference
- MeshPhysicalMaterial replaced with custom ShaderMaterial for glass text (more control)
- Logo text overlay added then removed per user preference
- Extensive iterative tuning of lighting/shader values based on user feedback

## Self-Check: PASS
All Phase 3 success criteria met:
1. ✅ "Tom Bernys" as extruded refractive 3D glass text centered in viewport
2. ✅ Mouse/touch causes text rotation; auto-float and drift when idle
3. ✅ Instanced cylindrical tiles with procedural animated patterns wrapping around viewer
4. ✅ Canvas fixed behind scrollable DOM content
5. ✅ Bloom and vignette post-processing active
