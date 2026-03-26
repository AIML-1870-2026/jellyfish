# Weather Dashboard — Product Specification

## Overview

A single-file HTML weather dashboard that fetches live data from the OpenWeatherMap API and presents it in a polished, animated dark-sky interface. No build step, no server required — open the file in any modern browser.

---

## API

- **Provider:** [OpenWeatherMap](https://openweathermap.org)
- **API key:** Baked directly into the HTML file (hardcoded constant `API_KEY`)
- **Endpoints used:**
  - `geo/1.0/direct` — city name → lat/lon lookup
  - `data/2.5/weather` — current conditions
  - `data/2.5/forecast` — 3-hour interval forecast (56 slots ≈ 7 days)
  - `data/2.5/air_pollution` — AQI and pollutant breakdown
- **Fetch strategy:** Current, forecast, and AQI are fetched in parallel via `Promise.all`. Data is always fetched in metric units and converted client-side, enabling instant unit switching without re-fetching.

---

## Starting Screen (Splash)

Shown on first load before any city is searched.

- Animated SVG hero: spinning sun with 8 radiating rays, pulsing golden core, two drifting cloud layers
- 22 floating glowing orbs (warm gold + cool blue palette) that drift upward and fade — pure CSS, no emoji
- Radial glow halo behind the hero element
- Twinkling star field (100 stars, randomised size, opacity, and blink timing)
- Daytime sky gradient set on the canvas background at init
- Headline: "What's the weather like in **your city?**"
- Six quick-city shortcut buttons, each with a small inline SVG icon (no emoji):
  - Tokyo — sun rays icon
  - London — cloud icon
  - Reykjavik — snowflake icon
  - Miami — lightning bolt icon
  - Sydney — partly cloudy icon
  - New York — rain drops icon
- Clicking a chip auto-populates the search field and triggers a fetch

---

## Top Bar (always visible)

- **Search input** — free-text city name entry; submits on Enter or button click
- **Search button**
- **+ Save button** — appears after first successful fetch; saves the current city to the saved cities bar
- **Unit toggle** — three-button segmented control: `°C` / `°F` / `K`; switches immediately by re-rendering from cached data

---

## Saved Cities Bar

- Horizontally scrollable chip row above the top bar
- Each chip shows the city name and country code
- Clicking a chip loads that city instantly
- Each chip has an `✕` remove button
- Persists across browser sessions via `localStorage` (key: `wx_saved`)
- Shows a muted placeholder text when no cities are saved

---

## Dynamic Background

A `<canvas>` element fills the viewport and draws a sky gradient that responds to the searched city's **local time of day** and **weather conditions**.

### Time phases

| Phase | Condition |
|---|---|
| `night` | More than 90 min before sunrise or after blue hour ends |
| `astronomical` | 60–90 min before sunrise |
| `nautical` | 30–60 min before sunrise |
| `civil` | 0–30 min before sunrise |
| `golden` | Within 60 min of sunrise or sunset |
| `day_clear` | Daytime, clear or light cloud |
| `day_cloudy` | Daytime, OWM weather ID indicates cloud/rain |
| `sunset_gold` | 0–30 min after sunset |
| `dusk` | 30–40 min after sunset |

### Weather overlays
- Stormy conditions (IDs 200–599) darken the palette regardless of time
- Cloudy conditions add a semi-transparent radial cloud overlay
- Golden/sunset phases add a warm horizon glow
- Star field opacity fades to 0 during daytime and rises to full during night phases

---

## Current Conditions Card

- City name and country code (from geo API)
- Large temperature display (80px, light weight)
- Weather description (capitalised)
- Feels-like, daily high, and daily low
- Animated SVG weather icon (see Icon System below)
- Current date (localised, e.g. "Thursday, Mar 26")

---

## Stats Row

Six stat cards in a responsive auto-fit grid:

| Stat | Detail |
|---|---|
| Humidity | Percentage with a fill bar |
| Wind | Speed (km/h or mph) + cardinal direction |
| Visibility | In km with a fill bar |
| Pressure | In hPa |
| Sunrise | Local time (UTC-offset) |
| Sunset | Local time (UTC-offset) |

---

## Air Quality Card

- AQI level badge (1–5 scale: Good / Fair / Moderate / Poor / Very Poor) with colour-coded background
- Pollutant breakdown: PM2.5, PM10, O₃, NO₂, CO

---

## Sunrise / Sunset Arc

A full-width SVG arc diagram showing the sun's current position across the day:

- Semicircular track arc
- Gold progress arc filled from sunrise to current position
- Animated sun dot at current position (only shown during daytime)
- Sunrise and sunset times labelled below the arc endpoints
- Golden hour marker dots on the arc

---

## Golden & Blue Hour Card

Two-column grid (morning / evening), each showing:

- Blue hour window (45 min before sunrise / immediately after sunset)
- Golden hour window (sunrise to +60 min / sunset −60 min to sunset)

All times derived from sunrise/sunset timestamps + city timezone offset; no additional API call needed.

---

## Hourly Forecast

Horizontally scrollable row of 9 items (3-hour intervals, ~24 hours):

- "Now" label on the first item (highlighted with blue tint)
- Local time for subsequent items
- Animated SVG weather icon
- Temperature in selected unit
- Rain probability (shown if > 0%)

---

## 7-Day Forecast

Vertical list of up to 7 days, grouped from the 56-slot 3-hour forecast:

- Day label ("Today" or short weekday name)
- Animated SVG weather icon
- Weather description
- Rain probability (shown if > 0%)
- Low / high temperatures in selected unit
- Proportional temperature range bar (gradient blue → amber), positioned relative to the week's global min/max

---

## Animated SVG Icon System

All weather icons are inline SVG with CSS keyframe animations — no emoji, no external image assets.

| Condition | Animation |
|---|---|
| Clear day | Spinning rays + pulsing core circle |
| Clear night | Crescent moon with glowing drop-shadow |
| Partly cloudy (day) | Sun behind drifting cloud ellipses |
| Partly cloudy (night) | Moon behind drifting cloud ellipses |
| Overcast | Two layered drifting cloud groups |
| Drizzle | Cloud + 2 staggered falling rain lines |
| Rain | Cloud + 4 staggered falling rain lines |
| Thunderstorm | Cloud + rain lines + flashing lightning path |
| Snow | Cloud + rotating/falling snowflake dots |
| Fog / mist | Three horizontally drifting fog bars |

Icons are rendered at three sizes: 80px (current card hero), 28px (hourly row), 24px (weekly row).

---

## Unit System

| Unit | Temperature | Wind speed |
|---|---|---|
| Metric (°C) | Celsius | km/h |
| Imperial (°F) | Fahrenheit | mph |
| Standard (K) | Kelvin | km/h |

Conversion is done client-side from cached metric values. Switching units re-renders all temperature and wind displays instantly.

---

## Loading State

A full-screen overlay with a spinning ring and contextual text ("Looking up [city]…" → "Loading weather data…") covers the viewport during fetch.

---

## Error Handling

- Invalid or not-found city: inline error bar below the top bar with a red-tinted message
- 401 response from API (bad key): specific "Invalid API key" message
- All other errors: raw error message displayed

---

## Design System

- **Fonts:** DM Sans (300, 400, 500) + DM Mono (400, 500) via Google Fonts
- **Colour palette:** Dark navy base (`#0a1628`) with translucent white card surfaces; accent blue `#50b8f0`; warm gold `#ffd84a`
- **Cards:** `rgba(255,255,255,0.07)` background, `rgba(255,255,255,0.13)` border, `border-radius: 10–16px`
- **Responsive:** Single-column layout below 620px; description column and some bar widths collapse on mobile

---

## Technical Notes

- Single HTML file — no framework, no build tooling, no external JS libraries
- Background rendered on a `<canvas>` element, redrawn on window resize
- All data cached in JS module-scope variables; unit switching never triggers a network request
- Saved cities stored in `localStorage` as a JSON array of `"City, CC"` strings
- Star field generated programmatically at init (100 `<div>` elements with CSS custom properties)
- Splash floater particles generated programmatically (22 `<div>` elements)
