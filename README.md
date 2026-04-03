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
