# Phase 4: Scroll Integration - Context

**Gathered:** 2026-03-24
**Status:** Ready for planning

<domain>
## Phase Boundary

Transform the hero into a scrollable cinematic experience — the WebGL world fades away with parallax, content sections emerge naturally with scroll-triggered animations. No fixed nav dots. A CTA button invites users to scroll. Touch and desktop scroll both work identically.

</domain>

<decisions>
## Implementation Decisions

### Hero Transition
- **Parallax + fade**: Canvas scrolls at ~0.5x speed (parallax) while fading to invisible — creates depth
- **Distance**: Transition completes over 100vh (one full viewport of scroll)
- **Text 3D behavior**: The "Tom Bernys" glass text moves away in Z (recedes into the scene) as user scrolls — separate from the canvas parallax
- **Tiles behavior**: Tiles spread apart slightly during scroll — visible but smooth gap expansion between tiles, like they're dispersing
- **Symmetrical return**: Scrolling back up reverses the transition identically
- **Mouse interaction stays active**: Touch/mouse rotation of the 3D text continues working even while scrolling — both coexist

### CTA Button "Voir mon travail"
- **Position**: Bottom-left of the hero section
- **Style**: Semi-transparent background with backdrop-blur (glassmorphism), includes downward arrow
- **Action**: Clicking auto-scrolls smoothly to just past the hero section
- **Scroll behavior**: Button stays visible for the first ~30% of scroll, then fades out
- **Destination**: Scrolls to just after the hero (first content section)

### Navigation
- **No fixed dot navigation** — the CTA button and natural scroll are sufficient
- **No scroll indicator/bar** — keep the UI minimal

### Scroll Behavior
- **Free scroll** — no snap between sections, continuous natural scrolling
- **Scroll speed**: Completely natural, no scroll hijacking
- **Brief hero hold**: ~0.5s brief moment where scroll feels slightly delayed before starting the hero transition — lets user see the scene
- **Mobile**: Identical behavior to desktop — same parallax, same fade

### Content Section Appearance
- **Animation**: Fade in + slide up from below for each section
- **Trigger**: Animation starts when 25% of the section is visible in the viewport
- **Play once**: Animations only play the first time a section enters — stays visible after
- **Section enters as one block**: Entire section animates as a single unit (not staggered elements)
- **First section special**: The first content section (video grid) gets a more pronounced entrance — Claude decides the specific treatment, knowing it's a video grid

### Background
- **Behind content sections**: Very dark background with subtle film grain/noise texture — cohesive with the CRT theme of the hero tiles

### Claude's Discretion
- Exact parallax speed ratio for the canvas
- Exact tile spread distance during scroll
- Specific treatment for the first section (video grid) entrance
- RAF optimization threshold for pausing rendering
- Easing curves for all scroll-driven animations
- Film grain implementation for content background

</decisions>

<specifics>
## Specific Ideas

- The hero parallax should feel cinematic — like the world is falling behind you as you scroll forward
- The tile spread during scroll should feel like the wall of screens is gently dispersing/breaking apart
- "Voir mon travail" button with glassmorphism style and arrow ↓ — modern portfolio CTA feel
- The 3D text receding in Z should feel like it's slowly sinking into the scene as you leave
- First content section will be a video grid — the entrance animation should work well for grid content
- Film grain on content background should be very subtle — just enough to avoid flat black

</specifics>

<deferred>
## Deferred Ideas

- Video grid layout and content — Phase 5 (Content Sections)
- Specific section content design — Phase 5+

</deferred>

---

*Phase: 04-scroll-integration*
*Context gathered: 2026-03-24*
