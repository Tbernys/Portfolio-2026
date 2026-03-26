# Phase 7: Responsive and Launch - Research

**Researched:** 2026-03-26
**Domain:** CSS responsive layout, Three.js mobile WebGL performance, static site deployment
**Confidence:** HIGH

---

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions

**Grille projets mobile (RESP-05)**
- D-01: 1 colonne sur mobile (<768px) — chaque projet prend toute la largeur pour maximiser la visibilité des thumbnails vidéo
- D-02: Override de la décision Phase 5 D-01 (3 colonnes partout) — 3 colonnes reste sur desktop, 1 colonne sur mobile
- D-03: Le ratio 16:9 est conservé en 1 colonne

**Breakpoints (RESP-01, RESP-05)**
- D-04: Un seul breakpoint à 768px — desktop (≥768px) vs mobile (<768px)
- D-05: Aligné avec le seuil `isMobile` JS existant (`window.innerWidth < 768`)
- D-06: Pas de breakpoint tablet intermédiaire — le portfolio single-page ne le justifie pas

**Texte 3D mobile (RESP-04)**
- D-07: Combo scale réduit + FOV ajusté — les deux pour un résultat optimal sur 375px
- D-08: Le code existant fait `finalScale = isMobile ? scale * 0.85 : scale` (15% réduction) — insuffisant, besoin d'aller plus loin (~0.6-0.65x)
- D-09: FOV élargi sur mobile pour que le texte tienne naturellement dans le viewport

**WebGL mobile (RESP-02, RESP-03)**
- D-10: DPR max 1.5 sur mobile — déjà implémenté (`maxDPR = isMobile ? 1.5 : 2`)
- D-11: Bloom 2 niveaux sur mobile — déjà implémenté (`isMobile ? 2 : 3`)
- D-12: Tile count et shader complexity à réduire davantage si nécessaire pour atteindre 30fps

**Layout reflow mobile (RESP-01, RESP-05)**
- D-13: Toutes les sections content passent en single-column sur mobile
- D-14: Logos clients : rangée horizontale avec wrap ou réduction de taille
- D-15: Formulaire contact : déjà max-width 500px centré, devrait être OK sur mobile
- D-16: Section-inner (max-width 900px) s'adapte à 100% width sur mobile avec padding

**Déploiement 2e host (RESP-05)**
- D-17: GitHub Pages comme 2e host (Vercel déjà fait)
- D-18: Deploy via `gh-pages` branch du repo GitHub

### Claude's Discretion
- Valeurs exactes de scale et FOV pour le glass text mobile
- Nombre de tiles réduit sur mobile si nécessaire pour 30fps
- Ajustements typographiques (font-size) pour mobile
- Padding/margins mobile
- Lightbox adaptation mobile
- Performance optimizations spécifiques pour atteindre 30fps
- Glassmorphism adjustments si backdrop-blur trop coûteux sur mobile

### Deferred Ideas (OUT OF SCOPE)
None — discussion stayed within phase scope
</user_constraints>

---

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|------------------|
| RESP-01 | Site layout adapts gracefully to desktop (1024px+) and mobile (<768px) viewports | CSS `@media (max-width: 767px)` block; single breakpoint maps to existing `isMobile` JS threshold |
| RESP-02 | WebGL renders at reduced DPR (max 1.5) on mobile devices | Already implemented — `maxDPR = isMobile ? 1.5 : 2` at line 794; no code change needed, only verification |
| RESP-03 | Instanced tile count and shader complexity are reduced on mobile for 30fps+ target | `generateQuadTree()` currently runs same logic for all devices; add `isMobile` branch to reduce subdivision depth or base grid size; measure with DevTools Rendering > FPS Meter |
| RESP-04 | 3D text scales appropriately for mobile viewport (smaller size, adjusted FOV) | Line 2155 `scale * 0.85` must change to `scale * 0.6`–`0.65`; line 2382 FOV formula needs mobile-specific value (65–70°); also rebuild glassText on resize |
| RESP-05 | All content sections reflow to single-column layout on mobile | `.projects-grid` needs `grid-template-columns: 1fr` at <768px; `.section-title` padding; lightbox close button positioning; logo-row gap reduction |
</phase_requirements>

---

## Summary

Phase 7 completes all five RESP requirements plus deploys to a second static host. The implementation splits into three parallel work areas: (1) CSS responsive layout — adding a single `@media (max-width: 767px)` block that is the ONLY place new mobile styles live; (2) Three.js WebGL mobile tuning — reducing tile count and adjusting camera FOV/glass text scale; (3) GitHub Pages deployment.

The project is a single `index.html` with all CSS and JS inline. No build step exists. This simplifies deployment radically — GitHub Pages can serve the file directly from a branch. The DPR cap and bloom reduction are already live; Phase 7 fills the remaining gaps: no `@media` queries exist yet (confirmed by code audit — only one exists: `prefers-reduced-motion`), the glass text scale of 0.85 is insufficient on 375px, and the tile quad-tree does not vary by device tier.

**Primary recommendation:** Write all mobile CSS in one `@media (max-width: 767px)` block appended to the existing `<style>` tag. Adjust the glass text scale and FOV inline via the existing `isMobile` ternaries. Reduce tile count by limiting `generateQuadTree()` subdivision depth on mobile. Deploy to GitHub Pages by pushing a `gh-pages` branch with the single `index.html` plus `.nojekyll`.

---

## Standard Stack

### Core (already in project — no new dependencies)

| Library | Version | Purpose | Status |
|---------|---------|---------|--------|
| Three.js | r175 | 3D rendering, camera, WebGL renderer | In import map |
| GSAP + ScrollTrigger | 3.14.2 | Scroll animation (not directly involved in responsive) | In import map |
| Vanilla CSS | — | `@media` queries, layout | Inline `<style>` tag |
| Vanilla JS | — | `isMobile` flag, conditional rendering | Inline `<script>` |

### Deployment Tools

| Tool | Version | Purpose | Available |
|------|---------|---------|-----------|
| Vercel CLI | 50.9.5 | Primary host (already deployed) | Yes — `/Users/tom/.npm-global/bin/vercel` |
| Git | — | Manual `gh-pages` branch push | Yes |
| Node.js | v24.13.0 | Scripts if needed | Yes |
| npm | 11.6.2 | Package installation if needed | Yes |

No new npm packages required. GitHub Pages does not need `gh-pages` npm package — a manual `git push origin gh-pages` from the repo root suffices when the repo has a GitHub remote.

**Note:** No GitHub remote is currently configured (`git remote get-url origin` returns empty). The `gh-pages` branch does not yet exist. These must be set up as part of the deployment task.

---

## Architecture Patterns

### Pattern 1: Single `@media` Block at Bottom of `<style>`

**What:** All mobile overrides in one `@media (max-width: 767px)` block, appended after all existing rules.

**Why:** The existing CSS has zero `@media` queries (only one `prefers-reduced-motion` exception). Adding a single block at the end keeps specificity simple — later rules naturally override earlier desktop rules without needing `!important`.

**Example:**
```css
@media (max-width: 767px) {
  .projects-grid {
    grid-template-columns: 1fr;
  }

  #clients,
  #contact {
    padding: 4rem 1.25rem;
  }

  .section-inner {
    padding: 0;
  }

  .logo-row {
    gap: 1.5rem;
  }

  .section-title {
    padding: 0 1.25rem 1.5rem;
    font-size: clamp(1rem, 4vw, 1.5rem);
  }

  .lightbox-inner {
    width: 96vw;
  }

  .lightbox-close {
    top: 12px;
    right: 12px;
  }

  .cta-btn {
    left: 1rem;
    bottom: 1.5rem;
  }
}
```

### Pattern 2: `isMobile` Ternary for Three.js Camera and Text Scale

**What:** Extend existing `isMobile` branching to adjust FOV and glass text scale at the JS level.

**Current code (line 2155):**
```javascript
const finalScale = isMobile ? scale * 0.85 : scale;
```

**Required change:**
```javascript
const finalScale = isMobile ? scale * 0.62 : scale;
```

**FOV — current `initScene()` and resize handler (lines 2053, 2382):**
```javascript
const fov = aspect < 0.8 ? 55 : 40;
```

**Required change:** On mobile, use a wider FOV to guarantee the text fits in the viewport regardless of exact scale. The aspect-ratio heuristic `aspect < 0.8` already catches portrait phones (375/812 ≈ 0.46), but the FOV value of 55° is not wide enough to show the text at a readable size. Widen to approximately 65°:
```javascript
const fov = isMobile ? 65 : (aspect < 0.8 ? 55 : 40);
```

This must be applied in both `initScene()` (line 2053) and the resize handler (line 2382), and `camera.updateProjectionMatrix()` must be called after. The `buildGlassText()` function is not called on resize — it runs once on init. The scale is computed using `camera.fov` at the moment `buildGlassText()` runs, so the camera FOV must be set before calling `buildGlassText()`.

### Pattern 3: Tile Count Reduction on Mobile

**What:** `generateQuadTree()` (lines 1160–1210) uses a 6×5 base grid with recursive subdivision. The resulting tile count is deterministic and always the same on all devices. On mobile, reduce count by either lowering base grid or reducing `splitProb`.

**Approach (Claude's discretion):** The safest, least-disruptive change is to pass an `isMobile` argument and halve the base grid to 4×3 (from 6×5). This reduces instance count by ~56% without touching shader code.

```javascript
function generateQuadTree(mobile = false) {
  const rng = mulberry32(42);
  const tiles = [];
  const splitProb = mobile ? [0.35, 0.15] : [0.55, 0.25]; // less subdivision on mobile
  const cols = mobile ? 4 : 6;
  const rows = mobile ? 3 : 5;
  const baseW = 1 / cols;
  const baseH = 1 / rows;
  // ... rest unchanged
  for (let gx = 0; gx < cols; gx++) {
    for (let gy = 0; gy < rows; gy++) {
      subdivide(gx * baseW, gy * baseH, baseW, baseH, 0);
    }
  }
  return tiles;
}
```

Call as: `const tiles = generateQuadTree(isMobile);`

The corner links geometry (`buildCornerLinks`) is already commented out in context restore handler (line 2366). Confirm it is also not called on mobile in the main init path if tile count reduction is insufficient.

### Pattern 4: GitHub Pages Deployment (Manual Branch Push)

**What:** Push the repo to GitHub with a `gh-pages` branch containing only `index.html` and `.nojekyll`.

**Steps:**
1. Add a GitHub remote: `git remote add origin https://github.com/<username>/<repo>.git`
2. Push `main` branch
3. Create orphan `gh-pages` branch: `git checkout --orphan gh-pages`
4. Remove all files except `index.html`: `git rm -rf . && git checkout main -- index.html`
5. Add `.nojekyll` empty file (prevents Jekyll from processing the HTML)
6. Commit and push: `git push -u origin gh-pages`
7. In GitHub repo Settings > Pages > Source: select `gh-pages` branch, `/ (root)`

**Why `.nojekyll`:** Without it, GitHub Pages runs Jekyll on the directory, which does nothing harmful to a plain HTML file but adds ~30s latency to deployments and can fail if the file contains Liquid-like syntax (`{{ }}`). The `index.html` uses GLSL template literals with `${}` interpolation — no conflict, but `.nojekyll` is still best practice.

**Alternative — simpler single-branch approach:** Configure GitHub Pages to serve from `main` branch directly (Settings > Pages > Source: `main`). This avoids the orphan branch complexity and is appropriate since the repo already has `index.html` at the root. Simpler and requires zero extra git operations.

### Anti-Patterns to Avoid

- **Separate `@media` blocks scattered through the CSS:** Makes maintenance hard. One block at the end is authoritative.
- **Rebuilding glassText on every resize:** `buildGlassText()` disposes and recreates geometry/material; it should only be called once. The camera FOV change on resize via `camera.updateProjectionMatrix()` is sufficient — the scale was computed at init time using the initial FOV.
- **Blocking mobile scroll with `preventDefault`:** Already avoided (canvas-scoped passive listeners since Phase 4). Do not reintroduce.
- **Testing responsive only in browser DevTools emulation:** Chrome/Safari device emulation does not accurately simulate mobile GPU fill-rate pressure. Must test on a real device for RESP-03 verification.

---

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| GitHub Pages deploy | Custom deploy script | `git push origin gh-pages` or main-branch Pages config | Native git + GitHub handles everything |
| Responsive images | Custom JS resize observer | CSS `object-fit: cover` + `aspect-ratio` already in place | Already works correctly at any width |
| Performance profiling | Custom FPS counter | Chrome DevTools Rendering panel > Frame Rendering Stats | Built-in, shows GPU time per frame |
| Mobile breakpoint detection | A second JS `isMobile` check | The one at line 793 is already the canonical source | Duplicate detection creates drift |

---

## Common Pitfalls

### Pitfall 1: Glass Text Scale Computed Once at Init
**What goes wrong:** The glass text scale is derived from `camera.fov` inside `buildGlassText()`, which runs once during initialization. If FOV is changed in the resize handler without also changing the FOV at init time (before `buildGlassText()` runs), the mobile scale will be wrong on first load.

**Why it happens:** `initScene()` sets the FOV. `buildGlassText()` reads `camera.fov`. The resize handler also sets FOV but never rebuilds the text. So the initial load path determines the scale permanently.

**How to avoid:** Apply the `isMobile` FOV override inside `initScene()` (line 2053), not only in the resize handler. Verify the FOV is set before `buildGlassText()` is called in the init sequence.

**Warning signs:** Glass text appears at wrong size only on mobile first load; refreshing after manual resize gives a different (correct) size.

### Pitfall 2: Viewport Width at Init Time vs. Resize
**What goes wrong:** `isMobile` is computed once as a `const` at module level (line 793) using `window.innerWidth` at the moment the script runs. On some browsers, `window.innerWidth` during early JS execution reflects the layout viewport before the browser chrome collapses, giving a wider value than the visual viewport. This is unlikely to cause a real bug here but explains why a 375px phone might initialise with `isMobile = false` if the UA string regex also fails to match.

**How to avoid:** The UA regex check (`/Android|iPhone|iPad|iPod|webOS|BlackBerry|IEMobile|Opera Mini/i`) is the primary guard and catches all real mobile devices reliably. The `|| window.innerWidth < 768` is a fallback for desktop DevTools simulation. No change needed.

### Pitfall 3: backdrop-blur GPU Cost on Old Android
**What goes wrong:** `backdrop-filter: blur(12px)` is applied to `.cta-btn`, `.back-to-top`, `.project-overlay`, `.lightbox-backdrop`, and `.contact-submit`. On Android Chrome < 76 and older mid-range GPUs (Adreno 505/506), backdrop-blur triggers a costly offscreen compositing pass. Combined with the WebGL canvas already consuming GPU, this can drop frame rates below 30fps for unrelated CSS animations.

**Why it happens:** Backdrop blur forces the browser to composite the WebGL texture into a separate raster layer before applying the blur — this stacks on top of the WebGL render budget.

**How to avoid:** Claude's discretion allows removing `backdrop-filter` from `.project-overlay` on mobile (it is purely aesthetic on small cards). Keep it on the lightbox backdrop (full-screen effect worth the cost) and the CTA button (always visible, static). Add a `@media (max-width: 767px)` override:
```css
@media (max-width: 767px) {
  .project-overlay {
    backdrop-filter: none;
    -webkit-backdrop-filter: none;
  }
}
```

**Warning signs:** DevTools Performance trace shows "Compositing" taking >8ms per frame on Android.

### Pitfall 4: GitHub Pages `og:image` Absolute URL Points to Vercel
**What goes wrong:** `og:image` is currently hardcoded to `https://portfolio-2026-three-pi.vercel.app/og-image.jpg` (lines 37, 43). This absolute URL will work correctly when the page is served from GitHub Pages — the image is still fetched from Vercel. This is not a bug but means the GitHub Pages deployment relies on Vercel staying alive for social preview images.

**How to avoid:** No action required for v1. Document the dependency. If Vercel URL changes, both deployments need updating simultaneously.

### Pitfall 5: `.nojekyll` Omission Breaks Directories with Underscores
**What goes wrong:** GitHub Pages Jekyll processing by default ignores directories starting with `_` (e.g., `_layouts`). The project has no such directories, but Jekyll also rewrites certain HTML if Liquid template syntax appears. The `index.html` contains GLSL with `${...}` interpolation inside template literals — Jekyll does not process JS template literals, so no conflict exists.

**How to avoid:** Add `.nojekyll` anyway as standard practice. It is a zero-cost empty file.

### Pitfall 6: Tile Count Reduction Changes Visual Appearance
**What goes wrong:** Reducing the base grid from 6×5 to 4×3 changes the tile density visible behind the glass text. On a phone where the camera covers the full cylinder, fewer tiles means more empty space between tiles or larger individual tiles.

**How to avoid:** Tune `splitProb` rather than grid size alone. Lower splitProb increases average tile size (fewer subdivisions = fewer but larger tiles), which can look intentional. Visually test at 375px width. The seeded RNG (mulberry32 with seed 42) is deterministic, so the mobile layout will be consistent across reloads.

---

## Code Examples

### Glass Text FOV + Scale Fix (lines 2053, 2155, 2382)

```javascript
// In initScene() — apply mobile FOV before buildGlassText() runs
function initScene() {
  scene = new THREE.Scene();
  scene.background = new THREE.Color(0x000000);

  const aspect = window.innerWidth / window.innerHeight;
  // Mobile: wider FOV so text fits naturally in portrait viewport
  const fov = isMobile ? 65 : (aspect < 0.8 ? 55 : 40);
  camera = new THREE.PerspectiveCamera(fov, aspect, 0.1, 100);
  camera.position.z = 7;
}

// In buildGlassText() — increase mobile scale reduction
const finalScale = isMobile ? scale * 0.62 : scale;

// In resize handler — mirror the mobile FOV logic
camera.fov = isMobile ? 65 : (aspect < 0.8 ? 55 : 40);
camera.updateProjectionMatrix();
```

### Mobile CSS Block (append to `<style>`)

```css
/* ─── Responsive: mobile (<768px) ──────────────────────────────────────── */
@media (max-width: 767px) {

  /* Projects grid: 1 column (D-01, D-02) */
  .projects-grid {
    grid-template-columns: 1fr;
    gap: 4px;
  }

  /* Section titles: tighter padding */
  .section-title {
    padding: 0 1.25rem 1.5rem;
  }

  /* Content sections: reduce vertical padding, tighter horizontal */
  #clients,
  #contact {
    padding: 4rem 1.25rem;
  }

  /* Section inner: full width with padding */
  .section-inner {
    padding: 0;
  }

  /* Logo row: tighter gap on small screens */
  .logo-row {
    gap: 1.5rem;
  }

  /* Lightbox: nearly full screen */
  .lightbox-inner {
    width: 96vw;
  }

  .lightbox-close {
    top: 10px;
    right: 10px;
  }

  /* CTA button: bring in from edge */
  .cta-btn {
    left: 1rem;
    right: 1rem;
    bottom: 1.5rem;
    justify-content: center;
  }

  /* Project overlay: remove backdrop-blur (GPU cost) */
  .project-overlay {
    backdrop-filter: none;
    -webkit-backdrop-filter: none;
  }

  /* Back-to-top: reduce size */
  .back-to-top {
    width: 2.5rem;
    height: 2.5rem;
    bottom: 1.25rem;
    right: 1rem;
  }
}
```

### GitHub Pages Deployment Commands

```bash
# 1. Add GitHub remote (Tom must supply correct URL)
git remote add origin https://github.com/tombernys/portfolio-2026.git

# 2. Push main branch
git push -u origin main

# SIMPLER OPTION: Configure Pages from main branch directly
# Settings > Pages > Source: Deploy from branch > main > / (root)
# No gh-pages branch needed — index.html is already at repo root

# OR: Manual gh-pages orphan branch
git checkout --orphan gh-pages
git reset --hard
git checkout main -- index.html
touch .nojekyll
git add index.html .nojekyll
git commit -m "chore: initial GitHub Pages deployment"
git push -u origin gh-pages
git checkout main
```

---

## State of the Art

| Old Approach | Current Approach | Impact on This Phase |
|--------------|------------------|---------------------|
| Separate stylesheet file | Inline `<style>` in single HTML | All CSS changes go in `<style>` — no separate file |
| `gh-pages` npm package for deploy | Direct `git push` to branch | No new npm dependency needed |
| Complex build pipeline for static deploy | Single file, push to repo root | Main-branch Pages works with zero tooling |
| backdrop-filter not supported on Android | Supported in Chrome 76+ (Android 9+) | Safe to keep; can still remove as optimization |

---

## Open Questions

1. **What is the GitHub repo URL?**
   - What we know: No remote is currently configured (`git remote get-url origin` returns empty)
   - What's unclear: Tom has not provided the GitHub repo name/URL
   - Recommendation: Plan should include a placeholder step "configure GitHub remote" with a note that Tom must supply the URL. The deploy plan cannot complete without it.

2. **What exact tile count results from `generateQuadTree()` currently?**
   - What we know: 6×5 base grid with `splitProb = [0.55, 0.25]`, depth ≤ 2 — this is deterministic
   - What's unclear: The exact count (likely 80–200 tiles). The corner links (`buildCornerLinks`) appears commented out in the context restore path but may still be called on first init
   - Recommendation: The planner should include a task to `console.log(tiles.length)` on first run to document the baseline, then implement the mobile reduction

3. **Is `buildCornerLinks` called on first init?**
   - What we know: Line 2366 in the context-restore handler has it commented out: `// buildCornerLinks(tiles);`
   - What's unclear: Whether the original init path (not shown in the read window) also calls it or not
   - Recommendation: Search for all `buildCornerLinks` calls in `index.html` before planning the tile reduction task

4. **Antialias on mobile**
   - What we know: `antialias: true` is hardcoded in `initRenderer()` (line 2041). The STATE.md notes "antialias disabled on GPU tier 0/1" as a decision, but the code does not implement this
   - What's unclear: Whether this was deliberately deferred or accidentally omitted
   - Recommendation: Disabling antialias on mobile (`antialias: !isMobile`) in `WebGLRenderer` options reduces fill rate pressure significantly. This should be investigated and included in the RESP-03 performance plan

---

## Environment Availability

| Dependency | Required By | Available | Version | Fallback |
|------------|------------|-----------|---------|----------|
| Node.js | Vercel CLI, scripts | Yes | v24.13.0 | — |
| npm | Package management | Yes | 11.6.2 | — |
| Vercel CLI | Primary host (already deployed) | Yes | 50.9.5 | — |
| Git | GitHub Pages deploy | Yes | (system) | — |
| GitHub remote | gh-pages deployment | No | — | Tom must add remote |
| Real mobile device | RESP-02/03 verification | Unknown | — | Chrome DevTools for initial tuning; real device required for final sign-off |

**Missing dependencies with no fallback:**
- GitHub remote URL — cannot deploy to GitHub Pages without it. Plan must gate the deploy task on Tom providing the URL.
- Real physical device — Chrome DevTools emulation does not simulate GPU fill-rate. Required for RESP-02 and RESP-03 acceptance criteria ("verified by browser DevTools performance trace" on "a real mid-range Android device").

**Missing dependencies with fallback:**
- None beyond above.

---

## Validation Architecture

### Test Framework

| Property | Value |
|----------|-------|
| Framework | Manual browser testing (no automated test framework in this project) |
| Config file | None — single HTML file, no test tooling |
| Quick run command | Open `index.html` via `npx serve .` or Vercel preview URL on device |
| Full suite command | Full device matrix: desktop Chrome, mobile Chrome (Android), mobile Safari (iOS) |

No automated test framework exists or is needed for this project (single HTML file, no build tools per TECH-01). All validation is manual with DevTools.

### Phase Requirements → Test Map

| Req ID | Behavior | Test Type | Automated Command | Verification Method |
|--------|----------|-----------|-------------------|---------------------|
| RESP-01 | Layout adapts to mobile (<768px) — no horizontal overflow | Manual | `npx serve . -p 3000` + Chrome DevTools 375px | Check for `overflow-x` scroll in DevTools > Elements |
| RESP-02 | WebGL DPR max 1.5 on mobile | Manual | DevTools console: `renderer.getPixelRatio()` on mobile UA | Log output must be ≤ 1.5 |
| RESP-03 | 30fps+ on real Android | Manual | DevTools Rendering > Frame Rendering Stats on physical device | Sustained green bars in FPS meter |
| RESP-04 | 3D text legible at 375px | Manual | Physical device or DevTools 375px emulation | Visual inspection — text fills ~75–85% of viewport width |
| RESP-05 | All sections reflow single-column | Manual | DevTools 375px + scroll through page | No horizontal overflow; grid shows 1 column |

### Wave 0 Gaps
None — no test infrastructure needs to be created. Validation is via browser DevTools and physical device testing as specified in acceptance criteria.

---

## Sources

### Primary (HIGH confidence)
- `index.html` — code audit of all `isMobile` references, tile generation, camera FOV, CSS rules (lines 793–794, 1160–1210, 1920, 2053, 2155, 2382)
- `07-CONTEXT.md` — locked user decisions D-01 through D-18
- `REQUIREMENTS.md` — RESP-01 through RESP-05 acceptance criteria

### Secondary (MEDIUM confidence)
- [GitHub Docs — Configuring a publishing source for GitHub Pages](https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site) — gh-pages branch setup, `.nojekyll` requirement
- [Can I Use — backdrop-filter](https://caniuse.com/css-backdrop-filter) — Android Chrome 76+ support confirmed
- [MDN — backdrop-filter](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Properties/backdrop-filter) — -webkit- prefix no longer required in Safari 17+

### Tertiary (LOW confidence)
- WebSearch: Three.js mobile instanced geometry performance — general principles confirmed; no project-specific benchmarks
- WebSearch: FOV/scale for 375px viewport — math confirmed via Three.js docs; exact values 0.62 and 65° are estimates requiring real-device tuning

---

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH — no new dependencies; everything is already in the project
- Architecture (CSS responsive): HIGH — direct code audit, clear pattern
- Architecture (WebGL tuning): MEDIUM — scale/FOV values are estimated; require real-device validation
- Deployment: MEDIUM — GitHub remote URL unknown; deployment steps are standard but cannot be fully verified without a configured remote
- Pitfalls: HIGH — derived from direct code reading, not speculation

**Research date:** 2026-03-26
**Valid until:** 2026-04-26 (stable domain — CSS, Three.js, GitHub Pages all unlikely to change meaningfully)
