---
phase: 6
slug: content-sections
status: draft
nyquist_compliant: false
wave_0_complete: false
created: 2026-03-26
---

# Phase 6 — Validation Strategy

> Per-phase validation contract for feedback sampling during execution.

---

## Test Infrastructure

| Property | Value |
|----------|-------|
| **Framework** | None — single static HTML file, no automated test runner |
| **Config file** | none |
| **Quick run command** | Open `index.html` in browser, scroll to section |
| **Full suite command** | Manual smoke test checklist (see below) |
| **Estimated runtime** | ~60 seconds manual |

---

## Sampling Rate

- **After every task commit:** Visual check in browser (scroll to section, confirm render)
- **After every plan wave:** Full smoke checklist below
- **Before `/gsd:verify-work`:** All checklist items green
- **Max feedback latency:** ~60 seconds (page reload + scroll)

---

## Per-Task Verification Map

| Task ID | Plan | Wave | Requirement | Test Type | Automated Command | File Exists | Status |
|---------|------|------|-------------|-----------|-------------------|-------------|--------|
| 06-01-01 | 01 | 1 | CLNT-01 | manual smoke | Scroll to #clients, confirm section visible | N/A | ⬜ pending |
| 06-01-02 | 01 | 1 | CLNT-02 | manual smoke | Inspect .logo-row layout at >=1024px | N/A | ⬜ pending |
| 06-01-03 | 01 | 1 | CTCT-01 | manual smoke | Fill form, submit, check Network tab for POST to formspree.io | N/A | ⬜ pending |
| 06-01-04 | 01 | 1 | CTCT-02 | manual smoke | Click mailto link, confirm mail client opens | N/A | ⬜ pending |
| 06-01-05 | 01 | 1 | CTCT-03 | manual smoke | Submit valid form → green feedback; block formspree → red feedback | N/A | ⬜ pending |

*Status: ⬜ pending · ✅ green · ❌ red · ⚠️ flaky*

---

## Wave 0 Requirements

- [ ] Formspree account created, form registered for contact@tombernys.com, FORM_ID recorded

*No test framework infrastructure needed — project has no automated test runner.*

---

## Manual-Only Verifications

| Behavior | Requirement | Why Manual | Test Instructions |
|----------|-------------|------------|-------------------|
| Logo section visible on scroll | CLNT-01 | Visual check — no DOM test runner | Load page, scroll to #clients, confirm entrance animation fires |
| Logos in horizontal row | CLNT-02 | Layout check — needs visual viewport | Inspect at >=1024px, confirm flex row |
| Form submits to Formspree | CTCT-01 | Network check — requires real HTTP | Fill form, submit, verify POST in Network tab |
| mailto link works | CTCT-02 | OS integration — opens mail client | Click link, confirm OS mail client opens |
| Success feedback | CTCT-03 | Visual check | Submit valid form, confirm green text appears |
| Error feedback | CTCT-03 | Network block | DevTools → block formspree.io, submit, confirm red text |

---

## Validation Sign-Off

- [ ] All tasks have manual verification instructions
- [ ] Sampling continuity: visual check after every task commit
- [ ] Wave 0 covers Formspree FORM_ID dependency
- [ ] No watch-mode flags
- [ ] Feedback latency < 60s
- [ ] `nyquist_compliant: true` set in frontmatter

**Approval:** pending
