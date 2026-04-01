# Asteroid Watch — Product Spec

**Last updated:** April 1, 2026  
**Data source:** NASA / JPL CNEOS APIs  
**Delivery format:** Single self-contained HTML file (no build step, no server required)

---

## Overview

A single-page asteroid tracking dashboard that pulls live data from two NASA JPL APIs and presents it across four tabs. The visual design is retro/pixelated, inspired by Galaga and Asteroids arcade games, with a warm light background (not dark). Plain English is used throughout — no scientific jargon without explanation.

---

## Data Sources

### 1. SBDB Close-Approach Data API
- **URL:** `https://ssd-api.jpl.nasa.gov/cad.api`
- **Parameters used:** `date-min`, `date-max`, `dist-max=0.05`, `neo=true`, `sort=date`, `limit=300`
- **Fields consumed:** `des` (designation), `cd` (close-approach date), `dist` (distance in AU), `v_rel` (relative velocity km/s), `h` (absolute magnitude)

### 2. Sentry Impact Risk API
- **URL:** `https://ssd-api.jpl.nasa.gov/sentry.api`
- **Parameters used:** `ps-min=-4` (Palermo Scale filter — returns ~40 highest-concern objects, keeps payload fast)
- **Fields consumed:** `des`, `fullname`, `ip` (impact probability), `ps_cum` (cumulative Palermo Scale), `h`, `range` (impact year window), `diameter`

### CORS Strategy
The JPL APIs do not send CORS headers, so direct `fetch()` calls from a local file (`file://`) are blocked by browsers. The solution is to **bake the data in at build time**: fetch live data server-side during development, embed it as a JavaScript constant in the HTML, and ship the file with no runtime network requests. This means the file works when opened directly from disk with no server.

Date strings from the CAD API use the format `"2026-Apr-01 16:03"` which is not reliably parsed by `new Date()` across browsers. A manual parser is required:
```js
function parseCd(cd) {
  const months = {Jan:0,Feb:1,Mar:2,Apr:3,May:4,Jun:5,Jul:6,Aug:7,Sep:8,Oct:9,Nov:10,Dec:11};
  const m = cd.match(/(\d{4})-([A-Za-z]{3})-(\d{2})\s+(\d{2}):(\d{2})/);
  if (!m) return new Date(cd);
  return new Date(Date.UTC(+m[1], months[m[2]], +m[3], +m[4], +m[5]));
}
```

---

## Visual Design

### Aesthetic
- Retro/pixelated — inspired by Galaga and Asteroids arcade games
- **Not dark:** warm cream/parchment background (`#f0ead8`), white panels
- Typography: `Press Start 2P` (headers, labels) + `VT323` (body text, numbers)
- Pixel-shadow borders on all cards and panels (`box-shadow: 3px 3px 0 #1a1a2e`)
- Dot-grid background pattern via CSS `radial-gradient`

### Color Palette
| Variable | Hex | Usage |
|---|---|---|
| `--bg` | `#f0ead8` | Page background |
| `--card` | `#ffffff` | Panel backgrounds |
| `--border` | `#1a1a2e` | All borders and shadows |
| `--green` | `#00b388` | Safe / far / positive |
| `--yellow` | `#f0a500` | Warning / moderate closeness |
| `--red` | `#e03030` | Danger / very close / hazardous |
| `--blue` | `#2d6be4` | Info boxes, tip borders |
| `--purple` | `#7c3aed` | Watch list accent |

### Proximity Color Coding (used across tabs)
- 🟢 **Green** — more than 10 lunar distances away
- 🟡 **Yellow** — 5–10 lunar distances away  
- 🔴 **Red** — fewer than 5 lunar distances away

---

## Unit Conversions

All distances are shown in **Lunar Distances (LD)** as the primary human-readable unit.

```
1 AU  = 149,597,870.7 km
1 LD  = 384,400 km  (Earth to Moon)
```

Speed context: velocity shown as `X km/s` plus `X× bullet speed` (bullet = 1 km/s).

Size estimation from absolute magnitude H (assumed albedo 0.14):
```
diameter_metres = (1329 / sqrt(0.14)) × 10^(-H/5)
```

---

## Tab Specifications

### Tab 1 — 🌍 Orbital Map *(landing tab)*

An interactive 3D globe showing this month's close approaches plotted around Earth at their relative miss distances, with the Moon as a reference point.

**Technology:** Three.js (r128) via CDN  
**CDN:** `https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js`

**Earth globe:**
- Real NASA earth texture: `https://threejs.org/examples/textures/land_ocean_ice_cloud_2048.jpg`
- Pixelation applied via canvas downscale trick: draw at 128×64, upscale to 1024×512 with `imageSmoothingEnabled = false`, overlay pixel grid lines
- Two-layer atmosphere: inner halo (sphere radius 4.22, blue, `opacity: 0.10, side: BackSide`) + outer rim (radius 4.45, `opacity: 0.04`)
- Earth rotates continuously on Y axis

**Moon:**
- Pixelated canvas texture (grey with dark square craters, grid overlay)
- Orbits at `LD_SCENE = 7` scene units (representing 1 LD)
- Orbits continuously with a sprite label: `🌙 MOON (1 LD)`
- Reference orbit ring drawn at the moon's radius

**Asteroids:**
- All baked CAD rows shown (up to 12, sorted by date)
- Evenly distributed around Earth at fixed angles (no random jitter)
- Distance from Earth centre in scene = `clamp(ld × 7, 6, 28)` scene units
- Shape: cluster of 7 voxel cubes (BoxGeometry 0.28 units) in a cross pattern
- Large glow sprite behind cluster (scale 2.2, opacity 0.5)
- Orbit ring drawn at asteroid's radius
- Sprite label with designation name above each
- On hover: tooltip shows designation, miss distance (LD), speed (km/s), estimated size, date

**Interaction:**
- Mouse drag → orbit rotation (spherical coordinates)
- Scroll wheel → zoom (radius 8–60)
- Touch drag supported
- Raycaster hover detection on asteroid cube children

**Legend:** Green = >10 LD, Yellow = 5–10 LD, Red = <5 LD, Light blue = Moon

---

### Tab 2 — ⭐ Fast Facts

A scoreboard of headline metrics for the current month.

**Stat cards (6 total):**
1. 🪨 **Flybys this month** — total count from CAD data
2. 📏 **Closest approach** — minimum `dist`, shown in LD, with designation
3. 💨 **Fastest flyby** — maximum `v_rel` in km/s, with designation
4. ⚠ **Potentially hazardous** — count where `auToLd(dist) < 19.5` AND `H < 22`
5. 🔭 **Top Sentry risk** — highest `ip` object, shown as "1-in-X", with year range
6. 🌍 **On watch list** — total Sentry object count

**Next Flyby card:** Shows the first upcoming CAD entry with: designation, date, miss distance (LD + million km), speed (km/s + bullet multiple), and estimated size.

---

### Tab 3 — 🪨 This Month

Scrollable list of all close approaches this month.

**Columns:** Name + date | Distance bar | Distance in LD + km | Speed

**Row color coding** (left border stripe):
- Red border: < 5 LD
- Yellow border: 5–10 LD
- Green border: > 10 LD

Each row includes:
- Designation and date
- "X lunar distances" with a filled bar (scaled to 20 LD max)
- Estimated size from H magnitude
- Speed in km/s + bullet multiple

Max scroll height: 440px with styled scrollbar.

**Explainer box** at top: plain English explanation that none are on a collision course, definition of LD, color legend.

---

### Tab 4 — ⚠ Watch List

Top 15 Sentry objects ranked by impact probability (highest first).

**Each row shows:**
- Rank + designation
- Full name (if available)
- Possible impact window (`range` field)
- Relative impact probability bar (scaled to highest IP in list = 100%)
- "1-in-X chance" in plain English
- Risk level badge: `✓ ROUTINE` (PS < -2) / `◆ LOW` (PS -2 to -1) / `⚠ NOTABLE` (PS > -1)
- Raw Palermo Scale value

**Explainer box:** Explains that Sentry monitors theoretical future impacts, that chances are tiny, and that no asteroid is confirmed on a collision course.

**Palermo Scale explained in footnote:** Below −2 = routine monitoring level.

---

### Tab 5 — 📐 How Big?

Interactive size comparison: pick an asteroid from a list and see it drawn to scale alongside everyday reference objects.

**Reference objects (left side):**
| Name | Diameter |
|---|---|
| Person | 1.8 m |
| Bus | 12 m |
| Blue Whale | 30 m |
| 747 Jet | 64 m |
| Eiffel Tower | 330 m |
| Empire State Building | 443 m |
| 1km City Block | 1,000 m |

**Asteroid (right side, separated by a bold "VS" divider):**
- Isolated in its own red-tinted zone
- Red downward arrow + "ASTEROID" label floating above
- Red name banner below with size
- Shape has pixel highlights (lighter top-left) and shadow (darker right/bottom)
- 4 pixel craters with inner highlight
- Glow halo (ellipse gradients)

**Reference object labels:** Each has a colored floating banner above the shape with an arrow pointing down, plus size in metres/km below the ground line.

**SVG scene layout:**
- Sky gradient background with pixel grid overlay
- Grass/ground strip at bottom
- All shapes rendered as SVG `<rect>` elements in pixel-block style (no smooth curves)
- SVG dimensions: full panel width × 460px
- Ground line at Y = SVG_H − 55

**Pixel shapes by type:**
- `person` — square head + rectangular body + two leg blocks
- `bus` — wide rectangle with pixel windows
- `whale` — horizontal body + tail blocks + dorsal fin
- `plane` — fuselage + wide wing + tail fin + nose
- `tower` — tapered stacked blocks + spire (Eiffel silhouette)
- `building` — stepped pyramid blocks + yellow lit windows (Empire State silhouette)
- `city` — 4 varied-height buildings with windows (skyline)
- `asteroid` — irregular cross of rects + highlights + shadow + 4 crater pits + red glow

**Asteroid list:** Populated from all unique designations in CAD data + top 30 Sentry objects, sorted by H (largest first / lowest H value first).

---

## Technical Constraints

- **Single HTML file** — no external CSS files, no JS modules, no build step
- **No runtime network requests** — all data baked in as JS constants
- **Works from `file://`** — opened directly from disk, no local server needed
- **Three.js r128** — only version available on cdnjs that matches known API; do not use `CapsuleGeometry` (added in r142)
- **Google Fonts** — loaded via `@import` in CSS (`Press Start 2P`, `VT323`); requires internet for font loading but degrades gracefully
- **No `localStorage`** — not supported in some environments; all state in memory

---

## File Structure

```
asteroid-dashboard.html
├── <style>           CSS variables, layout, all component styles
├── <body>
│   ├── .hdr          Header bar (title + live pill)
│   ├── .tabs         Tab buttons
│   └── .panels
│       ├── #tab-earth     Orbital map (Three.js canvas)
│       ├── #tab-facts     Fast facts (rendered by renderFacts())
│       ├── #tab-month     Flyby list (rendered by renderMonth())
│       ├── #tab-watch     Watch list (rendered by renderWatch())
│       └── #tab-big       How Big? (rendered by renderBig() + drawSilhouette())
└── <script>
    ├── RAW_CAD            Baked flyby data (30 rows)
    ├── RAW_SENTRY         Baked Sentry data (15 rows)
    ├── Utility functions  auToLd(), hToDiam(), fmtSize(), parseCd()
    ├── go()               Tab switching
    ├── initEarth()        Three.js globe (lazy, runs on first tab visit)
    ├── makeLabel()        Three.js sprite label generator
    ├── renderFacts()      Fast Facts tab
    ├── renderMonth()      This Month tab
    ├── renderWatch()      Watch List tab
    ├── renderBig()        How Big? tab (builds picker list + blank canvas)
    ├── drawSilhouette()   Renders full SVG comparison scene for selected asteroid
    └── drawShape()        Draws a single pixel-art silhouette shape in SVG
```

---

## Known Limitations

- Baked data goes stale — reflects the fetch date (April 1, 2026). To refresh, re-fetch both APIs and update `RAW_CAD` and `RAW_SENTRY` constants.
- Asteroid sizes are estimates from H magnitude assuming albedo 0.14. Real sizes can vary significantly.
- Orbital map is not astronomically accurate — distances are scaled for visibility, and asteroids are distributed evenly around Earth for clarity rather than placed at true orbital positions.
- Google Fonts require an internet connection. Without it, the browser falls back to system monospace fonts.
