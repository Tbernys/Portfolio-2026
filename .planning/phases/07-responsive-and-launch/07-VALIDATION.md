---
phase: 7
slug: responsive-and-launch
status: draft
nyquist_compliant: true
wave_0_complete: false
created: 2026-03-26
---

# Phase 7 — Validation Strategy

> Per-phase validation contract for feedback sampling during execution.

---

## Test Infrastructure

| Property | Value |
|----------|-------|
| **Framework** | Browser DevTools + Preview tools (no test framework — static site) |
| **Config file** | none — single index.html |
| **Quick run command** | `preview_snapshot` + `preview_console_logs` |
| **Full suite command** | `preview_resize` (375px) + `preview_snapshot` + `preview_console_logs` + `preview_inspect` |
| **Estimated runtime** | ~10 seconds |

---

## Sampling Rate

- **After every task commit:** Run `preview_snapshot` + `preview_console_logs`
- **After every plan wave:** Run full suite (resize to mobile, snapshot, check console, inspect CSS)
- **Before `/gsd:verify-work`:** Full suite must be green on both desktop and mobile viewports
- **Max feedback latency:** 10 seconds

---

## Per-Task Verification Map

| Task ID | Plan | Wave | Requirement | Test Type | Automated Command | File Exists | Status |
|---------|------|------|-------------|-----------|-------------------|-------------|--------|
| 07-01-01 | 01 | 1 | RESP-01, RESP-05 | visual | `grep -c "@media (max-width: 767px)" index.html` + `preview_resize(375,812)` + `preview_inspect --selector "body" --property "overflow-x"` | ✅ | ⬜ pending |
| 07-01-02 | 01 | 1 | RESP-02, RESP-03, RESP-04 | structural | `grep -c "isMobile ? 65" index.html` + `grep -c "antialias: !isMobile" index.html` + `grep -c "generateQuadTree(isMobile)" index.html` | ✅ | ⬜ pending |
| 07-02-01 | 02 | 2 | RESP-05 | deploy | `git remote get-url origin` + `git branch -r \| grep -c gh-pages` | ✅ | ⬜ pending |
| 07-02-02 | 02 | 2 | RESP-01, RESP-02, RESP-03, RESP-04, RESP-05 | visual+perf | `preview_resize(375,812)` + `preview_snapshot` + `preview_console_logs` | ✅ | ⬜ pending |

*Status: ⬜ pending · ✅ green · ❌ red · ⚠️ flaky*

---

## Wave 0 Requirements

*Existing infrastructure covers all phase requirements — no test framework needed for a single-file static site. Validation uses browser preview tools and visual inspection.*

---

## Manual-Only Verifications

| Behavior | Requirement | Why Manual | Test Instructions |
|----------|-------------|------------|-------------------|
| 30fps on real mid-range Android | RESP-03 | Requires physical device | Open site on Android, Chrome DevTools > Performance > record 5s scroll, verify avg fps >= 30 |
| 30fps on real iPhone | RESP-02 | Requires physical device | Open site on iPhone Safari, Web Inspector > Timeline > verify no dropped frames |
| DPR 1.5 cap on iPhone | RESP-02 | Requires Safari Web Inspector | Check `renderer.getPixelRatio()` returns <= 1.5 |
| Deploy on GitHub Pages | RESP-05 | Requires GitHub repo URL | Push to gh-pages branch, verify site loads at pages URL |

---

## Validation Sign-Off

- [x] All tasks have `<automated>` verify or Wave 0 dependencies
- [x] Sampling continuity: no 3 consecutive tasks without automated verify
- [x] Wave 0 covers all MISSING references
- [x] No watch-mode flags
- [x] Feedback latency < 10s
- [x] `nyquist_compliant: true` set in frontmatter

**Approval:** approved
