# ReviewForge — Product Specification

**Version:** 1.0  
**Last Updated:** 2026-04-09  
**Status:** Implemented

---

## 1. Overview

ReviewForge is a single-file, browser-based AI product review generator powered by OpenAI. Users configure a set of parameters — product details, writing style, and per-aspect sentiment — then receive a streamed, markdown-rendered review from an OpenAI language model.

The application ships as a single `.html` file with no build step, no server, and no dependencies beyond a browser and an OpenAI API key.

---

## 2. Deployment

| Attribute | Value |
|---|---|
| Delivery format | Single standalone `.html` file |
| Runtime | Any modern browser (Chrome, Firefox, Safari, Edge) |
| Server required | No |
| Build step required | No |
| Node.js server variant | `server.js` — optional, single-file, no npm install |
| API key storage | In-memory only (never persisted, never transmitted except to OpenAI) |

### 2.1 Node.js server variant (optional)

A companion `server.js` was also produced. It:

- Reads `OPENAI_API_KEY` from a `.env` file or environment variable (in-memory only)
- Serves the full HTML page at `/`
- Proxies streaming OpenAI requests at `/api/generate`
- Requires only Node.js — no `npm install`
- Runs with: `node server.js` or `OPENAI_API_KEY=sk-... node server.js`

---

## 3. AI Integration

| Attribute | Value |
|---|---|
| Provider | OpenAI only |
| Endpoint | `https://api.openai.com/v1/chat/completions` |
| Response mode | Streaming (SSE / `stream: true`) |
| Output format | Unstructured free-form text (no JSON schema / structured output) |
| Markdown rendering | Yes — model output is rendered as formatted HTML via `marked.js` |
| Max tokens | 1600 |

### 3.1 Supported models

Models are grouped into families displayed as an accordion picker.

| Family | Models |
|---|---|
| GPT-4o | `gpt-4o` (default), `gpt-4o-mini` |
| GPT-4 Turbo | `gpt-4-turbo` |
| GPT-3.5 | `gpt-3.5-turbo` |
| o-series (Reasoning) | `o1-mini`, `o3-mini` |

---

## 4. UI Architecture

### 4.1 Layout

Two-column grid layout (responsive — stacks to single column on narrow viewports):

- **Left panel** — all configuration controls
- **Right panel** — streaming review output

Sticky header with logo, tagline, and provider badge.

### 4.2 Design system

| Token | Value |
|---|---|
| Display font | Playfair Display (serif) |
| Body font | DM Sans |
| Monospace font | DM Mono |
| Primary accent | Amber (`#c8882a`) |
| Background | Warm paper (`#f5f0e8`) |
| Theme | Editorial / print — warm, textured, typographic |
| Grain overlay | SVG fractal noise at low opacity |
| Animations | Fade-up panel entrance; streaming cursor blink |

---

## 5. Configuration Controls

### 5.1 API Access

- Password input field for OpenAI API key
- Show/Hide toggle button
- Key used client-side only; never stored or forwarded

### 5.2 Model picker

- Accordion-style family groups; one family open at a time
- Radio buttons for individual model selection within a family
- Default: `gpt-4o`

### 5.3 Product details

| Control | Type | Notes |
|---|---|---|
| Product name | Text input | Free-form, max 120 chars |
| Category | Dropdown select | Electronics, Home & Kitchen, Clothing & Apparel, Beauty & Personal Care, Sports & Outdoors, Books, Toys & Games, Food & Beverage, Software / Apps, Other |
| Overall star rating | Interactive star row | 1–5 stars; click to set; amber fill animation |

### 5.4 Aspect sentiment (multi-layer sentiment)

The core differentiating feature. Allows the user to control the sentiment expressed about each product dimension independently.

**Aspect catalogue (10 aspects):**

| Key | Label |
|---|---|
| `price` | Price / Value |
| `features` | Features |
| `usability` | Usability |
| `build` | Build Quality |
| `performance` | Performance |
| `design` | Design |
| `support` | Customer Support |
| `durability` | Durability |
| `compatibility` | Compatibility |
| `packaging` | Packaging |

**Default active aspects:** price, features, usability, build, performance

**Interaction model:**

1. Tag pills — click to toggle an aspect on or off; active aspects render a slider card below
2. Each active aspect displays:
   - Aspect name label
   - Sentiment badge (color-coded, text label: Very Negative / Negative / Neutral / Positive / Very Positive)
   - Custom range track: colored fill bar that updates live (red → amber → green as value increases 1→5)
   - Invisible range input overlaid on track for dragging
   - Tick marks: `−−  −  ·  +  ++`
3. All active aspect sentiments are injected into the prompt, instructing the model to calibrate tone differently per section rather than applying a uniform overall tone

**Sentiment color scale:**

| Value | Label | Fill color | Badge style |
|---|---|---|---|
| 1 | Very Negative | `#e09090` (red) | Red tint |
| 2 | Negative | `#e0b878` (amber) | Amber tint |
| 3 | Neutral | `#c8c0b0` (gray) | Neutral |
| 4 | Positive | `#8ec882` (light green) | Green tint |
| 5 | Very Positive | `#4ab870` (green) | Strong green tint |

### 5.5 Writing style controls

| Control | Type | Options |
|---|---|---|
| Tone | Segment control (5-way) | Balanced, Positive, Critical, Expert, Casual |
| Writing style | Segment control (4-way) | Narrative, Listicle, Technical, Story |
| Formality | Labeled range slider (1–5) | Informal → Formal; live label readout |
| Detail level | Labeled range slider (1–5) | Surface → In-depth; live label readout |

All segment controls use a pill-row design: radio inputs hidden, labels styled as connected buttons; selected option fills with ink/dark background.

### 5.6 Length controls

| Control | Type | Notes |
|---|---|---|
| Length preset | Segment control (4-way) | Short (~120w), Medium (~250w, default), Long (~450w), Custom |
| Custom word count | Range slider (60–800, step 20) | Revealed only when "Custom" is selected; live `~N` readout |

### 5.7 Extra context

Free-form textarea for supplementary information (usage period, comparisons, standout moments, specific use case).

---

## 6. Output Panel

### 6.1 Streaming display

- Response streams token-by-token via the OpenAI SSE API
- Markdown is parsed with `marked.js` and rendered as formatted HTML on each chunk
- Animated amber blinking cursor appended during streaming
- Auto-scrolls to bottom during stream
- Final render removes cursor; full markdown output shown

### 6.2 Markdown rendering styles

Rendered elements include: `h1/h2` (Playfair Display), `h3` (monospace uppercase amber label), paragraphs, `ul/ol` lists, `strong`, `em`, `code`, `blockquote`, `hr`.

### 6.3 Sentiment summary bar

After generation completes, a row of colored chips appears above the review showing which aspects were active and their sentiment, for quick reference. Chips use the same color scale as the slider cards.

### 6.4 Utility controls

- **Model badge** — fades in after generation showing the model used
- **Copy Markdown** button — copies raw markdown to clipboard; shows "Copied!" confirmation

### 6.5 Error states

Errors (missing API key, OpenAI API errors, network failures) are displayed inline in the output panel with a styled error card showing the error message.

---

## 7. Prompt Engineering

The prompt sent to OpenAI is constructed from all active controls:

```
You are an experienced product reviewer writing for a reputable consumer publication.

Product: {name}
Category: {category}
Overall Star Rating: {1-5}/5
Target Length: ~{N} words
Tone: {tone description}
Writing style: {style description}
Formality: {formality description}
Detail level: {depth description}
Additional context: {ctx}  ← omitted if empty

Per-aspect sentiment (calibrate each section accordingly):  ← omitted if no aspects active
- {Aspect Label}: {Sentiment Label} ({value}/5)
...

Structure:
- ## Compelling headline
- Engaging opening paragraph
- ### Pros  (bullet list)
- ### Cons  (bullet list)
- ### Verdict
- Closing sentence naturally mentioning the star rating

Use markdown. Write authentically, as if you have personally used this product.
```

The aspect block explicitly instructs the model not to force a uniform tone across all aspects, enabling mixed sentiment (e.g. positive about features, critical about price).

---

## 8. External Dependencies

All loaded from CDN — no local npm packages required.

| Library | Version | Source | Purpose |
|---|---|---|---|
| marked.js | 9.1.6 | cdnjs.cloudflare.com | Markdown → HTML rendering |
| Google Fonts | — | fonts.googleapis.com | Playfair Display, DM Mono, DM Sans |

---

## 9. Reference Implementation Notes

The original reference (user's `temp/` folder, LLM Switchboard project) informed:

- `.env` file parsing pattern (in-memory, no third-party dotenv)
- `fetch()` call structure for OpenAI chat completions
- Error handling patterns for failed API requests
- Single-page LLM tool architecture

The following Switchboard features were explicitly excluded:

- Anthropic / multi-provider integration
- Model selection dropdown / provider switching UI
- Structured output mode and JSON schema handling

---

## 10. Iteration History

| Iteration | Description |
|---|---|
| v1 | Initial `server.js` — Node.js single-file server with embedded HTML, `.env` loader, OpenAI streaming proxy, model family accordion, star rating, length slider, markdown rendering |
| v2 | Standalone `review-forge.html` — same feature set ported to pure client-side HTML; API key entered directly in browser; no server required |
| v3 | Enhanced `review-forge.html` — added aspect sentiment system (10 aspects, toggle tags, colored range tracks, live badge updates, sentiment summary chips); added segment controls for tone/style/length; added formality and detail-level labeled sliders; custom length option |
