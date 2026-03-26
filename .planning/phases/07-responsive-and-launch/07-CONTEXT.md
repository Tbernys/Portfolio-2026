# Phase 7: Responsive and Launch - Context

**Gathered:** 2026-03-26
**Status:** Ready for planning

<domain>
## Phase Boundary

Mobile responsive adaptation de tout le site (layout, WebGL, typography) + audit performance sur vrais appareils + déploiement sur un 2e host statique. Chaque requirement v1 doit être satisfait sur desktop ET mobile avant mise en ligne.

</domain>

<decisions>
## Implementation Decisions

### Grille projets mobile (RESP-05)
- **D-01:** 1 colonne sur mobile (<768px) — chaque projet prend toute la largeur pour maximiser la visibilité des thumbnails vidéo
- **D-02:** Override de la décision Phase 5 D-01 (3 colonnes partout) — 3 colonnes reste sur desktop, 1 colonne sur mobile
- **D-03:** Le ratio 16:9 est conservé en 1 colonne

### Breakpoints (RESP-01, RESP-05)
- **D-04:** Un seul breakpoint à 768px — desktop (≥768px) vs mobile (<768px)
- **D-05:** Aligné avec le seuil `isMobile` JS existant (`window.innerWidth < 768`)
- **D-06:** Pas de breakpoint tablet intermédiaire — le portfolio single-page ne le justifie pas

### Texte 3D mobile (RESP-04)
- **D-07:** Combo scale réduit + FOV ajusté — les deux pour un résultat optimal sur 375px
- **D-08:** Le code existant fait `finalScale = isMobile ? scale * 0.85 : scale` (15% réduction) — insuffisant, besoin d'aller plus loin (~0.6-0.65x)
- **D-09:** FOV élargi sur mobile pour que le texte tienne naturellement dans le viewport

### WebGL mobile (RESP-02, RESP-03)
- **D-10:** DPR max 1.5 sur mobile — déjà implémenté (`maxDPR = isMobile ? 1.5 : 2`)
- **D-11:** Bloom 2 niveaux sur mobile — déjà implémenté (`isMobile ? 2 : 3`)
- **D-12:** Tile count et shader complexity à réduire davantage si nécessaire pour atteindre 30fps

### Layout reflow mobile (RESP-01, RESP-05)
- **D-13:** Toutes les sections content passent en single-column sur mobile
- **D-14:** Logos clients : rangée horizontale avec wrap ou réduction de taille
- **D-15:** Formulaire contact : déjà max-width 500px centré, devrait être OK sur mobile
- **D-16:** Section-inner (max-width 900px) s'adapte à 100% width sur mobile avec padding

### Déploiement 2e host (RESP-05)
- **D-17:** GitHub Pages comme 2e host (Vercel déjà fait)
- **D-18:** Deploy via `gh-pages` branch du repo GitHub

### Claude's Discretion
- Valeurs exactes de scale et FOV pour le glass text mobile
- Nombre de tiles réduit sur mobile si nécessaire pour 30fps
- Ajustements typographiques (font-size) pour mobile
- Padding/margins mobile
- Lightbox adaptation mobile
- Performance optimizations spécifiques pour atteindre 30fps
- Glassmorphism adjustments si backdrop-blur trop coûteux sur mobile

</decisions>

<canonical_refs>
## Canonical References

**Downstream agents MUST read these before planning or implementing.**

### Project architecture
- `.planning/PROJECT.md` — Vision, stack constraints (single HTML, vanilla JS, Three.js CDN)
- `.planning/REQUIREMENTS.md` — RESP-01 through RESP-05 acceptance criteria

### Prior phase context
- `.planning/phases/02-webgl-foundation/02-CONTEXT.md` — Device tier detection, GPU infrastructure
- `.planning/phases/03-hero-scene/03-CONTEXT.md` — Glass text appearance, tile background, post-processing
- `.planning/phases/05-project-showcase/05-CONTEXT.md` — Project grid layout decisions (D-01 overridden by D-02 here)
- `.planning/phases/06-content-sections/06-CONTEXT.md` — Contact form, client logos layout

### Existing responsive code
- `index.html` lines 783-794 — `gpuTier`, `isMobile`, `maxDPR` detection
- `index.html` line 1920 — bloom level mobile reduction
- `index.html` line 2155 — glass text mobile scale (0.85x, to be increased)

</canonical_refs>

<code_context>
## Existing Code Insights

### Reusable Assets
- `isMobile` flag — already detected via UA + width < 768, used throughout
- `gpuTier` detection — 3-tier system (0, 1, 2) already influences rendering
- `.content-section` + `.is-visible` pattern — all sections already use entrance animations
- `.section-inner` wrapper — max-width 900px container exists on clients/contact

### Established Patterns
- Conditional rendering via `isMobile` ternary — bloom levels, DPR, scale, sensitivity
- ScrollTrigger two-track architecture — CSS timeline + Three.js onUpdate
- Glassmorphism style — backdrop-blur, semi-transparent bg, used on CTA + project overlay + contact button

### Integration Points
- CSS: no `@media` queries exist yet — all responsive CSS is new
- JS: `isMobile` already threaded through all WebGL code
- Layout: `.section-inner`, `.projects-grid`, `#clients`, `#contact` all need media query overrides

</code_context>

<specifics>
## Specific Ideas

- La grille projets doit rester visuellement impactante sur mobile — 1 colonne pleine largeur avec les mêmes thumbnails 16:9
- Le glass text doit rester lisible et impressionnant sur 375px — pas juste "visible" mais "wow"
- Performance 30fps sur mid-range Android est le vrai test — pas juste iPhone

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope

</deferred>

---

*Phase: 07-responsive-and-launch*
*Context gathered: 2026-03-26*
