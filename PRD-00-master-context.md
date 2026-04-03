# PRD-00 — Moonrise: Master Agent Context

> **For the coding agent:** Read this first. It defines constraints, philosophy, and conventions that all other PRDs inherit. Do not deviate from these without explicit instruction.

---

## Project Overview

**Moonrise** is a single-file, browser-based interactive visualization that answers the question: *"Why does the Moon rise in a different spot on the horizon every night?"*

It is built for local macOS tinkering first, GitHub Pages deployment second. No build step. No frameworks. No package manager. Everything runs from one HTML file opened directly in a browser or served from a simple static host.

---

## Hard Constraints (non-negotiable)

| Constraint | Detail |
|---|---|
| **Single HTML file** | All HTML, CSS, and JS in one `.html` file |
| **No build step** | No webpack, Vite, Rollup, npm scripts, etc. |
| **No frameworks** | No React, Vue, Svelte, Angular |
| **CDN imports only** | Use `<script src="https://cdnjs.cloudflare.com/...">` |
| **Named fonts only** | e.g. `font-family: Georgia, serif` — no @import, no Google Fonts |
| **GitHub Pages compatible** | Must work as a static file with no server-side code |
| **Works offline** | After first CDN load, should function without network (use cached CDN) |

---

## Observer Location

Hardcoded for Phase 1 and Phase 2. Do not add a UI for changing this until explicitly instructed.

```javascript
const OBSERVER = {
  lat: 21.3069,   // Waimanalo, Hawaiʻi (North)
  lng: -157.7111  // West
};
```

---

## Primary CDN Dependency

**SunCalc v1.9.0** — the only required external library.

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/suncalc/1.9.0/suncalc.min.js"></script>
```

Key functions used throughout the project:

```javascript
// Moon position at a moment: altitude (radians above horizon), azimuth (radians)
SunCalc.getMoonPosition(date, lat, lng)
// → { altitude: Number, azimuth: Number }

// Moon rise/set times and azimuths for a given calendar day
SunCalc.getMoonTimes(date, lat, lng)
// → { rise: Date, set: Date, alwaysUp: Boolean, alwaysDown: Boolean }
// Note: rise/set objects carry .azimuth in radians when available

// Moon illumination: phase fraction, angle, phase name approximation
SunCalc.getMoonIllumination(date)
// → { fraction: Number (0–1), phase: Number (0–1), angle: Number }

// Sun position
SunCalc.getPosition(date, lat, lng)
// → { altitude: Number, azimuth: Number }

// Sun rise/set
SunCalc.getTimes(date, lat, lng)
// → { sunrise: Date, sunset: Date, ... }
```

**Azimuth convention in SunCalc:** 0 = South, positive = West, negative = East (radians).
Convert to compass bearing (0=N, clockwise): `bearing = (azimuth * 180/Math.PI + 180) % 360`

---

## Date / Time Model

All calculations use JavaScript `Date` objects. The scrubber is the master clock — one `currentDate` variable drives every view.

```javascript
let currentDate = new Date(); // starts at "now"
```

The scrubber spans **±365 days** from today (i.e., one full year in either direction).

```javascript
const MS_PER_DAY = 86400000;
const TODAY = new Date();
const scrubberMin = -365;
const scrubberMax = 365;
// currentDate = new Date(TODAY.getTime() + offsetDays * MS_PER_DAY)
```

---

## App Structure (both phases)

```
moonrise/
  index.html          ← the entire app (Phase 1)
  moonrise-p2.html    ← Phase 2 immersive version (separate file)
```

Both files are standalone. Phase 2 does not depend on Phase 1.

---

## Two Views (toggle between them)

Both phases implement the same two logical views. What changes between phases is the *visual treatment*, not the underlying data or structure.

### View A — Horizon Panorama
A flat, wide strip representing the observer's 360° horizon, laid out left-to-right as:
```
N ... NE ... E ... SE ... S ... SW ... W ... NW ... N
```
The Moon's rise azimuth and set azimuth are plotted as dots on this strip. The arc of the Moon's path above the horizon is drawn as a curve between them.

### View B — Top-Down Orbital
A simplified clockface view showing Sun, Earth, and Moon positions. The Moon orbits Earth; Earth orbits the Sun (or Sun can be fixed off-screen to the right for simplicity). Phase shading is drawn on the Moon circle. Scrubbing moves the Moon around its orbit.

### Toggle
A single button switches between the two canvas views. Label it **"Switch view"** or use an icon. No animation needed for the toggle itself.

---

## Shared UI Controls (both phases)

```
[ ← ]  [Date label: Mon Apr 7, 2025]  [ → ]   ← prev/next day buttons
[————————●————————————————————] ← range input scrubber (±365 days)
[ ▶ Play ]  [ Today ]               ← playback + reset buttons
```

Play auto-increments `currentDate` by 1 day per frame at ~30fps (requestAnimationFrame). A `playing` boolean flag controls it.

---

## Phase Indicator (always visible)

A small inset circle (bottom-right of canvas or below controls) showing the Moon's current illumination as a crescent/gibbous shape. Always synced to `currentDate`.

Draw it as two overlapping circles:
- Full circle = the Moon disc
- Offset/clipped circle = the shadow portion
- Fill lit portion with white/yellow, shadow with dark gray/near-black

---

## Azimuth / Compass Math Reference

```javascript
// Convert SunCalc azimuth (radians, 0=S, +W) to degrees clockwise from North
function azToBearing(azRad) {
  return (azRad * 180 / Math.PI + 180) % 360;
}

// Convert bearing (0–360, 0=N) to canvas X position on horizon strip
function bearingToX(bearing, canvasWidth) {
  return (bearing / 360) * canvasWidth;
}
```

---

## Pedagogical Principles

This app is an **explorable explanation** in the tradition of Bret Victor. These shape every design decision:

1. **The scrubber is the teacher.** Dragging it is the moment of insight — the Moon's rise point visibly migrating across the horizon over time.
2. **Static is readable, interactive is deeper.** The app should make sense as a frozen screenshot, but reward interaction.
3. **Label the "why," not just the "what."** Brief inline text (e.g. "Moon rises NE tonight — furthest north in this cycle") contextualizes what the user sees.
4. **Progressive complexity.** Phase 1 = clean diagram. Phase 2 = felt experience. Future phases = more phenomena.

---

## Future Phases (do not build yet, but do not block)

- **Sun overlay** on the horizon strip (same strip, dimmer arc, shows the comparison)
- **Nodal cycle toggle** — extend scrubber to ±10 years to see the 18.6-year wobble
- **Eclipse / supermoon flags** — flag dates when special events occur
- **3D view** — Three.js orthographic camera, toggle from 2D
- **Artemis 2 trajectory** — live data pipe (NASA API TBD)
- **User location** — geolocation API or map pin

---

## Code Style for the Agent

- Comments in plain English. Explain the astronomy, not just the code.
- Group code into clearly labeled sections with `// ===` dividers
- No minification
- No transpilation (write ES2020+ directly — modern browsers handle it)
- `const` by default, `let` only when mutation is needed
- All drawing happens in a `render()` function called on every scrubber change

---

*End of PRD-00*
