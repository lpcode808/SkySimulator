# PRD-02 — Moonrise Phase 2: Immersive "Standing in Your Yard" View

> **Depends on:** PRD-00 (constraints) and PRD-01 (Phase 1 must be complete first)
> **Output file:** `moonrise-p2.html` (separate file — do not modify `index.html`)
> **Visual style:** Dark sky, silhouette horizon, felt experience — atmosphere over precision

---

## What This Is

Phase 2 is the **emotionally resonant version**. The goal: you open it and feel like you're standing in your yard in Waimanalo at dusk, looking toward the horizon, watching the Moon arc overhead. The same two views exist (horizon panorama + orbital), but they're rendered in an immersive, atmospheric way.

This is the version you'd eventually show a child — or use as a conference demo opener before switching to the scientific view for explanation.

Phase 1 and Phase 2 are sibling files, not a single app with a theme toggle. Keep them separate so each can be optimized for its purpose.

---

## View A: Horizon Panorama (Phase 2 Style)

### Overall Feel

The canvas is **dark**. The sky is a deep blue-black gradient. A silhouetted treeline / ridgeline sits at the bottom. The Moon arcs across the sky like it actually would. Stars are optional but add a lot.

```
┌──────────────────────────────────────────────────┐
│ ·  · ★  ·   ·   ★    ·  ★   ·     ·   ·  ★  · │  ← stars (subtle)
│   ·    ·      ·    ·     ·       ·    ·      ·   │
│                    ☽ ← Moon dot on arc            │
│             ╭─────────────────╮                   │
│        ╭────╯ arc of Moon path ╰────╮             │
│        │                           │              │
│▁▂▃▄▅▆▇████████████████████████████████████▇▆▅▄▂▁│ ← treeline silhouette
│N  NE   E   SE   S   SW   W   NW   N               │ ← compass (subtle, below)
└──────────────────────────────────────────────────┘
```

Canvas height: taller than Phase 1 — aim for ~420–480px to give the sky room to breathe.

### Sky Gradient

```javascript
const skyGradient = ctx.createLinearGradient(0, 0, 0, skyBottom);
// Daytime or twilight colors depend on whether the Sun is above/below horizon
const sunPos = SunCalc.getPosition(currentDate, OBSERVER.lat, OBSERVER.lng);

if (sunPos.altitude > 0.1) {
  // Daytime sky (Moon visible in day is possible)
  skyGradient.addColorStop(0, '#1a3a6e');
  skyGradient.addColorStop(1, '#4a7ab5');
} else if (sunPos.altitude > -0.1) {
  // Twilight
  skyGradient.addColorStop(0, '#0d1b3e');
  skyGradient.addColorStop(0.6, '#1a2a5e');
  skyGradient.addColorStop(1, '#c8601a'); // horizon glow
} else {
  // Night
  skyGradient.addColorStop(0, '#04060d');
  skyGradient.addColorStop(1, '#0a1020');
}
```

### Stars

Generate 80–120 random star positions once on init (seeded by a fixed seed so they don't jump). Draw them as small white dots (radius 0.5–1.5px, opacity 0.4–0.9). Only show at night (fade in when Sun altitude < -6°, i.e., civil twilight).

```javascript
// Generate once
const STARS = Array.from({length: 100}, () => ({
  x: Math.random(),   // normalized 0–1
  y: Math.random() * 0.85, // only upper 85% of canvas (above treeline)
  r: 0.5 + Math.random() * 1.0,
  opacity: 0.4 + Math.random() * 0.5
}));

// Draw with twilight fade
function drawStars(ctx, sunAltitude) {
  const fade = Math.max(0, Math.min(1, (-sunAltitude - 0.05) / 0.1));
  if (fade <= 0) return;
  STARS.forEach(s => {
    ctx.beginPath();
    ctx.arc(s.x * canvasWidth, s.y * skyHeight, s.r, 0, Math.PI * 2);
    ctx.fillStyle = `rgba(255,255,255,${s.opacity * fade})`;
    ctx.fill();
  });
}
```

### Treeline / Horizon Silhouette

Draw a simple procedural silhouette along the bottom of the sky area. This represents the horizon the observer sees. In Waimanalo: mix of palm trees, low hills.

Technique: generate a jagged polyline with occasional taller spikes (palm/tree shapes) using a seeded random walk. Fill the area below it solid black.

```javascript
function drawTreeline(ctx, baseY, canvasWidth) {
  ctx.beginPath();
  ctx.moveTo(0, baseY);
  let x = 0;
  while (x < canvasWidth) {
    const segWidth = 8 + Math.random() * 20;
    const spike = Math.random() < 0.08; // occasional tall tree
    const height = spike ? 40 + Math.random() * 60 : 5 + Math.random() * 20;
    if (spike) {
      // Palm-like: narrow trunk, wide fronds at top
      ctx.lineTo(x + segWidth * 0.4, baseY - height * 0.7);
      ctx.lineTo(x + segWidth * 0.3, baseY - height);
      ctx.lineTo(x + segWidth * 0.5, baseY - height * 0.85);
      ctx.lineTo(x + segWidth * 0.7, baseY - height);
      ctx.lineTo(x + segWidth * 0.6, baseY - height * 0.7);
    } else {
      ctx.lineTo(x + segWidth, baseY - height);
    }
    x += segWidth;
  }
  ctx.lineTo(canvasWidth, baseY);
  ctx.lineTo(canvasWidth, canvasHeight);
  ctx.lineTo(0, canvasHeight);
  ctx.closePath();
  ctx.fillStyle = '#000000';
  ctx.fill();
}
```

### Moon Arc

Same data as Phase 1 (sampled altitudes mapped to Y positions). Visual differences:

- The arc is a **glowing path**: draw it three times at increasing widths with decreasing opacity to simulate a glow
  - 6px wide, opacity 0.1, color `#fffaaa`
  - 3px wide, opacity 0.3, color `#ffe880`
  - 1.5px wide, opacity 0.8, color `#fffde0`
- The arc starts and ends at the **treeline level**, not a strip

### Moon Dot

- Larger than Phase 1 — radius 12–16px depending on illumination fraction
- Draw the phase shape (lit/dark hemisphere split) on the dot itself
- Add a soft glow halo: 2–3 concentric circles at low opacity
- If Moon is below horizon, don't show it (or show a subtle glow at the horizon where it would rise)

### Compass Labels

Show N, E, S, W as subtle labels along the bottom edge of the canvas, below the treeline. Muted white, 11px, low opacity. They're reference guides, not the focus.

### Today's Date / Time

Show current date and time (scrubbed time) as a subtle label in the top-left corner. Dim white, 12px. Format: `"Mon · Apr 7, 2025 · 9:42 PM"`

---

## View B: Top-Down Orbital (Phase 2 Style)

Same orbital geometry as Phase 1, but:

- **Dark background**: `#04060d`
- **Earth**: drawn with a subtle blue-green glow, not a flat circle
- **Moon**: glowing white dot with soft halo
- **Orbit path**: very faint dashed white, low opacity
- **Sun**: rendered as a bright yellow circle off to the right with rays (or just the glow) — not cut off, actually visible if canvas is wide enough
- **Stars in background**: same as View A
- **Phase shading on Moon**: same algorithm as Phase 1, just styled dark-on-dark

---

## Toggle Between Views

The toggle button is styled differently in Phase 2:

- Small, pill-shaped button in the top-right corner of the canvas
- `border: 1px solid rgba(255,255,255,0.2)`, `color: rgba(255,255,255,0.6)`, `background: rgba(0,0,0,0.4)`
- Label: "⬤ Horizon" / "◎ Orbital"

---

## Controls (Phase 2 Style)

The controls live **below** the canvas (not overlaid, which would ruin the immersion).

- Dark pill background: `background: #0d1225`, `border-radius: 12px`, `padding: 12px 20px`
- Text in muted white
- Range input styled with a custom track: dark gray, thumb in gold
- Play button styled as a circular icon button

### Custom Scrubber Styling

```css
input[type=range] {
  -webkit-appearance: none;
  width: 100%;
  height: 4px;
  background: #1a2540;
  border-radius: 2px;
  outline: none;
}
input[type=range]::-webkit-slider-thumb {
  -webkit-appearance: none;
  width: 16px;
  height: 16px;
  border-radius: 50%;
  background: #c8a000;
  cursor: pointer;
}
```

---

## Phase Indicator (Phase 2 Style)

In Phase 2, render the phase indicator as a larger, more beautiful Moon glyph — ~40px radius — using the same phase-drawing algorithm but with:

- Dark disc background
- Soft glow halo around the lit portion
- Phase name label below: "Waxing Gibbous"
- Fraction label: "67% illuminated"

Position it in the **bottom-right corner of the canvas**, overlaid on the sky with slight transparency in the background.

---

## Atmospheric Effects (build if time allows, not required for v1)

These are optional enhancements for Phase 2:

1. **Moon glow on horizon** — when Moon is just below the horizon, paint a soft amber/white glow at the rise azimuth point on the treeline
2. **Horizon glow from Moon** — the treeline is faintly lit in the direction of the Moon (ambient light simulation)
3. **Clouds** — slowly drifting semi-transparent blobs, purely cosmetic. Animated with `requestAnimationFrame` independently of the scrubber. Toggle off during play mode to avoid distraction.

---

## Performance Notes

- Treeline is generated once on init (seeded), stored as a path, not regenerated on every frame
- Stars are generated once on init
- Sky gradient is recreated each frame (needed for twilight transitions) — cheap, fine
- Moon arc is computed once per `currentDate` change (not every animation frame)
- Clouds (if implemented): use separate low-fps `setInterval` (every 100ms), not the main render loop

---

## Emotional Design Goals

When someone opens Phase 2 and presses Play, they should feel:

1. *Orientation* — "I recognize this. I'm looking at a horizon."
2. *Discovery* — "Wait, the Moon is moving... and it's coming up in a different place every night."
3. *Wonder* — "It's swinging north and south over weeks. This is actually beautiful."

The scientific view (Phase 1) answers *how*. This view answers *why it matters*.

---

## Acceptance Criteria (Phase 2)

- [ ] Canvas is visually dark with a sky gradient that shifts at twilight
- [ ] Stars appear at night and fade at twilight
- [ ] Treeline silhouette is present and has organic variation
- [ ] Moon arc has a visible glow effect
- [ ] Moon dot shows phase shading
- [ ] All Phase 1 functional behavior is preserved (scrubber, play, toggle, phase indicator, live text)
- [ ] Works as a local file (`file://`)
- [ ] No layout overflow on a 1280px wide macOS browser window

---

*End of PRD-02*
