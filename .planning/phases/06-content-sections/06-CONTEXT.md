# Phase 6: Content Sections - Context

**Gathered:** 2026-03-26
**Status:** Ready for planning

<domain>
## Phase Boundary

Section logos clients et section contact : logos en grille horizontale, formulaire de contact (Formspree), lien email direct, feedback de soumission. Cohérent avec l'esthétique dark/CRT des phases précédentes.

</domain>

<decisions>
## Implementation Decisions

### Logos clients (CLNT-01, CLNT-02)
- **D-01:** Logos affichés en rangée horizontale unique (pas de grille multi-lignes)
- **D-02:** Logos en SVG inline ou images haute qualité — monochromes (blancs/accent) pour cohérence avec le thème dark
- **D-03:** Titre de section visible ("Clients" ou "Ils m'ont fait confiance")
- **D-04:** Logos placeholder — Tom fournira les vrais logos plus tard ; utiliser des rectangles stylisés ou le nom texte en attendant
- **D-05:** Pas d'animation au hover sur les logos — simple et sobre
- **D-06:** Entrée animée cohérente avec le pattern `.content-section` + `.is-visible` existant

### Formulaire de contact (CTCT-01, CTCT-03)
- **D-07:** Backend = Formspree (fonctionne sur n'importe quel hébergeur statique, contrairement à Netlify Forms)
- **D-08:** Champs : nom, email, message — pas de téléphone, pas de sujet
- **D-09:** Soumission via fetch() (pas de redirect) — afficher success/error inline
- **D-10:** Message de succès : texte de confirmation visible dans le formulaire
- **D-11:** Message d'erreur : texte d'erreur visible si le fetch échoue
- **D-12:** Validation HTML5 basique (required, type="email") — pas de validation JS custom

### Email direct (CTCT-02)
- **D-13:** Lien mailto: visible en dessous ou à côté du formulaire
- **D-14:** Texte du lien = l'adresse email elle-même (pas "Envoyez-moi un email")
- **D-15:** Email = contact@tombernys.com

### Layout contact
- **D-16:** Section contact sur fond dark cohérent (#050508)
- **D-17:** Formulaire centré, largeur max ~500px
- **D-18:** Style des inputs cohérent avec l'esthétique : fond semi-transparent, bordure subtile, texte clair
- **D-19:** Bouton d'envoi avec style glassmorphism (cohérent avec le CTA "Voir mon travail" de Phase 4)

### Claude's Discretion
- Nombre et disposition exacte des logos placeholder
- Espacement entre les logos
- Taille et police du titre de section logos
- Animation d'entrée spécifique pour chaque section
- Style exact des champs de formulaire (border-radius, padding, focus state)
- Icône ou non sur le bouton d'envoi
- Espacement entre le formulaire et le lien mailto

</decisions>

<canonical_refs>
## Canonical References

**Downstream agents MUST read these before planning or implementing.**

### Project architecture
- `.planning/PROJECT.md` — Vision, stack constraints (single HTML, vanilla JS, Three.js CDN)
- `.planning/REQUIREMENTS.md` — CLNT-01, CLNT-02, CTCT-01, CTCT-02, CTCT-03 acceptance criteria
- `.planning/ROADMAP.md` §Phase 6 — Success criteria and dependency on Phase 5

### Prior phase context
- `.planning/phases/05-project-showcase/05-CONTEXT.md` — Grid layout, glassmorphism pattern, section entrance animations
- `.planning/phases/04-scroll-integration/04-CONTEXT.md` — CTA glassmorphism style, content section entrance pattern, film grain background

### Technical references
- `index.html` — Existing `#clients` section (line 602), `#contact` section (line 607), `.content-section` CSS pattern, glassmorphism `.cta-btn` CSS, import map

</canonical_refs>

<code_context>
## Existing Code Insights

### Reusable Assets
- **Content section entrance**: `.content-section` + `.is-visible` CSS pattern with fade-in + slide-up already wired via IntersectionObserver
- **Glassmorphism CSS**: `.cta-btn` uses `backdrop-filter: blur()` + semi-transparent background — reuse for submit button
- **Film grain background**: `.content-bg::before` already applied to all content sections
- **Dark background**: `#050508` consistent across all content sections

### Established Patterns
- **Single HTML file**: All CSS inline in `<style>`, all JS inline in `<script type="module">`
- **Section structure**: `<section id="..." class="content-section" data-lang="fr" aria-label="...">`
- **No framework**: Vanilla JS, direct DOM manipulation
- **CDN imports**: Via import map — no new dependencies needed for Phase 6

### Integration Points
- `#clients` section (line 602): Replace placeholder with logo row
- `#contact` section (line 607): Replace placeholder with form + mailto link
- Section order: projects → clients → contact (already in HTML)
- IntersectionObserver for section entrances already covers these sections

</code_context>

<specifics>
## Specific Ideas

- Logos monochromes (blancs ou accent #D4C5B2) sur fond dark — pas de logos couleur qui casseraient l'esthétique
- Formulaire minimaliste — 3 champs max, pas de fioritures, cohérent avec le ton sobre du site
- Glassmorphism sur le bouton d'envoi pour le lien visuel avec le reste du site
- Formspree plutôt que Netlify Forms car le site doit rester portable (Vercel, GitHub Pages, etc.)

</specifics>

<deferred>
## Deferred Ideas

- Social links (Vimeo, Instagram, LinkedIn) — v2 requirement CONT-01, not in v1 scope
- Animated logo marquee/carousel — over-engineering for a static row of logos

</deferred>

---

*Phase: 06-content-sections*
*Context gathered: 2026-03-26*
