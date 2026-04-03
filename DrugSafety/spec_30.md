# Drug Safety Explorer — Product Specification

## Overview

Drug Safety Explorer is a single-page web application that lets users query the OpenFDA API to explore and compare publicly available drug safety information. It is intended for **educational and informational purposes only** and does not constitute medical advice.

---

## Requirements

### Mandatory compliance

| # | Requirement | Implementation |
|---|-------------|----------------|
| 1 | Query at least one OpenFDA endpoint using live API calls (no hardcoded data) | Three endpoints queried per drug via `fetch()` on every search |
| 2 | Single-page web application | Self-contained `drug-safety-explorer.html` — no server required, opens directly in any browser |
| 3 | Clear educational disclaimer | Permanent red banner directly below the header, visible before any results load |
| 4 | Required OpenFDA attribution (verbatim) | Footer on every page load: *"This product uses publicly available data from the U.S. Food and Drug Administration (FDA). FDA is not responsible for the product and does not endorse or recommend this or any other product."* |

---

## Data Sources

All data is fetched live from `https://api.fda.gov`. No data is hardcoded.

### `/drug/label.json`
Official FDA-approved prescribing information (package insert), reviewed before a drug is approved. Used for:
- Boxed warnings and general warnings text
- Brand name lookup

### `/drug/event.json`
FDA Adverse Event Reporting System (FAERS) — voluntary self-reports submitted by patients, doctors, pharmacists, and manufacturers after a drug is on the market. Used for:
- Top adverse reactions ranked by report count
- Total report count per drug

### `/drug/enforcement.json`
Drug recall and enforcement reports. Used for:
- Recall reason and description
- Recall severity classification (Class I / II / III)

---

## Features

### Search

- **Drug A field** — required; monospace text input, Enter key triggers search
- **Drug B field** — optional; enables comparison mode when filled; Enter key triggers search
- **Analyze button** — fetches all three endpoints in parallel for each drug using `Promise.all`
- **Loading bar** — animated 3px gradient bar (teal → blue → amber) shown during fetch
- **7 preset pairs** — one-click shortcuts that populate both fields and auto-run the search:
  - Ibuprofen vs Naproxen
  - Metformin vs Glipizide
  - Atorvastatin vs Simvastatin
  - Sertraline vs Fluoxetine
  - Lisinopril vs Amlodipine
  - Amoxicillin (single)
  - Warfarin vs Aspirin

### Results — Single drug mode

When only Drug A is entered, a single full-width card is rendered (max-width 660px).

### Results — Comparison mode

When both Drug A and Drug B are entered, two cards are rendered side by side in a 1fr 1fr grid, followed by a comparison chart panel.

### Drug card

Each drug card contains three sections:

**1. Label warnings**
- Displays the first 420 characters of the drug's warning or boxed warning text from label data
- Amber left-border callout box with scroll if content overflows 108px
- Falls back to a "no data" message if label endpoint returns nothing

**2. Top adverse events — FAERS**
- Horizontal bar chart (CSS bars, not canvas) showing up to 8 reactions ranked by report count
- Bar width is proportional to the top result (100% = highest count)
- Report count shown in monospace at right
- Drug A bars: teal (`#009966`); Drug B bars: amber (`#c27000`)
- Falls back gracefully if FAERS returns no results

**3. Recalls & enforcement**
- Up to 4 most recent recall records
- Each item shows a severity badge + truncated reason text (130 chars)
- Falls back to "no data" message if no recalls found

**Data quality chips** (shown under label warnings section):
- `label: ok / none` — green or gray
- `events: ok / sparse / none` — green, amber, or gray
- `recalls: ok / sparse / none` — green, amber, or gray

**Report count** — shown in the card header in monospace (e.g. `1,234,567 reports`)

### Recall severity badges

| Badge | Color | Meaning |
|-------|-------|---------|
| `CLASS I` | Red | Reasonable probability of serious health consequences or death |
| `CLASS II` | Amber | May cause temporary or reversible harm; remote probability of serious harm |
| `CLASS III` | Teal | Unlikely to cause adverse health consequences |

### Adverse event comparison chart (comparison mode only)

- Chart.js 4.4.1 horizontal bar chart rendered on `<canvas>`
- Up to 9 unique reaction terms drawn from the top 6 results of each drug
- Drug A: teal bars; Drug B: amber bars
- Custom tooltip: white background, dark text, report count formatted with `toLocaleString()`
- X-axis ticks abbreviated (e.g. `120k`) using IBM Plex Mono
- Chart height auto-scales: `max(280, terms.length × 46 + 60)px`
- Destroyed and re-created on each new search to avoid Chart.js conflicts
- Followed by an amber FAERS disclaimer note

---

## Help & Education System

### "How to read this data" button

Located in the header. Opens a **tabbed modal** covering six topics:

| Tab | Content |
|-----|---------|
| About | Overview of all three data sources and the tool's purpose |
| Adverse events | FAERS limitations — voluntary reporting, correlation ≠ causation, popularity bias |
| Label warnings | FDA-approved label structure — boxed warnings, contraindications, adverse reactions |
| Recalls | Class I / II / III definitions with concrete examples |
| Report counts | Why popular drugs accumulate more reports; why raw counts are misleading |
| Interactions | Classic high-risk drug pairs (Warfarin + NSAIDs, MAO inhibitors + SSRIs, etc.) |

### Contextual `i` buttons

Each section head in every drug card has a small circular `i` button that opens the relevant modal:
- Label warnings → "What drug labels actually tell you"
- Top adverse events → "How to interpret adverse event data"
- Recalls & enforcement → "Understanding recall classifications"
- Comparison chart → "How to interpret adverse event data"
- Comparison chart legend → "Drug pairs with known dangerous interactions" (red `!` button)

### Modal behavior
- Backdrop click closes modal
- Escape key closes all open modals
- ✕ button closes modal
- Body scroll is locked while any modal is open

---

## Layout

```
┌─────────────────────────────────────────────────────┐
│  HEADER  [logo]         [? How to read] [Live badge]│  56px sticky
├─────────────────────────────────────────────────────┤
│  DISCLAIMER BANNER (educational use only)           │  always visible
├────────────┬────────────────────────────────────────┤
│            │                                        │
│  SIDEBAR   │  MAIN CONTENT                          │
│  300px     │                                        │
│  sticky    │  Hero heading                          │
│            │  Loading bar                           │
│  Search    │  Drug grid (1 or 2 columns)            │
│  Presets   │  Comparison chart                      │
│  Notice    │  FAERS note                            │
│            │                                        │
├────────────┴────────────────────────────────────────┤
│  FOOTER  [OpenFDA attribution] [data source info]   │  48px
└─────────────────────────────────────────────────────┘
```

Sidebar is sticky (`top: 56px`, `height: calc(100vh - 56px - 48px)`) so it stays in view while main content scrolls.

---

## Visual Design

### Theme
Light mode. Clean white card surfaces on a soft slate background.

### Color palette

| Token | Value | Usage |
|-------|-------|-------|
| `--bg` | `#f0f4f8` | Page background |
| `--bg-panel` | `#ffffff` | Header, footer, sidebar |
| `--bg-card` | `#ffffff` | Drug cards, compare panel |
| `--bg-sidebar` | `#f8fafc` | Sidebar background |
| `--teal` | `#009966` | Drug A bars, primary action, accents |
| `--amber` | `#c27000` | Drug B bars, warnings, data notices |
| `--red` | `#c62828` | Class I recalls, disclaimer, errors |
| `--blue` | `#1565c0` | Info buttons, links, hero italic |
| `--purple` | `#6a1b9a` | "How to read" button |
| `--text` | `#0d1f35` | Primary text |
| `--text-muted` | `#4a6080` | Secondary text |
| `--text-faint` | `#8da5be` | Labels, placeholders, metadata |

Each accent has a matching `-bg` and `-border` tint variant used for badges, chips, and callout boxes.

### Typography

| Font | Weights | Usage |
|------|---------|-------|
| IBM Plex Sans | 300, 400, 500, italic | All body text, UI labels |
| IBM Plex Mono | 400, 500 | Inputs, counts, badges, section labels, buttons |

Base font size: 15px, line-height: 1.6.

### Key UI details
- Cards: 8px border-radius, 1px border, `box-shadow: 0 1px 4px rgba(0,0,0,.06)`
- Header: sticky, `box-shadow: 0 1px 3px rgba(0,0,0,.06)`
- Inputs: focus ring `0 0 0 3px rgba(0,153,102,.12)` with teal border
- Analyze button: solid teal, darkens on hover, `scale(0.98)` on active
- Card fade-in animation: `translateY(8px) → none`, 300ms, second card delayed 70ms
- Bar fills: `transition: width 700ms cubic-bezier(.23,1,.32,1)`
- Loading bar animation: gradient slides across at 1.4s ease-in-out loop

### Responsive breakpoint (`≤ 860px`)
- Sidebar unsticks and stacks above main content
- Two-column drug grid collapses to single column
- Horizontal padding reduces to 1.1rem

---

## Dependencies

| Library | Version | How loaded |
|---------|---------|------------|
| Chart.js | 4.4.1 | `<script src="https://cdnjs.cloudflare.com/...">` |
| IBM Plex Sans | latest | Google Fonts `<link>` |
| IBM Plex Mono | latest | Google Fonts `<link>` |

No build step. No framework. No server. Pure HTML + CSS + vanilla JS.

---

## Data Handling

### API query patterns

```
/drug/label.json?search=openfda.generic_name:"<drug>"&limit=1
/drug/event.json?search=patient.drug.medicinalproduct:"<DRUG>"&count=patient.reaction.reactionmeddrapt.exact&limit=10
/drug/enforcement.json?search=product_description:"<drug>"&limit=5
/drug/event.json?search=patient.drug.medicinalproduct:"<DRUG>"&limit=1  (for total count)
```

Note: FAERS queries use the drug name uppercased, which matches more records in the medicinalproduct field.

### Sparse/missing data handling
- Each `apiFetch()` call is wrapped in try/catch — network failures return `null` silently
- Every card section has a graceful "no data" fallback message
- Data quality chips communicate availability at a glance without breaking layout
- No section is hidden or collapsed when data is missing — the absence of data is surfaced explicitly

### Text truncation
- Label warnings: 420 characters, scrollable to 108px max-height
- Recall descriptions: 130 characters
- All truncation appended with `…`

---

## Disclaimers & Limitations Communicated to Users

The tool surfaces data literacy information in four places:

1. **Disclaimer banner** (always visible) — educational use only, not medical advice
2. **Sidebar data notice** — adverse event counts are reports, not confirmed causal links
3. **Section-level `i` buttons** — contextual modals explaining each data type
4. **FAERS note** (below comparison chart) — report volume ≠ incidence rate or severity
5. **OpenFDA attribution** (footer) — FDA is not responsible for this product

---

## Delivery

- **Format:** Single `.html` file
- **Filename:** `drug-safety-explorer.html`
- **Runtime:** Any modern browser (Chrome, Firefox, Safari, Edge)
- **Network:** Requires internet connection to query `api.fda.gov` and load fonts/Chart.js from CDN
