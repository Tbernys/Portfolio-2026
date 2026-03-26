# Phase 7: Responsive and Launch - Discussion Log

> **Audit trail only.** Do not use as input to planning, research, or execution agents.
> Decisions are captured in CONTEXT.md — this log preserves the alternatives considered.

**Date:** 2026-03-26
**Phase:** 07-responsive-and-launch
**Areas discussed:** Grille projets mobile, Breakpoints, Texte 3D mobile, 2e host déploiement

---

## Grille projets mobile

| Option | Description | Selected |
|--------|-------------|----------|
| 1 colonne | Chaque projet prend toute la largeur, scroll vertical, max visibilité | ✓ |
| 2 colonnes | Compromis densité/lisibilité | |
| 3 colonnes | Fidèle à Phase 5 D-01, mais ~115px par carte sur 375px | |

**User's choice:** 1 colonne
**Notes:** Override de Phase 5 D-01 qui imposait 3 colonnes même sur mobile — trop petit pour des thumbnails vidéo sur 375px

---

## Breakpoints & layout reflow

| Option | Description | Selected |
|--------|-------------|----------|
| Un seul à 768px | Desktop vs mobile, aligné avec isMobile JS existant | ✓ |
| Deux (768px + 480px) | Mobile + très petit écran | |
| Trois (1024px + 768px + 480px) | Tablet intermédiaire en plus | |

**User's choice:** Un seul breakpoint à 768px
**Notes:** Portfolio single-page ne justifie pas plus de complexité. Cohérent avec le seuil JS existant.

---

## Texte 3D mobile

| Option | Description | Selected |
|--------|-------------|----------|
| Réduire le scale | ~0.6-0.65x, proportion visuelle réduite | |
| Ajuster le FOV | Élargir le champ de vision de la caméra | |
| Les deux | Scale réduit + FOV ajusté pour résultat optimal | ✓ |
| Claude décide | Lisible et beau sur 375px, détails techniques secondaires | |

**User's choice:** Les deux (scale + FOV)
**Notes:** Le 0.85x actuel est insuffisant — besoin d'aller plus loin

---

## 2e host de déploiement

| Option | Description | Selected |
|--------|-------------|----------|
| GitHub Pages | Gratuit, gh-pages branch | ✓ |
| Netlify | Gratuit, drag & drop ou git deploy | |

**User's choice:** GitHub Pages
**Notes:** Vercel déjà fait en Phase 1. GitHub Pages comme 2e host pour satisfaire RESP-05.

---

## Claude's Discretion

- Valeurs exactes scale/FOV mobile
- Tile count reduction mobile
- Typography mobile adjustments
- Performance optimizations for 30fps target

## Deferred Ideas

None — discussion stayed within phase scope
