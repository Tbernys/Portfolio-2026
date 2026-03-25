# Phase 5: Project Showcase - Context

**Gathered:** 2026-03-25
**Status:** Ready for planning

<domain>
## Phase Boundary

Section showcase de projets vidéo : une grille 3 colonnes de 9 projets avec thumbnails Vimeo, preview vidéo au hover, overlay glassmorphism, et modal lightbox pour la lecture complète. Lazy-loading des players via IntersectionObserver.

</domain>

<decisions>
## Implementation Decisions

### Layout (grille)
- **D-01:** Grille 3 colonnes — même sur mobile (pas de reflow en 1 colonne)
- **D-02:** 9 projets au total (3 rangées de 3)
- **D-03:** Ratio 16:9 (cinématique) pour chaque carte
- **D-04:** Espacement serré entre les cartes (8-12px gap)
- **D-05:** Pleine largeur (bord à bord, pas de max-width centré)
- **D-06:** Angles droits sur les cartes (pas d'arrondi) — cohérent avec l'esthétique CRT/glitch
- **D-07:** Titre de section visible ("Projets" ou similaire) au-dessus de la grille

### Thumbnail & Player
- **D-08:** Thumbnail par défaut récupérée automatiquement via l'API Vimeo (oEmbed)
- **D-09:** Au hover : la thumbnail est remplacée par une preview vidéo Vimeo (muted, autoplay) — effet "Netflix"
- **D-10:** Au clic : ouverture d'une modal lightbox in-page avec le player Vimeo agrandi
- **D-11:** La lightbox affiche le player + métadonnées (titre, rôle, client) en dessous du player
- **D-12:** Fermeture de la lightbox : bouton X + clic sur le fond sombre (pas de touche Escape mentionnée)

### Hover interaction
- **D-13:** Overlay glassmorphism (bande floutée backdrop-blur) en bas de la carte au hover — cohérent avec le bouton CTA glassmorphism de Phase 4
- **D-14:** L'overlay affiche le titre du projet et le rôle (pas le client)
- **D-15:** Sur mobile (pas de hover) : premier tap révèle l'overlay, deuxième tap ouvre la lightbox

### Métadonnées projet
- **D-16:** Informations par projet : titre, rôle(s)/crédits, client/marque (pas d'année)
- **D-17:** Titre + rôle visibles dans l'overlay glassmorphism au hover
- **D-18:** Client visible uniquement dans la lightbox (pas dans l'overlay)
- **D-19:** Dans la lightbox, les métadonnées sont affichées sous le player

### Claude's Discretion
- Taille et police du titre de section
- Hauteur exacte de la bande glassmorphism
- Animation d'ouverture/fermeture de la lightbox
- Style du bouton X de fermeture
- Taille du player dans la lightbox
- Traitement de la preview vidéo au hover (délai avant chargement, transition)
- Espacement exact dans le gap (entre 8-12px)
- Ajout de la touche Escape pour fermer la lightbox (bonne pratique a11y)

</decisions>

<canonical_refs>
## Canonical References

**Downstream agents MUST read these before planning or implementing.**

### Project architecture
- `.planning/PROJECT.md` — Vision, stack constraints (single HTML, vanilla JS, Three.js CDN)
- `.planning/REQUIREMENTS.md` — PROJ-01 through PROJ-04 acceptance criteria
- `.planning/ROADMAP.md` §Phase 5 — Success criteria and dependency on Phase 4

### Prior phase context
- `.planning/phases/04-scroll-integration/04-CONTEXT.md` — Scroll behavior, content section entrance animations, film grain background, glassmorphism CTA pattern

### Technical references
- `index.html` — Existing #projects section placeholder (line 375), IntersectionObserver pattern for section entrances, content-section CSS, import map (GSAP, Three.js)

</canonical_refs>

<code_context>
## Existing Code Insights

### Reusable Assets
- **IntersectionObserver pattern**: Already used for content section entrance animations — can be extended for lazy-loading Vimeo players
- **Glassmorphism CSS**: CTA button `.cta-btn` uses backdrop-blur + semi-transparent background — reusable pattern for card overlay
- **Content section entrance**: `.content-section` + `.is-visible` CSS pattern with fade-in + slide-up already wired
- **GSAP + ScrollTrigger**: Available in import map, registered in JS

### Established Patterns
- **Single HTML file**: All CSS inline in `<style>`, all JS inline in `<script type="module">`
- **Dark background**: `#050508` with film grain overlay on `.content-bg`
- **No framework**: Vanilla JS, DOM manipulation direct
- **CDN imports**: Three.js and GSAP via import map — `@vimeo/player` will need adding

### Integration Points
- `#projects` section (line 375): Replace placeholder `<h2>Projets</h2>` with full grid
- `<main class="content-bg">`: Projects section lives inside this wrapper
- Import map (line 9): Add `@vimeo/player` CDN entry
- Section order: projects → clients → contact (already in HTML)

</code_context>

<specifics>
## Specific Ideas

- L'overlay glassmorphism doit être cohérent avec le bouton "Voir mon travail" de Phase 4 — même langage visuel
- L'effet "Netflix" au hover : la thumbnail se transforme en preview vidéo muette — le visiteur voit le travail vivre avant même de cliquer
- Grille bord à bord, serrée, angles droits : l'esthétique est brute et dense, pas aérée
- Le double-tap mobile (reveal puis open) évite les ouvertures accidentelles de lightbox

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope

</deferred>

---

*Phase: 05-project-showcase*
*Context gathered: 2026-03-25*
