# PRD-01 — Moonrise Phase 1: Scientific Diagram View

> **Depends on:** PRD-00 (read it first — all constraints and shared conventions apply)
> **Output file:** `index.html`
> **Visual style:** Clean, labeled, diagram-like — precision over atmosphere

---

## What This Is

Phase 1 is the **legible, scientific version**. Think: a well-designed astronomy textbook diagram come to life. Clean lines, compass labels, degree markings, neutral colors. It should feel like something a planetarium would display on a monitor. It prioritizes clarity and correctness over emotional impact.

Phase 2 (see PRD-02) adds the atmospheric, "standing in a dark yard" treatment as a separate file. Build Phase 1 first and fully before touching Phase 2.

---

## Layout

```
┌─────────────────────────────────────────────────┐
│  [Horizon Panorama] / [Orbital View]  ← toggle  │
│                                                   │
│  ┌───────────────────────────────────────────┐   │
│  │         CANVAS (view renders here)        │   │
│  │         ~600px wide, ~320px tall          │   │
│  └───────────────────────────────────────────┘   │
│                                                   │
│  [← Prev]  Mon Apr 7, 2025  [Next →]             │
│  ●━━━━━━━━━━━━━━━━━━━━━━━━━━━  ← scrubber        │
│  [▶ Play]  [Today]                               │
│                                                   │
│  ┌──────────────────────┐  ← phase indicator     │
│  │  Phase: Waxing ◑ 67% │                        │
│  └──────────────────────┘                        │
└─────────────────────────────────────────────────┘
```

Canvas width: `Math.min(window.innerWidth - 40, 780)` — responsive but capped.
Canvas height: roughly half the width (16:9 feel).

---

## View A: Horizon Panorama (Phase 1 Style)

### The Strip

Draw a full-width horizontal strip representing the 360° horizon. The strip sits in the vertical middle of the canvas.

- Strip height: ~40px
- Strip background: very light gray (`#f0f0f0`) or transparent with a baseline stroke
- Compass labels along the top edge: **N, NE, E, SE, S, SW, W, NW, N**
  - Spaced evenly at 0°, 45°, 90°, 135°, 180°, 225°, 270°, 315°, 360°
  - Font: 11px sans-serif, muted color
- Minor tick marks every 10° (short) and every 30° (taller)
- The strip repeats N at both left and right edges (the horizon wraps)

### Moon Rise / Set Dots

For `currentDate`, compute the Moon's rise and set azimuth using SunCalc.

```javascript
const moonTimes = SunCalc.getMoonTimes(currentDate, OBSERVER.lat, OBSERVER.lng);
// moonTimes.rise is a Date; compute its azimuth via getMoonPosition at that moment
const risePos = SunCalc.getMoonPosition(moonTimes.rise, OBSERVER.lat, OBSERVER.lng);
const setBearing = azToBearing(SunCalc.getMoonPosition(moonTimes.set, OBSERVER.lat, OBSERVER.lng).azimuth);
const riseBearing = azToBearing(risePos.azimuth);
```

- Rise dot: filled circle, `#c8a000` (gold), radius 8px, labeled "Rise" above with bearing in degrees (e.g. "Rise 62°")
- Set dot: filled circle, `#6080c0` (blue-gray), radius 8px, labeled "Set" above

### Moon Arc

Draw a smooth arc *above* the strip representing the Moon's path across the sky.

Approach: sample Moon altitude every 30 minutes between rise and set time. Map each sample's azimuth → X position on strip, altitude → Y position above strip (higher altitude = higher on canvas, capped at some max height like 160px above strip).

```javascript
function drawMoonArc(ctx, riseTime, setTime) {
  const steps = 48; // samples between rise and set
  const duration = setTime - riseTime;
  ctx.beginPath();
  let started = false;
  for (let i = 0; i <= steps; i++) {
    const t = new Date(riseTime.getTime() + (duration * i / steps));
    const pos = SunCalc.getMoonPosition(t, OBSERVER.lat, OBSERVER.lng);
    if (pos.altitude < 0) continue;
    const x = bearingToX(azToBearing(pos.azimuth), canvasWidth);
    const y = stripY - (pos.altitude / (Math.PI / 2)) * maxArcHeight;
    if (!started) { ctx.moveTo(x, y); started = true; }
    else ctx.lineTo(x, y);
  }
  ctx.strokeStyle = '#c8a000';
  ctx.lineWidth = 2;
  ctx.stroke();
}
```

### Moon Position Dot on Arc

An additional dot on the arc showing where the Moon is *right now* (at the exact `currentDate` time, not just the day). If the Moon is below the horizon, show a dimmed dot on the strip at its azimuth with a downward arrow.

### Trailing History (optional enhancement, build if time allows)

Draw the rise-dot positions for the previous 14 days as smaller, fading dots along the strip. This makes the migration pattern immediately visible. Use opacity from 0.1 (14 days ago) to 0.7 (yesterday). Color: same gold.

### Annotations

- A dashed vertical line at 90° (E) and 270° (W) labeled "Due East" / "Due West" — this is the reference: the Sun and Moon only rise exactly here at equinoxes.
- A small info label below the strip: `"Moon rises {N/S} of due East — {X}° from East"` — updates live.

---

## View B: Top-Down Orbital (Phase 1 Style)

### Layout

Center the Earth in the canvas. The Moon orbits it. The Sun is positioned off to one edge (or implied by a directional arrow labeled "Sun →").

For Phase 1, **fix the Sun off the right edge** to keep the diagram clean. Draw a yellow glow / label on the right side: "☀ Sun" with rays or just a label. The key insight is the angle between Earth→Moon and Earth→Sun.

### Earth

- Circle, radius ~20px, center of canvas
- Fill: `#2244aa` with a `#44aa44` landmass suggestion (or just solid blue — keep it simple)
- Label: "Earth" below

### Moon Orbit

- Dashed circle around Earth, radius ~120px
- Color: light gray, 1px stroke
- Label: "Moon's orbit (29.5 days)" at top of circle

### Moon

- Circle, radius ~10px, on the orbit circle at the correct angular position
- Position derived from `SunCalc.getMoonIllumination(currentDate).phase`
  - Phase 0 = new moon (Moon between Earth and Sun = top of orbit if Sun is to the right)
  - Phase 0.25 = first quarter
  - Phase 0.5 = full moon (Moon opposite Sun)
  - Phase 0.75 = last quarter
- Angle from phase: `moonAngle = phase * 2 * Math.PI`
  - With Sun to the right: `moonX = earthX + orbitRadius * Math.cos(moonAngle - Math.PI/2)`
  - `moonY = earthY + orbitRadius * Math.sin(moonAngle - Math.PI/2)`

### Phase Shading on Moon Circle

Draw the Moon as two overlapping circles — one lit, one dark — to show the current phase.

```javascript
function drawPhasedMoon(ctx, cx, cy, r, phase) {
  // Full dark circle first
  ctx.beginPath();
  ctx.arc(cx, cy, r, 0, Math.PI * 2);
  ctx.fillStyle = '#222';
  ctx.fill();

  // Lit half: always the right hemisphere relative to Sun direction
  // phase 0–0.5: waxing (right side lit), phase 0.5–1: waning (left side lit)
  const lit = phase <= 0.5
    ? { startAngle: Math.PI/2, endAngle: -Math.PI/2 } // right half
    : { startAngle: -Math.PI/2, endAngle: Math.PI/2 }; // left half
  ctx.beginPath();
  ctx.arc(cx, cy, r, lit.startAngle, lit.endAngle);
  ctx.fillStyle = '#ffffcc';
  ctx.fill();
}
```

(A more accurate phase shape requires ellipse arcs — acceptable to approximate for Phase 1.)

### Sun Direction Arrow

Draw a long arrow from Earth pointing right (or toward the Sun's direction). Label it "Sunlight →" in yellow.

### Lines

- A thin line from Earth to Moon (showing the Earth→Moon vector)
- A thin line from Earth toward Sun direction
- The angle between these two lines is what determines the phase — consider labeling this angle

### Earth's Orbit (optional)

A large dashed circle representing Earth's orbit around the Sun, with Earth on it. This helps explain seasons / the yearly cycle. Include only if it doesn't clutter the view — make it toggleable if needed.

---

## Phase Indicator Widget

Displayed below the canvas controls, always visible in both views.

```
Moon phase: Waxing Gibbous  ◕  67% illuminated
Rises: 3:42 PM  at  62° ENE
Sets:  3:18 AM  at  298° WNW
```

Draw the crescent/gibbous shape as a small SVG or canvas mini-render (radius ~18px). This is a simplified version of the orbital phase shading above.

---

## Color Palette (Phase 1)

| Element | Color |
|---|---|
| Canvas background | `#ffffff` or `#fafafa` |
| Horizon strip | `#eeeeee` with `#cccccc` border |
| Compass labels | `#888888` |
| Moon rise dot | `#c8a000` |
| Moon set dot | `#5070b0` |
| Moon arc | `#c8a000`, 2px, opacity 0.8 |
| Moon (orbital view) | `#ffffcc` lit, `#1a1a1a` dark |
| Earth | `#2244aa` |
| Sun indicator | `#ffcc00` |
| Reference lines | `#cccccc` dashed |
| Labels / text | `#333333` |

---

## Typography (Phase 1)

```css
body {
  font-family: Georgia, 'Times New Roman', serif;
  font-size: 15px;
  color: #333;
  background: #f8f8f5;
  max-width: 820px;
  margin: 0 auto;
  padding: 20px;
}
canvas {
  display: block;
  border: 1px solid #ddd;
  border-radius: 4px;
}
```

Use `sans-serif` for compass labels and data readouts, serif for prose/explanatory text.

---

## Explanatory Text (inline, updates with scrubber)

Below the phase indicator, show a brief paragraph that updates live:

> *"On [date], the Moon rises at [time] at [bearing]° ([compass direction]). This is [X]° north/south of due East. The Moon is in its [phase name] phase, [fraction]% illuminated. At this time of year, the Moon's rising point is [migrating north / migrating south / near its northernmost / near its southernmost point]."*

This is the Bret Victor moment — the text is an active description of what's happening, not a static caption.

---

## Edge Cases to Handle

- **Moon doesn't rise or set today** (rare at tropical latitudes, more common further north): Show a message "Moon above horizon all day" or "Moon below horizon all day" and skip the arc.
- **Date close to new moon**: Rise and set may be very close in time and azimuth — arc will be short. Handle gracefully.
- **Play mode near scrubber limits**: Stop playback when reaching ±365 days.

---

## File Structure (inside index.html)

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Moonrise</title>
  <style>/* all CSS here */</style>
</head>
<body>

  <!-- Controls: toggle, date label, scrubber, play/today buttons -->
  <!-- Canvas -->
  <!-- Phase indicator -->
  <!-- Live explanatory text paragraph -->

  <script src="https://cdnjs.cloudflare.com/ajax/libs/suncalc/1.9.0/suncalc.min.js"></script>
  <script>
  // === CONSTANTS ===
  // === OBSERVER LOCATION ===
  // === DATE / SCRUBBER STATE ===
  // === MATH HELPERS ===
  // === DRAW: HORIZON PANORAMA ===
  // === DRAW: ORBITAL VIEW ===
  // === DRAW: PHASE INDICATOR ===
  // === RENDER (calls active view) ===
  // === CONTROLS / EVENT LISTENERS ===
  // === PLAY LOOP ===
  // === INIT ===
  </script>
</body>
</html>
```

---

## Acceptance Criteria

- [ ] Scrubbing the date slider moves the Moon rise/set dots visibly across the horizon strip
- [ ] Toggling views switches between horizon panorama and orbital view without page reload
- [ ] Play button animates the scrubber at a comfortable speed (1 day/frame ≈ 1 month per second at 30fps)
- [ ] Today button resets to current date
- [ ] Phase indicator always matches the scrubbed date
- [ ] Live text paragraph updates on every scrubber tick
- [ ] Works in Safari, Chrome, Firefox on macOS
- [ ] Opens as a local file (`file://`) without errors
- [ ] No console errors on load or during interaction

---

*End of PRD-01*
