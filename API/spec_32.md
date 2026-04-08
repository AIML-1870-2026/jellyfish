# LLM Switchboard — Product Specification

## Overview

LLM Switchboard is a single-page web application that serves as a multi-provider AI inference console. It allows users to send prompts to OpenAI and Anthropic models directly from the browser using their own API keys, compare model outputs side by side, track usage metrics, manage a prompt library, and validate JSON schemas against model responses. All API calls are made client-side — no server required.

---

## Requirements

| # | Requirement | Implementation |
|---|-------------|----------------|
| 1 | Support at least two LLM providers | OpenAI and Anthropic both supported via direct browser fetch |
| 2 | Single-page web application | Self-contained `index.html` — no build step, no server |
| 3 | API keys stored client-side only | Keys held in JS variables for the session; never transmitted to any third party |
| 4 | Privacy-first badge visible in header | Green `no data leaves your browser` badge shown at all times |

---

## Providers & Models

### OpenAI
- Endpoint: `https://api.openai.com/v1/chat/completions`
- Key format: `sk-...`
- Models: `gpt-4o`, `gpt-4o-mini`, `gpt-4-turbo`, `gpt-3.5-turbo`

### Anthropic
- Endpoint: `https://api.anthropic.com/v1/messages`
- Key format: `sk-ant-...`
- Models: `claude-opus-4-5`, `claude-sonnet-4-5`, `claude-haiku-4-5-20251001`, `claude-opus-4-20250514`, `claude-sonnet-4-20250514`
- Header: `anthropic-dangerous-direct-browser-access: true` required for browser calls

> **Note:** Anthropic blocks direct browser requests by default (CORS). A CORS notice with a curl fallback command is surfaced when the key is Anthropic-format and a CORS error is detected.

---

## Views (Navigation Tabs)

### 1. Single Model
The primary inference view. A two-column layout: left sidebar (265px) for configuration, right column for prompt input and response output.

**Sidebar — Provider + Credentials**
- Provider tab buttons: OpenAI / Anthropic
- API key input (password type) with show/hide toggle
- Key file upload (drag-to-drop zone accepts `.txt` / `.env` files, extracts the key)
- Key status chip: green `key looks valid` / red `key format unexpected`

**Sidebar — Model & Parameters**
- Model select dropdown (populates based on active provider)
- Mode toggle: `chat` / `structured` (structured mode enables JSON schema input)
- Temperature slider: 0.0–2.0 (default 0.7), value displayed in monospace
- Max tokens select: 256 / 512 / 1024 / 2048 / 4096

**Sidebar — System Prompt**
- Optional textarea for a system prompt (3 rows, resizable)

**Prompt Area**
- Full-width monospace textarea
- Character count displayed live (`~N chars`)
- Example prompt pills (one-click inserts): `summarize`, `translate`, `explain code`, `pros & cons`, `step-by-step`, `json output`, `creative story`, `debug this`
- `+ library` button saves the current prompt to the Prompt Library

**Structured Mode (JSON Schema)**
- Schema editor textarea (blue monospace text)
- Schema template buttons: `person`, `product`, `list`, `analysis` (insert pre-built schemas)
- Separate prompt textarea for the schema view: prompt is sent with the schema auto-injected

**Send Bar**
- Token/char info
- `send →` button (primary green); shows spinner + "sending" while in flight; disabled during request

**Response Panel**
- Scrollable monospace response box (max-height 340px)
- Empty state: centered hexagon placeholder with "awaiting transmission" label
- Error boxes styled by type: CORS (amber), auth (red), rate limit (blue), generic (muted)
- CORS notice with copy-ready curl command shown when Anthropic CORS is detected

---

### 2. Side-by-Side (Compare)
Sends the same prompt to two independently configured models simultaneously and displays responses in a 1fr 1fr grid.

**Configuration**
- Slot A and Slot B each have: provider tabs, model select, max tokens select
- Slot A default: OpenAI; Slot B default: Anthropic
- Shared prompt textarea and `compare →` button
- Both slots use the API keys entered in the Single Model view

**Results**
- Two response panels rendered side by side, labeled A and B with colored badges
- Each panel shows the response text and elapsed time

---

### 3. Metrics
Tracks all requests made in the current session.

**Summary cards (4-up grid)**
- Total requests
- Total tokens (input + output combined)
- Average latency (ms)
- Average words per response

**Request history table**
- Up to 60 most recent requests, newest first
- Columns: timestamp, provider, model, prompt preview (60 chars), tokens in, tokens out, latency (ms), word count
- Empty state shown when no requests have been sent

---

### 4. Prompt Library
A personal prompt store, persisted in `localStorage`.

**Left panel — Saved prompts list**
- Each item shows: name, tag badge (prompt / schema), truncated content preview
- Filter toggle buttons: `all` / `prompts` / `schemas`
- Click an item to load it into the preview panel

**Right panel — Preview & load**
- Shows selected prompt name and full content
- `load into editor` button populates the Single Model prompt area and switches to that view
- `delete` button removes the entry

**Add new prompt form**
- Name input + content textarea + Save button
- Entries tagged automatically as `schema` if content begins with `{`

---

### 5. Schema Validator
Tests whether a model's response conforms to a given JSON schema, without leaving the app.

**Left panel — Input**
- JSON schema textarea
- JSON response textarea (paste model output here)

**Right panel — Validation result**
- `validate →` button runs client-side JSON parse + schema structure check
- Result displayed as pass (green) or fail (red) with error details

---

## Visual Design

### Theme
Dark-tinted sage green. Soft grid background (faint horizontal + vertical lines at 40px intervals) overlaid on a radial green gradient.

### Color Palette

| Token | Value | Usage |
|-------|-------|-------|
| `--bg` | `#f0f4f0` | Page background |
| `--bg2` | `#e8efe8` | Panels |
| `--bg3` | `#dde7dd` | Inputs, section headers |
| `--bg4` | `#d0ddd0` | Active mode buttons, disabled states |
| `--accent` | `#3d7a4a` | Primary buttons, active tabs, links |
| `--accent2` | `#2d5e38` | Button hover |
| `--green` | `#2e7d46` | Key valid badge, privacy badge |
| `--red` | `#b85450` | Auth errors, key invalid |
| `--yellow` | `#8a7a30` | CORS warnings |
| `--blue` | `#3a6e7a` | Rate limit errors, schema text |
| `--text` | `#1e2d1e` | Primary text |
| `--text2` | `#4a6349` | Secondary text |
| `--text3` | `#8aaa88` | Labels, placeholders, metadata |

### Typography

| Font | Weights | Usage |
|------|---------|-------|
| Syne | 400, 500, 600, 700 | All UI labels, headings, navigation |
| IBM Plex Mono | 400, 500, italic | Inputs, responses, badges, buttons, counts |

### Key UI Details
- Panels: 6px border-radius, 1px border, `background: var(--bg2)`
- Panel headers: uppercase monospace label, dot accent, `background: var(--bg3)`
- Provider tab buttons: full-width flex; active = solid `--accent` fill
- Hexagon logo: CSS `clip-path: polygon(...)` shape in accent green
- Spinner: CSS animation on send button during in-flight requests
- Responsive breakpoint (≤ 800px): sidebar stacks above main content; compare and metric grids collapse to single column

---

## Data Handling

### API request patterns

**OpenAI**
```
POST https://api.openai.com/v1/chat/completions
{ model, messages: [{role:'user', content: prompt}], temperature, max_tokens }
```

**Anthropic**
```
POST https://api.anthropic.com/v1/messages
{ model, max_tokens, messages: [{role:'user', content: prompt}], temperature }
Headers: x-api-key, anthropic-version: 2023-06-01, anthropic-dangerous-direct-browser-access: true
```

### Token tracking
- OpenAI: `usage.prompt_tokens` / `usage.completion_tokens`
- Anthropic: `usage.input_tokens` / `usage.output_tokens`
- Both normalized into a shared metrics record per request

### Error classification
| Type | Detection | Style |
|------|-----------|-------|
| CORS | `TypeError` / fetch failure on Anthropic key | Amber, curl fallback shown |
| Auth | HTTP 401 | Red |
| Rate limit | HTTP 429 | Blue |
| Generic | All other errors | Muted green |

### Prompt Library persistence
- Stored in `localStorage` as a JSON array
- Maximum display: 60 metrics history entries per session (not persisted across reload)

---

## Dependencies

| Library | Version | How loaded |
|---------|---------|------------|
| IBM Plex Mono | latest | Google Fonts `<link>` |
| Syne | latest | Google Fonts `<link>` |

No build step. No framework. No server. No external JS libraries. Pure HTML + CSS + vanilla JS.

---

## Delivery

- **Format:** Single `.html` file
- **Filename:** `index.html`
- **Runtime:** Any modern browser (Chrome, Firefox, Safari, Edge)
- **Network:** Requires internet connection to reach OpenAI / Anthropic APIs and load fonts from Google Fonts CDN
