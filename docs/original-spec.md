# Ruler Watch Face — Web Demo

## Context

Jon has had a long-standing idea for a watch face: instead of hands, two linear
ruler-style scales — one for hours (subdivided into half/quarter hours), one
for minutes (subdivided into 10-second increments) — slide vertically past a
fixed horizontal hairline, and you read the time off wherever each scale
crosses the hairline. He originally meant to build it for Pebble, but the
platform was discontinued before he got to it, and Apple Watch doesn't allow
third-party apps to install real system watch faces (confirmed in prior
discussion — no public face API, unlike Wear OS).

Rather than build a constrained native app that can't actually replace the
watch face, the near-term goal is a **standalone, self-contained web page**
that demonstrates the mechanism live. It should be embeddable via `<iframe>`
in a blog post, requiring no build step or dependencies.

## Approach

Single self-contained file: `~/Projects/ruler-watch/index.html` (inline CSS +
JS, no external libraries, no build tooling — opens directly in a browser and
iframes cleanly).

**Layout** (per Jon's spec):
- A watch-face container with two vertical scale tracks side by side,
  touching at the center — hour scale on the left, minute scale on the right.
- A horizontal hairline fixed at the vertical center of the container,
  spanning both tracks, drawn on top as a static overlay.
- Style: functional wireframe — black ticks/numerals on a plain background,
  no texture or skeuomorphism. Optimize for clarity of the mechanism first.

**Scale mechanics:**
- Hour track: continuous linear scale labeled 12,1,2,...,12,1... (12-hour
  numbering) with major ticks + numerals at each hour, medium ticks at the
  half-hour, minor ticks at the quarter-hours. Built long enough (spanning
  ~24h worth of hour positions) that it scrolls continuously with no reset
  jump as hours roll over.
- Minute track: continuous linear scale labeled 0–59, major tick + numeral
  every 5 minutes, medium tick at each whole minute, minor tick at every
  10-second increment. Built long enough to scroll smoothly across multiple
  hours without a visible seam.
- Both tracks translate vertically via CSS `transform: translateY()`, driven
  by a `requestAnimationFrame` loop reading `Date.now()` each frame so the
  motion is continuous (sub-second), not a discrete per-second tick — this is
  what will make it read as "sliding" rather than jumping.
- Tick/label generation is done in JS (loop that appends positioned
  `<div>`s) rather than hand-authored markup, since the pattern is regular
  and repetitive.

**Known simplification for this first pass:** ticks are pre-rendered for a
fixed window (rather than virtualized/recycled), which is fine for a demo but
would need revisiting if this ever needs to run for very long unattended
periods or on low-power devices.

## Files

- `~/Projects/ruler-watch/index.html` — everything: markup, CSS, JS, in one
  file for easy portability/iframe embedding.

## Verification

- Open `index.html` directly in a browser (`open ~/Projects/ruler-watch/index.html`)
  and confirm:
  - The hour and minute scales align correctly with the current system time
    under the hairline.
  - Motion is smooth/continuous, not jumpy.
  - Ticks and numerals are legible and correctly subdivided (half/quarter
    hour; 10-second minute increments).
- Resize the browser window / embed in a small `<iframe>` test page to confirm
  it degrades reasonably at blog-post iframe dimensions (e.g. ~400×400).

---
*Note: this was the original build spec, written before the Safari/Chrome
rendering fix (see `safari-chrome-rendering-fix.md`) and before later
sessions added the Apple Watch case mockup, CSS custom properties, and the
switch to a discrete once-per-second tick (superseding the "continuous,
not discrete" motion decision described above — see project history for
why that changed).*
