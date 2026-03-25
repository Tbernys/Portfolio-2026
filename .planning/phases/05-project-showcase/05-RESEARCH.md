# Phase 5: Project Showcase - Research

**Researched:** 2026-03-25
**Domain:** Vimeo embed lazy-loading, CSS grid card layout, glassmorphism overlay, modal lightbox, mobile touch interactions
**Confidence:** HIGH (core patterns verified; Vimeo account tier caveat noted)

---

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions

**Layout (grille)**
- D-01: Grille 3 colonnes — même sur mobile (pas de reflow en 1 colonne)
- D-02: 9 projets au total (3 rangées de 3)
- D-03: Ratio 16:9 (cinématique) pour chaque carte
- D-04: Espacement serré entre les cartes (8-12px gap)
- D-05: Pleine largeur (bord à bord, pas de max-width centré)
- D-06: Angles droits sur les cartes (pas d'arrondi) — cohérent avec l'esthétique CRT/glitch
- D-07: Titre de section visible ("Projets" ou similaire) au-dessus de la grille

**Thumbnail & Player**
- D-08: Thumbnail par défaut récupérée automatiquement via l'API Vimeo (oEmbed)
- D-09: Au hover: la thumbnail est remplacée par une preview vidéo Vimeo (muted, autoplay) — effet "Netflix"
- D-10: Au clic: ouverture d'une modal lightbox in-page avec le player Vimeo agrandi
- D-11: La lightbox affiche le player + métadonnées (titre, rôle, client) en dessous du player
- D-12: Fermeture de la lightbox: bouton X + clic sur le fond sombre (pas de touche Escape mentionnée)

**Hover interaction**
- D-13: Overlay glassmorphism (bande floutée backdrop-blur) en bas de la carte au hover — cohérent avec le bouton CTA glassmorphism de Phase 4
- D-14: L'overlay affiche le titre du projet et le rôle (pas le client)
- D-15: Sur mobile (pas de hover): premier tap révèle l'overlay, deuxième tap ouvre la lightbox

**Métadonnées projet**
- D-16: Informations par projet: titre, rôle(s)/crédits, client/marque (pas d'année)
- D-17: Titre + rôle visibles dans l'overlay glassmorphism au hover
- D-18: Client visible uniquement dans la lightbox (pas dans l'overlay)
- D-19: Dans la lightbox, les métadonnées sont affichées sous le player

### Claude's Discretion
- Taille et police du titre de section
- Hauteur exacte de la bande glassmorphism
- Animation d'ouverture/fermeture de la lightbox
- Style du bouton X de fermeture
- Taille du player dans la lightbox
- Traitement de la preview vidéo au hover (délai avant chargement, transition)
- Espacement exact dans le gap (entre 8-12px)
- Ajout de la touche Escape pour fermer la lightbox (bonne pratique a11y)

### Deferred Ideas (OUT OF SCOPE)
None — discussion stayed within phase scope
</user_constraints>

---

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|------------------|
| PROJ-01 | Visitor sees a showcase section displaying 6-10 projects with titles and thumbnails | CSS Grid 3-col, oEmbed thumbnail fetch, data-driven project array |
| PROJ-02 | Each project embeds a Vimeo player that loads lazily (IntersectionObserver) | IntersectionObserver pattern (existing in codebase), @vimeo/player ES module, data-vimeo-defer |
| PROJ-03 | Project thumbnails/cards have hover animation (title reveal, scale, or color shift) | Glassmorphism overlay CSS, CSS transition on hover, mobile tap-state pattern |
| PROJ-04 | Each project displays role/credits metadata (direction, motion, editing, etc.) | Data object per project, overlay shows title+role, lightbox shows title+role+client |
</phase_requirements>

---

## Summary

Phase 5 builds the projects grid inside the existing `#projects` section placeholder (index.html line 375). All work is inline CSS and inline JS in a single HTML file — no build tools, no framework. The existing IntersectionObserver, GSAP, and glassmorphism patterns from Phases 2-4 are directly reusable.

The three key technical challenges are: (1) lazy-loading Vimeo thumbnails via oEmbed at page startup (fire-and-forget `fetch` calls, CORS is open), (2) swapping a static thumbnail for a muted autoplay Vimeo iframe on hover to achieve the "Netflix" preview effect, and (3) a lightbox modal that creates a full Vimeo player on click and destroys it on close to avoid background audio leaks.

**Critical account tier caveat:** `background=1` (hides controls + forces autoplay/mute/loop) and `controls=0` (hides UI) require a Vimeo Starter+ paid account. The hover-preview approach must use `autoplay=1&muted=1&loop=1&title=0&byline=0&portrait=0` on the iframe src instead — this works on free accounts and achieves the same visual effect (Vimeo controls briefly appear but the card is small enough that they are minimally distracting). Tom should confirm his Vimeo plan tier before implementation so the plan can use `background=1` if available.

**Primary recommendation:** Build project data as a JS array, generate cards via `innerHTML`, fetch thumbnails via oEmbed in parallel (`Promise.all`), use IntersectionObserver (extend existing pattern) to trigger iframe injection on hover/viewport, and manage the lightbox with pure DOM + CSS transitions.

---

## Standard Stack

### Core

| Library | Version | Purpose | Why Standard |
|---------|---------|---------|--------------|
| @vimeo/player | 2.30.3 (verified) | Programmatic player control (play, pause, volume, events) | Official Vimeo SDK; needed for `player.pause()` on lightbox close to stop audio |
| Vanilla IntersectionObserver | Browser built-in | Lazy-load player iframes as cards scroll into view | Already used in project for section entrances; zero cost |
| GSAP 3.14.2 | Already in importmap | Lightbox open/close animation, overlay transition | Already registered; consistent with project motion language |

### Supporting

| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| Vimeo oEmbed API | REST endpoint (no SDK) | Fetch thumbnail URLs at startup | Called once per project via `fetch()` at DOMContentLoaded |
| CSS Grid + aspect-ratio | Browser built-in | 3-column 16:9 grid | Baseline 2022, 97%+ support; no polyfill needed |
| CSS backdrop-filter | Browser built-in | Glassmorphism overlay on card hover | Baseline 2024, 95%+ support; matches existing `.cta-btn` pattern |

### Alternatives Considered

| Instead of | Could Use | Tradeoff |
|------------|-----------|----------|
| @vimeo/player SDK | Raw iframe only | SDK needed for `pause()` on lightbox close — otherwise audio continues after modal dismissed |
| oEmbed fetch for thumbnails | Hardcoded thumbnail URLs | oEmbed auto-resolves even if Tom changes video; hardcoded is fragile but removes the async fetch |
| CSS Grid | CSS Flexbox | Grid enforces strict 3-column lock even on mobile (D-01); flexbox would wrap which violates the decision |

**Installation (import map addition):**
```html
"@vimeo/player": "https://cdn.jsdelivr.net/npm/@vimeo/player@2.30.3/dist/player.es.js"
```

The `player.es.js` file exports `export default Player` with no bare specifier imports — safe for browser import maps. Verified 2026-03-25.

**oEmbed endpoint (verified working, CORS open):**
```
https://vimeo.com/api/oembed.json?url=https://vimeo.com/{VIDEO_ID}
```
Returns `thumbnail_url`, `title`, `width`, `height`. `Access-Control-Allow-Origin: *` confirmed. Use with `?width=640` to request a specific thumbnail size.

---

## Architecture Patterns

### Recommended Project Structure (inline in index.html)

```
<style>
  /* Projects grid — appended to existing <style> block */
  #projects { ... }           /* override placeholder min-height */
  .projects-grid { ... }      /* CSS Grid 3-col */
  .project-card { ... }       /* aspect-ratio: 16/9, position:relative, overflow:hidden */
  .project-thumb { ... }      /* img fills card */
  .project-preview { ... }    /* iframe, hidden until hover */
  .project-overlay { ... }    /* glassmorphism band, bottom of card */
  .project-card:hover .project-overlay { ... }
  .project-card.touch-revealed .project-overlay { ... }
  .lightbox { ... }           /* fixed overlay, z-index high */
  .lightbox.is-open { ... }
  .lightbox-player-wrap { ... }
  .lightbox-meta { ... }
</style>

<section id="projects">
  <h2 class="section-title">Projets</h2>
  <div class="projects-grid">
    <!-- 9 .project-card elements, generated by JS -->
  </div>
</section>

<div class="lightbox" id="lightbox" aria-modal="true" role="dialog" hidden>
  <button class="lightbox-close" id="lightbox-close">×</button>
  <div class="lightbox-inner">
    <div class="lightbox-player-wrap" id="lightbox-player-wrap"></div>
    <div class="lightbox-meta" id="lightbox-meta"></div>
  </div>
  <div class="lightbox-backdrop" id="lightbox-backdrop"></div>
</div>
```

### Pattern 1: Project Data Array

**What:** All project metadata lives in a JS array — drives both card HTML generation and lightbox content.

**When to use:** Always. No CMS, no fetch — just edit the array to add/remove projects.

```javascript
// Source: project convention (single HTML file, no CMS)
const PROJECTS = [
  {
    id: '76979871',          // Vimeo video ID
    title: 'Nom du projet',
    role: 'Direction · Montage',
    client: 'Marque XYZ',
  },
  // ... 8 more
];
```

### Pattern 2: oEmbed Thumbnail Fetch at Startup

**What:** Fetch all 9 thumbnails in parallel at DOMContentLoaded, then assign to `<img>` elements.

**When to use:** Initial page load. Non-blocking — uses `Promise.all` so thumbnails resolve together.

```javascript
// Source: Vimeo oEmbed API (verified CORS open, 2026-03-25)
async function fetchThumbnails() {
  const requests = PROJECTS.map(p =>
    fetch(`https://vimeo.com/api/oembed.json?url=https://vimeo.com/${p.id}&width=640`)
      .then(r => r.json())
      .then(data => ({ id: p.id, url: data.thumbnail_url }))
      .catch(() => ({ id: p.id, url: null }))  // graceful fallback
  );
  const results = await Promise.all(requests);
  results.forEach(({ id, url }) => {
    if (!url) return;
    const img = document.querySelector(`[data-vimeo-id="${id}"] .project-thumb`);
    if (img) img.src = url;
  });
}
```

**Fallback:** If oEmbed fails (private video, network issue), card shows dark background — no broken image icon.

### Pattern 3: IntersectionObserver for Lazy Player Init

**What:** Extend the existing IntersectionObserver pattern. Do NOT create a second global observer — add a callback to the existing one or create one dedicated observer for cards.

**When to use:** Cards entering the viewport trigger a "ready for hover" state. The Vimeo iframe is NOT created until hover (or tap on mobile). The observer just marks the card as `data-near-viewport="true"` to enable hover injection.

```javascript
// Source: extends existing IntersectionObserver pattern in index.html
const cardObserver = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      entry.target.dataset.nearViewport = 'true';
      cardObserver.unobserve(entry.target); // fire once
    }
  });
}, { rootMargin: '200px' }); // pre-load 200px before visible

document.querySelectorAll('.project-card').forEach(card => cardObserver.observe(card));
```

### Pattern 4: Hover Preview — "Netflix" Iframe Swap

**What:** On `mouseenter` (desktop) or first `touchstart` (mobile), inject a muted autoplay iframe into the card if `data-near-viewport` is set. On `mouseleave`, remove the iframe.

**CRITICAL — Vimeo account tier:**
- If Tom has Starter+ plan: use `?background=1` (hides controls, forces autoplay/mute/loop — cleanest result)
- If Tom is on free plan: use `?autoplay=1&muted=1&loop=1&title=0&byline=0&portrait=0` (Vimeo controls briefly visible, but acceptable)

```javascript
// Source: Vimeo Player Parameters docs (help.vimeo.com)
function buildPreviewSrc(vimeoId, hasPaidPlan) {
  const base = `https://player.vimeo.com/video/${vimeoId}`;
  if (hasPaidPlan) return `${base}?background=1`;
  return `${base}?autoplay=1&muted=1&loop=1&title=0&byline=0&portrait=0`;
}

card.addEventListener('mouseenter', () => {
  if (!card.dataset.nearViewport) return;
  const iframe = document.createElement('iframe');
  iframe.src = buildPreviewSrc(card.dataset.vimeoId, VIMEO_PAID_PLAN);
  iframe.className = 'project-preview';
  iframe.allow = 'autoplay; fullscreen; picture-in-picture; playsinline';
  iframe.setAttribute('frameborder', '0');
  card.appendChild(iframe);
});

card.addEventListener('mouseleave', () => {
  const iframe = card.querySelector('.project-preview');
  if (iframe) iframe.remove(); // stops network request immediately
});
```

### Pattern 5: Mobile Double-Tap (D-15)

**What:** First tap toggles `.touch-revealed` class (shows overlay). Second tap opens lightbox. On tap-away, remove `.touch-revealed`.

```javascript
// Source: project decision D-15
card.addEventListener('touchstart', (e) => {
  if (!card.classList.contains('touch-revealed')) {
    e.preventDefault();
    // Close any other revealed cards
    document.querySelectorAll('.project-card.touch-revealed')
      .forEach(c => c.classList.remove('touch-revealed'));
    card.classList.add('touch-revealed');
  } else {
    openLightbox(card.dataset.vimeoId, card.dataset.index);
  }
}, { passive: false });
```

**Note:** `{ passive: false }` is needed only for the `preventDefault()` call. The canvas-scoped passive touch listener from Phase 4 is on the canvas element — no conflict here.

### Pattern 6: Lightbox — Create Player on Open, Destroy on Close

**What:** On click (desktop) or second tap (mobile), append a new `@vimeo/player` instance to `#lightbox-player-wrap` and show the lightbox. On close, call `player.pause()` then remove the iframe.

```javascript
// Source: @vimeo/player v2.30.3 README (github.com/vimeo/player.js)
import Player from '@vimeo/player';

let activePlayer = null;

async function openLightbox(vimeoId, projectIndex) {
  const proj = PROJECTS[projectIndex];
  const wrap = document.getElementById('lightbox-player-wrap');

  activePlayer = new Player(wrap, {
    id: parseInt(vimeoId),
    responsive: true,
    playsinline: true,
  });

  // Populate meta
  document.getElementById('lightbox-meta').innerHTML = `
    <p class="lightbox-title">${proj.title}</p>
    <p class="lightbox-role">${proj.role}</p>
    <p class="lightbox-client">${proj.client}</p>
  `;

  const lightbox = document.getElementById('lightbox');
  lightbox.removeAttribute('hidden');
  // Use requestAnimationFrame to trigger CSS transition
  requestAnimationFrame(() => lightbox.classList.add('is-open'));
  document.body.style.overflow = 'hidden'; // prevent background scroll
}

async function closeLightbox() {
  if (activePlayer) {
    await activePlayer.pause().catch(() => {});
    activePlayer.destroy().catch(() => {});
    activePlayer = null;
  }
  document.getElementById('lightbox-player-wrap').innerHTML = '';
  const lightbox = document.getElementById('lightbox');
  lightbox.classList.remove('is-open');
  // Wait for transition before hiding
  setTimeout(() => lightbox.setAttribute('hidden', ''), 350);
  document.body.style.overflow = '';
}
```

### Pattern 7: Glassmorphism Card Overlay (extends existing .cta-btn pattern)

```css
/* Source: extends existing .cta-btn glassmorphism — index.html line 113 */
.project-overlay {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  padding: 1rem 1.25rem 0.875rem;
  background: rgba(5, 5, 8, 0.65);      /* darker than cta-btn, more legible on video */
  backdrop-filter: blur(10px);
  -webkit-backdrop-filter: blur(10px);
  border-top: 1px solid rgba(255, 255, 255, 0.08);
  transform: translateY(100%);
  transition: transform 0.25s ease-out, opacity 0.25s ease-out;
  opacity: 0;
}

.project-card:hover .project-overlay,
.project-card.touch-revealed .project-overlay {
  transform: translateY(0);
  opacity: 1;
}
```

### Anti-Patterns to Avoid

- **Creating Player instance for hover preview:** Use a raw iframe for hover preview — the full `@vimeo/player` SDK is only needed in the lightbox where you need `pause()` control. Instantiating 9 SDK players is memory-heavy.
- **Attaching multiple IntersectionObservers per card:** One observer for all cards, `unobserve()` after first trigger.
- **`body.overflow: hidden` without cleanup:** Always restore in `closeLightbox()` — if the user closes via Escape (which we add as a11y per Claude's discretion), ensure the cleanup still runs.
- **Leaving hover iframes in DOM on lightbox open:** Remove the hover iframe before opening lightbox — two simultaneous Vimeo iframes for the same video ID causes audio bleed.
- **Not removing `hidden` attribute before adding `is-open` class:** The CSS transition won't fire on a `[hidden]` element. Remove `hidden` first, then add `is-open` on next frame.

---

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Vimeo player control (pause on close) | Custom postMessage to iframe | `@vimeo/player` SDK `.pause()` | Player SDK wraps postMessage with promise API, handles timing, cross-browser |
| Thumbnail fetching | Screen-scraping Vimeo HTML | oEmbed API `vimeo.com/api/oembed.json` | Official endpoint, CORS open, stable |
| 16:9 aspect ratio enforcement | JS-calculated heights | CSS `aspect-ratio: 16 / 9` | Browser-native, no layout shift, no resize listener needed |
| Scroll-triggered lazy load | `scroll` event listener | `IntersectionObserver` | Zero scroll jank, already used in project |

**Key insight:** The Vimeo oEmbed API is a simple `fetch` — no auth, no SDK, no API key for public videos. Thumbnail resolution through oEmbed is more reliable than constructing vimeocdn.com URLs manually (those embed a hash that can change).

---

## Runtime State Inventory

Step 2.5: SKIPPED — This is a greenfield content phase, not a rename/refactor/migration. No runtime state to inventory.

---

## Environment Availability

Step 2.6: No external tools beyond browser APIs and CDN resources.

| Dependency | Required By | Available | Version | Fallback |
|------------|-------------|-----------|---------|----------|
| Vimeo oEmbed API (CDN) | Thumbnail fetch (PROJ-01) | ✓ (CORS open, verified) | N/A | Hardcode thumbnail URLs per project |
| @vimeo/player (jsDelivr CDN) | Lightbox player control (PROJ-02) | ✓ (player.es.js confirmed) | 2.30.3 | Raw iframe without pause control |
| IntersectionObserver | Lazy load (PROJ-02) | ✓ (browser built-in, baseline 2016) | N/A | — |
| CSS aspect-ratio | Card dimensions (PROJ-01) | ✓ (baseline 2022, 97% support) | N/A | padding-bottom: 56.25% hack |
| CSS backdrop-filter | Glassmorphism overlay (PROJ-03) | ✓ (baseline 2024, 95% support) | N/A | Semi-transparent background without blur |

**Missing dependencies with no fallback:** None.

**Vimeo account tier (action required before implementation):**
Tom must confirm whether his Vimeo account is free or Starter+. This determines which iframe parameter set is used for hover preview:
- Free plan: `?autoplay=1&muted=1&loop=1&title=0&byline=0&portrait=0` (controls visible but small)
- Starter+ plan: `?background=1` (controls hidden, pure background video — cleanest)

---

## Common Pitfalls

### Pitfall 1: Hover Iframe Left Active When Lightbox Opens
**What goes wrong:** User hovers a card, preview iframe starts playing, then clicks to open lightbox. Both iframes are now active — background audio from the card preview continues behind the lightbox.
**Why it happens:** The `mouseenter` iframe is in the card DOM; `click` event opens lightbox without cleaning up.
**How to avoid:** In `openLightbox()`, remove any `.project-preview` iframe from the clicked card before creating the lightbox player.
**Warning signs:** Hearing two audio sources when lightbox is open.

### Pitfall 2: Vimeo oEmbed Fails Silently for Private Videos
**What goes wrong:** If a Vimeo video is set to "Private" or "Password protected", oEmbed returns 403/404. The `<img>` src remains empty, showing a broken layout.
**Why it happens:** oEmbed only works for publicly accessible videos (or domain-whitelisted with Referer header).
**How to avoid:** Wrap each fetch in `.catch(() => ({ id, url: null }))`. When `url` is null, leave the card with the dark `#050508` background — visually consistent with the grid aesthetic.
**Warning signs:** Any 403 errors in the network panel during dev.

### Pitfall 3: `player.destroy()` Called After iframe Already Removed
**What goes wrong:** `player.destroy()` fails if the iframe was already removed from DOM (e.g., by quickly opening/closing lightbox). Throws an uncaught promise rejection.
**Why it happens:** `destroy()` posts a message to the iframe's contentWindow. If the iframe is gone, this throws.
**How to avoid:** Always call `player.pause()` and `player.destroy()` before `innerHTML = ''`. Or catch the rejection: `activePlayer.destroy().catch(() => {})`.
**Warning signs:** Uncaught promise rejections in console.

### Pitfall 4: body.overflow Hidden Not Restored on Page Reload / Context Loss
**What goes wrong:** If the page throws a JS error while the lightbox is open, `body.style.overflow = 'hidden'` stays set, making the page unscrollable.
**Why it happens:** Synchronous errors skip the cleanup code in `closeLightbox()`.
**How to avoid:** In `closeLightbox()`, always run the cleanup in a `finally` block, or ensure the close handler is idempotent and always restores overflow.
**Warning signs:** Page stops scrolling after dev console errors.

### Pitfall 5: CSS Grid on Mobile — 3 Columns Become Very Narrow
**What goes wrong:** 3 columns of 16:9 cards on a 375px iPhone = each card is ~119px wide × 67px tall. Title text is unreadable, overlay cramped.
**Why it happens:** D-01 locks 3 columns even on mobile. This is intentional per user decision — the "dense brute aesthetic."
**How to avoid:** Set `font-size` for overlay text to clamp to something legible at 119px width (e.g., `clamp(0.55rem, 2vw, 0.875rem)`). The overlay title should be single-line truncated with `text-overflow: ellipsis`. Accept that hover interaction gives way entirely to tap-to-reveal on mobile.
**Warning signs:** Text overflowing cards on small viewport in dev tools.

### Pitfall 6: IntersectionObserver rootMargin and Cross-Frame Issues
**What goes wrong:** If a card observer fires before the project grid has fully rendered in the DOM, `querySelectorAll('.project-card')` returns 0 elements.
**Why it happens:** Observer setup runs before card generation completes if async thumbnail fetch is awaited before card injection.
**How to avoid:** Generate all card HTML first (synchronously), then fetch thumbnails asynchronously, then set up the observer. Order: (1) generate DOM, (2) attach observer, (3) fetch thumbnails.

---

## Code Examples

### Grid CSS — 3 Columns, 16:9, Edge-to-Edge, Sharp Corners

```css
/* Source: project decisions D-01 through D-06, CSS aspect-ratio MDN */
#projects {
  min-height: unset;       /* override Phase 4 placeholder */
  display: block;
  padding: 3rem 0 4rem;
  background: #050508;
}

.section-title {
  padding: 0 2rem 2rem;
  font-family: 'Inter', sans-serif;
  font-size: clamp(1.25rem, 3vw, 2rem);
  font-weight: 300;
  letter-spacing: 0.2em;
  text-transform: uppercase;
  color: var(--color-accent);  /* #D4C5B2 */
}

.projects-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 10px;            /* within D-04 range: 8-12px */
  padding: 0;           /* D-05: edge-to-edge */
}

.project-card {
  position: relative;
  aspect-ratio: 16 / 9;  /* D-03: cinematic ratio */
  overflow: hidden;
  border-radius: 0;      /* D-06: sharp corners */
  cursor: pointer;
  background: #0a0a0d;   /* fallback if thumbnail fails */
}

.project-thumb {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
  transition: opacity 0.3s ease;
}

.project-preview {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  border: none;
  pointer-events: none;  /* hover state only — no interaction with iframe */
}
```

### Lightbox CSS

```css
/* Source: project decisions D-10 through D-12 */
.lightbox {
  position: fixed;
  inset: 0;
  z-index: 500;
  display: flex;
  align-items: center;
  justify-content: center;
  opacity: 0;
  transition: opacity 0.3s ease;
  pointer-events: none;
}

.lightbox.is-open {
  opacity: 1;
  pointer-events: auto;
}

.lightbox-backdrop {
  position: absolute;
  inset: 0;
  background: rgba(5, 5, 8, 0.92);
  backdrop-filter: blur(8px);
  -webkit-backdrop-filter: blur(8px);
  z-index: 0;
}

.lightbox-inner {
  position: relative;
  z-index: 1;
  width: min(90vw, 1200px);
  max-height: 90vh;
  overflow-y: auto;
}

.lightbox-player-wrap {
  width: 100%;
  aspect-ratio: 16 / 9;
}

/* @vimeo/player SDK injects an iframe directly into this div */
.lightbox-player-wrap iframe {
  width: 100%;
  height: 100%;
  border: none;
}

.lightbox-meta {
  padding: 1.5rem 0 1rem;
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
}

.lightbox-close {
  position: absolute;
  top: 1.5rem;
  right: 1.5rem;
  z-index: 2;
  background: rgba(255, 255, 255, 0.08);
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  border: 1px solid rgba(255, 255, 255, 0.15);
  border-radius: 50%;
  width: 2.5rem;
  height: 2.5rem;
  color: var(--color-text);
  font-size: 1.25rem;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: background 0.2s;
}
```

### Import Map Addition

```html
<!-- Add to existing importmap in index.html -->
"@vimeo/player": "https://cdn.jsdelivr.net/npm/@vimeo/player@2.30.3/dist/player.es.js"
```

```javascript
// ES module import at top of <script type="module">
import Player from '@vimeo/player';
```

---

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| `padding-bottom: 56.25%` hack for 16:9 | `aspect-ratio: 16 / 9` | CSS 2021/Baseline 2022 | Cleaner, no positioned children needed |
| `scroll` event for lazy load | `IntersectionObserver` | 2016 (mainstream ~2019) | Zero main-thread cost, already used in project |
| Vimeo oEmbed via JSONP | `fetch()` (CORS open) | oEmbed CORS policy change | Direct fetch, no library needed |
| `opacity: 0; visibility: hidden` toggle | `hidden` attribute + CSS `opacity` transition | HTML spec evolution | `hidden` is semantic; combine with transition via rAF trick |

**Deprecated/outdated:**
- `padding-bottom` aspect ratio hack: still works but `aspect-ratio` is cleaner — use `aspect-ratio` everywhere in this phase
- `<div data-vimeo-id="">` auto-initialization: `@vimeo/player` will auto-scan the DOM if not using `data-vimeo-defer` — always use `data-vimeo-defer` or programmatic init to avoid this

---

## Open Questions

1. **Tom's Vimeo plan tier**
   - What we know: `background=1` (hides controls, pure background video) requires Starter+ plan; `controls=0` also requires Starter+
   - What's unclear: Tom's current Vimeo subscription level
   - Recommendation: Plan should include a single constant `const VIMEO_PAID = false` at the top of the projects JS section — Tom sets it to `true` if he has Starter+; the rest of the code branches accordingly

2. **Hover preview delay**
   - What we know: Injecting an iframe immediately on `mouseenter` fires for any accidental mouse-through; Claude has discretion on this
   - What's unclear: Optimal delay (100-200ms debounce is standard)
   - Recommendation: Use a 150ms `setTimeout` before iframe injection; clear it on `mouseleave`. Prevents flicker on fast cursor movement.

3. **Vimeo video IDs for all 9 projects**
   - What we know: The PROJECTS array needs real Vimeo IDs to test oEmbed fetches
   - What's unclear: Tom's actual video IDs
   - Recommendation: Plan should include a Wave 0 task to populate the PROJECTS array with real IDs, or use placeholder IDs with a comment. Implementation can use a known public video (76979871) as stand-in for dev.

---

## Validation Architecture

`nyquist_validation` key is absent from `.planning/config.json` — treated as enabled. However, this project is a single HTML file with no build tools, no test runner, and no module system that can be tested in Node. Traditional automated test frameworks (Jest, Vitest, Playwright) are disproportionate for this phase.

### Test Framework

| Property | Value |
|----------|-------|
| Framework | None — single HTML file, vanilla JS, no build system |
| Config file | N/A |
| Quick run command | `open index.html` in browser (or `npx serve .` + localhost) |
| Full suite command | Manual browser checklist (see below) |

### Phase Requirements → Test Map

| Req ID | Behavior | Test Type | Automated Command | File Exists? |
|--------|----------|-----------|-------------------|-------------|
| PROJ-01 | 9 project cards visible with titles and thumbnails | Manual visual | N/A — single HTML file | N/A |
| PROJ-02 | Vimeo iframes absent on page load; appear only when card enters viewport | Manual + DevTools Network tab | N/A | N/A |
| PROJ-03 | Hover triggers overlay animation; mobile tap-reveal works | Manual — desktop hover + mobile DevTools emulation | N/A | N/A |
| PROJ-04 | Role/credits visible in overlay; client visible in lightbox | Manual visual | N/A | N/A |

### Sampling Rate

- **Per task commit:** Open `index.html` in Chrome — confirm grid renders, no console errors
- **Per wave merge:** Full manual checklist: thumbnails loaded, lazy load verified in Network tab (Preserve Log, confirm no Vimeo requests on initial load), hover preview plays, lightbox opens/closes cleanly, audio stops on close
- **Phase gate:** All 4 checklist items green before `/gsd:verify-work`

### Wave 0 Gaps

- None for test infrastructure (no framework needed).
- Action item: populate `PROJECTS` array with real Vimeo IDs before running manual tests.

---

## Sources

### Primary (HIGH confidence)
- Vimeo oEmbed API — verified via Node.js `fetch` equivalent (HTTP 200, `Access-Control-Allow-Origin: *`, `thumbnail_url` returned) — 2026-03-25
- @vimeo/player 2.30.3 `player.es.js` — confirmed via jsDelivr CDN, `export default Player`, no bare specifier imports — 2026-03-25
- `npm view @vimeo/player version` — confirmed 2.30.3 is latest — 2026-03-25
- Vimeo Help Center "About Player Parameters" — `background=1`, `controls=0` require Starter+; `autoplay`, `muted`, `loop` free on all plans — https://help.vimeo.com/hc/en-us/articles/12426260232977
- MDN `aspect-ratio` — Baseline 2022, 97%+ support
- MDN `backdrop-filter` — Baseline 2024, 95%+ support
- Existing `index.html` codebase — glassmorphism `.cta-btn` CSS, IntersectionObserver pattern, import map structure

### Secondary (MEDIUM confidence)
- Vimeo player.js GitHub README — `data-vimeo-defer`, programmatic `new Player(element, options)`, `player.pause()`, `player.destroy()` — https://github.com/vimeo/player.js/
- Vimeo Help "Background and Chromeless videos" — `background=1` behavior and plan requirements — https://help.vimeo.com/hc/en-us/articles/12426285089681

### Tertiary (LOW confidence)
- WebSearch results on hover preview patterns — aligned with official docs but not independently verified against Vimeo changelog

---

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH — @vimeo/player version verified against npm registry; ES module confirmed functional; oEmbed CORS verified by live HTTP request
- Architecture: HIGH — patterns derived from existing codebase + official Vimeo SDK docs; no experimental features
- Vimeo account tier caveat: HIGH — officially documented in Vimeo Help Center; plan-gated features clearly identified

**Research date:** 2026-03-25
**Valid until:** 2026-06-25 (stable APIs; re-verify if Vimeo changes oEmbed CORS policy)
