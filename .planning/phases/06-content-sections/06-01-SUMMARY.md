---
phase: 06-content-sections
plan: 01
subsystem: content-sections
tags: [html, css, javascript, formspree, contact-form, logo-row, glassmorphism]
dependency_graph:
  requires: [05-02]
  provides: [clients-section, contact-form, formspree-integration]
  affects: [index.html]
tech_stack:
  added: []
  patterns: [Formspree fetch AJAX, glassmorphism button reuse, .content-section entrance animation]
key_files:
  created: []
  modified:
    - index.html
decisions:
  - Formspree FORM_ID left as REPLACE_WITH_FORM_ID placeholder — Tom must create Formspree account before deploy
  - initContactForm() placed after setupLightboxListeners() and before initScrollIntegration to match existing pattern
  - .section-inner wrapper (max-width: 900px) added to both sections for consistent layout width
metrics:
  duration_minutes: 6
  completed_date: "2026-03-26"
  tasks_completed: 2
  files_modified: 1
---

# Phase 06 Plan 01: Clients Logo Row and Contact Form Summary

Clients logo row (6 placeholder brands) and Formspree-backed contact form with inline success/error feedback, direct mailto link, and glassmorphism submit button — all wired into the existing .content-section entrance animation and init chain.

## What Was Built

### Task 1: HTML markup + CSS

Replaced the two empty placeholder sections in index.html with full markup:

**#clients section** — a horizontal flex row of 6 styled `<span class="logo-placeholder">` elements (Nike, Canal+, Dior, Red Bull, Ubisoft, Netflix) under the heading "Ils m'ont fait confiance". Uses `.logo-row` with `flex-wrap: wrap`, `gap: 3rem`, and `.logo-placeholder` with `color: var(--color-accent)`, `opacity: 0.6`, and a subtle accent border.

**#contact section** — a centered form (max-width 500px) with name, email, and message fields, a glassmorphism submit button (matching `.cta-btn` values exactly), an `aria-live="polite"` feedback div, and a mailto link below the form.

**CSS additions:**
- `.section-inner` — constrains content width to 900px, centered
- `.logo-row` / `.logo-placeholder` — horizontal flex, monochrome accent styling
- `.contact-form`, `.form-field`, `input`/`textarea` — dark semi-transparent inputs with focus highlight
- `.contact-submit` — glassmorphism button (backdrop-filter: blur(12px), rgba whites)
- `.form-feedback`, `--success`, `--error` — green/red inline feedback states
- `.contact-mailto` — email link below form

Removed `min-height: 100vh` from `#clients` and `#contact` (was placeholder scroll space). Replaced with `padding: 6rem 2rem`.

### Task 2: initContactForm() JS function

Added `initContactForm()` after `setupLightboxListeners()` in the JS module block. The function:
- Guards with `if (!form) return` for safety
- Clears feedback and disables button at start of each submission (prevents stale state and double-submit)
- Calls `fetch(form.action, ...)` with `headers: { 'Accept': 'application/json' }` (required for Formspree JSON response)
- On success: shows green confirmation text, calls `form.reset()`
- On Formspree error: shows red text from `json.errors` or fallback
- On network catch: shows red network error text
- Restores button in `finally` block

Wired `initContactForm()` into `initScrollIntegration()` after `initProjectsShowcase()` and before `sectionObserver` setup.

## Deviations from Plan

None — plan executed exactly as written.

## Known Stubs

- `action="https://formspree.io/f/REPLACE_WITH_FORM_ID"` — the Formspree form ID is a placeholder. Form submission will return 404 until Tom creates a Formspree account and replaces this value. This is explicitly documented in the plan (Pitfall 3) and is a deployment prerequisite, not a code stub affecting rendering.
- Logo placeholder spans (Nike, Canal+, Dior, Red Bull, Ubisoft, Netflix) — intentional per D-04; Tom will provide real logo SVGs/images in a future update without requiring JS changes.

## Self-Check: PASSED

Files created/modified:
- FOUND: index.html (modified)

Commits:
- FOUND: 16a8d5b — feat(06-01): clients logo row and contact form — HTML markup + CSS
- FOUND: 7684fff — feat(06-01): initContactForm() — Formspree fetch submission with inline feedback

Key patterns verified:
- `logo-row` appears 2 times (CSS + HTML)
- `initContactForm` appears 2 times (definition + call)
- `min-height: 100vh` only appears once (for #hero, not #clients/#contact)
- `formspree.io` present in form action
- `mailto:contact@tombernys.com` present with email as link text
- `backdrop-filter: blur(12px)` in `.contact-submit`
- `aria-live="polite"` on `#form-feedback`
