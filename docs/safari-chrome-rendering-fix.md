# Fix Safari/Chrome rendering mismatch in ruler-watch

## Context
`ruler-watch/index.html` is a single-file animated watch face (two scrolling
ruler-style strips: hours and minutes) that Jon is building to look identical
in Safari and Chrome. Permissions for browser automation were already granted
in a prior session (required a Ghostty restart), so this session could
directly screenshot both browsers running the page at `localhost:8934`.

Comparing the two renders side by side turned up a real visual bug, not a
permissions issue: the minute track's minor sub-tick marks (the short ticks
between each minute label, built in `buildMinuteStrip()`) render as crisp,
individually-visible dashes in Chrome, but blur into a hazy, merged light
band in Safari. See the zoomed screenshots taken this session — Chrome shows
distinct 1px dashes with visible gaps; Safari shows a near-continuous smear.

## Root cause
`frame()` (index.html:290-307) drives the whole strip's position via
`transform: translateY(...)` every animation frame, computed directly from
the continuous (fractional) wall-clock time — never rounded to a pixel
boundary. The tick marks themselves are also positioned with fractional
`top` values (from `hourPos()`/`minutePos()` math, index.html:215-220,
227, 256, 268, 277).

Chrome/Blink snaps thin (1px) elements to the device pixel grid fairly well
even under fractional transforms. Safari/WebKit anti-aliases hairline
elements more aggressively at fractional offsets, and since minor sub-ticks
sit only ~9.7px apart (`MINUTE_PX_PER_MIN = 58`, subdivided into sixths),
that extra blur causes adjacent ticks to visually merge — an effect that
doesn't show up on the more widely-spaced major/medium ticks.

## Fix
Snap all pixel values to the device pixel grid before they hit the DOM, in
two places in `index.html`:

1. **Tick build time** — in `buildHourStrip()` (index.html:222-250) and
   `buildMinuteStrip()` (index.html:252-288), round each tick's computed `y`
   to the nearest device pixel before assigning `tick.style.top`, using
   `window.devicePixelRatio` (`Math.round(y * dpr) / dpr`).
2. **Per-frame transform** — in `frame()` (index.html:301-302), round the
   `translateY` offset the same way for both `hourStrip` and `minuteStrip`
   before writing `style.transform`.

This keeps the sweep visually smooth (sub-device-pixel jitter is
imperceptible at these speeds) while forcing both engines to rasterize the
same pixel-aligned geometry, eliminating the Safari-only blur.

## Verification
- Reload `localhost:8934/index.html` in both Safari and Chrome (already
  open in this session).
- Screenshot/zoom the minute track in both and confirm the minor sub-ticks
  now render as distinct dashes in Safari, matching Chrome.
- Toggle the "Scales move up/down" radio in both browsers and confirm ticks
  stay crisp during motion, not just at rest.
- Let it run a few seconds in both to confirm the sweep still looks smooth
  (no visible stepping/jitter from the rounding).

---
*Note: line numbers above reflect `index.html` as it existed when this fix
was made. The file has since grown (Apple Watch case mockup, CSS custom
properties, discrete-per-second tick motion), so treat line references as
historical rather than current.*
