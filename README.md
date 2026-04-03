# Moonrise

An interactive Moon simulator that shows why the Moon rises in a different place on the horizon every night.

## Open It Online

- Live scientific view: [https://lpcode808.github.io/SkySimulator/](https://lpcode808.github.io/SkySimulator/)
- Live immersive view: [https://lpcode808.github.io/SkySimulator/moonrise-p2.html](https://lpcode808.github.io/SkySimulator/moonrise-p2.html)
- GitHub repo: [https://github.com/lpcode808/SkySimulator](https://github.com/lpcode808/SkySimulator)

## What This Project Is

This project is a small astronomy explainer built as plain HTML files.

There is no build step, no framework, and no package manager.

You can open the files directly in a browser, host them on GitHub Pages, or run a tiny local server if you want cleaner URLs while testing.

## What Each Page Does

- `index.html` is the Phase 1 scientific version. It is clean, labeled, and diagram-focused.
- `moonrise-p2.html` is the Phase 2 immersive version. It feels more like standing outside at dusk and watching the Moon move across the sky.

Both pages use the same astronomy idea:

- drag the date scrubber
- watch the Moon's rise point shift north and south
- switch between the horizon view and orbital view

## Files In This Repo

- `index.html` — scientific diagram view
- `moonrise-p2.html` — immersive atmospheric view
- `PRD-00-master-context.md` — overall project rules and constraints
- `PRD-01-phase1-scientific.md` — Phase 1 product spec
- `PRD-02-phase2-immersive.md` — Phase 2 product spec
- `ASTRONOMY-REF.md` — astronomy notes and formulas behind the app
- `LEARNING-DESIGN-REVIEW.md` — extra design notes kept in the repo

## Quick Code Map (single-file friendly)

Both app pages are intentionally single-file so they can still be copy/pasted into simple hosts (including Google Sites style workflows).

When editing either HTML page:

- **Intro + controls markup**: search for `<section class="app-shell">`
- **Look and feel (CSS)**: in the top `<style>` block
- **Keyboard + button wiring**: search for `function bindEvents()`
- **Main teaching copy updates**: search for `function updateCopy(data)`
- **Astronomy calculations**: search for `function getMoonDataForDate`
- **Canvas drawing**: search for `renderHorizonView` / `renderOrbitalView`

This keeps local opening and GitHub Pages deployment simple while still making the file easier to navigate.

## How To Run It Locally

### Easiest option

Double-click `index.html` or `moonrise-p2.html` and open it in your browser.

### Better option for testing

Run this in the project folder:

```bash
python3 -m http.server
```

Then open:

- `http://localhost:8000/` for the scientific view
- `http://localhost:8000/moonrise-p2.html` for the immersive view

## How GitHub Pages Works Here

This repo is already set up in a GitHub Pages-friendly way:

- the repo root can be published directly
- `index.html` becomes the main page
- `moonrise-p2.html` becomes a second page in the same site

If GitHub Pages is enabled for the repository root, the expected links are:

- `https://lpcode808.github.io/SkySimulator/`
- `https://lpcode808.github.io/SkySimulator/moonrise-p2.html`

## Notes

- The only external dependency is SunCalc v1.9.0 from cdnjs.
- Offline use after the first successful load depends on the browser caching that CDN script.
- The observer location is currently fixed to Waimanalo, Hawaiʻi.

## Mobile Strategy (portrait-first)

Yes — a mobile version is absolutely possible, and this project is already close because both pages are single-file and have a basic mobile media query.

The strategic decision is to ship in **two layers**:

1. **Responsive baseline (must-have)**
   - Keep one code path per page and make layout adapt to small portrait screens.
   - Prioritize learning flow over parity with desktop chrome (fewer visible controls at once is fine).
   - Preserve astronomy correctness and date-scrubbing as the core interaction.

2. **Mode-specific optimization (nice-to-have)**
   - Add a dedicated compact mobile mode for immersive interactions when needed.
   - Keep scientific and immersive pages distinct, but share common interaction rules and sizing tokens.

### Why portrait feels hard (and what to do)

- The canvas competes with controls for vertical space.
- Horizon visuals are naturally wide, but phones are narrow.
- Scrubber + status + explanatory text can push key visuals below the fold.

Recommended mitigation:

- Use a fixed viewport budget on phones (for example ~45–55vh for the main canvas region).
- Move secondary text into collapsible panels ("What to notice", "Moon phase details").
- Keep primary controls sticky at the bottom (Prev / Next / Play / Today + scrubber).
- Reduce simultaneous UI density rather than shrinking everything.

### Implementation roadmap

**Phase A — responsive hardening (lowest risk)**
- Expand mobile breakpoints for button sizing, spacing, and typography.
- Introduce canvas presets by breakpoint (desktop/tablet/phone).
- Reorder DOM sections on phone so users see: canvas → date controls → key insight → details.

**Phase B — mobile interaction polish**
- Add touch-first affordances (larger hit targets, swipe day-to-day).
- Add "quick jump" dates (new moon, full moon, ±14 days).
- Optionally support haptic-friendly micro-steps for scrubber adjustments.

**Phase C — optional dedicated mobile mode**
- Add a "Mobile focus mode" toggle that hides non-essential chrome.
- Keep mode state in URL params (`?mode=mobile-focus`) so links are shareable.

### Product guardrails

- Do not fork astronomy logic by device.
- Keep one source of truth for date state and Moon data.
- Treat mobile as a constrained teaching surface, not a compressed desktop clone.
