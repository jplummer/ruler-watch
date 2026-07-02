# Ruler Watch

A watch face concept: instead of hands, two vertical ruler-style scales —
hours on the left, minutes on the right — slide past a fixed horizontal
hairline. Read the time off wherever each scale crosses the line.

Originally conceived for Pebble; built here as a standalone web demo since
Apple Watch has no public API for custom system watch faces. This repo is a
single self-contained `index.html` (no build step, no dependencies) so it
can be opened directly in a browser or dropped into an `<iframe>`.

## Try it

```
open index.html
```

or serve the directory and visit it in a browser — either way, no build
step is required.

## How it works

- Two scale tracks (hour, minute) are generated in JS as absolutely
  positioned tick elements, long enough to scroll across a full day with no
  visible seam.
- Both tracks are driven by a single `transform: translateY()` update once
  per second, snapped to the device pixel grid so tick marks stay crisp in
  both Safari and Chrome (see `docs/safari-chrome-rendering-fix.md`).
- The whole thing is composited into a pure-CSS Apple Watch case mockup —
  aluminum body, digital crown, side button, tapered band — no external
  image assets.
- Colors, spacing, and the watch's physical scale are all CSS custom
  properties under `:root` in `index.html`, so the theme and size can be
  retuned from one place.

## Docs

- [`docs/original-spec.md`](docs/original-spec.md) — the initial build spec:
  concept, layout, and scale mechanics.
- [`docs/safari-chrome-rendering-fix.md`](docs/safari-chrome-rendering-fix.md)
  — investigation and fix for a Safari-only tick-blurring bug.
