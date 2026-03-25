# Phase 5: Project Showcase - Discussion Log

> **Audit trail only.** Do not use as input to planning, research, or execution agents.
> Decisions are captured in CONTEXT.md — this log preserves the alternatives considered.

**Date:** 2026-03-25
**Phase:** 05-project-showcase
**Areas discussed:** Card layout, Thumbnail vs player, Hover interaction, Project metadata

---

## Card Layout

| Option | Description | Selected |
|--------|-------------|----------|
| Grille 2 colonnes | Grille régulière desktop 2 cols, 1 col mobile | |
| Empilé pleine largeur | Chaque projet prend toute la largeur | |
| Alternance gauche/droite | Zigzag player/texte | |
| Grille 3 colonnes (user) | 3 colonnes, même sur mobile | ✓ |

**User's choice:** Grille sur 3 colonnes, qui reste sur 3 colonnes sur mobile
**Notes:** Custom answer — user wants consistent 3-col layout across all breakpoints

| Option | Description | Selected |
|--------|-------------|----------|
| Titre visible | Titre "Projets" au-dessus de la grille | ✓ |
| Pas de titre | Grille directe après hero | |
| Claude décide | | |

| Option | Description | Selected |
|--------|-------------|----------|
| 16:9 (cinématique) | Format vidéo standard | ✓ |
| Carré (1:1) | Style Instagram | |
| 4:5 (portrait) | Plus haut que large | |

| Option | Description | Selected |
|--------|-------------|----------|
| Serré (8-12px) | Cartes quasi collées, mosaïque dense | ✓ |
| Moyen (20-30px) | Espacement confortable | |
| Aéré (40-60px) | Beaucoup d'espace | |
| Claude décide | | |

| Option | Description | Selected |
|--------|-------------|----------|
| Contenue (max-width) | Grille centrée avec marges | |
| Pleine largeur | Bord à bord | ✓ |
| Claude décide | | |

| Option | Description | Selected |
|--------|-------------|----------|
| Angles droits | Bords francs, cohérent CRT/glitch | ✓ |
| Légèrement arrondis | Petit radius (4-8px) | |
| Claude décide | | |

| Option | Description | Selected |
|--------|-------------|----------|
| 6 projets (2 rangées) | Compact | |
| 9 projets (3 rangées) | Grille 3×3 | ✓ |
| Claude décide | | |

---

## Thumbnail vs Player

| Option | Description | Selected |
|--------|-------------|----------|
| Thumbnail → player au clic | Image fixe, clic charge le player | |
| Player embedé lazy-load | Player auto quand visible | |
| Thumbnail → player au hover | Preview vidéo muette au survol, effet Netflix | ✓ |

| Option | Description | Selected |
|--------|-------------|----------|
| Thumbnail Vimeo auto | Via API oEmbed | ✓ |
| Image custom hardcodée | Image spécifique par projet | |
| Claude décide | | |

| Option | Description | Selected |
|--------|-------------|----------|
| Plein écran Vimeo | Clic lance fullscreen natif | |
| Agrandissement in-page | Modal lightbox avec player | ✓ |
| Lecture dans la carte | Vidéo continue dans la carte | |

| Option | Description | Selected |
|--------|-------------|----------|
| Player seul | Modal ne montre que le player | |
| Player + métadonnées | Titre, rôle, client sous le player | ✓ |
| Claude décide | | |

| Option | Description | Selected |
|--------|-------------|----------|
| Croix + clic extérieur + Escape | 3 méthodes classiques | |
| Bouton X + clic fond sombre (user) | 2 méthodes sélectionnées | ✓ |

**Notes:** User selected subset — X button and backdrop click. Did not explicitly mention Escape key.

---

## Hover Interaction

| Option | Description | Selected |
|--------|-------------|----------|
| Scale + élévation | Carte grossit avec ombre | |
| Overlay titre/infos | Overlay semi-transparent avec titre | ✓ |
| Bordure lumineuse | Contour glow | |
| Minimal (cursor change) | Juste le curseur qui change | |

| Option | Description | Selected |
|--------|-------------|----------|
| Gradient bas sombre | Dégradé noir en bas | |
| Fond semi-transparent uniforme | Voile sombre sur toute la carte | |
| Glassmorphism (flou) | Bande floutée backdrop-blur en bas | ✓ |

**Notes:** Cohérent avec le bouton CTA glassmorphism de Phase 4

| Option | Description | Selected |
|--------|-------------|----------|
| Toujours visibles | Overlay permanent sur mobile | |
| Tap pour révéler | 1er tap → overlay, 2e tap → lightbox | ✓ |
| Claude décide | | |

---

## Project Metadata

| Option | Description | Selected |
|--------|-------------|----------|
| Titre du projet | Nom du projet | ✓ |
| Rôle(s) / crédits | Direction, montage, motion | ✓ |
| Client / marque | Nom du client | ✓ |
| Année | Année de réalisation | |

**Notes:** Pas d'année — 3 infos par projet

| Option | Description | Selected |
|--------|-------------|----------|
| Tout dans l'overlay | Titre + rôle + client dans glassmorphism | |
| Titre + rôle dans overlay (user) | Client réservé à la lightbox | ✓ |

| Option | Description | Selected |
|--------|-------------|----------|
| Sous le player | Métadonnées sous le player dans la modal | ✓ |
| À côté du player | Layout horizontal dans la modal | |
| Claude décide | | |

---

## Claude's Discretion

- Taille et police du titre de section
- Hauteur de la bande glassmorphism
- Animation lightbox (ouverture/fermeture)
- Style du bouton X
- Taille du player dans la lightbox
- Délai avant chargement de la preview hover
- Espacement exact (8-12px)
- Ajout Escape key pour a11y

## Deferred Ideas

None — discussion stayed within phase scope
