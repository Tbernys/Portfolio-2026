---
phase: 04-scroll-integration
verified: 2026-03-25T01:00:00Z
status: human_needed
score: 12/12 must-haves verified
re_verification: true
previous_status: gaps_found
previous_score: 10/12
gaps_closed:
  - "SCRL-03 persistent orientation — fixed back-to-top button added by plan 04-03"
gaps_remaining: []
human_verification:
  - test: "Open in Chrome desktop — scroll to 100% of hero scroll distance, then scroll back to top. Verify glass text returns to original Z position and tile spread fully collapses."
    expected: "Hero scene looks identical to initial page load after scrolling fully back to top."
    why_human: "entranceComplete flag is never reset; scrub:true handles CSS but Three.js Z reversal depends on GSAP correctly re-driving the onUpdate callback to progress=0."
  - test: "Open on mobile (or Chrome DevTools device emulation) and touch-scroll down from hero."
    expected: "Page scrolls normally — no stuck/locked behaviour. Hero parallax and fade still occurs. CTA button touch target works."
    why_human: "Touch passive listener fix removes window-level preventDefault but the actual absence of scroll locking can only be confirmed on device."
  - test: "Scroll past the hero completely (hero canvas invisible). Open Chrome DevTools Performance tab. Check if there is continuous GPU/CPU activity."
    expected: "RAF loop paused — no continuous frame renders from the Three.js scene after canvas fades out."
    why_human: "stopRAFLoop() call at progress > 0.95 is in the code; actual GPU idle state requires runtime measurement."
---

# Phase 04: Scroll Integration — Re-Verification Report

**Phase Goal:** Scrolling down from the hero feels cinematic — the WebGL world fades away and content sections emerge naturally, with orientation cues present throughout
**Verified:** 2026-03-25
**Status:** human_needed — all automated checks pass, 3 runtime items pending human confirmation
**Re-verification:** Yes — after gap closure by plan 04-03

---

## Re-Verification Summary

| Item | Previous Status | Current Status | Change |
|------|----------------|----------------|--------|
| SCRL-03 persistent orientation (Truth #12) | FAILED | VERIFIED | Gap closed |
| Scroll reversal (Truth #4) | PARTIAL | VERIFIED (code) | Promoted — code is correct; human visual check remains |
| All other truths (1-3, 5-11) | VERIFIED | VERIFIED | No regression |

**Previous score:** 10/12 — **Current score:** 12/12

---

## Goal Achievement

### Observable Truths

| #  | Truth | Status | Evidence |
|----|-------|--------|----------|
| 1  | Visitor can scroll vertically from hero into content sections on both desktop and mobile | VERIFIED | No `pin:true`; touch fix applied; window touch listeners removed |
| 2  | Hero WebGL canvas fades and parallaxes upward as visitor scrolls, revealing content beneath | VERIFIED | `heroTl.to('#canvas', {y:'-50vh'})` + `{opacity:0}`; scrub:true |
| 3  | Glass text recedes in Z and tiles spread apart during scroll — cinematic exit | VERIFIED | `glassGroup.position.z = 3 - progress * 5`; `uScrollProgress` uniform updated in Track B |
| 4  | Scrolling back up reverses the transition identically | VERIFIED (code) | scrub:true on both triggers; Track B onUpdate receives progress=0 on full scroll-up. Human visual confirmation still recommended |
| 5  | A glassmorphism CTA button 'Voir mon travail' is visible at bottom-left of hero | VERIFIED | Button at line 370; CSS glassmorphism styles; bob animation; correct position |
| 6  | CTA button click smooth-scrolls to first content section | VERIFIED | `firstSection.scrollIntoView({behavior:'smooth'})` at line 2228 |
| 7  | RAF loop stops when canvas is invisible (scroll progress > 95%) | VERIFIED | `if (progress > 0.95 && rafId !== null) stopRAFLoop()` at line 2210 |
| 8  | Content sections fade in with slide-up animation when 25% visible | VERIFIED | IntersectionObserver at threshold:0.25; `.content-section.is-visible` CSS |
| 9  | Animations play once — sections stay visible after first entrance | VERIFIED | `sectionObserver.unobserve(entry.target)` present |
| 10 | First section (video grid) has more pronounced spring-like entrance | VERIFIED | `#projects` CSS: `translateY(80px)` + `cubic-bezier(0.16, 1, 0.3, 1)` |
| 11 | Content background has subtle film grain texture matching CRT theme | VERIFIED | `.content-bg::before` with feTurbulence SVG; `opacity:0.04`; `@keyframes grain` |
| 12 | Persistent orientation element visible throughout the page | VERIFIED | Fixed back-to-top button at `position:fixed; bottom:2rem; right:2rem`; `z-index:1000`; appears at progress > 0.95; visible through all content sections |

**Score: 12/12 truths verified**

---

### Required Artifacts

| Artifact | Expected | Status | Details |
|----------|----------|--------|---------|
| `index.html` | ScrollTrigger integration, CTA button, touch fix, tile spread uniform, back-to-top button | VERIFIED | All elements present and substantive |

**Artifact levels:**
- Level 1 (exists): index.html present — PASS
- Level 2 (substantive): Full implementation present — PASS
- Level 3 (wired): `initScrollIntegration()` called inside `main()`; back-to-top toggle inside Track B onUpdate — PASS
- Level 4 (data flows): scrub progress drives CSS transforms and Three.js uniforms; back-to-top visibility driven by live scroll progress — PASS

---

### Key Link Verification

| From | To | Via | Status | Details |
|------|----|-----|--------|---------|
| ScrollTrigger timeline (Track A) | `#canvas` CSS transform + opacity | `gsap.timeline` with `scrub:true` | VERIFIED | Both y and opacity tweens confirmed |
| ScrollTrigger.create (Track B) | `glassGroup.position.z` and `tilesMesh uScrollProgress` | `onUpdate` callback | VERIFIED | `progress` drives both mutations |
| Track B `onUpdate` progress | `back-to-top` `.is-visible` class | `progress > 0.95` threshold | VERIFIED | Lines 2216-2220: classList.add/remove inside existing callback — no new ScrollTrigger added |
| CTA button click | First content section | `scrollIntoView` | VERIFIED | `{behavior:'smooth'}` confirmed |
| Back-to-top click | `window.scrollTo` | `{ top: 0, behavior: 'smooth' }` | VERIFIED | Line 2234 |
| Tile vertex shader `uScrollProgress` | Tile spread visual effect | `uniform float uScrollProgress` | VERIFIED | Uniform declared, used in spread logic, updated in Track B onUpdate |

---

### Data-Flow Trace (Level 4)

| Artifact | Data Variable | Source | Produces Real Data | Status |
|----------|---------------|--------|--------------------|--------|
| `#canvas` (CSS) | `y`, `opacity` | GSAP scrub timeline from scroll position | Yes — live browser scroll | FLOWING |
| `glassGroup.position.z` | `progress` | Track B onUpdate `self.progress` | Yes — live scroll progress 0..1 | FLOWING |
| `tilesMesh.material.uniforms.uScrollProgress` | `progress` | Same onUpdate callback | Yes — same live source | FLOWING |
| `.back-to-top` visibility | `.is-visible` class | Track B `progress > 0.95` | Yes — same live source | FLOWING |
| `.content-section` visibility | `is-visible` class | IntersectionObserver `isIntersecting` | Yes — real DOM intersection events | FLOWING |

---

### Behavioral Spot-Checks

Step 7b: SKIPPED for runtime behaviour (cannot start server in verification). Key static checks performed:

| Check | Result | Status |
|-------|--------|--------|
| `grep -c "back-to-top" index.html` | 5 matches (HTML id, CSS class x3, JS x1) | PASS |
| Button placed after `</main>` (outside all section containers) | Line 391, after `</main>` at line 389 | PASS |
| `position: fixed` in `.back-to-top` CSS block | Line 142 | PASS |
| `bottom: 2rem` and `right: 2rem` | Lines 143-144 | PASS |
| `border-radius: 50%` (circular button) | Line 154 | PASS |
| `backdrop-filter: blur(12px)` count | 4 matches (CTA x2 + back-to-top x2 with -webkit) | PASS |
| `rgba(255, 255, 255, 0.08)` count | 2 matches (CTA + back-to-top) | PASS |
| `rgba(255, 255, 255, 0.15)` border count | 2 matches (CTA + back-to-top) | PASS |
| `scrollTo.*smooth` at line 2234 | `window.scrollTo({ top: 0, behavior: 'smooth' })` | PASS |
| `progress > 0.95` appears twice (RAF stop + button toggle) | 2 matches | PASS |
| `is-visible` in CSS and JS | 5 matches | PASS |
| `aria-label="Retour en haut"` on button | Line 391 | PASS |
| SVG arrow-up with `stroke="currentColor"` | Line 392 | PASS |
| `backToTopBtn` declared inside `initScrollIntegration()` | Line 2165 — first line of function body | PASS |
| No new ScrollTrigger.create added for button | Toggle is inside existing Track B onUpdate | PASS |
| SCRL-03 marked Complete in REQUIREMENTS.md | `[x] **SCRL-03**` confirmed | PASS |

---

### Requirements Coverage

| Requirement | Source Plan | Description | Status | Evidence |
|-------------|-------------|-------------|--------|----------|
| SCRL-01 | 04-01-PLAN.md, 04-02-PLAN.md | Visitor scrolls vertically from hero into content sections | SATISFIED | Free scroll (no pin); touch fix; sections visible below hero |
| SCRL-02 | 04-01-PLAN.md, 04-02-PLAN.md | Hero WebGL canvas fades/scales as visitor scrolls down, revealing content beneath | SATISFIED | Canvas parallax + fade (Track A); glass text Z recession + tile spread (Track B); IntersectionObserver content entrance |
| SCRL-03 | 04-01-PLAN.md, 04-03-PLAN.md | Fixed navigation or scroll indicator helps visitor orient on the page | SATISFIED | Fixed back-to-top button at bottom-right: `position:fixed`, glassmorphism style, `z-index:1000`, visible throughout all content sections (progress > 0.95). REQUIREMENTS.md marks as `[x] Complete`. |

**Orphaned requirements check:** REQUIREMENTS.md maps SCRL-01, SCRL-02, SCRL-03 to Phase 4. All three appear in plan frontmatter. No orphaned requirements.

---

### Anti-Patterns Found

| File | Line | Pattern | Severity | Impact |
|------|------|---------|----------|--------|
| index.html | ~2248 | `// Future phases: call ScrollTrigger.refresh() after adding section content` | Info | Intentional reminder comment for Phase 5 — not a gap |

No blocker anti-patterns. No TODO/FIXME/PLACEHOLDER/HACK comments in implementation code. No empty returns or hardcoded empty data flowing to rendered output. No regression from previous verified items.

---

### Human Verification Required

#### 1. Three.js scroll reversal quality

**Test:** Scroll slowly to 100% hero scroll distance (canvas fully faded), then scroll back to top completely. Observe the glass text.
**Expected:** Glass text restores to its original Z position and tile spread fully collapses back to initial state, making the hero look identical to first load.
**Why human:** `scrub:true` drives CSS DOM values symmetrically, but Three.js `glassGroup.position.z` is mutated imperatively in the `onUpdate` callback. GSAP does not manage this value's reset — it relies on the callback receiving `progress=0` on full scroll-up. Correct in code but requires visual confirmation.

#### 2. Mobile touch scroll behaviour

**Test:** Open on a real iOS or Android device (or Chrome DevTools device emulation) and attempt to scroll down through the hero.
**Expected:** Page scrolls normally without sticking. Hero parallax/fade occurs. Glass text rotation responds to touch drag. CTA touch target is tappable.
**Why human:** The passive listener fix is in the code, but elimination of scroll lock can only be confirmed on a real touch device.

#### 3. RAF pause after hero scroll-out

**Test:** Scroll past the hero completely (hero canvas invisible). Open Chrome DevTools Performance tab. Check for continuous GPU/CPU activity.
**Expected:** RAF loop paused — no continuous frame renders from the Three.js scene.
**Why human:** `stopRAFLoop()` is called at `progress > 0.95` but the actual performance impact requires runtime measurement.

---

### Gaps Summary

No code gaps remain. All 12 truths are verified at the code level.

**Gap 1 (SCRL-03) — CLOSED:** Plan 04-03 added a fixed glassmorphism back-to-top button (`position:fixed; bottom:2rem; right:2rem; z-index:1000`) that appears when scroll progress exceeds 95% — meaning it is visible throughout all content sections. Glassmorphism style exactly matches the CTA button (`rgba(255,255,255,0.08)` background, `blur(12px)`, `rgba(255,255,255,0.15)` border). Click handler calls `window.scrollTo({top:0, behavior:'smooth'})`. REQUIREMENTS.md updated to `[x] Complete` for SCRL-03. All 14 acceptance criteria from the plan pass.

**Gap 2 (scroll reversal) — PROMOTED TO HUMAN CHECK:** The `entranceComplete` flag concern from the initial verification was re-examined. The flag being permanently true on second scroll-down is the desired behaviour (skip re-entrance). The Three.js reversal path (Track B onUpdate receiving `progress=0`) is structurally correct. Remaining concern is purely a visual quality check, not a code defect.

Three runtime human checks carry over: scroll reversal visual quality, mobile touch behaviour, and RAF pause confirmation.

---

*Verified: 2026-03-25*
*Verifier: Claude (gsd-verifier)*
*Re-verification: Yes — after gap closure by plan 04-03*
