# Feature Research

**Domain:** Creative portfolio — freelance video editor / motion designer, WebGL single-page site
**Researched:** 2026-03-22
**Confidence:** MEDIUM (features grounded in multiple WebSearch sources; some findings cross-verified against industry references)

---

## Feature Landscape

### Table Stakes (Users Expect These)

Features every prospective client assumes exist. Missing these = site feels amateur or broken.

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| Showreel / demo reel in hero | Clients decide in 10 seconds whether to keep scrolling; reel must be the first thing seen | MEDIUM | Autoplay, muted, looping. For this site the WebGL hero IS the brand statement — reel lives in the project section below. The hero visual replaces the traditional autoplay video. |
| Project showcase (6–10 works) | Portfolio without work samples is meaningless; clients expect curated, quality-over-quantity selection | MEDIUM | Vimeo embeds via iframe. Thumbnails must be high-quality; lazy-load iframes so initial page load is not blocked by 10 simultaneous embed requests. |
| Clear contact path (email + form) | Clients need a frictionless way to reach out; hidden contact = lost jobs | LOW | Both email link and form; form doubles conversion vs email-only. Static site constraint means form needs a third-party backend (Netlify Forms, Formspree, or mailto fallback). |
| Responsive design (mobile + desktop) | >50% of portfolio views happen on mobile; broken layout loses clients | HIGH | WebGL needs allege mode on mobile: reduced DPR (max 1.5), simplified fluid, fewer bloom passes. Layout must reflow gracefully for all screen widths. |
| Social / professional links | Instagram, LinkedIn, Vimeo profile expected for freelance credibility | LOW | Footer or contact section. Vimeo profile link especially important in motion design. |
| Fast enough initial load | Slow load = bounce before hero even renders; heavy WebGL sites must use skeleton/loader | MEDIUM | Glitch loader in project spec serves this purpose. Target: interactive within 3–5s on desktop broadband. |
| Dark aesthetic throughout | Motion design / video work always shown against dark backgrounds — industry standard | LOW | Already decided in PROJECT.md. Light theme would make the work look washed out. |
| Open Graph / meta tags | Clients share portfolio links; unfurled preview matters on Slack, WhatsApp, email | LOW | Single HTML file — add og:title, og:description, og:image (static preview screenshot), og:url. |

### Differentiators (Competitive Advantage)

Features that set this portfolio apart. Not baseline requirements, but the reason clients remember and share it.

| Feature | Value Proposition | Complexity | Notes |
|---------|-------------------|------------|-------|
| WebGL 3D glass name hero (TextGeometry + glass shader) | Immediately signals technical sophistication and premium craft; most competitors use static text or simple CSS animations | HIGH | Three.js TextGeometry + custom refraction/transmission shader. This is the signature brand statement — every interaction should reinforce "this person makes things look incredible." |
| GPU fluid simulation (Navier-Stokes, mouse/touch interactive) | Creates a "living" hero that reacts to the visitor; sticks in memory; referenced PromptHQ as benchmark | HIGH | Encode velocity + dye fields as render targets; advect, diffuse, pressure-solve. Performance gate required: disable or reduce on low-end GPU. |
| Instanced cylindrical tile background with bloom | Adds depth and texture to hero without relying on stock assets; procedural = unique to this site | HIGH | Instanced mesh with procedural shader; bloom post-processing pipeline. |
| Glitch-style loading screen with character reveal | Sets the tone before WebGL renders; prevents blank-screen anxiety during asset load; memorable first impression | MEDIUM | Character-by-character text scramble → resolve to name. Classic technique in motion design portfolios. |
| Scroll-triggered hero-to-content transition | Smooth cinematic handoff from WebGL world to project gallery; feels like a film cut, not a webpage jump | MEDIUM | Scroll listener drives opacity/scale of WebGL canvas; content fades in beneath. GSAP ScrollTrigger or IntersectionObserver. |
| Per-project hover reveal / thumbnail motion | Gives the gallery life before a Vimeo embed loads; differentiates from static grid | MEDIUM | CSS or JS-driven hover: title reveal, subtle scale, or color shift on thumbnail. Does not require WebGL. |
| Client / brand logos section | Social proof; signals professional-grade work for recognizable brands | LOW | SVG logos preferred (scalable, small). Horizontal scroll or static grid. Industry-standard for freelance motion designers. |

### Anti-Features (Commonly Requested, Often Problematic)

Features that seem desirable but create disproportionate cost or undermine the site's goals.

| Feature | Why Requested | Why Problematic | Alternative |
|---------|---------------|-----------------|-------------|
| CMS / admin panel for content updates | "I want to update projects without touching code" | Breaks the single-HTML-file constraint; adds backend complexity, hosting requirements, auth, and ongoing maintenance. For 6–10 projects that rarely change, the ROI is near zero. | Hardcode projects in HTML/JS. When projects change, do a targeted edit. Document the data structure clearly so edits take <10 minutes. |
| Contact form with server-side processing | Seems professional and necessary | Requires a backend or serverless function; adds deployment complexity. Risk: form breaks silently, leads lost. | Use Netlify Forms (zero-config for static HTML) or Formspree (hosted endpoint). Both handle spam filtering, email forwarding, and work with static files. |
| Password-protected case study pages | "Some work is NDA" | Multi-page routing breaks the single-page architecture. Password UX is friction for legitimate clients. | Show NDA projects as locked thumbnails with "Available on request" text. Clients email to ask. |
| Blog / articles section | "Good for SEO" | Content marketing requires ongoing effort; a stale blog hurts credibility more than no blog. Motion design work speaks louder than blog posts. | Let the portfolio and external profiles (Vimeo, Instagram) do SEO work. A single well-crafted About blurb is enough long-form content. |
| Custom video player (replacing Vimeo) | "Full control over player UI" | Self-hosting video requires CDN, encoding pipeline, bandwidth costs, and playback reliability work. Vimeo provides all of this plus HLS adaptive streaming, analytics, and privacy controls for free. | Keep Vimeo. Use Vimeo's embed API for custom controls if needed (play/pause, no branding on Pro accounts). |
| Chat widget / live chat | "Instant communication" | Interrupts the visual experience; chat services load heavy scripts; creates expectation of real-time response that a solo freelancer can't maintain | Contact form + email link is sufficient; add response time expectation in the form ("I respond within 24h"). |
| Cursor replacement (custom CSS cursor) | Popular in creative portfolios, looks impressive in screenshots | Degrades perceived performance; cursor lag feels broken on anything but high-end machines; can interfere with native touch behavior on mobile | If cursor customization is desired, limit to subtle trail effect using requestAnimationFrame rather than CSS cursor swap. Low priority. |
| Horizontal scroll for project gallery | "More cinematic, like Justin Buisson reference" | Significantly harder to make accessible and mobile-friendly; scroll hijacking frustrates users on trackpads; adds complexity without clear benefit for a video-first portfolio | Vertical scroll is the correct choice for this project (already decided in PROJECT.md). Projects stack naturally in vertical flow. |
| Preloader that blocks all content | "Ensures assets are ready before showing anything" | If load takes >3s, users bounce. A glitch loader that shows immediately while WebGL initialises in background is better than a blocking white screen. | Use the glitch text reveal as the loader — it IS the content, not a waiting screen. Progressive enhancement: show HTML sections even if WebGL is still loading. |

---

## Feature Dependencies

```
[WebGL Hero]
    └──requires──> [Three.js + post-processing pipeline]
                       └──requires──> [CDN import maps setup]

[Glass name (TextGeometry)]
    └──requires──> [WebGL Hero canvas]
    └──requires──> [Font JSON (typeface.js format)]

[GPU Fluid Simulation]
    └──requires──> [WebGL Hero canvas]
    └──requires──> [Render target / FBO support]

[Glitch Loader]
    └──precedes──> [WebGL Hero] (loader must show while WebGL initialises)

[Scroll Transition (hero → content)]
    └──requires──> [WebGL Hero]
    └──requires──> [Content sections exist in DOM]

[Project Showcase (Vimeo embeds)]
    └──requires──> [Content section structure]
    └──enhances──> [Scroll Transition] (destination for the hero-to-content handoff)

[Project Thumbnails / Hover State]
    └──enhances──> [Project Showcase]

[Client Logos Section]
    └──no dependencies] (standalone HTML/CSS section)

[Contact Section]
    └──requires──> [Form backend (Netlify Forms / Formspree)] OR [mailto fallback]

[Mobile Allege Mode]
    └──requires──> [WebGL Hero] (graceful degradation path must be coded alongside the hero, not after)
    └──enhances──> [Responsive Design]

[Open Graph Meta Tags]
    └──requires──> [Static screenshot / og:image asset]
```

### Dependency Notes

- **Glitch Loader precedes WebGL Hero:** The loader must be visible the instant HTML parses — before Three.js is even imported. Build the loader with zero JS dependencies or minimal inline script, then kick off WebGL initialisation in parallel.
- **Mobile Allege requires WebGL Hero:** Do not build the hero first and add mobile fallback later. The device capability check (GPU tier, DPR, touch detection) must be part of the initial render loop, or you will need a full refactor.
- **Font JSON requires sourcing upfront:** TextGeometry requires a typeface.js-format JSON font. If the chosen typeface is not available in this format, conversion is a blocker. Resolve font choice before building the glass name shader.
- **Contact form backend is independent of everything else:** Can be added or swapped at any time. Use mailto as a fallback during development; upgrade to Netlify Forms or Formspree before launch.

---

## MVP Definition

### Launch With (v1)

Minimum viable product — what validates the site as a professional-grade portfolio.

- [ ] WebGL hero with glass name + fluid simulation + tile background — the core brand statement; without this the site is just another portfolio
- [ ] Glitch loading screen — required to mask WebGL initialisation delay; prevents blank-screen first impression
- [ ] Project showcase (6–10 Vimeo embeds) — the actual evidence of skill; nothing else matters if the work isn't visible
- [ ] Contact section (email link + Formspree/Netlify form) — the entire purpose of the site is to generate inquiries
- [ ] Client logos section — social proof that requires minimal implementation time for significant credibility gain
- [ ] Responsive layout with WebGL mobile allege — majority of portfolio views are mobile; broken mobile = lost clients
- [ ] Open Graph meta tags + page title — every shared link needs a proper preview

### Add After Validation (v1.x)

Features to add once the core is live and working.

- [ ] Per-project hover / thumbnail reveal animations — adds delight to the gallery but is not required for it to function
- [ ] Scroll-triggered micro-animations on section entry (fade-in, translate-up) — polish pass; adds perceived quality
- [ ] Vimeo player API integration for custom play/pause controls — only if Vimeo's default embed UI feels off-brand

### Future Consideration (v2+)

Features to defer until there is a clear reason to add them.

- [ ] Analytics integration (Plausible / Fathom) — useful but adds a script tag and privacy consideration; defer until there is traffic worth measuring
- [ ] Custom project detail sections — inline case study expansions below each embed; adds significant content authoring work; only valuable if clients request more context
- [ ] Dark/light mode toggle — conflicts with the deliberately dark aesthetic; revisit only if feedback suggests demand

---

## Feature Prioritization Matrix

| Feature | User Value | Implementation Cost | Priority |
|---------|------------|---------------------|----------|
| WebGL hero (glass name + fluid) | HIGH | HIGH | P1 |
| Project showcase (Vimeo embeds) | HIGH | MEDIUM | P1 |
| Contact section | HIGH | LOW | P1 |
| Glitch loader | HIGH | MEDIUM | P1 |
| Mobile responsive + WebGL allege | HIGH | HIGH | P1 |
| Open Graph meta tags | MEDIUM | LOW | P1 |
| Client logos section | MEDIUM | LOW | P1 |
| Scroll hero-to-content transition | MEDIUM | MEDIUM | P2 |
| Hover reveal on project thumbnails | MEDIUM | LOW | P2 |
| Post-processing pipeline (bloom, vignette) | MEDIUM | MEDIUM | P2 |
| Scroll-triggered section animations | LOW | LOW | P2 |
| Vimeo API custom controls | LOW | MEDIUM | P3 |
| Analytics | LOW | LOW | P3 |
| Inline case study expansions | LOW | HIGH | P3 |

**Priority key:**
- P1: Must have for launch
- P2: Should have, add when possible
- P3: Nice to have, future consideration

---

## Competitor Feature Analysis

| Feature | Justin Buisson (justinbuisson.com) | PromptHQ (prompthq.fr) | Tom Bernys Approach |
|---------|-----------------------------------|-----------------------|---------------------|
| Hero | Minimalist white, static typography, horizontal scroll entry | Full WebGL: 3D glass logo, fluid sim, cylindrical tiles, bloom | Full WebGL dark hero (PromptHQ-level), vertical scroll |
| Project showcase | Full-bleed project pages, one per page | Not a portfolio showcase site | Vertical scrolling grid of 6–10 Vimeo embeds |
| Video hosting | Not applicable (visual design portfolio) | Not applicable | Vimeo embeds (Pro quality, no ads) |
| Navigation | Minimal: Work + About only | Single-page, no nav | Single-page, anchor scroll, minimal nav |
| Loading experience | Instant (static, no WebGL) | Glitch/scramble text reveal | Glitch character-by-character reveal |
| Mobile | ReadyMag handles it | Simplified allege mode | WebGL allege: reduced DPR, fewer bloom passes |
| Contact | Simple email | Not a client-facing portfolio | Email link + contact form |
| Client logos | Not present | Not present | Dedicated logos section for social proof |
| SEO | Standard | Minimal (single HTML file) | Meta tags, OG image, semantic HTML landmarks |

---

## Sources

- Tobias van Schneider — Dos and don'ts for motion design portfolios: https://vanschneider.com/blog/portfolio-tips/motion-design-portfolio-tips/
- Vimeo — Video editor portfolio guide: https://vimeo.com/blog/post/video-editor-portfolio
- Fueler.io — How to build a standout video editing portfolio 2025: https://fueler.io/blog/how-to-build-a-standout-video-editing-portfolio
- Codrops — Letting the creative process shape a WebGL portfolio (2025): https://tympanus.net/codrops/2025/11/27/letting-the-creative-process-shape-a-webgl-portfolio/
- TakeFlyte — Contact form vs email link: https://www.takeflyte.com/contact-form-vs-email-link-which-is-better
- Matt Olpinski — Supercharge your portfolio contact form: https://mattolpinski.com/articles/supercharge-your-website-contact-form/
- SiteBuilderReport — Motion design portfolio examples 2026: https://www.sitebuilderreport.com/inspiration/motion-design-portfolios
- Google Search Central — Core Web Vitals: https://developers.google.com/search/docs/appearance/core-web-vitals
- Fast.io — How to build a motion graphics portfolio 2025: https://fast.io/resources/motion-graphics-portfolio/
- PremiumBeat — Demo reel / showreel tips: https://www.premiumbeat.com/blog/demo-reel-showreel-tips/

---

*Feature research for: freelance video editor / motion designer portfolio — WebGL single-page site*
*Researched: 2026-03-22*
