# ASTRONOMY-REF — Moonrise: Astronomy Concepts for the Agent

> **For the coding agent:** This doc explains the astronomy the app is visualizing. Read it so your code comments and inline labels are accurate, not just technically functional. Understanding *why* these numbers mean what they mean will help you make better implementation decisions.

---

## The Core Question the App Answers

**Why does the Moon rise in a different place on the horizon every night?**

Short answer: because the Moon's orbital plane is tilted relative to Earth's equator, and as the Moon orbits Earth, its position in the sky (its *declination* — how far north or south it sits) swings up and down over a 29.5-day cycle. When the Moon is far north in its declination, it rises in the northeast. When it's far south, it rises in the southeast.

This is the same reason the *Sun* rises in different places — due north-east at summer solstice (Northern Hemisphere), due east at equinoxes, and southeast at winter solstice. The Moon does this same migration, but faster (29.5 days instead of 365) and with a wider swing (can go further north or south than the Sun ever does).

---

## Key Concepts Defined

### Azimuth
The compass bearing of an object on the horizon. Measured in degrees, clockwise from North.
- 0° = North
- 90° = East (where an object rises at the equinoxes)
- 180° = South
- 270° = West

**SunCalc note:** SunCalc returns azimuth in radians with 0 = South and positive = West. Convert to standard bearing: `bearing = (azRad * 180/Math.PI + 180) % 360`

### Altitude
How high an object is above the horizon. Measured in degrees (or radians in SunCalc).
- 0° = on the horizon
- 90° = directly overhead (the zenith)
- Negative = below the horizon

### Declination
How far north or south of the celestial equator an object sits. Like latitude, but for the sky.
- Positive = north of celestial equator
- Negative = south
- The Sun's declination ranges from about +23.5° (summer solstice) to -23.5° (winter solstice)
- The Moon's declination ranges from about +28.5° to -28.5° (at *major lunar standstill*) — wider than the Sun's because of the Moon's own orbital tilt of ~5.1°

### Phase (0–1 scale, as SunCalc returns it)
- 0 = New Moon (Moon between Earth and Sun, not illuminated from our perspective)
- 0.25 = First Quarter (right half lit, in Northern Hemisphere view)
- 0.5 = Full Moon (Moon opposite Sun, fully illuminated)
- 0.75 = Last Quarter (left half lit)
- Back to 1/0 = New Moon again

**Synodic month:** 29.53 days — the time from new moon to new moon. This is what the phase cycle tracks.

### Perigee / Apogee (Moon–Earth distance)
- **Perigee**: Moon closest to Earth (~356,000–370,000 km). Moon appears ~14% larger.
- **Apogee**: Moon furthest from Earth (~404,000–407,000 km).
- A full moon at perigee = "supermoon" (media term). It's measurably larger but the difference is subtle to the eye.

### Perihelion / Aphelion (Earth–Sun distance)
- **Perihelion**: Earth closest to Sun (~147.1M km). Occurs around January 3.
- **Aphelion**: Earth furthest from Sun (~152.1M km). Occurs around July 4.
- Northern Hemisphere is *closer* to the Sun in winter — counterintuitive but true.
- The 3.3% distance difference produces ~7% more solar energy at perihelion, but Earth's axial tilt effect is far larger and determines seasons.

### Axial Tilt (Obliquity)
Earth's spin axis is tilted 23.5° relative to its orbital plane around the Sun. This is the primary cause of seasons and the primary cause of why the Sun (and Moon) rise in different compass directions at different times of year.

### The 18.6-Year Nodal Cycle
The Moon's orbital plane slowly precesses (wobbles) over an 18.6-year cycle. At *major lunar standstill* (like ~2025–2026), the Moon's declination reaches its maximum swing: up to +28.5° and down to -28.5°. This means it rises further north and further south on the horizon than at any other point in the cycle. At *minor lunar standstill* (9.3 years later), the swing is reduced to about ±18.5°.

**In practical terms for the app:** The Moon's rise azimuth migrates further from due East at major standstill years than it does mid-cycle. This is visible in the app if the scrubber spans multiple years.

---

## Why the Moon Rises in Different Places: The Math Chain

1. The Moon orbits Earth once every 29.5 days (synodic period from our perspective).
2. The Moon's orbit is inclined ~5.1° to the ecliptic (the plane of Earth's orbit around the Sun).
3. The ecliptic is already tilted 23.5° to Earth's equator (that's the axial tilt).
4. These two tilts can add or subtract, giving the Moon a total declination range of roughly 18.5° to 28.5° (depending on where the nodal cycle is).
5. An observer on the ground sees the Moon rise at an azimuth determined by its declination and their latitude.
6. As the Moon's declination changes (over 29.5 days), its rise azimuth migrates north-south along the horizon.

**Rule of thumb for Waimanalo (lat 21.3°N):**
- Moon due east (90°) = Moon's declination is ~+21.3° (equal to observer latitude) — wait, actually this is more complex. At equator, object rises due east when declination = 0°. At any latitude L, an object with declination D rises at azimuth: `cos(azimuth) = sin(D) / cos(L)`. So at lat 21.3°N:
  - D = 0° → rises due East (90°) — equinox behavior
  - D = +28° → rises about 20° north of East (azimuth ~70°)
  - D = -28° → rises about 20° south of East (azimuth ~110°)

---

## Phase Names (for the live text label)

| Phase value (0–1) | Name |
|---|---|
| 0–0.03 or 0.97–1.0 | New Moon |
| 0.03–0.22 | Waxing Crescent |
| 0.22–0.28 | First Quarter |
| 0.28–0.47 | Waxing Gibbous |
| 0.47–0.53 | Full Moon |
| 0.53–0.72 | Waning Gibbous |
| 0.72–0.78 | Last Quarter |
| 0.78–0.97 | Waning Crescent |

```javascript
function getPhaseName(phase) {
  if (phase < 0.03 || phase > 0.97) return 'New Moon';
  if (phase < 0.22) return 'Waxing Crescent';
  if (phase < 0.28) return 'First Quarter';
  if (phase < 0.47) return 'Waxing Gibbous';
  if (phase < 0.53) return 'Full Moon';
  if (phase < 0.72) return 'Waning Gibbous';
  if (phase < 0.78) return 'Last Quarter';
  return 'Waning Crescent';
}
```

---

## Compass Direction Labels

For labeling bearings in human-readable form:

```javascript
function bearingToCompass(deg) {
  const dirs = ['N','NNE','NE','ENE','E','ESE','SE','SSE','S','SSW','SW','WSW','W','WNW','NW','NNW'];
  return dirs[Math.round(deg / 22.5) % 16];
}
```

---

## Simultaneous Full Moon Worldwide

Yes — the full moon phase is a specific geometric instant (Moon exactly opposite Sun as seen from Earth center). That instant occurs simultaneously everywhere on Earth. However:
- Whether that instant falls on a Tuesday or Wednesday depends on your time zone.
- The **orientation** of the lit crescent differs by hemisphere (Northern vs Southern observers see it "upside down" relative to each other).
- At high latitudes, the full moon may be above or below the horizon at that exact moment, but the phase itself is simultaneous.

---

## Blood Moon / Lunar Eclipse

A lunar eclipse occurs when the Earth passes directly between the Sun and the Moon, and Earth's shadow falls on the Moon. The Moon turns red because:
- Earth's atmosphere bends sunlight around the planet (refraction)
- The blue wavelengths scatter away (Rayleigh scattering)
- The red/orange wavelengths bend through and illuminate the Moon
- Essentially: every sunrise and sunset on Earth, simultaneously, paints the Moon

Lunar eclipses are visible from anywhere on Earth where the Moon is above the horizon. They are not caused by the Moon being close (that would be a supermoon), though both can coincide (super blood moon).

SunCalc does not compute eclipse events. A future version of the app could use a precomputed eclipse table or the Astronomical Algorithms formulae for this.

---

## Equation of Time (for future reference)

The "equation of time" explains why solar noon (when the Sun is highest) doesn't always match clock noon. Two causes:
1. **Orbital eccentricity** — Earth moves faster at perihelion (Jan), slower at aphelion (Jul). The Sun appears to move faster across the sky in Jan than Jul.
2. **Axial tilt** — the ecliptic (Sun's apparent path) is tilted to the equator, causing unequal apparent motion.

These two effects combine in a figure-8 pattern called an **analemma**. If you photographed the Sun at the same clock time every day for a year, it traces this figure-8 in the sky. This is a rich future layer for the app.

---

*End of ASTRONOMY-REF*
