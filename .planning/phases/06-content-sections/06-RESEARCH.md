# Phase 6: Content Sections — Research

**Researched:** 2026-03-26
**Domain:** Vanilla JS form submission (Formspree), SVG logo display, CSS glassmorphism, single-HTML inline patterns
**Confidence:** HIGH

---

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions

#### Logos clients (CLNT-01, CLNT-02)
- **D-01:** Logos displayed in a single horizontal row (no multi-line grid)
- **D-02:** Logos in inline SVG or high-quality images — monochrome (white / accent #D4C5B2) for dark theme consistency
- **D-03:** Section title visible ("Clients" or "Ils m'ont fait confiance")
- **D-04:** Placeholder logos — Tom will provide real logos later; use styled rectangles or text names in the interim
- **D-05:** No hover animation on logos — simple and sober
- **D-06:** Animated entrance consistent with the existing `.content-section` + `.is-visible` pattern

#### Contact form (CTCT-01, CTCT-03)
- **D-07:** Backend = Formspree (works on any static host, unlike Netlify Forms)
- **D-08:** Fields: name, email, message — no phone, no subject
- **D-09:** Submit via fetch() (no redirect) — display success/error inline
- **D-10:** Success message: visible confirmation text inside the form
- **D-11:** Error message: visible error text if fetch fails
- **D-12:** Basic HTML5 validation only (required, type="email") — no custom JS validation

#### Direct email (CTCT-02)
- **D-13:** mailto: link visible below or beside the form
- **D-14:** Link text = the email address itself (not "Send me an email")
- **D-15:** Email = contact@tombernys.com

#### Contact layout
- **D-16:** Contact section on dark background consistent with #050508
- **D-17:** Form centered, max-width ~500px
- **D-18:** Input style consistent with dark aesthetic: semi-transparent background, subtle border, light text
- **D-19:** Submit button with glassmorphism style (consistent with "Voir mon travail" CTA from Phase 4)

### Claude's Discretion
- Exact number and arrangement of placeholder logos
- Spacing between logos
- Size and font of the logos section title
- Specific entrance animation for each section
- Exact input field style (border-radius, padding, focus state)
- Icon or no icon on the submit button
- Spacing between the form and mailto link

### Deferred Ideas (OUT OF SCOPE)
- Social links (Vimeo, Instagram, LinkedIn) — v2 requirement CONT-01, not in v1 scope
- Animated logo marquee/carousel — over-engineering for a static row of logos
</user_constraints>

---

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|------------------|
| CLNT-01 | Visitor sees a section displaying client/brand logos | Static HTML section, horizontal flex row, `.content-section` entrance pattern |
| CLNT-02 | Logos are SVG or high-quality images, in a grid or horizontal row | Inline SVG technique, placeholder rectangles as stand-ins |
| CTCT-01 | Visitor can submit a contact form (name, email, message) via Formspree | Formspree fetch() pattern documented below |
| CTCT-02 | Visitor sees a direct email link as alternative to the form | Simple `<a href="mailto:...">` — no dependencies |
| CTCT-03 | Form provides clear feedback on submission (success/error state) | Formspree JSON response shape; response.ok / data.errors pattern |
</phase_requirements>

---

## Summary

Phase 6 adds two content sections to the existing single-HTML portfolio: a clients logo row and a contact section with a Formspree-backed form plus a direct mailto link. Both sections already exist as empty placeholders (`#clients` at line 602, `#contact` at line 607) and are already observed by the `sectionObserver` IntersectionObserver that fires `.is-visible` on scroll — no new JavaScript infrastructure is needed for entrance animations.

The work is purely HTML markup + CSS additions inside the existing `<style>` block, plus a small JS function `initContactForm()` wired into the existing init chain. No new CDN imports are needed; Formspree is a plain HTTP endpoint called with `fetch()`. The main technical question is the Formspree fetch pattern (confirmed below), and correctly reusing the glassmorphism design tokens already defined for `.cta-btn`.

**Primary recommendation:** Write `initContactForm()` alongside the existing `initProjectsShowcase()` pattern. Keep all CSS in the existing `<style>` block. Placeholder logos as styled `<span>` text labels or SVG rectangles — real assets swapped in later without touching JS.

---

## Standard Stack

### Core

| Library / API | Version | Purpose | Why Standard |
|---------------|---------|---------|--------------|
| Formspree | hosted | Form backend for static sites | No server needed; works on Vercel, GitHub Pages, Netlify; free tier 50 submissions/mo |
| Vanilla fetch() | native | AJAX form submission | Already used throughout project; no new imports |
| IntersectionObserver | native | Section entrance animations | Already wired via `sectionObserver` — zero new code for entrance |

### Supporting

| Pattern | Purpose | When to Use |
|---------|---------|-------------|
| HTML5 form validation | `required`, `type="email"` | Built into browser; no JS needed per D-12 |
| `<a href="mailto:...">` | Direct email link | No dependencies; instant mailto fallback |
| Inline SVG / `<img>` | Logo display | Per D-02; SVG preferred for crisp monochrome rendering |

### Alternatives Considered

| Instead of | Could Use | Tradeoff |
|------------|-----------|----------|
| Formspree | Netlify Forms | Netlify Forms only works when deployed to Netlify — breaks portability (D-07 locked) |
| Formspree | EmailJS | Requires JS SDK, exposes API key in client; Formspree hides key server-side |
| Placeholder text spans | Actual logo SVG files | Real files require asset delivery; placeholder spans need zero assets |

**No new installation needed.** Formspree is a remote endpoint — no npm package.

---

## Architecture Patterns

### Recommended Project Structure

All changes land in `index.html` only. No new files.

```
index.html
  <style>           ← add: #clients, #contact, .logo-row, .contact-form, .form-feedback CSS
  <body>
    #clients        ← replace placeholder with logo row markup
    #contact        ← replace placeholder with form + mailto markup
  <script type="module">
    initContactForm()    ← new function, called in init chain
```

### Pattern 1: Clients Logo Row

**What:** A `<div class="logo-row">` inside `#clients` containing placeholder spans (later replaced with `<img>` or inline `<svg>`). The outer `<section>` already has `class="content-section"` which the existing `sectionObserver` picks up automatically.

**When to use:** As soon as Tom provides real SVG assets, swap `<span class="logo-placeholder">Brand Name</span>` for `<img src="logo.svg" alt="Brand Name" class="logo-img">` — no JS changes required.

**Example:**
```html
<!-- Source: existing pattern from #projects section -->
<section id="clients" class="content-section" data-lang="fr" aria-label="Clients">
  <div class="section-inner">
    <h2 class="section-title">Ils m'ont fait confiance</h2>
    <div class="logo-row">
      <span class="logo-placeholder">Brand A</span>
      <span class="logo-placeholder">Brand B</span>
      <span class="logo-placeholder">Brand C</span>
      <span class="logo-placeholder">Brand D</span>
      <span class="logo-placeholder">Brand E</span>
    </div>
  </div>
</section>
```

**CSS pattern** (reusing existing design tokens):
```css
.logo-row {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  align-items: center;
  gap: 3rem;
  padding: 2rem 0;
}
.logo-placeholder {
  font-family: 'Inter', sans-serif;
  font-size: 0.875rem;
  letter-spacing: 0.12em;
  text-transform: uppercase;
  color: var(--color-accent);   /* #D4C5B2 per D-02 */
  opacity: 0.6;
  border: 1px solid rgba(212, 197, 178, 0.2);
  padding: 0.5rem 1.2rem;
  border-radius: 2px;
}
```

### Pattern 2: Formspree fetch() Submission

**What:** On form `submit` event, `preventDefault()`, build `FormData`, call `fetch('https://formspree.io/f/{FORM_ID}', {...})`. Show inline success or error message.

**Formspree endpoint format:** `https://formspree.io/f/{FORM_ID}` — FORM_ID obtained from Formspree dashboard after creating a free form.

**Required header:** `Accept: 'application/json'` — without this Formspree returns a redirect HTML page instead of JSON.

**Success check:** `response.ok` (HTTP 200–299).

**Error shape:** `{ errors: [{ message: "..." }, ...] }` — join messages for display.

**Source:** [Formspree AJAX docs](https://help.formspree.io/hc/en-us/articles/360013470814-Submit-forms-with-JavaScript-AJAX) (MEDIUM confidence — confirmed by multiple community sources)

**Example:**
```javascript
// Source: Formspree AJAX documentation pattern
function initContactForm() {
  const form = document.getElementById('contact-form');
  if (!form) return;

  form.addEventListener('submit', async (e) => {
    e.preventDefault();
    const data = new FormData(form);
    const feedback = document.getElementById('form-feedback');

    try {
      const response = await fetch(form.action, {
        method: 'POST',
        body: data,
        headers: { 'Accept': 'application/json' }
      });

      if (response.ok) {
        feedback.textContent = 'Message envoyé ! Je vous réponds rapidement.';
        feedback.className = 'form-feedback form-feedback--success';
        form.reset();
      } else {
        const json = await response.json();
        const msg = json.errors
          ? json.errors.map(err => err.message).join(', ')
          : 'Une erreur est survenue.';
        feedback.textContent = msg;
        feedback.className = 'form-feedback form-feedback--error';
      }
    } catch {
      const feedback = document.getElementById('form-feedback');
      feedback.textContent = 'Erreur réseau. Veuillez réessayer.';
      feedback.className = 'form-feedback form-feedback--error';
    }
  });
}
```

### Pattern 3: Glassmorphism Submit Button (reuse `.cta-btn`)

**What:** The submit button reuses exactly the same `background / backdrop-filter / border / border-radius` values as `.cta-btn`. No new design token needed — copy the rule or extend `.cta-btn`.

**Exact values from existing code (line 115–135):**
```css
.contact-submit {
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
  transition: background 0.2s, border-color 0.2s;
}
.contact-submit:hover {
  background: rgba(255, 255, 255, 0.14);
  border-color: rgba(255, 255, 255, 0.25);
}
```

### Pattern 4: Input Field Dark Style (D-18)

**What:** Inputs and textarea use semi-transparent dark background, subtle border, `var(--color-text)` text, no box shadows that clash with the WebGL canvas behind.

```css
.contact-form input,
.contact-form textarea {
  width: 100%;
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.12);
  border-radius: 0.375rem;
  color: var(--color-text);
  font-family: 'Inter', sans-serif;
  font-size: 0.875rem;
  padding: 0.75rem 1rem;
  outline: none;
  transition: border-color 0.2s;
}
.contact-form input:focus,
.contact-form textarea:focus {
  border-color: rgba(212, 197, 178, 0.4);  /* accent at 40% opacity */
}
.contact-form input::placeholder,
.contact-form textarea::placeholder {
  color: rgba(245, 240, 235, 0.35);
}
```

### Pattern 5: HTML Markup for Contact Section

```html
<section id="contact" class="content-section" data-lang="fr" aria-label="Contact">
  <div class="section-inner">
    <h2 class="section-title">Contact</h2>
    <form
      id="contact-form"
      class="contact-form"
      action="https://formspree.io/f/REPLACE_WITH_FORM_ID"
      method="POST"
    >
      <div class="form-field">
        <label for="contact-name" class="sr-only">Nom</label>
        <input type="text" id="contact-name" name="name" placeholder="Nom" required>
      </div>
      <div class="form-field">
        <label for="contact-email" class="sr-only">Email</label>
        <input type="email" id="contact-email" name="email" placeholder="Email" required>
      </div>
      <div class="form-field">
        <label for="contact-message" class="sr-only">Message</label>
        <textarea id="contact-message" name="message" placeholder="Message" rows="5" required></textarea>
      </div>
      <div id="form-feedback" class="form-feedback" aria-live="polite"></div>
      <button type="submit" class="contact-submit">Envoyer</button>
    </form>
    <p class="contact-mailto">
      Ou directement : <a href="mailto:contact@tombernys.com">contact@tombernys.com</a>
    </p>
  </div>
</section>
```

**Notes on markup:**
- `action` attribute on `<form>` stores the Formspree URL — the JS reads `form.action` directly, so there is a single source of truth
- `aria-live="polite"` on `#form-feedback` announces success/error to screen readers without requiring focus
- `.sr-only` labels satisfy accessibility while keeping the visual design clean (placeholder text serves as the visible label)
- `method="POST"` on the form is a progressive enhancement fallback; the JS intercepts submit and prevents default

### Anti-Patterns to Avoid

- **Reading `form.action` before calling `fetch`:** Always use `form.action` (set in HTML attribute) — hardcoding the Formspree URL in JS creates a second source of truth to keep in sync.
- **Sending Content-Type: application/json:** Formspree expects `multipart/form-data` or `application/x-www-form-urlencoded` via `FormData` — only the `Accept` header should be `application/json`.
- **Blocking `pointer-events` on `<form>`:** The `main` element has `pointer-events: none`. The existing rule `main form, main input, main textarea` already restores `pointer-events: auto` (line 82–87) — verify the new form elements are covered by this rule.
- **Calling `initContactForm()` before DOM is ready:** Add the call inside the existing init sequence (after `initProjectsShowcase()`), not at module top level.

---

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Form backend | Custom server/serverless function | Formspree hosted endpoint | Edge cases: spam filtering, rate limiting, email deliverability, CORS — Formspree handles all of these |
| Success/error UX | Complex state machine | Simple `textContent` + CSS class toggle | Two states only; no framework needed |
| Email validation | Custom regex | `type="email"` HTML5 attribute | Browser engines have battle-tested RFC 5322 parsers; custom regex gets edge cases wrong |
| Logo animation | CSS marquee / JS scroll | Static flex row | Decided as out of scope (D-05, deferred) |
| CSRF protection | Custom token | Formspree handles server-side | Formspree uses form ID as secret; adding a second CSRF layer adds complexity without benefit |

**Key insight:** Formspree's free tier (50 submissions/month) is more than sufficient for a freelancer portfolio. The fetch + Accept: application/json pattern gives full inline control with zero dependencies beyond a remote HTTP call.

---

## Common Pitfalls

### Pitfall 1: Missing Accept: application/json Header

**What goes wrong:** Formspree returns a 302 redirect to its own thank-you page. `fetch()` follows the redirect, `response.ok` reads `true`, but the form is never properly acknowledged.

**Why it happens:** Formspree's default behavior without the header is to redirect browsers to a confirmation page — it predates the AJAX paradigm.

**How to avoid:** Always include `headers: { 'Accept': 'application/json' }` in the `fetch()` call.

**Warning signs:** Response body is HTML, not JSON; `response.url` differs from the submitted URL.

### Pitfall 2: pointer-events Blocked on Form Elements

**What goes wrong:** Inputs and buttons receive no clicks/focus because `main { pointer-events: none }` is inherited.

**Why it happens:** The main element disables pointer events globally so the WebGL canvas below receives mouse input.

**How to avoid:** The existing rule `main a, main button, main form, main input, main textarea { pointer-events: auto }` (lines 80–87) already covers standard form elements. **Confirm the new form's `<textarea>` and `<button type="submit">` match these selectors.** If a new wrapper element blocks events, add it to the selector list.

**Warning signs:** Inputs are visually present but cannot be clicked or typed into.

### Pitfall 3: Formspree Form ID Not Replaced

**What goes wrong:** Form submits to `https://formspree.io/f/REPLACE_WITH_FORM_ID` — Formspree returns a 404.

**Why it happens:** The planner uses a placeholder that the implementer forgets to replace.

**How to avoid:** Include a Wave 0 task: "Create Formspree account, create form for contact@tombernys.com, paste real FORM_ID into `form.action`." The rest of the implementation can proceed with a test FORM_ID.

**Warning signs:** Console error `404 Not Found` on form submit; Formspree dashboard shows no submissions.

### Pitfall 4: Feedback Element Not Cleared Between Submissions

**What goes wrong:** A previous error message persists when the user corrects and resubmits; a success message stays visible after `form.reset()`.

**Why it happens:** The feedback `textContent` is only set on completion, never reset on new submission.

**How to avoid:** At the top of the submit handler (after `e.preventDefault()`), immediately set `feedback.textContent = ''` and remove the state class before the `await`.

### Pitfall 5: Section min-height Constraint Breaking Layout

**What goes wrong:** The existing CSS rule `#clients, #contact { min-height: 100vh; }` (lines 421–428) forces full-viewport-height sections. Once real content replaces the placeholder, the section may look awkwardly stretched with too much empty space below the content.

**Why it happens:** The `min-height: 100vh` was added as a placeholder scroll-space holder, not as a final layout constraint.

**How to avoid:** Replace with `padding: 6rem 2rem` (or similar comfortable vertical rhythm) and remove the `min-height` for these two sections, or reduce it to `min-height: auto`. The `display: flex; align-items: center; justify-content: center` can be kept.

---

## Code Examples

### Formspree Verified fetch Pattern

```javascript
// Source: Formspree AJAX documentation — https://formspree.io
// Pattern confirmed by multiple community sources (MEDIUM confidence)

form.addEventListener('submit', async (e) => {
  e.preventDefault();
  feedback.textContent = '';
  feedback.className = 'form-feedback';

  const response = await fetch(form.action, {
    method: 'POST',
    body: new FormData(form),
    headers: { 'Accept': 'application/json' }
  });

  if (response.ok) {
    feedback.textContent = 'Message envoyé !';
    feedback.className = 'form-feedback form-feedback--success';
    form.reset();
  } else {
    const json = await response.json().catch(() => ({}));
    const msg = json.errors?.map(e => e.message).join(', ') || 'Erreur. Réessayez.';
    feedback.textContent = msg;
    feedback.className = 'form-feedback form-feedback--error';
  }
});
```

### Existing sectionObserver — No Changes Required

```javascript
// Source: index.html line 2733–2744
// Already observes ALL .content-section elements including #clients and #contact
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

No changes to this block — `#clients` and `#contact` are already `.content-section` elements. The observer fires automatically once content is in the DOM.

### Design Tokens Already Available

```css
/* Source: index.html :root (lines 46–50) */
--color-bg: #000000;
--color-text: #F5F0EB;
--color-accent: #D4C5B2;

/* Glassmorphism — from .cta-btn (lines 115–140) */
background: rgba(255, 255, 255, 0.08);
backdrop-filter: blur(12px);
-webkit-backdrop-filter: blur(12px);
border: 1px solid rgba(255, 255, 255, 0.15);
border-radius: 2rem;
/* hover: */
background: rgba(255, 255, 255, 0.14);
border-color: rgba(255, 255, 255, 0.25);
```

---

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| `<form action="...">` full-page POST | `fetch()` + AJAX inline | ~2015 | No redirect, inline feedback possible |
| Netlify Forms (deploy-locked) | Formspree (host-agnostic) | Decided Phase 6 | Preserves TECH-03 static host portability |
| Logo carousels / marquee | Static monochrome flex row | Phase 6 decision | Simpler, faster, matches sober aesthetic |

**No deprecated approaches in scope for this phase.**

---

## Open Questions

1. **Formspree FORM_ID**
   - What we know: The endpoint is `https://formspree.io/f/{FORM_ID}` — the ID comes from the Formspree dashboard
   - What's unclear: Tom has not yet created a Formspree account / form
   - Recommendation: Wave 0 plan task — "Create Formspree account, register form for contact@tombernys.com, record FORM_ID." A test form ID can be used during dev; the real one replaces it before deploy.

2. **Logo assets**
   - What we know: D-04 explicitly defers real logos — placeholders go in now
   - What's unclear: Final logo count (5? 8? more?)
   - Recommendation: Implement 5–6 placeholder slots. The planner can pick a number; HTML is trivially extended later.

---

## Environment Availability

Step 2.6: SKIPPED — Phase 6 is purely HTML/CSS/JS additions plus a remote HTTP endpoint (Formspree). No local tools, runtimes, databases, or CLI utilities are required beyond the browser and an internet connection.

---

## Validation Architecture

`workflow.nyquist_validation` key is absent from `.planning/config.json` — treated as enabled.

### Test Framework

| Property | Value |
|----------|-------|
| Framework | None detected — portfolio is a single static HTML file |
| Config file | None |
| Quick run command | Open `index.html` in browser; submit form with DevTools Network tab open |
| Full suite command | Manual smoke test checklist (see below) |

No automated test runner exists in this project. All validation is manual browser testing.

### Phase Requirements → Test Map

| Req ID | Behavior | Test Type | Automated Command | File Exists? |
|--------|----------|-----------|-------------------|-------------|
| CLNT-01 | Logo section visible on scroll | manual smoke | Load page, scroll to #clients, confirm section appears | N/A |
| CLNT-02 | Logos/placeholders render in horizontal row | manual smoke | Inspect layout at ≥1024px viewport | N/A |
| CTCT-01 | Form submits to Formspree, no page redirect | manual smoke | Fill form, submit, check Network tab for POST to formspree.io | N/A |
| CTCT-02 | mailto link opens mail client | manual smoke | Click email link, confirm OS mail client opens | N/A |
| CTCT-03 | Success message appears on valid submit | manual smoke | Submit valid form, confirm green feedback text | N/A |
| CTCT-03 | Error message appears on network failure | manual smoke | DevTools → Network → block formspree.io, submit, confirm error text | N/A |

### Sampling Rate

- **Per task commit:** Manual visual check in browser (scroll to section, confirm render)
- **Per wave merge:** Full smoke checklist above
- **Phase gate:** All checklist items green before `/gsd:verify-work`

### Wave 0 Gaps

- [ ] Formspree account + form created — provides real FORM_ID to insert into `form.action`

*(No test file infrastructure gaps — project has no automated test framework to configure.)*

---

## Sources

### Primary (HIGH confidence)
- `index.html` (project codebase) — existing CSS variables, glassmorphism patterns, sectionObserver implementation, pointer-events rules, section placeholder markup

### Secondary (MEDIUM confidence)
- [Formspree AJAX docs](https://help.formspree.io/hc/en-us/articles/360013470814-Submit-forms-with-JavaScript-AJAX) — fetch pattern, Accept header, response.ok, error shape; confirmed by multiple community sources
- [Formspree home](https://formspree.io) — endpoint URL format `https://formspree.io/f/{ID}`; free tier terms

### Tertiary (LOW confidence)
- None

---

## Project Constraints (from CLAUDE.md)

CLAUDE.md does not exist in this project. No additional project-level directives apply beyond those captured in CONTEXT.md decisions above.

---

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH — all tools are native browser APIs or a well-established remote endpoint; no new dependencies
- Architecture: HIGH — directly derived from existing patterns in the codebase (lines verified)
- Pitfalls: HIGH — pointer-events trap verified in source, Formspree header requirement confirmed by official docs

**Research date:** 2026-03-26
**Valid until:** 2026-09-26 (Formspree API is stable; vanilla JS patterns don't change)
