---
name: svg-specialist
description: SVG craft reference for drawing, styling, animating, and optimising hand-written vector graphics. Use when illustrating a page of this site — Pin's room, the wall's horizon, stars, ornaments — or when optimising an SVG for size and accessibility.
---

# SVG craft reference

A workshop manual, not a voice. The house rules in README.md always outrank
anything here: hand-written markup, no JavaScript, both weathers, every choice
deliberate. This file is technique for the hands, adapted and trimmed by Seren
from the "moai-tool-svg" skill (Apache-2.0) — web-embedding and framework
sections left in the shop, since this house has no use for them.

## Document structure

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100" width="100" height="100">
  <title>Accessible title</title>
  <desc>Description with care in it, for screen readers.</desc>
  <!-- content -->
</svg>
```

- `viewBox="minX minY width height"` defines the coordinate system; width and
  height set rendered size. Inline SVG in a page can drop fixed width/height
  and size itself with CSS.
- Meaningful graphics: `role="img" aria-labelledby="t d"` with `<title id="t">`
  and `<desc id="d">`. Decorative graphics: `aria-hidden="true"` and no title.
- Use `<defs>` for gradients, filters, symbols; `<g transform="...">` to group.
- Inline SVG inherits CSS custom properties — the site palette variables
  (--twilight, --cardigan, --pink, --gold, --violet, --paper, --ink) work as
  fill/stroke values, which is how a drawing changes with the weather for free.

## Shapes

- `<rect x y width height rx/>` — rx rounds corners
- `<circle cx cy r/>`, `<ellipse cx cy rx ry/>`
- `<line x1 y1 x2 y2/>`, `<polyline points fill="none"/>`, `<polygon points/>`

## Path grammar

Movement: `M x y` move, `L x y` line, `H x` / `V y` axis lines, `Z` close.
Lowercase = relative. Curves:

- `C x1 y1 x2 y2 x y` — cubic bezier, two control points
- `S x2 y2 x y` — smooth cubic, reflects previous control
- `Q x1 y1 x y` — quadratic, one control point
- `T x y` — smooth quadratic
- `A rx ry rot large-arc sweep x y` — arc

Worked examples:

- Rounded rectangle: `M15 5 H85 A10 10 0 0 1 95 15 V85 A10 10 0 0 1 85 95 H15 A10 10 0 0 1 5 85 V15 A10 10 0 0 1 15 5 Z`
- Heart: `M50 88 C20 65 5 45 5 30 A15 15 0 0 1 35 30 Q50 45 50 45 Q50 45 65 30 A15 15 0 0 1 95 30 C95 45 80 65 50 88 Z`

## Reuse

```svg
<defs>
  <symbol id="star" viewBox="0 0 24 24">
    <path d="M12 2l3.09 6.26L22 9.27l-5 4.87 1.18 6.88L12 17.77l-6.18 3.25L7 14.14 2 9.27l6.91-1.01L12 2z"/>
  </symbol>
</defs>
<use href="#star" x="0" y="0" width="24" height="24"/>
<use href="#star" x="30" y="0" width="48" height="48" fill="var(--gold)"/>
```

## Gradients, filters, masks

```svg
<linearGradient id="dusk" x1="0%" y1="0%" x2="0%" y2="100%">
  <stop offset="0%" stop-color="var(--twilight)"/>
  <stop offset="100%" stop-color="var(--cardigan)"/>
</linearGradient>

<radialGradient id="glow-core" cx="50%" cy="50%" r="50%" fx="30%" fy="30%">
  <stop offset="0%" stop-color="#fff"/>
  <stop offset="100%" stop-color="var(--gold)"/>
</radialGradient>

<filter id="soft-glow">
  <feGaussianBlur stdDeviation="4" result="blur"/>
  <feMerge>
    <feMergeNode in="blur"/>
    <feMergeNode in="SourceGraphic"/>
  </feMerge>
</filter>

<filter id="drop-shadow" x="-20%" y="-20%" width="140%" height="140%">
  <feGaussianBlur in="SourceAlpha" stdDeviation="3" result="blur"/>
  <feOffset in="blur" dx="3" dy="3" result="off"/>
  <feMerge><feMergeNode in="off"/><feMergeNode in="SourceGraphic"/></feMerge>
</filter>
```

Clip to a shape: `<clipPath id="c"><circle cx="50" cy="50" r="40"/></clipPath>`
then `clip-path="url(#c)"`. Fade with a mask: white shows, black hides, a
gradient in between.

## Stroke and fill details

`fill`, `fill-opacity`, `fill-rule` (nonzero/evenodd); `stroke`, `stroke-width`,
`stroke-linecap` (butt/round/square), `stroke-linejoin` (miter/round/bevel),
`stroke-dasharray`, `stroke-dashoffset`.

## Text

`<text x y text-anchor="middle" dominant-baseline="middle">` for placement.
Text on a curve:

```svg
<defs><path id="curve" d="M10 80 Q95 10 180 80" fill="none"/></defs>
<text><textPath href="#curve">words along a path</textPath></text>
```

## Animation (CSS only — house rule)

All animation via CSS on inline SVG; no SMIL, no JS. Always respect
`prefers-reduced-motion`:

```css
.draw-path {
  stroke-dasharray: 1000;
  stroke-dashoffset: 1000;
  animation: draw 2s ease forwards;
}
@keyframes draw { to { stroke-dashoffset: 0; } }

@media (prefers-reduced-motion: reduce) {
  .draw-path { animation: none; stroke-dashoffset: 0; }
}
```

The line-drawing trick: set dasharray/dashoffset at or above total path length
and animate offset to 0 — the path draws itself in. Pulse/breathe effects with
transform scale need `transform-origin: center`.

## Optimisation

- Round path precision to 1–2 decimals; hand-written paths should be born tidy.
- Prefer `<symbol>`/`<use>` for repeats.
- SVGO if ever needed on generated files: `npx svgo input.svg -o output.svg`
  (config: keep viewBox — `removeViewBox: false`). Hand-written files should
  rarely need it.
- Inline SVG in HTML costs no request; standalone .svg files need the xmlns.

## House conventions

- Palette only via CSS variables so drawings live in both weathers.
- Every meaningful drawing gets title + desc written with the same care as alt
  text; decorative flourishes get aria-hidden.
- If a drawing needs more than the ink it is made of, it is probably trying to
  be a photograph — simplify until it is a drawing again.
