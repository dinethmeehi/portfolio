# Design System

Reference for colours, typography, and the recurring interaction patterns used across the site. Keep new work consistent with this unless the person explicitly asks for a change.

## Brief / brand direction (as given by the person)

- "Minimal, clean and not too colourful" (explicit, early request).
- Later requests added a dark, ambient, moody background "with white tint" — still minimal, just inverted from light-on-dark to dark-on-dark-with-glow rather than becoming colourful. The particle effects and gold accent are the only saturated/warm notes in the whole site, and even those are restrained (outlined shapes, not filled; a soft warm-silver/gold, not bright yellow).

## Colour tokens

```css
:root{
  --paper: #0A0A0B;                 /* base background (dark) */
  --ink:   #F2F0EA;                 /* primary text (off-white) */
  --muted: #A6A199;                 /* secondary/label text */
  --accent:#C9BFA0;                 /* soft warm silver/gold accent */
  --line:  rgba(255,255,255,0.14);  /* hairline borders */
  --card:  rgba(255,255,255,0.05);  /* translucent card backgrounds */
}
```

Note on naming: `--paper` and `--ink` are named after an earlier **light** theme (paper = light background, ink = dark text) and were later flipped in value when the site moved to a dark theme, but the variable *names* were kept as-is rather than renamed throughout the codebase. When editing, remember `--paper` is now the dark colour and `--ink` is now the light colour — the names are historical, not descriptive.

Particle colours (not CSS variables, defined directly in JS as RGB triples):

```js
const inkColor    = '242,240,234'; // default particle colour (white/ivory)
const accentColor = '201,191,160'; // default particle accent
const goldInk      = '120,88,32';  // "All" filter only
const goldAccent   = '184,140,64'; // "All" filter only
```

**Rule:** only the "All" category button uses the gold pair. Every other button (3D, 2D, Projects, and all their sub-filters) uses the white/ivory pair. This was implemented, then accidentally made global, then explicitly corrected back — don't regress this.

## Typography

A single custom font, embedded via `@font-face`:

```css
@font-face{
  font-family:'Retron2000';
  src:url('data:font/ttf;base64,...') format('truetype');
  font-weight:normal;
  font-style:normal;
  font-display:swap;
}
```

- Source file: `Retron2000.ttf`, uploaded by the person (© 2018 Vasily Draigo aka Daymarius).
- Applied via `font-family:'Retron2000', sans-serif;` on **every** text element — body copy, headings, labels, buttons, nav. There is no secondary/body typeface; this was an explicit, deliberate request to replace *all* fonts, not just headings.
- The font only has a single weight/style (regular). Where the old design called for italic display type (e.g. hero/section headings originally set in an italic serif), the browser will synthesize a slanted version. This was flagged to the person as a known cosmetic side effect; they have not asked for it to be changed.

## Layout language

- Content width capped via `.wrap { max-width:1180px; margin:0 auto; padding:0 40px; }`.
- Generous whitespace, hairline borders (`--line`) rather than filled dividers.
- Pill-shaped buttons (`border-radius:999px`) for all filter/category chips.
- Grid-based work galleries (`.work-grid`, CSS grid, responsive column count).

## Recurring interaction patterns

### 1. Ambient background

**v2 (current):** a looping VFX video clip, embedded as a base64 `data:video/mp4` source inside a fixed, full-viewport `<video id="bg-layer">` (`autoplay muted loop playsinline`, `z-index:-2`), with a separate `#bg-overlay` gradient div (`z-index:-1`) on top for text contrast. The original procedurally-generated ambient JPEG (below) is kept as the video's `poster`.

- CSS applies `blur(2.5px)` plus `brightness`/`saturate` to the video layer so body text stays legible over the busier moving footage — the brightness range was also darkened (was `0.72–1.14`, now `0.5–0.85`) and the `#bg-overlay` gradient stops darkened to match.
- **Interactive:** unchanged from v1 — on scroll, the layer's `rotate`, `scale`, and `brightness` (CSS `filter`) all interpolate based on scroll progress (0→1 down the page). On mousemove, it also translates slightly for a parallax feel. All driven by a single `requestAnimationFrame`-throttled `updateBackground()` function. If `prefers-reduced-motion: reduce`, the parallax loop is skipped **and** the video is explicitly `.pause()`d (falls back to its static poster frame).
- The same `<video>` element also feeds a `THREE.VideoTexture` used as the glass orb's environment map — see the orb description in `CLAUDE.md`'s v2 architecture section.

**v1 origin (still true of the underlying image, now used only as the poster):** a procedurally generated (not stock) dark image with a soft off-white radial glow and fine film grain, generated via Python/PIL — base near-black fill, 2–3 large soft radial "glow" ellipses blurred heavily (Gaussian blur ~60px), a vignette, then fine Gaussian noise added as grain. Rendered at 1920×1200, exported as JPEG (quality ~88).

### 2. Particle burst

A canvas-based (`#particle-canvas`, `position:fixed`, `z-index:60`, `pointer-events:none`) burst of ~26 small outlined shapes fired from the click coordinates whenever a filter/category button is pressed. Shapes: `square`, `triangle`, `line`, `cube` (a simple hand-drawn pseudo-3D cube outline). Physics: random initial velocity + angle, mild gravity, air drag, rotation, and per-particle opacity decay (`life -= decay` each frame) until removed.

Shape choice is tied to the clicked category (see `makeParticle(x, y, mode)`):

| mode | shapes |
|---|---|
| `3d` | cube / triangle |
| `2d` | square / triangle / line (random mix) |
| `projects` | cube / square |
| `all` | random mix of all four **and** gold-coloured |

Respects `prefers-reduced-motion` (burst is skipped entirely, not just slowed).

### 3. Filter / category chips

Pill buttons, inactive state: `background:transparent; border:1px solid var(--line); color:var(--muted)`. Active state: filled (`background:var(--ink)` for top-level filters, `background:var(--accent)` for sub-filters), text flips to `var(--paper)` (dark-on-light, since `--ink`/`--accent` are both light colours in the current dark theme).

### 4. Work card hover

Grid images sit at full colour at all times (an earlier grayscale→colour hover effect was explicitly removed — don't reintroduce it). On hover: subtle `scale(1.035)` zoom on the image, and a gradient-scrim caption (`.work-meta`) fades/slides in from the bottom showing title + medium/caption.

Video pieces get a CSS-only play-button overlay (`::before`/`::after` pseudo-elements drawing a circle + triangle) instead of a static thumbnail treatment.

### 5. Galleries on individual pieces

Some pieces have extra images (alternate angles, framed mockups) beyond their main/front image. These render as a row of small clickable thumbnails *underneath* the main image once a piece is opened (in v1: inside the lightbox; in v2: on the detail page). **The front/main thumbnail and default opened view never change** — extra images are strictly additive and opt-in via click. This pattern was requested and repeated consistently across ~6 different pieces; treat it as a firm rule for any future gallery additions.

### 6. Video pieces

Pieces can carry a YouTube video ID instead of/alongside a static image. When present: the grid thumbnail uses the YouTube-hosted thumbnail (`img.youtube.com/vi/{id}/hqdefault.jpg`, linked externally, not embedded) with a play-button overlay, and opening the piece embeds a real `<iframe>` YouTube player (`youtube.com/embed/{id}?rel=0`) so it's actually playable in place, not just a link out.

## Things that were tried and explicitly reverted (don't redo)

- Grayscale-by-default, colour-on-hover treatment for work images → removed, now always full colour.
- Gold particle colour applied to *every* category → corrected, gold is "All" only.
- Separate "Concept Art" and "Graphic Design" as **top-level** filters → merged into a single top-level "2D" filter with those as sub-filters (alongside the added "Character Art").
- A large serif hero headline ("Built in three dimensions, felt in two.") → removed entirely at the person's request; the hero now leads straight into the bio.
- Two separate bio paragraphs (studio/mediums paragraph, location/availability paragraph) → removed, kept only the single lead paragraph.
- A visible "Work / About / Contact" text nav in the header → removed (redundant with in-page sections at the time); note that v2's navigation model supersedes this entirely.
