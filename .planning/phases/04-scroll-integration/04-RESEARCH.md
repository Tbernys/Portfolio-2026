# Phase 4: Scroll Integration - Research

**Researched:** 2026-03-24
**Domain:** GSAP ScrollTrigger, scroll-driven WebGL fade, fixed navigation dots
**Confidence:** HIGH

## Summary

Phase 4 wires GSAP ScrollTrigger to the existing hero scene so that scrolling down fades/scales the WebGL canvas and reveals DOM content sections beneath. The critical blocker is that the current codebase has `window` touch event listeners calling `e.preventDefault()` on touchstart and touchmove (lines 1906-1917), which will completely prevent native scrolling on mobile. This must be fixed before any scroll integration can work on touch devices.

The existing import map already has `"gsap/": "https://cdn.jsdelivr.net/npm/gsap@3.14.2/"` which resolves `gsap/ScrollTrigger` to the correct ESM file (verified on CDN). ScrollTrigger's `scrub` mode directly ties animation progress to scroll position, making it the right tool for a cinematic hero-to-content transition. A fixed nav dot indicator completes the orientation requirement.

**Primary recommendation:** Use ScrollTrigger with `scrub: true` on the `#hero` section to drive canvas opacity/scale from 1 to 0 over a 100vh scroll distance. Fix touch event handlers to only preventDefault inside the hero viewport zone. Add a minimal fixed dot-nav on the right edge.

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|-----------------|
| SCRL-01 | Visitor scrolls vertically from hero into content sections | Touch event fix (remove blanket preventDefault), ScrollTrigger with no scroll hijacking (no pin on hero) |
| SCRL-02 | Hero WebGL canvas fades/scales as visitor scrolls down | ScrollTrigger scrub tween on `#canvas` opacity + scale, driven by `#hero` trigger leaving viewport |
| SCRL-03 | Fixed navigation or scroll indicator helps visitor orient | Fixed-position dot nav with active state driven by ScrollTrigger section triggers |
</phase_requirements>

## Standard Stack

### Core
| Library | Version | Purpose | Why Standard |
|---------|---------|---------|--------------|
| GSAP | 3.14.2 | Animation engine (already in project) | Already imported, proven in Phase 2 |
| GSAP ScrollTrigger | 3.14.2 | Scroll-to-animation bridge | Free since GSAP 3.12, industry standard for scroll-driven animations, ESM available on same CDN path |

### Supporting
| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| None needed | - | - | Pure CSS + ScrollTrigger handles everything |

### Alternatives Considered
| Instead of | Could Use | Tradeoff |
|------------|-----------|----------|
| ScrollTrigger | IntersectionObserver | IO is binary (in/out), not progress-based; no scrub capability |
| ScrollTrigger | CSS scroll-driven animations | Browser support still incomplete (no Safari < 18.4), GSAP already in project |

**Import map addition:**
```
No changes needed. The existing "gsap/" prefix resolves gsap/ScrollTrigger to:
https://cdn.jsdelivr.net/npm/gsap@3.14.2/ScrollTrigger.js (verified ESM)
```

**Import code:**
```javascript
import { ScrollTrigger } from 'gsap/ScrollTrigger';
gsap.registerPlugin(ScrollTrigger);
```

## Architecture Patterns

### Current Layout (already in place)
```
#canvas          position: fixed, z-index: 0   (WebGL)
main             position: relative, z-index: 1 (DOM content)
  #hero          min-height: 100vh, pointer-events: none
  #clients       min-height: 100vh
  #projects      min-height: 100vh
  #contact       min-height: 100vh
```

This layout is already correct for scroll integration. The canvas sits behind scrollable content. As content scrolls up, it naturally covers the fixed canvas. The fade/scale on the canvas makes this transition feel intentional rather than accidental.

### Pattern 1: Scrub Tween for Canvas Fade
**What:** Tie canvas opacity and scale to scroll progress using ScrollTrigger's `scrub` mode
**When to use:** When an animation should track scroll position 1:1 (not trigger-and-play)
**Example:**
```javascript
// Source: GSAP ScrollTrigger docs — scrub + onUpdate
gsap.to('#canvas', {
  opacity: 0,
  scale: 0.95,
  scrollTrigger: {
    trigger: '#hero',
    start: 'top top',       // when hero top hits viewport top
    end: 'bottom top',      // when hero bottom hits viewport top (1 viewport of scroll)
    scrub: true,            // direct link to scroll position
  }
});
```

### Pattern 2: Section-Based Nav Dots
**What:** Fixed dot navigation that highlights the active section
**When to use:** Long single-page sites with distinct sections
**Example:**
```javascript
// Source: ScrollTrigger docs — standalone create()
const sections = ['hero', 'clients', 'projects', 'contact'];
sections.forEach((id, i) => {
  ScrollTrigger.create({
    trigger: `#${id}`,
    start: 'top center',
    end: 'bottom center',
    onToggle: (self) => {
      if (self.isActive) {
        document.querySelectorAll('.nav-dot').forEach(d => d.classList.remove('active'));
        document.querySelector(`[data-section="${id}"]`).classList.add('active');
      }
    }
  });
});
```

### Pattern 3: Touch Event Fix (Critical)
**What:** Replace blanket `window` preventDefault with scoped hero-only handling
**When to use:** When WebGL interaction and native scroll must coexist
**Example:**
```javascript
// BEFORE (blocks ALL scrolling on mobile):
window.addEventListener('touchstart', (e) => { e.preventDefault(); }, { passive: false });

// AFTER (only blocks scrolling when touching the canvas in hero zone):
const canvas = document.getElementById('canvas');
canvas.addEventListener('touchstart', (e) => {
  // Only prevent default while hero is visible (scroll progress < 1)
  if (heroScrollProgress < 1) {
    e.preventDefault();
  }
}, { passive: false });

canvas.addEventListener('touchmove', (e) => {
  if (heroScrollProgress < 1) {
    e.preventDefault();
    const touch = e.touches[0];
    mouseX = touch.clientX;
    mouseY = touch.clientY;
  }
}, { passive: false });
```

**Alternative simpler approach:** Move touch listeners to the `#canvas` element instead of `window`, and let touch events on `main` (the content layer) propagate normally. Since `main` has `z-index: 1` and sits above the canvas, touches on content sections will never reach the canvas listener. The hero section has `pointer-events: none` so touches pass through to canvas there -- but once the user scrolls past the hero, content sections with `pointer-events: auto` intercept touches first.

**IMPORTANT NUANCE:** The current hero section has `pointer-events: none` on `main`, which means ALL touch events in the hero zone go to `window` -> canvas. For scrolling to work on mobile in the hero zone, we need a different strategy. Options:

1. **Remove preventDefault entirely** -- let native scroll always work, use `touchmove` passively for mouse tracking only (no `preventDefault`). The canvas interaction (glass rotation) still works because we read touch coordinates, we just don't prevent the scroll.
2. **Use a "scroll unlock" zone** -- add a transparent touch-scroll layer over part of the hero that has `pointer-events: auto` and does NOT preventDefault.

Option 1 is simpler and recommended. The glass text rotation from touch movement still works perfectly without `preventDefault` -- you just also get scrolling, which is the desired behavior.

### Anti-Patterns to Avoid
- **Scroll hijacking / pinning the hero:** Don't pin `#hero` -- visitors should always feel in control of scrolling. The requirement explicitly says "without the page feeling stuck or hijacked."
- **requestAnimationFrame for scroll-driven animation:** Don't listen to `scroll` event and animate in RAF -- ScrollTrigger already handles this with debouncing and performance optimization.
- **Separate scroll library (Lenis, Locomotive):** Unnecessary complexity for this use case. Native scroll + ScrollTrigger is sufficient. No smooth-scroll library needed.

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Scroll-to-animation sync | Custom scroll listener + RAF + progress calc | ScrollTrigger `scrub` | Handles resize, refresh, direction reversal, mobile quirks |
| Section detection | IntersectionObserver for each section | ScrollTrigger `onToggle` | Already loading ScrollTrigger; consistent API |
| Scroll progress calculation | `window.scrollY / document.body.scrollHeight` | `self.progress` in ScrollTrigger callback | Accounts for dynamic content, resize, trigger boundaries |

**Key insight:** ScrollTrigger handles edge cases that hand-rolled scroll listeners miss: dynamic content resizing, orientation changes, rubber-band scrolling on iOS, and scroll direction reversal mid-animation.

## Common Pitfalls

### Pitfall 1: Touch preventDefault Blocking Native Scroll
**What goes wrong:** The current `window` touchstart/touchmove listeners call `e.preventDefault()` which completely blocks native scrolling on mobile/touch devices.
**Why it happens:** These listeners were added in Phase 3 for glass text interaction, before scroll was needed.
**How to avoid:** Move touch listeners to `#canvas` element, remove `preventDefault()`, or make it conditional on hero visibility.
**Warning signs:** Page appears stuck/unscrollable on mobile; works fine on desktop with mouse wheel.

### Pitfall 2: ScrollTrigger.refresh() After DOM Changes
**What goes wrong:** ScrollTrigger calculates trigger positions on creation. If section heights change later (content added in Phase 5/6), positions become stale.
**Why it happens:** ScrollTrigger caches measurements for performance.
**How to avoid:** Call `ScrollTrigger.refresh()` after any DOM change that affects section heights. Future phases adding content must call this.
**Warning signs:** Nav dots highlight wrong section, canvas fade happens too early/late.

### Pitfall 3: Canvas Opacity vs. Render Cost
**What goes wrong:** Setting canvas CSS opacity to 0 still runs the full WebGL render pipeline (bloom, fluid, etc.) -- wasting GPU while invisible.
**Why it happens:** CSS opacity is cosmetic; it doesn't affect JS execution.
**How to avoid:** When scroll progress reaches ~0.95, stop the render loop or switch to a lightweight no-op loop. Resume when scrolling back to hero.
**Warning signs:** High GPU usage even when scrolled fully past the hero.

### Pitfall 4: pointer-events: none on main
**What goes wrong:** `main` has `pointer-events: none` globally, with selective `pointer-events: auto` on interactive elements. This means ScrollTrigger's wheel/touch detection on content sections may behave unexpectedly if the scroller element is misconfigured.
**Why it happens:** The pattern was set up for hero-only WebGL interaction.
**How to avoid:** Content sections added in Phase 5/6 will have interactive elements (`pointer-events: auto`). The current setup already handles this with the CSS rule targeting `main a, main button, main form, main input, main textarea`. ScrollTrigger uses `window` scroll by default, which is unaffected by pointer-events.

### Pitfall 5: GSAP ScrollTrigger Import Fails Silently
**What goes wrong:** If `gsap.registerPlugin(ScrollTrigger)` is forgotten, scroll-linked animations simply don't trigger -- no error thrown.
**Why it happens:** GSAP's plugin system is opt-in.
**How to avoid:** Always register immediately after import. Verify by checking `ScrollTrigger.version` in console.
**Warning signs:** Animations play on load but don't respond to scrolling.

## Code Examples

### Full Canvas Fade Setup
```javascript
// Source: GSAP ScrollTrigger docs + project-specific integration
import { ScrollTrigger } from 'gsap/ScrollTrigger';
gsap.registerPlugin(ScrollTrigger);

// Track hero scroll progress for conditional logic elsewhere
let heroScrollProgress = 0;

// Fade and slightly scale down the WebGL canvas as hero scrolls away
gsap.to('#canvas', {
  opacity: 0,
  scale: 0.95,
  scrollTrigger: {
    trigger: '#hero',
    start: 'top top',
    end: 'bottom top',
    scrub: true,
    onUpdate: (self) => {
      heroScrollProgress = self.progress;
      // Optimization: pause RAF when canvas fully invisible
      if (self.progress > 0.95 && rafId !== null) {
        stopRAFLoop();
      } else if (self.progress <= 0.95 && rafId === null && !document.hidden) {
        startRAFLoop();
      }
    }
  }
});
```

### Nav Dot HTML + CSS
```html
<nav class="scroll-nav" aria-label="Page sections">
  <button class="nav-dot active" data-section="hero" aria-label="Hero"></button>
  <button class="nav-dot" data-section="projects" aria-label="Projects"></button>
  <button class="nav-dot" data-section="clients" aria-label="Clients"></button>
  <button class="nav-dot" data-section="contact" aria-label="Contact"></button>
</nav>
```

```css
.scroll-nav {
  position: fixed;
  right: 1.5rem;
  top: 50%;
  transform: translateY(-50%);
  z-index: 10;
  display: flex;
  flex-direction: column;
  gap: 1rem;
  pointer-events: auto;
}

.nav-dot {
  width: 10px;
  height: 10px;
  border-radius: 50%;
  border: 1.5px solid var(--color-accent);
  background: transparent;
  cursor: pointer;
  transition: background 0.3s, transform 0.3s;
  padding: 0;
}

.nav-dot.active {
  background: var(--color-accent);
  transform: scale(1.3);
}
```

### Nav Dot Click-to-Scroll
```javascript
// Smooth scroll to section on dot click
document.querySelectorAll('.nav-dot').forEach(dot => {
  dot.addEventListener('click', () => {
    const target = document.getElementById(dot.dataset.section);
    if (target) {
      gsap.to(window, {
        scrollTo: { y: target, autoKill: true },
        duration: 1,
        ease: 'power2.inOut'
      });
    }
  });
});
```

**Note:** `gsap.to(window, { scrollTo: ... })` requires the ScrollToPlugin. Alternative without extra plugin:
```javascript
dot.addEventListener('click', () => {
  const target = document.getElementById(dot.dataset.section);
  if (target) {
    target.scrollIntoView({ behavior: 'smooth' });
  }
});
```

The native `scrollIntoView` approach is preferred here to avoid adding another GSAP plugin.

### Touch Event Fix (Concrete)
```javascript
// Replace current window-level touch handlers (lines 1906-1917) with:
const canvasEl = document.getElementById('canvas');

canvasEl.addEventListener('touchstart', (e) => {
  if (e.target.tagName === 'A' || e.target.tagName === 'BUTTON') return;
  // Do NOT preventDefault — allow native scroll
}, { passive: true });

canvasEl.addEventListener('touchmove', (e) => {
  // Read touch position for glass interaction (passive — no preventDefault)
  const touch = e.touches[0];
  mouseX = touch.clientX;
  mouseY = touch.clientY;
}, { passive: true });

canvasEl.addEventListener('touchend', () => {
  hoverVelY += 0.03;
});
```

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| ScrollMagic + GSAP | GSAP ScrollTrigger | GSAP 3.3 (2020) | ScrollMagic deprecated; ScrollTrigger is first-party |
| ScrollTrigger (paid Club) | ScrollTrigger (free) | GSAP 3.12 (2024) | No license concern, free for all use |
| Lenis/Locomotive smooth scroll | Native scroll + ScrollTrigger | Ongoing shift | Less complexity, better accessibility, fewer mobile bugs |

**Deprecated/outdated:**
- ScrollMagic: unmaintained, replaced by ScrollTrigger
- GSAP ScrollTrigger as Club plugin: now free in standard GSAP package

## Open Questions

1. **Should the RAF loop fully stop when scrolled past hero?**
   - What we know: Setting canvas opacity to 0 is cosmetic; GPU still renders
   - What's unclear: Whether stopping RAF causes a visible hitch when scrolling back up
   - Recommendation: Stop RAF at progress > 0.95, restart at progress <= 0.95. Test for smoothness. If hitch is noticeable, keep RAF running but skip the heavy bloom/fluid passes instead.

2. **Section order: hero -> projects -> clients -> contact, or different?**
   - What we know: Current HTML order is hero -> clients -> projects -> contact
   - What's unclear: Whether user wants to reorder sections
   - Recommendation: Planner should follow existing HTML order unless user specifies otherwise. Nav dots match HTML order.

## Sources

### Primary (HIGH confidence)
- GSAP ScrollTrigger official docs (https://gsap.com/docs/v3/Plugins/ScrollTrigger/) -- API, scrub, pin, start/end syntax
- jsDelivr CDN verified: `https://cdn.jsdelivr.net/npm/gsap@3.14.2/ScrollTrigger.js` exists as ESM
- jsDelivr CDN verified: `https://cdn.jsdelivr.net/npm/gsap@3.14.2/Observer.js` exists (ScrollTrigger dependency)
- Project codebase: index.html lines 1906-1917 (touch handlers), lines 9-18 (import map), lines 64-85 (layout CSS)

### Secondary (MEDIUM confidence)
- GSAP installation docs (https://gsap.com/docs/v3/Installation/) -- ESM import patterns

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH -- GSAP already in project, ScrollTrigger verified on same CDN path
- Architecture: HIGH -- layout already supports scroll (fixed canvas + relative main), well-documented ScrollTrigger patterns
- Pitfalls: HIGH -- touch preventDefault issue identified directly in codebase; RAF optimization is standard practice

**Research date:** 2026-03-24
**Valid until:** 2026-04-24 (stable domain, GSAP 3.x is mature)
