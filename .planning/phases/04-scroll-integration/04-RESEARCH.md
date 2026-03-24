# Phase 4: Scroll Integration - Research

**Researched:** 2026-03-24 (updated from earlier draft)
**Domain:** GSAP ScrollTrigger, WebGL-driven scroll parallax, CSS section animations, film grain
**Confidence:** HIGH

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions

**Hero Transition**
- Parallax + fade: Canvas scrolls at ~0.5x speed (parallax) while fading to invisible — creates depth
- Distance: Transition completes over 100vh (one full viewport of scroll)
- Text 3D behavior: The "Tom Bernys" glass text moves away in Z (recedes into the scene) as user scrolls — separate from the canvas parallax
- Tiles behavior: Tiles spread apart slightly during scroll — visible but smooth gap expansion between tiles, like they're dispersing
- Symmetrical return: Scrolling back up reverses the transition identically
- Mouse interaction stays active: Touch/mouse rotation of the 3D text continues working even while scrolling — both coexist

**CTA Button "Voir mon travail"**
- Position: Bottom-left of the hero section
- Style: Semi-transparent background with backdrop-blur (glassmorphism), includes downward arrow
- Action: Clicking auto-scrolls smoothly to just past the hero section
- Scroll behavior: Button stays visible for the first ~30% of scroll, then fades out
- Destination: Scrolls to just after the hero (first content section)

**Navigation**
- No fixed dot navigation — the CTA button and natural scroll are sufficient
- No scroll indicator/bar — keep the UI minimal

**Scroll Behavior**
- Free scroll — no snap between sections, continuous natural scrolling
- Scroll speed: Completely natural, no scroll hijacking
- Brief hero hold: ~0.5s brief moment where scroll feels slightly delayed before starting the hero transition — lets user see the scene
- Mobile: Identical behavior to desktop — same parallax, same fade

**Content Section Appearance**
- Animation: Fade in + slide up from below for each section
- Trigger: Animation starts when 25% of the section is visible in the viewport
- Play once: Animations only play the first time a section enters — stays visible after
- Section enters as one block: Entire section animates as a single unit (not staggered elements)
- First section special: The first content section (video grid) gets a more pronounced entrance — Claude decides the specific treatment, knowing it's a video grid

**Background**
- Behind content sections: Very dark background with subtle film grain/noise texture — cohesive with the CRT theme of the hero tiles

### Claude's Discretion
- Exact parallax speed ratio for the canvas
- Exact tile spread distance during scroll
- Specific treatment for the first section (video grid) entrance
- RAF optimization threshold for pausing rendering
- Easing curves for all scroll-driven animations
- Film grain implementation for content background

### Deferred Ideas (OUT OF SCOPE)
- Video grid layout and content — Phase 5 (Content Sections)
- Specific section content design — Phase 5+
</user_constraints>

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|-----------------|
| SCRL-01 | Visitor scrolls vertically from hero into content sections | Touch event fix (remove blanket window-level preventDefault), free scroll with no hijacking — ScrollTrigger's `scrub` with no `pin` |
| SCRL-02 | Hero WebGL canvas fades/scales as visitor scrolls down | ScrollTrigger scrub tween: CSS `transform: translateY` for parallax + opacity fade on `#canvas`; Three.js `glassGroup.position.z` and tile `uSpread` uniform driven via `onUpdate` callback |
| SCRL-03 | Fixed navigation or scroll indicator helps visitor orient | CONTEXT.md decision: no dots, no indicator — requirement fulfilled by CTA button "Voir mon travail" which is visible for first 30% of scroll and allows user to find entry point |
</phase_requirements>

---

## Summary

Phase 4 wires GSAP ScrollTrigger to the existing hero scene to create a cinematic scroll-out: the WebGL canvas parallaxes at ~0.5x speed and fades to invisible over 100vh of scroll, while the glass text recedes in Z and the tile background spreads apart. A brief "hold" effect at the very start lets the user appreciate the scene before the transition begins. Content sections below animate in on first appearance using IntersectionObserver (or ScrollTrigger one-shots). A glassmorphism CTA button at bottom-left satisfies SCRL-03.

The existing codebase already has `glassGroup` (Three.js group containing the glass text, at `position.z = 3`) and `tilesMesh` (instanced geometry with `aPos` instance attribute for cylinder UV positions). The tile spread effect requires adding a new `uScrollProgress` uniform to the tile shader — there is no pre-existing spread uniform. The canvas is already `position: fixed` at z-index 0. Three.js objects cannot be animated with `gsap.to()` directly (no DOM node), so all Three.js mutations must happen in the `onUpdate` callback of a standalone `ScrollTrigger.create()` instance.

The critical existing blocker: `window.addEventListener('touchstart', ..., { passive: false })` at lines 1906-1917 calls `e.preventDefault()` on all touches, which completely blocks native scroll on mobile. This must be converted to canvas-scoped passive listeners before any mobile scroll integration can work.

**Primary recommendation:** Use a GSAP timeline + ScrollTrigger `scrub: true` to drive CSS canvas transform (parallax Y) and opacity, plus a standalone `ScrollTrigger.create()` with `onUpdate` to drive Three.js object mutations (glassGroup Z, tile spread uniform). Fix touch handlers. Use IntersectionObserver for content section one-shot fade/slide animations. Use CSS `::before` pseudo-element with animated SVG feTurbulence for film grain on the content background.

---

## Standard Stack

### Core
| Library | Version | Purpose | Why Standard |
|---------|---------|---------|--------------|
| GSAP | 3.14.2 | Animation engine | Already in project, import map configured |
| GSAP ScrollTrigger | 3.14.2 | Scroll-to-animation bridge | Free since 3.12, industry standard, works with existing GSAP import |
| IntersectionObserver | Web API | Section fade-in triggers | Built-in browser API, no dependency, perfect for play-once pattern |

### Supporting
| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| CSS custom properties | — | Parameterize film grain, section animation timing | Single source of truth for animation values |
| SVG feTurbulence (inline data URI) | — | Film grain texture for content background | Lightweight, CSS-only, already used in loader |

### Alternatives Considered
| Instead of | Could Use | Tradeoff |
|------------|-----------|----------|
| ScrollTrigger scrub | Manual scroll event + RAF | ScrollTrigger handles resize, refresh, direction reversal, mobile edge cases automatically |
| IntersectionObserver for sections | ScrollTrigger `once: true` | IntersectionObserver is built-in, no extra plugin; ScrollTrigger is fine too but heavier |
| CSS parallax translateY | GSAP tween position | CSS transform on the canvas element is composited on GPU — smoother parallax than animating JS properties |
| SVG data URI film grain | Canvas-drawn noise | CSS pseudo-element approach zero-cost (no JS), project already uses this pattern in the loader |

**Import code (no import map changes needed):**
```javascript
import { ScrollTrigger } from 'gsap/ScrollTrigger';
gsap.registerPlugin(ScrollTrigger);
```

The existing import map `"gsap/": "https://cdn.jsdelivr.net/npm/gsap@3.14.2/"` resolves to `ScrollTrigger.js` correctly.

---

## Architecture Patterns

### Current Layout (already in place)
```
#canvas          position: fixed, z-index: 0    (WebGL, already correct)
main             position: relative, z-index: 1 (DOM content layer)
  #hero          min-height: 100vh               (scroll distance zone)
    .cta-btn     NEW — CTA button bottom-left
  #projects      min-height: 100vh               (content sections, currently empty)
  #clients       min-height: 100vh
  #contact       min-height: 100vh
```

### Pattern 1: Two-Track Scroll Architecture

**What:** Split the scroll response into two independent tracks that run in parallel:

- **Track A (CSS):** ScrollTrigger timeline with `scrub: true` — drives CSS properties on `#canvas` (translateY for parallax, opacity for fade) and `.cta-btn` (opacity fade-out). CSS transforms are GPU-composited and don't trigger layout.
- **Track B (Three.js):** Standalone `ScrollTrigger.create()` — drives Three.js mutations in `onUpdate`: `glassGroup.position.z` for text recession, tile shader `uScrollProgress` uniform for spread. Cannot use `gsap.to()` because these are not DOM objects.

**Why two tracks:** GSAP can animate DOM CSS properties natively. Three.js objects require manual `onUpdate` callbacks. Keeping them separate avoids complex coupling.

**Example:**
```javascript
// Track A: CSS-driven (canvas + CTA button)
const heroTl = gsap.timeline({
  scrollTrigger: {
    trigger: '#hero',
    start: 'top top',
    end: 'bottom top',
    scrub: true,
  }
});

// Hold for first ~8% of scroll (simulates 0.5s pause before transition)
heroTl.to({}, { duration: 0.08 });

// Parallax: canvas moves up at 0.5x scroll speed
// translateY of -50vh when hero scrolls 100vh = 0.5x ratio
heroTl.to('#canvas', {
  y: '-50vh',
  ease: 'none',
  duration: 0.92
}, 0.08);

// Fade canvas to invisible over same scroll range
heroTl.to('#canvas', {
  opacity: 0,
  ease: 'power1.in',
  duration: 0.92
}, 0.08);

// CTA button fades out at 30% of scroll (position 0.08 + 0.22 = 0.30)
heroTl.to('.cta-btn', {
  opacity: 0,
  pointerEvents: 'none',
  ease: 'power1.out',
  duration: 0.22
}, 0.08);
```

```javascript
// Track B: Three.js-driven (glassGroup Z + tile spread)
let scrollProgress = 0;
ScrollTrigger.create({
  trigger: '#hero',
  start: 'top top',
  end: 'bottom top',
  scrub: true,
  onUpdate: (self) => {
    scrollProgress = self.progress;

    // Text recedes in Z: from 3 to -2 (5 units back) over full scroll
    if (glassGroup) {
      glassGroup.position.z = 3 - scrollProgress * 5;
    }

    // Tile spread: uniform drives gap expansion in shader
    if (tilesMesh) {
      tilesMesh.material.uniforms.uScrollProgress.value = scrollProgress;
    }

    // RAF optimization: stop expensive render when canvas invisible
    if (scrollProgress > 0.95 && rafId !== null) {
      stopRAFLoop();
    } else if (scrollProgress <= 0.95 && rafId === null && !document.hidden) {
      startRAFLoop();
    }
  }
});
```

### Pattern 2: Tile Spread via Shader Uniform

**What:** Add a `uScrollProgress` uniform to the existing tile vertex shader. Use it to displace each tile outward from the cylinder center.

**When to use:** Tiles are in a single `InstancedBufferGeometry`. Individual tile transforms cannot be set per-instance via GSAP. The shader uniform is the correct layer.

**Implementation:**

In `buildTiles()`, add the uniform:
```javascript
uniforms: {
  uTime: { value: 0 },
  uFluid: { value: null },
  uNoise: { value: noiseTex },
  uMouseUv: { value: new THREE.Vector2(0.5, 0.5) },
  uScrollProgress: { value: 0 },  // NEW
},
```

In the vertex shader (after `void main()`), use `uScrollProgress` to expand the `aPos.x` coordinate away from 0.5 (cylinder center):
```glsl
// In tileVert, add uniform declaration:
uniform float uScrollProgress;

// In main(), before computing theta:
// Spread tiles outward from center as scroll progresses
// aPos.x is in [0..1] where 0.5 is center
float spreadOffset = (aPos.x - 0.5) * uScrollProgress * 0.25;
float spreadY = (aPos.y - 0.5) * uScrollProgress * 0.15;

float vertUvX = aPos.x + spreadOffset + position.x * aScale.x * GAP;
float vertUvY = aPos.y + spreadY + position.y * aScale.y * GAP;
```

The 0.25 and 0.15 values control max spread distance (Claude's discretion area). This pushes tiles outward from center, creating a dispersing-wall effect.

### Pattern 3: Brief Hold Before Transition

**What:** Use a zero-property "padding" tween at the start of the ScrollTrigger timeline. The `scrub: true` timeline treats duration values as proportions — a `duration: 0.08` empty tween at the start means the first 8% of scroll does nothing visible.

**Why:** A brief hold (the equivalent of ~0.5s at a 100vh scroll distance) lets visitors see the scene before anything moves. ScrollTrigger does not have a native `delay` for scrubbed animations. The empty-tween technique is the documented community pattern.

```javascript
heroTl.to({}, { duration: 0.08 });  // 8% of scroll = hold zone
// Then main animations at 0.08 position
```

### Pattern 4: Content Section Fade-In (IntersectionObserver, play once)

**What:** Use `IntersectionObserver` with `threshold: 0.25` to trigger CSS class additions that animate sections in. Call `observer.unobserve(entry.target)` after trigger to play-once.

**When to use:** Section entrance animations that should play exactly once and stay visible. IntersectionObserver is zero-dependency and ideal for binary "entered viewport" events (vs scroll progress).

**CSS:**
```css
.content-section {
  opacity: 0;
  transform: translateY(40px);
  transition: opacity 0.7s ease-out, transform 0.7s ease-out;
}

.content-section.is-visible {
  opacity: 1;
  transform: translateY(0);
}

/* First section (video grid): more pronounced entrance */
#projects.is-visible {
  transition: opacity 1.0s ease-out, transform 1.0s cubic-bezier(0.16, 1, 0.3, 1);
  transform: translateY(0);
}
```

**JS:**
```javascript
const sectionObserver = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      entry.target.classList.add('is-visible');
      sectionObserver.unobserve(entry.target); // play once
    }
  });
}, { threshold: 0.25 });

document.querySelectorAll('.content-section').forEach(el => {
  sectionObserver.observe(el);
});
```

**First section (video grid) special treatment:** Apply a larger `translateY(80px)` start offset plus a more aggressive `cubic-bezier(0.16, 1, 0.3, 1)` spring-like ease (iOS-style bounce deceleration). This makes the video grid feel like it snaps into view rather than just slides up.

### Pattern 5: CTA Button (Glassmorphism)

**What:** Absolutely-positioned button at bottom-left of `#hero` with backdrop-blur + semi-transparent background.

**CSS:**
```css
.cta-btn {
  position: absolute;
  bottom: 2.5rem;
  left: 2rem;
  padding: 0.75rem 1.5rem;
  background: rgba(255, 255, 255, 0.08);
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  border: 1px solid rgba(255, 255, 255, 0.15);
  border-radius: 2rem;
  color: var(--color-text);
  font-family: 'Inter', sans-serif;
  font-size: 0.875rem;
  letter-spacing: 0.03em;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 0.5rem;
  pointer-events: auto;
  transition: background 0.2s, border-color 0.2s;
}

.cta-btn:hover {
  background: rgba(255, 255, 255, 0.14);
  border-color: rgba(255, 255, 255, 0.25);
}

.cta-btn .arrow {
  display: inline-block;
  animation: bob 1.5s ease-in-out infinite;
}

@keyframes bob {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(4px); }
}
```

**HTML:**
```html
<button class="cta-btn" id="cta-btn" aria-label="Voir mon travail">
  Voir mon travail <span class="arrow">↓</span>
</button>
```

**Click handler:**
```javascript
document.getElementById('cta-btn').addEventListener('click', () => {
  const firstSection = document.querySelector('.content-section');
  if (firstSection) {
    firstSection.scrollIntoView({ behavior: 'smooth' });
  }
});
```

### Pattern 6: Film Grain on Content Background

**What:** CSS `::before` pseudo-element on `main` (or a `.content-bg` wrapper) with animated SVG feTurbulence as `background-image`. The project already uses this exact pattern in the loader.

**Why this way:** Zero JS, GPU-composited when using `will-change: transform`, negligible cost. Identical to the loader's `has-grain` class pattern already in the codebase.

**CSS:**
```css
.content-bg {
  position: relative;
  background: #050508;  /* very dark, near-black with slight blue tint */
}

.content-bg::before {
  content: '';
  position: fixed;  /* fixed so grain doesn't scroll */
  inset: 0;
  pointer-events: none;
  z-index: 2;
  opacity: 0.04;
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='200' height='200'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='200' height='200' filter='url(%23n)'/%3E%3C/svg%3E");
  background-size: 200px 200px;
  animation: grain 0.3s steps(4) infinite;
}
```

The `grain` keyframes animation is already defined in the codebase (lines 180-186 of index.html). The same keyframe can be reused.

### Anti-Patterns to Avoid

- **`gsap.to(glassGroup, { ... })` for Three.js objects:** GSAP can animate plain JS objects and DOM nodes. Three.js `Object3D` instances have positional properties but GSAP's DOM selector (`'#canvas'`) cannot reference them. Use `onUpdate` callback instead.
- **Pinning `#hero`:** Don't use `pin: true` on the hero — creates a position:fixed stacking context that breaks the parallax and feels like scroll hijacking (explicitly ruled out by user).
- **Setting `body { overflow: hidden }` to "control" scroll:** Blocks native scroll entirely on mobile. The canvas is already `position: fixed` — no body overflow control needed.
- **Animating `canvas.style.opacity` in the RAF loop:** RAF reads `window.scrollY` every frame, creating jank. ScrollTrigger's scrub is batched and optimized.
- **Using `scrub: 0` or `scrub: false` with `toggleActions`:** This plays-and-holds the animation, preventing reverse on scroll-up. The user explicitly requires symmetrical return.

---

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Scroll progress to animation | Custom `window.scroll` + RAF + `scrollY / maxScroll` | ScrollTrigger `scrub: true` + `onUpdate self.progress` | Handles resize invalidation, dynamic content, orientation change, rubber-band scroll on iOS |
| Section entrance detection | `scroll` event + `getBoundingClientRect()` loop | `IntersectionObserver` | Browser-native, runs off main thread, no polling cost |
| Smooth scroll to section | GSAP ScrollToPlugin (extra dependency) | `element.scrollIntoView({ behavior: 'smooth' })` | Native API, no plugin needed for this simple use case |
| Film grain texture | Canvas-drawn noise in JS | CSS data URI SVG feTurbulence | Already in codebase (loader), zero JS, GPU composited |

**Key insight:** ScrollTrigger handles the edge cases that custom scroll listeners always miss: iOS rubber-band overscroll, resize/reflow after font load, section height changes from Phase 5/6 content additions, and scroll direction reversal mid-animation.

---

## Common Pitfalls

### Pitfall 1: Touch preventDefault Blocking Native Scroll (Critical)
**What goes wrong:** Lines 1906-1917 in index.html add `touchstart` and `touchmove` listeners to `window` with `e.preventDefault()` and `{ passive: false }`. This completely blocks native scrolling on mobile — the page appears frozen.
**Why it happens:** Added in Phase 3 for glass text touch rotation, before scroll was needed.
**How to avoid:** Remove `e.preventDefault()` calls. Touch coordinates (`e.touches[0].clientX/Y`) are still readable without calling `preventDefault`. The glass rotation still works — we just also allow native scroll. Move listeners to `canvas` element instead of `window` as a belt-and-suspenders measure.
**Warning signs:** Desktop (mouse wheel) scrolls fine; mobile touch appears completely stuck.

### Pitfall 2: CSS translateY Parallax + Position Fixed Canvas Interaction
**What goes wrong:** The canvas is `position: fixed`. Applying CSS `transform: translateY` to a `position: fixed` element is valid CSS but the coordinate system can create visual artifacts when combined with `backdrop-filter` on child elements.
**Why it happens:** `transform` on a fixed element moves it within the viewport coordinate system — this is correct behavior for parallax.
**How to avoid:** Verify on Chrome, Firefox, and Safari. On Safari, `backdrop-filter` on siblings/children of transformed fixed elements sometimes breaks. Since the CTA button has `backdrop-filter`, test this interaction specifically on iOS Safari.
**Warning signs:** CTA button backdrop-blur disappears or the button jumps on Safari.

### Pitfall 3: glassGroup.position.z Conflicts with Entry Animation
**What goes wrong:** The Phase 3 entry animation (lines 2004-2030) animates `glassGroup.position.z` from `5` down to `3` over 2.4s. If scroll starts before this animation completes, the `onUpdate` callback will overwrite `position.z` mid-entry-animation, causing a jump.
**Why it happens:** Both systems write to `glassGroup.position.z` concurrently.
**How to avoid:** Gate the scroll-driven Z mutation on `sceneReady && entranceComplete` where `entranceComplete` is a flag set after the 2.4s entry animation finishes. During the entry animation, disable the scroll-driven Z override.
**Warning signs:** Glass text jumps or snaps on page load if user scrolls quickly.

### Pitfall 4: ScrollTrigger.refresh() After DOM Changes
**What goes wrong:** Phases 5/6 will add content to the empty `#projects`, `#clients`, `#contact` sections. ScrollTrigger calculates trigger positions on creation. Added content changes section heights, making all trigger positions stale.
**Why it happens:** ScrollTrigger caches measurements for performance.
**How to avoid:** Any Phase 5/6 plan that adds section content MUST call `ScrollTrigger.refresh()` after DOM modification. Add a comment to the scroll init code: `// Future phases: call ScrollTrigger.refresh() after adding section content`.
**Warning signs:** Canvas fade completes too early/late; section animations trigger at wrong scroll positions.

### Pitfall 5: Canvas GPU Render Cost When Invisible
**What goes wrong:** Setting `#canvas` CSS opacity to 0 via ScrollTrigger makes the canvas visually invisible but the full WebGL render pipeline (fluid sim, bloom, glass refraction) still runs on every RAF tick.
**Why it happens:** CSS opacity is composited at paint stage — JS execution is unaffected.
**How to avoid:** In the `onUpdate` callback, call `stopRAFLoop()` when `self.progress > 0.95`. Call `startRAFLoop()` when `self.progress <= 0.95`. The existing `startRAFLoop()`/`stopRAFLoop()` functions already handle this cleanly (lines 1736/1821). Also verify `document.hidden` guard (already in codebase, line 1831).
**Warning signs:** GPU usage remains at 100% even scrolled far past the hero.

### Pitfall 6: `#hero` position:relative Required for Absolutely-Positioned CTA
**What goes wrong:** The CTA button uses `position: absolute` relative to `#hero`. Currently `#hero` does not have `position: relative` set explicitly (it inherits static).
**Why it happens:** `position: absolute` resolves to the nearest positioned ancestor. With `#hero` being static, the button would position against `main` (which is `position: relative`).
**How to avoid:** Add `position: relative` to `#hero` CSS rule.

### Pitfall 7: GSAP ScrollTrigger registering before Hero is in DOM
**What goes wrong:** If `gsap.registerPlugin(ScrollTrigger)` or `ScrollTrigger.create()` runs before the hero section exists in the DOM, trigger measurements will be wrong (zero-height elements).
**Why it happens:** The main() function is async and the script runs as a module.
**How to avoid:** Register ScrollTrigger and create triggers inside (or after) the `main()` function, after `document.fonts.ready` has resolved. The DOM is already present (HTML renders before the module script evaluates), so this is only a concern if ScrollTrigger setup is placed inside a deferred callback.

---

## Code Examples

### Full Scroll Integration Setup
```javascript
// Source: GSAP ScrollTrigger official docs + project-specific integration
import { ScrollTrigger } from 'gsap/ScrollTrigger';
gsap.registerPlugin(ScrollTrigger);

// ── Module-level scroll state ──────────────────────────────────────────
let scrollProgress = 0;
let entranceComplete = false;  // Guard against Phase 3 entry anim conflict

// Set this flag after Phase 3 entry animation finishes (2.4s from transition())
setTimeout(() => { entranceComplete = true; }, 2500);

// ── Track A: CSS-driven (canvas parallax + fade, CTA fade) ─────────────
const heroTl = gsap.timeline({
  scrollTrigger: {
    trigger: '#hero',
    start: 'top top',
    end: 'bottom top',
    scrub: true,
  }
});

// Hold zone (~8% = brief pause before anything moves)
heroTl.to({}, { duration: 0.08 });

// Canvas: parallax translateY (moves up at ~0.5x scroll speed)
heroTl.to('#canvas', {
  y: '-50vh',
  ease: 'none',
  duration: 0.92
}, 0.08);

// Canvas: fade to invisible
heroTl.to('#canvas', {
  opacity: 0,
  ease: 'power2.in',
  duration: 0.92
}, 0.08);

// CTA button: fade out by 30% of scroll
heroTl.to('#cta-btn', {
  opacity: 0,
  ease: 'power1.out',
  duration: 0.22
}, 0.08);

// ── Track B: Three.js mutations (via onUpdate) ─────────────────────────
ScrollTrigger.create({
  trigger: '#hero',
  start: 'top top',
  end: 'bottom top',
  scrub: true,
  onUpdate: (self) => {
    scrollProgress = self.progress;

    // Text Z recession (guard against entry anim conflict)
    if (glassGroup && entranceComplete) {
      glassGroup.position.z = 3 - scrollProgress * 5;  // 3 → -2
    }

    // Tile spread
    if (tilesMesh) {
      tilesMesh.material.uniforms.uScrollProgress.value = scrollProgress;
    }

    // RAF pause optimization
    if (scrollProgress > 0.95 && rafId !== null) {
      stopRAFLoop();
    } else if (scrollProgress <= 0.95 && rafId === null && !document.hidden) {
      startRAFLoop();
    }
  }
});
```

### Tile Shader Spread (vertex shader addition)
```glsl
// Add to tile vertex shader uniform block:
uniform float uScrollProgress;

// In void main(), REPLACE the existing vertUvX/vertUvY lines with:
float spreadX = (aPos.x - 0.5) * uScrollProgress * 0.3;
float spreadY = (aPos.y - 0.5) * uScrollProgress * 0.18;

float vertUvX = aPos.x + spreadX + position.x * aScale.x * GAP;
float vertUvY = aPos.y + spreadY + position.y * aScale.y * GAP;
float theta = (vertUvX - 0.5) * PI;
```

Note: `aPos` is in `[0..1]` range (cylinder UV coordinates). Tiles at the edges (aPos.x near 0 or 1) spread outward most; tiles at center (aPos.x ~0.5) barely move. This creates an organic dispersion from center.

### Touch Event Fix
```javascript
// Source: Replace lines 1906-1917 in index.html

// Remove the window-level passive:false handlers entirely.
// Add canvas-scoped passive listeners instead:
const canvasEl = document.getElementById('canvas');

canvasEl.addEventListener('touchmove', (e) => {
  // No preventDefault — allow native scroll
  const touch = e.touches[0];
  mouseX = touch.clientX;
  mouseY = touch.clientY;
}, { passive: true });

canvasEl.addEventListener('touchend', () => {
  hoverVelY += 0.03;
}, { passive: true });

// touchstart on window can stay as passive (just impulse trigger):
window.addEventListener('touchstart', () => {}, { passive: true });
```

### Content Section Animations
```javascript
// Source: IntersectionObserver Web API — play-once pattern
const sectionObserver = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      entry.target.classList.add('is-visible');
      sectionObserver.unobserve(entry.target);
    }
  });
}, { threshold: 0.25 });

document.querySelectorAll('.content-section').forEach(el => {
  sectionObserver.observe(el);
});
```

```css
/* Default state — invisible and shifted down */
.content-section {
  opacity: 0;
  transform: translateY(40px);
  transition: opacity 0.7s ease-out, transform 0.7s ease-out;
  transition-delay: 0.1s;
}

.content-section.is-visible {
  opacity: 1;
  transform: translateY(0);
}

/* First section (video grid) — more pronounced spring entrance */
#projects {
  transform: translateY(80px);
  transition: opacity 1.0s ease-out, transform 1.0s cubic-bezier(0.16, 1, 0.3, 1);
}
```

---

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| ScrollMagic + GSAP | GSAP ScrollTrigger | GSAP 3.3 (2020) | ScrollMagic deprecated; ScrollTrigger is first-party |
| ScrollTrigger (paid Club) | ScrollTrigger (free) | GSAP 3.12 (2024) | No license concern |
| Custom scroll + RAF opacity | ScrollTrigger scrub | Ongoing | Handles iOS rubber-band, resize, direction reversal |
| Locomotive Scroll / Lenis | Native scroll + ScrollTrigger | Ongoing shift in community | Less complexity, better a11y, fewer mobile bugs |
| Shader uniforms for scroll | CSS scroll-driven animations | CSS Scroll (Safari 18.4) | CSS scroll-driven has incomplete Safari support; GSAP safer |

**Deprecated/outdated:**
- ScrollMagic: unmaintained; ScrollTrigger replaced it
- CSS scroll-driven animations: still unsuitable — Safari support only reached parity in 18.4 (March 2025), too new for production use

---

## Open Questions

1. **Entry animation Z conflict: exact timing**
   - What we know: Phase 3 entry animation at lines 2004-2030 runs for 2.4s from `transition()`. It animates `glassGroup.position.z` from 5 → 3.
   - What's unclear: Whether there's a flag already set when the entry animation completes. There is no explicit callback — just a `requestAnimationFrame` loop that stops at `t >= 1`.
   - Recommendation: Add an `entranceComplete` flag that is set to `true` at the end of the `animateEntry()` function. Gate scroll-driven Z writes on this flag.

2. **Safari `backdrop-filter` on transformed fixed element**
   - What we know: `#canvas` is `position: fixed`. ScrollTrigger will apply `transform: translateY` to it. The `.cta-btn` uses `backdrop-filter: blur()`. Both are in the same stacking context area.
   - What's unclear: Whether Safari correctly applies backdrop-blur on a sibling element of a transformed fixed element.
   - Recommendation: Test on iOS Safari specifically. Fallback: use `background: rgba(0,0,0,0.6)` without backdrop-filter if the effect breaks.

3. **ScrollTrigger.refresh() for empty sections (Phases 5/6)**
   - What we know: Current sections have no content — they're `min-height: 100vh` placeholders. Phase 5 will add video grid content that changes actual heights.
   - What's unclear: Whether ScrollTrigger positions will need a refresh call as part of Phase 5's plan.
   - Recommendation: Add `// ScrollTrigger.refresh() required here after content added` comment to the scroll init code. Phase 5 planner must include this as a task.

---

## Sources

### Primary (HIGH confidence)
- GSAP ScrollTrigger official docs: https://gsap.com/docs/v3/Plugins/ScrollTrigger/ — scrub, onUpdate, start/end syntax, standalone create()
- Project codebase: `index.html` lines 1736–1824 (RAF loop, startRAFLoop/stopRAFLoop), lines 1896–1921 (touch handlers), lines 305–318 (scene object declarations), lines 947–991 (buildTiles, uniforms), lines 2004–2030 (entry animation)
- MDN IntersectionObserver: https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API — threshold, unobserve pattern
- CSS-Tricks Grainy Gradients: https://css-tricks.com/grainy-gradients/ — SVG feTurbulence data URI technique

### Secondary (MEDIUM confidence)
- Codrops "How to Build Cinematic 3D Scroll Experiences with GSAP" (Nov 2025): https://tympanus.net/codrops/2025/11/19/how-to-build-cinematic-3d-scroll-experiences-with-gsap/ — confirms GSAP onUpdate pattern for Three.js objects
- GSAP community forum "delay to scrollTrigger": https://gsap.com/community/forums/topic/28196-delay-to-scrolltrigger/ — empty-tween hold technique
- Codrops "Animate WebGL Shaders with GSAP" (Oct 2025): https://tympanus.net/codrops/2025/10/08/how-to-animate-webgl-shaders-with-gsap-ripples-reveals-and-dynamic-blur-effects/ — shader uniform animation via GSAP callbacks

### Tertiary (LOW confidence — validate if needed)
- backdrop-filter + fixed + transform Safari interaction: not officially documented as problematic; flagged for hands-on testing

---

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH — GSAP already in project; IntersectionObserver is a browser standard; film grain pattern already in codebase
- Architecture: HIGH — two-track pattern directly derived from codebase analysis; ScrollTrigger scrub API verified against official docs
- Pitfalls: HIGH — touch handler issue found directly in codebase at lines 1906-1917; entry animation conflict identified from code analysis; other pitfalls from ScrollTrigger official docs and community

**Research date:** 2026-03-24
**Valid until:** 2026-04-24 (stable domain; GSAP 3.x is mature, ScrollTrigger API is stable)
