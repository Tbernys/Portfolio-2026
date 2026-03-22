# Phase 1: Scaffolding - Research

**Researched:** 2026-03-22
**Domain:** Static HTML shell, CDN import maps, Three.js setup, font conversion, semantic HTML, meta tags
**Confidence:** HIGH

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions

**Typography & Fonts**
- 3D glass text font: Display/creative style, arrondi/organique (e.g., Comfortaa, Quicksand, or similar rounded modern display font). Must be converted to typeface.json format via facetype.js for TextGeometry.
- Loader text font: Monospace/terminal style for the glitch character reveal (e.g., JetBrains Mono, Fira Code, or Space Mono via Google Fonts)
- UI body font: Inter (Google Fonts CDN) for all section titles, descriptions, navigation, metadata
- Hierarchy: Display font for "Tom Bernys" only (3D + loader), Inter for everything else

**HTML Structure (top to bottom)**
- Section order: Hero → Clients → Projects → Contact
- Hero content: "Tom Bernys" as main element + "Monteur Vidéo & Motion Designer" as subtitle
- Language: French for v1. Structure should anticipate future English translation (data attributes or similar), but no i18n system needed now

**Identity & Color Palette**
- Background: Pure black (#000000)
- Primary text: Blanc cassé (#F5F0EB) — warm off-white, not harsh pure white
- Secondary/accent: Beige sable (#D4C5B2) — warm, organic, sophisticated
- Contrast level: Subtil et clean — no excessive glow, no neon. Accents are warm and understated
- Overall vibe: Sobre, classé, chaleureux — the work is the star, the UI stays elegant and quiet

**Meta & SEO**
- Page title: "Tom Bernys | Portfolio"
- Meta description: Claude to write an optimized French description for SEO
- OG image: Placeholder for now — will need a static screenshot once the site is built
- Domain: Not yet decided — use relative paths, no hardcoded domain
- Favicon: Inline SVG data URI with initials "TB"

### Claude's Discretion
- Exact display font choice (must be rounded/organic, available on Google Fonts, convertible to typeface.json)
- Exact mono font choice for loader
- Meta description copywriting
- Favicon design details
- HTML landmark structure and ARIA attributes

### Deferred Ideas (OUT OF SCOPE)
- English translation — noted for post-v1. Structure should be translation-friendly but no i18n implementation now.
</user_constraints>

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|-----------------|
| TECH-01 | Entire site is a single index.html file with inline CSS and JS (no build tools, no framework) | Single-file HTML pattern from PROMPT_REPRODUCTION.md; all CSS/JS inline in script/style tags |
| TECH-02 | Three.js loaded via CDN import maps (jsDelivr) | Import map syntax verified via threejs.org docs; exact v0.175.0 URL pattern confirmed |
| TECH-03 | Site deploys on any static host (GitHub Pages, Vercel, Netlify) | All three support zero-configuration static HTML; no build step required; relative paths only |
</phase_requirements>

---

## Summary

This phase establishes the deployable HTML shell before any WebGL rendering code is written. The core technical challenge is proving the CDN import map works on the production host — ES modules loaded via import maps are blocked in `file://` protocol but work correctly when served over HTTP/HTTPS. The entire project uses a single `index.html` with all CSS and JS inline, following the pattern proven in PROMPT_REPRODUCTION.md.

Three.js r175 (npm `0.175.0`) is the pinned version. STATE.md documents the decision to use r175 rather than the latest (currently r183): r183 renamed `PostProcessing` to `RenderPipeline` and deprecated `Clock` in favor of `Timer`, which would require migration work before the project's glass shader and bloom pipeline are even built. r175 is the WebGL 2 baseline, stable, and all addons (`TextGeometry`, `FontLoader`) are importable from the `three/addons/` path alias.

Font conversion is a Phase 1 deliverable and a blocker for Phase 3. The display font (rounded organic, e.g., Comfortaa) must be downloaded as a TTF from Google Fonts and converted to `typeface.json` using facetype.js or Vextrude. The resulting JSON must be embedded inline in the HTML (as a JS variable or fetched from a relative path) since TECH-01 prohibits external asset files beyond CDN-loaded libraries.

**Primary recommendation:** Build the index.html with the import map, a test Three.js module (cube or sphere), semantic HTML shell with visually-hidden text, Google Fonts link tags, inline SVG favicon, and OG/Twitter meta tags — then deploy to Vercel or Netlify to confirm the import map resolves correctly in production before touching any glass shader or TextGeometry code.

---

## Standard Stack

### Core
| Library | Version | Purpose | Why Standard |
|---------|---------|---------|--------------|
| Three.js | 0.175.0 | WebGL rendering engine | Project decision; avoids r182/r183 API renames; stable WebGL2 baseline |
| (none) | — | No build tools, no framework | TECH-01 constraint; single index.html approach |

### Supporting
| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| Inter | via Google Fonts CDN | UI body font (all text except "Tom Bernys" 3D) | Locked decision; load via `<link>` tag |
| Comfortaa (recommended) | via Google Fonts CDN + TTF download | Display font for "Tom Bernys" (3D geometry + loader) | Rounded/organic, available on Google Fonts, confirmed TTF downloadable for facetype.js conversion |
| JetBrains Mono (recommended) | via Google Fonts CDN | Monospace font for glitch loader text | Available on Google Fonts; clean terminal aesthetic; CSS discretion |

### Alternatives Considered
| Instead of | Could Use | Tradeoff |
|------------|-----------|----------|
| Comfortaa | Quicksand | Quicksand is slightly more neutral; Comfortaa has stronger personality and more distinctive rounded caps — better visual match for the organic/artistic brief |
| JetBrains Mono | Space Mono | Space Mono is more geometric/retro; JetBrains Mono is more refined; either works — Claude's discretion |
| facetype.js (browser tool) | Vextrude font converter | Both produce typeface.json; Vextrude has a cleaner UI and processes client-side so the font is never uploaded to a server; either works |

**Installation:**
```bash
# No npm install — zero dependencies
# Three.js loaded via CDN import map in HTML
# Fonts loaded via Google Fonts <link> tags
# typeface.json produced offline via browser tool, then inlined/hosted as a file
```

---

## Architecture Patterns

### Recommended Project Structure
```
PORTFOLIO 2026/
├── index.html           # Single file — all CSS and JS inline
├── fonts/
│   └── comfortaa.typeface.json   # Converted display font (TECH-01 allows separate font asset)
└── .planning/           # GSD planning files (not deployed)
```

**Note on TECH-01:** "Single index.html with inline CSS and JS" means no separate `.js` or `.css` files. However the typeface.json font file is a data asset, not a code file. It is acceptable to host it as a sibling file and load via `FontLoader.load('./fonts/comfortaa.typeface.json')`. Alternatively it can be inlined as a JS object literal inside the `<script type="module">` tag (adds ~50-150KB to HTML but removes the HTTP round-trip and avoids any CORS concern). Decision is left to the planner — inline is simpler for Phase 1, external file is cleaner for Phase 3.

### Pattern 1: Import Map + ES Module Script
**What:** A `<script type="importmap">` block before any module scripts maps bare specifiers to CDN URLs. Then a `<script type="module">` block uses standard ES import syntax.
**When to use:** Always — required for Three.js CDN usage with addons.

```html
<!-- Source: https://threejs.org/manual/en/installation.html -->
<script type="importmap">
{
  "imports": {
    "three": "https://cdn.jsdelivr.net/npm/three@0.175.0/build/three.module.js",
    "three/addons/": "https://cdn.jsdelivr.net/npm/three@0.175.0/examples/jsm/"
  }
}
</script>

<script type="module">
  import * as THREE from 'three';
  import { TextGeometry } from 'three/addons/geometries/TextGeometry.js';
  import { FontLoader } from 'three/addons/loaders/FontLoader.js';
  // All application code goes here — inline in the HTML
</script>
```

**CRITICAL:** The import map must appear before any `<script type="module">` tags. The browser processes import maps before resolving module specifiers. Placing it after a module script causes a parse error.

### Pattern 2: Semantic HTML Shell with Visually-Hidden Content
**What:** The WebGL canvas is cosmetic. Semantic HTML provides the accessible content layer that search crawlers and screen readers consume.
**When to use:** Required — success criteria item 3 requires a crawler to see semantic landmarks.

```html
<!-- Source: https://www.a11yproject.com/posts/how-to-hide-content/ -->
<style>
  .sr-only {
    position: absolute;
    width: 1px;
    height: 1px;
    padding: 0;
    margin: -1px;
    overflow: hidden;
    clip: rect(0, 0, 0, 0);
    clip-path: inset(50%);
    white-space: nowrap;
    border: 0;
  }
</style>

<body>
  <canvas id="canvas"></canvas>

  <main>
    <section id="hero" aria-label="Hero">
      <h1>Tom Bernys</h1>
      <p class="sr-only">Monteur Vidéo &amp; Motion Designer</p>
    </section>

    <section id="clients" aria-label="Clients">
      <h2 class="sr-only">Clients</h2>
    </section>

    <section id="projects" aria-label="Projets">
      <h2 class="sr-only">Projets</h2>
    </section>

    <section id="contact" aria-label="Contact">
      <h2 class="sr-only">Contact</h2>
    </section>
  </main>
</body>
```

### Pattern 3: Open Graph + Twitter Card Meta Block
**What:** A complete set of social preview meta tags in the `<head>`.
**When to use:** Required — success criteria item 4 requires correct link previews.

```html
<!-- Source: https://ogp.me/ -->
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Tom Bernys | Portfolio</title>
  <meta name="description" content="Tom Bernys — Monteur Vidéo &amp; Motion Designer. Découvrez ses réalisations pour des marques exigeantes.">

  <!-- Open Graph -->
  <meta property="og:type" content="website">
  <meta property="og:title" content="Tom Bernys | Portfolio">
  <meta property="og:description" content="Tom Bernys — Monteur Vidéo &amp; Motion Designer. Découvrez ses réalisations pour des marques exigeantes.">
  <meta property="og:image" content="og-image.jpg">

  <!-- Twitter Card -->
  <meta name="twitter:card" content="summary_large_image">
  <meta name="twitter:title" content="Tom Bernys | Portfolio">
  <meta name="twitter:description" content="Tom Bernys — Monteur Vidéo &amp; Motion Designer. Découvrez ses réalisations pour des marques exigeantes.">
  <meta name="twitter:image" content="og-image.jpg">

  <!-- Favicon: inline SVG with "TB" initials -->
  <link rel="icon" href="data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 100 100'><rect width='100' height='100' rx='20' fill='%23F5F0EB'/><text x='50' y='72' font-family='sans-serif' font-size='52' font-weight='700' text-anchor='middle' fill='%23000000'>TB</text></svg>">
</head>
```

**Note on og:image placeholder:** Use a relative path `og-image.jpg` for now. Slack/iMessage debuggers cache previews — test with the validator tools after the OG image is set, not before. For Phase 1, a placeholder 1200×630 solid-color image is sufficient to prove the meta tags are wired correctly.

### Pattern 4: Google Fonts Loading
**What:** Load Inter and the display font via `<link>` tag. Place before all other styles.
**When to use:** Required for UI fonts.

```html
<!-- Source: https://fonts.google.com -->
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Comfortaa:wght@700&family=Inter:wght@100..900&family=JetBrains+Mono:wght@400;700&display=swap" rel="stylesheet">
```

### Pattern 5: Canvas Layering (WebGL behind DOM)
**What:** The canvas is `position: fixed; inset: 0; z-index: 0`. The content overlay is `position: relative; z-index: 1`.

```css
/* Canvas sits behind all DOM content */
#canvas {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 0;
}

body {
  background: #000000;
  color: #F5F0EB;
  margin: 0;
}

main {
  position: relative;
  z-index: 1;
  pointer-events: none; /* Let mouse events pass through to canvas by default */
}

/* Re-enable pointer events on interactive elements */
main a,
main button,
main form,
main input,
main textarea {
  pointer-events: auto;
}
```

### Anti-Patterns to Avoid
- **Placing the import map after a module script:** The browser will throw a `TypeError: Cannot resolve module specifier` because module resolution happens before the import map is parsed.
- **Mixing Three.js versions in the same import map:** E.g., `"three"` pointing to r175 but an addon URL pointing to r170 causes duplicate class registrations and instanceof failures.
- **Hardcoding the production domain in og:image:** Until a domain is decided, use a relative path. Absolute URLs are recommended for production OG images but only once the domain is known.
- **Loading typeface.json via fetch from file:// protocol:** ES module scripts and fetch both fail under `file://`. Development requires a local HTTP server (e.g., `python3 -m http.server 8080`).
- **Using `display: none` for the canvas-covering h1:** This removes it from the accessibility tree. Use `sr-only` / `.visually-hidden` instead to keep it discoverable by crawlers and screen readers.

---

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Font → 3D geometry | Custom path parser | facetype.js or Vextrude → typeface.json | Glyph path triangulation is complex; existing tools handle bézier-to-polygon conversion correctly |
| Three.js CDN loading | Custom module resolver | Standard `<script type="importmap">` | Browser-native; handles version pinning, consistent with Three.js docs |
| Monospace font metrics | Custom CSS | Google Fonts CDN `<link>` | Subsetting, FOUT handling, caching all managed |
| SVG favicon | External favicon.ico | Inline `data:image/svg+xml` in `<link href>` | Single file requirement; SVG favicons have full cross-browser support as of 2023 |
| OG image generation | Canvas API rendering | Static placeholder image | Dynamic OG images are a Phase 2+ concern; static JPEG is sufficient for Phase 1 validation |

**Key insight:** The single-file constraint (TECH-01) makes the import map + inline script pattern non-negotiable — there is no alternative module loading mechanism that works cross-browser without a bundler.

---

## Common Pitfalls

### Pitfall 1: ES Modules Blocked Under file:// Protocol
**What goes wrong:** Opening index.html directly in the browser (double-click) shows a blank canvas and console errors like `Cross-origin request blocked` or `Failed to resolve module specifier "three"`.
**Why it happens:** ES module imports and the import map require HTTP/HTTPS. The `file://` protocol is blocked by CORS policies.
**How to avoid:** Always develop with a local server: `python3 -m http.server 8080` or `npx serve .` or VS Code Live Server extension.
**Warning signs:** Console shows `TypeError: Failed to fetch dynamically imported module` when loading from disk.

### Pitfall 2: Import Map Ordering Error
**What goes wrong:** Console error: `Cannot set import map after one or more module scripts have already begun loading`.
**Why it happens:** Any `<script type="module">` that appears before the `<script type="importmap">` starts loading immediately — by the time the import map is parsed, module resolution has already started.
**How to avoid:** Place the `<script type="importmap">` as the FIRST script tag in `<head>`, before any `<script type="module">` or `<link rel="modulepreload">` tags.
**Warning signs:** The error appears in the browser console on first load, before any application code runs.

### Pitfall 3: FontLoader / TextGeometry Not Found in Three Core
**What goes wrong:** `import { FontLoader } from 'three'` throws `SyntaxError: The requested module does not provide an export named 'FontLoader'`.
**Why it happens:** FontLoader and TextGeometry were moved out of the core `three` package and into addons many versions ago. They must be imported from the addons path.
**How to avoid:** Always import from `'three/addons/loaders/FontLoader.js'` and `'three/addons/geometries/TextGeometry.js'`.
**Warning signs:** Import from `'three'` works for `THREE.WebGLRenderer` but throws for FontLoader/TextGeometry.

### Pitfall 4: typeface.json Needs HTTP to Load (same as Pitfall 1)
**What goes wrong:** `FontLoader.load('./fonts/comfortaa.typeface.json')` silently fails or throws a CORS error when opened from `file://`.
**Why it happens:** FontLoader uses `fetch()` internally; blocked under file protocol.
**How to avoid:** Same as Pitfall 1 — use a local HTTP server.
**Warning signs:** The font load callback is never called; Three.js logs a network error.

### Pitfall 5: Vercel / Netlify Serving index.html Without MIME Type Issues
**What goes wrong:** On some hosts, `.js` modules may be served with the wrong MIME type and blocked.
**Why it happens:** Static hosts generally serve files correctly, but the CDN modules (Three.js) are already served from jsDelivr with correct MIME types, so this is not a risk for the CDN import map approach.
**How to avoid:** No action needed — Three.js loads entirely from jsDelivr which sets `Content-Type: application/javascript`. Only a concern if hosting your own `.js` files.
**Warning signs:** Console error `Failed to load module script: Expected a JavaScript module script but the server responded with a MIME type of "text/plain"`.

### Pitfall 6: Outdated Three.js Forum Answers Using Wrong Import Paths
**What goes wrong:** Following tutorial code that imports from `'three/examples/jsm/...'` instead of `'three/addons/...'`.
**Why it happens:** The alias `three/addons/` was introduced in r148 as a cleaner path. Many tutorials and Stack Overflow answers predate this and still show the old path.
**How to avoid:** Use `three/addons/` exclusively. Both paths currently resolve to the same files on CDN, but `three/addons/` is the canonical path going forward.
**Warning signs:** Tutorial uses `examples/jsm/` in import paths.

---

## Code Examples

Verified patterns from official sources:

### Minimal Phase 1 Proof-of-Concept Structure
```html
<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Tom Bernys | Portfolio</title>

  <!-- Source: https://threejs.org/manual/en/installation.html -->
  <script type="importmap">
  {
    "imports": {
      "three": "https://cdn.jsdelivr.net/npm/three@0.175.0/build/three.module.js",
      "three/addons/": "https://cdn.jsdelivr.net/npm/three@0.175.0/examples/jsm/"
    }
  }
  </script>

  <!-- Google Fonts -->
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Comfortaa:wght@700&family=Inter:wght@100..900&family=JetBrains+Mono:wght@400;700&display=swap" rel="stylesheet">

  <!-- Favicon: inline SVG with TB initials -->
  <link rel="icon" href="data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 100 100'><rect width='100' height='100' rx='20' fill='%23F5F0EB'/><text x='50' y='72' font-family='sans-serif' font-size='52' font-weight='700' text-anchor='middle' fill='%23000'>TB</text></svg>">

  <!-- Meta / SEO -->
  <meta name="description" content="Tom Bernys — Monteur Vidéo &amp; Motion Designer. Découvrez ses réalisations pour des marques exigeantes.">
  <meta property="og:type" content="website">
  <meta property="og:title" content="Tom Bernys | Portfolio">
  <meta property="og:description" content="Tom Bernys — Monteur Vidéo &amp; Motion Designer. Découvrez ses réalisations pour des marques exigeantes.">
  <meta property="og:image" content="og-image.jpg">
  <meta name="twitter:card" content="summary_large_image">
  <meta name="twitter:title" content="Tom Bernys | Portfolio">
  <meta name="twitter:description" content="Tom Bernys — Monteur Vidéo &amp; Motion Designer. Découvrez ses réalisations pour des marques exigeantes.">
  <meta name="twitter:image" content="og-image.jpg">

  <style>
    *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
    body { background: #000; color: #F5F0EB; font-family: 'Inter', sans-serif; }
    canvas { position: fixed; top: 0; left: 0; width: 100%; height: 100%; z-index: 0; }
    main { position: relative; z-index: 1; pointer-events: none; }
    .sr-only {
      position: absolute; width: 1px; height: 1px; padding: 0;
      margin: -1px; overflow: hidden; clip: rect(0 0 0 0);
      clip-path: inset(50%); white-space: nowrap; border: 0;
    }
  </style>
</head>
<body>
  <canvas id="canvas"></canvas>

  <main>
    <section id="hero" aria-label="Hero">
      <h1>Tom Bernys</h1>
      <p>Monteur Vidéo &amp; Motion Designer</p>
    </section>
    <section id="clients" aria-label="Clients"><h2 class="sr-only">Clients</h2></section>
    <section id="projects" aria-label="Projets"><h2 class="sr-only">Projets</h2></section>
    <section id="contact" aria-label="Contact"><h2 class="sr-only">Contact</h2></section>
  </main>

  <script type="module">
    // Source: https://threejs.org/manual/en/installation.html
    import * as THREE from 'three';

    const canvas = document.getElementById('canvas');
    const renderer = new THREE.WebGLRenderer({ canvas, antialias: true });
    renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
    renderer.setSize(window.innerWidth, window.innerHeight);

    const scene = new THREE.Scene();
    const camera = new THREE.PerspectiveCamera(40, window.innerWidth / window.innerHeight, 0.1, 100);
    camera.position.z = 5;

    // Phase 1 test: confirm import map works — simple cube
    const geometry = new THREE.BoxGeometry(1, 1, 1);
    const material = new THREE.MeshStandardMaterial({ color: 0xF5F0EB });
    const cube = scene.add(new THREE.Mesh(geometry, material));
    scene.add(new THREE.AmbientLight(0xffffff, 1));
    scene.add(new THREE.DirectionalLight(0xffffff, 2));

    function animate() {
      requestAnimationFrame(animate);
      scene.children[0].rotation.x += 0.01;
      scene.children[0].rotation.y += 0.01;
      renderer.render(scene, camera);
    }
    animate();

    window.addEventListener('resize', () => {
      camera.aspect = window.innerWidth / window.innerHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(window.innerWidth, window.innerHeight);
    });
  </script>
</body>
</html>
```

### Font Conversion Workflow
```
1. Go to: https://fonts.google.com/specimen/Comfortaa
2. Click "Download family" → extract ZIP → find Comfortaa-Bold.ttf
3. Go to: https://gero3.github.io/facetype.js/  (or https://vextrude.com/font_converter)
4. Upload Comfortaa-Bold.ttf
5. Enable "Restrict to basic Latin" to reduce file size (~50KB vs ~1MB+)
6. Download → comfortaa_bold.typeface.json
7. Place in ./fonts/ directory alongside index.html
8. Load in Phase 3 via:
     import { FontLoader } from 'three/addons/loaders/FontLoader.js';
     const loader = new FontLoader();
     loader.load('./fonts/comfortaa_bold.typeface.json', (font) => { ... });
```

### Inline SVG Favicon Pattern
```html
<!-- Source: https://austingil.com/svg-favicons/ -->
<!-- URL-encode # as %23, space as %20 -->
<link rel="icon" href="data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 100 100'><rect width='100' height='100' rx='20' fill='%23F5F0EB'/><text x='50' y='72' font-family='sans-serif' font-size='52' font-weight='700' text-anchor='middle' fill='%23000000'>TB</text></svg>">
```

---

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| `<script src="three.min.js">` (global) | `<script type="importmap">` + `import * as THREE from 'three'` | ~r148 (2022) | Clean module imports, no global pollution, addons import naturally |
| `import { FontLoader } from 'three/examples/jsm/loaders/FontLoader.js'` | `import { FontLoader } from 'three/addons/loaders/FontLoader.js'` | ~r148 | Canonical alias; both resolve on CDN but `three/addons/` is forward-compatible |
| Separate HTML/CSS/JS files with bundler | Single index.html with inline CSS+JS | Project decision (TECH-01) | Zero build toolchain; deploy anywhere; PROMPT_REPRODUCTION.md pattern |
| Six favicon files (ico, png, apple-touch...) | Single SVG favicon via data URI | Browser support matured ~2023 | One tag, no external file, works in Chrome/Firefox/Safari/Edge |

**Deprecated/outdated:**
- `import { FontLoader } from 'three'` (core): Removed from core many versions ago — always use addons path
- Three.js r182+: Renamed `PostProcessing` → `RenderPipeline`, deprecated `Clock` — project deliberately stays at r175 to avoid these
- `require()` / CommonJS: Not usable in browser ES modules without a bundler

---

## Open Questions

1. **typeface.json hosting strategy: inline vs separate file**
   - What we know: TECH-01 says "single index.html with inline CSS and JS." typeface.json is data, not JS/CSS code.
   - What's unclear: Whether to inline the JSON as a JS object literal inside `<script type="module">` (simpler, larger HTML) or serve it as `./fonts/comfortaa_bold.typeface.json` (cleaner, but adds a second HTTP request and a second file to deploy).
   - Recommendation: Use a separate `./fonts/` directory. This is consistent with PROMPT_REPRODUCTION.md's approach (`three/addons/fonts/helvetiker_bold.typeface.json` is loaded from the CDN). A `./fonts/` sibling directory doesn't violate TECH-01's spirit (no build tools, no framework) and is significantly simpler for Phase 3 when modifying font parameters.

2. **Exact meta description copy**
   - What we know: Claude's discretion; must be French, SEO-optimized for Tom Bernys as "Monteur Vidéo & Motion Designer"
   - What's unclear: Target keywords — should the description mention specific clients, or stay general?
   - Recommendation: Write a general 155-character French description covering role + quality signal. E.g.: "Tom Bernys, Monteur Vidéo & Motion Designer freelance. Réalisations pour des marques exigeantes — direction artistique, post-production et motion design."

3. **OG image placeholder strategy**
   - What we know: Phase 1 success criterion 4 requires OG tags to "render a correct link preview." The image itself is a placeholder.
   - What's unclear: What placeholder to use — a blank black image, a text-only design, or reference to a future screenshot?
   - Recommendation: Create a minimal 1200×630 black PNG with "Tom Bernys | Portfolio" text in white. This satisfies the validator tools and shows a real preview. Can be replaced with a real screenshot in a later phase.

4. **Display font: Comfortaa vs Quicksand**
   - What we know: Both are on Google Fonts, both are rounded/organic, both are available as TTF for facetype.js conversion
   - What's unclear: Which converts better at TextGeometry extrusion sizes (bevel rendering quality depends on font path complexity)
   - Recommendation: Comfortaa. It has stronger rounded terminals and distinctive organic character that matches "chaleureux, classé" better than Quicksand's more neutral geometry. Comfortaa Bold (weight 700) is available and converts via facetype.js.

---

## Sources

### Primary (HIGH confidence)
- `https://threejs.org/manual/en/installation.html` — Import map syntax, CDN URL pattern, version placeholder guidance
- `https://www.jsdelivr.com/package/npm/three` — Verified Three.js 0.175.0 exists; latest is 0.183.2 (confirmed r175 is not latest)
- `https://github.com/mrdoob/three.js/wiki/Migration-Guide` — r174→r175 and r182→r183 breaking changes confirmed
- `https://caniuse.com/import-maps` — Import map browser support: all modern browsers since 2023

### Secondary (MEDIUM confidence)
- `https://gero3.github.io/facetype.js/` — Font conversion tool; output format verified; input accepts TTF/OTF
- `https://vextrude.com/blog-font-converter-tutorial` — Alternative converter; confirmed client-side processing, same typeface.json output
- `https://austingil.com/svg-favicons/` — Inline SVG data URI favicon syntax verified
- `https://www.a11yproject.com/posts/how-to-hide-content/` — `.sr-only` CSS pattern verified
- `https://sbcode.net/threejs/importmap/` — Single HTML file import map + module script structure confirmed

### Tertiary (LOW confidence)
- WebSearch results for Comfortaa vs Quicksand suitability for TextGeometry — based on general font aesthetic assessment, not tested conversion output. Validate by attempting conversion of both in Phase 1.

---

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH — Three.js version pinned in STATE.md; CDN URLs verified against jsDelivr; import map syntax confirmed via official docs
- Architecture: HIGH — Pattern proven in PROMPT_REPRODUCTION.md; single-file approach fully documented
- Font conversion: MEDIUM — Tools verified; exact Comfortaa conversion quality untested until Phase 1 execution
- Pitfalls: HIGH — file:// ES module restriction is documented behavior; import map ordering is spec-defined; FontLoader addon path confirmed

**Research date:** 2026-03-22
**Valid until:** 2026-04-22 (Three.js minor versions release monthly; import map spec is stable; CDN URLs are stable at pinned version)
