---
phase: 5
slug: project-showcase
status: draft
nyquist_compliant: false
wave_0_complete: false
created: 2026-03-25
---

# Phase 5 — Validation Strategy

> Per-phase validation contract for feedback sampling during execution.

---

## Test Infrastructure

| Property | Value |
|----------|-------|
| **Framework** | Browser-based manual + CLI verification |
| **Config file** | none — single HTML file, no test framework |
| **Quick run command** | `grep -c 'project-card' index.html` |
| **Full suite command** | `python3 -m http.server 8080 & sleep 1 && curl -s http://localhost:8080 | grep -o 'project-card' | wc -l` |
| **Estimated runtime** | ~3 seconds |

---

## Sampling Rate

- **After every task commit:** Run `grep -c 'project-card' index.html`
- **After every plan wave:** Visual browser check + network tab verification
- **Before `/gsd:verify-work`:** Full visual + network audit
- **Max feedback latency:** 5 seconds

---

## Per-Task Verification Map

| Task ID | Plan | Wave | Requirement | Test Type | Automated Command | File Exists | Status |
|---------|------|------|-------------|-----------|-------------------|-------------|--------|
| TBD | TBD | TBD | PROJ-01 | grep | `grep 'project-card' index.html` | ❌ W0 | ⬜ pending |
| TBD | TBD | TBD | PROJ-02 | grep | `grep 'IntersectionObserver' index.html` | ❌ W0 | ⬜ pending |
| TBD | TBD | TBD | PROJ-03 | grep | `grep 'mouseenter\|hover' index.html` | ❌ W0 | ⬜ pending |
| TBD | TBD | TBD | PROJ-04 | grep | `grep 'vimeo.com/video' index.html` | ❌ W0 | ⬜ pending |

*Status: ⬜ pending · ✅ green · ❌ red · ⚠️ flaky*

---

## Wave 0 Requirements

*Existing infrastructure covers all phase requirements — single HTML file with inline JS, no test framework needed.*

---

## Manual-Only Verifications

| Behavior | Requirement | Why Manual | Test Instructions |
|----------|-------------|------------|-------------------|
| Vimeo lazy-loading | PROJ-02 | Network tab needed | Open DevTools Network, load page, verify no Vimeo requests until scroll |
| Hover animation | PROJ-03 | Visual interaction | Hover project card, verify glassmorphism overlay appears |
| Vimeo playback | PROJ-04 | Player interaction | Click card, verify video plays in lightbox |
| Mobile double-tap | PROJ-03 | Touch device needed | First tap reveals overlay, second tap opens lightbox |

---

## Validation Sign-Off

- [ ] All tasks have `<automated>` verify or Wave 0 dependencies
- [ ] Sampling continuity: no 3 consecutive tasks without automated verify
- [ ] Wave 0 covers all MISSING references
- [ ] No watch-mode flags
- [ ] Feedback latency < 5s
- [ ] `nyquist_compliant: true` set in frontmatter

**Approval:** pending
