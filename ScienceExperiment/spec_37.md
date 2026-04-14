# Science Lab — Project Specification

> Grade-aligned science experiment generator powered by the OpenAI API.
> Single-file HTML deployment — no build step, no server, no dependencies to install.

---

## Overview

Science Lab is a browser-based tool that lets teachers and students generate complete, grade-appropriate science experiment plans from a list of available supplies. The user selects a grade level, enters (or quick-selects) materials, and the app calls the OpenAI chat completions API to produce a structured, markdown-formatted experiment plan rendered as HTML in the page.

---

## Delivery Format

- **Single `.html` file** — all CSS, JavaScript, and HTML in one file
- No backend, no Node.js, no build toolchain
- Fonts loaded from Google Fonts CDN
- Markdown rendered client-side via `marked.js` from jsDelivr CDN
- API key entered by the user in-browser; stored in memory only, never persisted to disk or localStorage

---

## Reference Implementation

The LLM Switchboard (`llm-switchboard.html`) was used as a visual and structural reference for:

- CSS variable system and color token naming conventions
- Panel / section / header component patterns
- `.panel-header` with `ph-left` / `ph-dot` structure
- `.btn`, `.btn-primary`, `.btn-secondary`, `.btn.loading` button classes
- `.error-box` with `auth` / `rate` / `generic` type variants and icon prefixes (`✗`, `◷`, `!`)
- Response meta footer (words · elapsed · tokens · model)
- Copy button with "copied!" flash feedback
- Spinner animation
- `fadeUp` entrance animation on panels
- API key show/hide toggle with eye button
- Key format validation (checks `sk-` prefix, rejects `sk-ant-`)
- `fetch()` call structure for OpenAI chat completions
- Typed error handling: `401` → `auth`, `429` → `rate`, other → `generic`

**Excluded from this project (Switchboard features not needed):**
- Anthropic provider integration
- Model selection provider tabs / switching
- Structured JSON / schema mode
- Side-by-side comparison view
- Metrics tab
- Prompt library
- Schema validator

---

## API Integration

### Provider
- **OpenAI only** — no Anthropic support

### Endpoint
```
POST https://api.openai.com/v1/chat/completions
```

### Request structure
```json
{
  "model": "<selected model>",
  "temperature": "<slider value>",
  "max_tokens": "<dropdown value>",
  "messages": [
    { "role": "system", "content": "<buildSystemPrompt(gradeLabel)>" },
    { "role": "user",   "content": "Generate a science experiment for <gradeLabel> students using these available supplies:\n\n<supplies>" }
  ]
}
```

**Key fix:** Supplies are passed in the **user message**, not the system prompt. Putting supplies only in the system prompt caused the model to respond as if it was still waiting for supplies.

### Response handling
- Extract: `data.choices[0].message.content`
- Render markdown to HTML using `marked.parse(rawText)`
- Display word count, elapsed seconds, total token count, and model name in the response meta footer
- Extract experiment title from the first `# Heading` in the response for use in history saving

### Models available (dropdown)
- `gpt-4o` (default)
- `gpt-4o-mini`
- `gpt-4-turbo`
- `gpt-3.5-turbo`

---

## System Prompt

The system prompt sets the assistant's role, output format, and grade-level language guidance. Supplies are **not** included in the system prompt — they travel in the user message.

```
You are an enthusiastic, experienced science teacher who designs safe,
curriculum-aligned science experiments.

When given a list of available supplies, generate a complete experiment plan
for <gradeLabel> students using ONLY those supplies (plus common items like
water, tape, scissors, and paper if needed).

Your response must include these sections:

1. Experiment Title — A catchy, memorable name
2. Scientific Concept — Core concept being explored, aligned to grade level
3. Learning Objectives — 2–3 clear, age-appropriate goals
4. Materials List — Exactly what's needed from the available supplies
5. Safety Notes — Age-appropriate precautions
6. Step-by-Step Procedure — Clear numbered steps at the right reading level
7. What to Observe — What students should watch for
8. Discussion Questions — 3–5 post-experiment reflection questions
9. Extension Activity — One optional challenge for advanced students
10. Supply Substitutions — For each key material, suggest 1–2 alternatives

Language guidance by grade:
- K–3: Simple words, short sentences, concrete ideas
- 4–5: Accessible language, introduce basic scientific terms
- 6–8: Proper scientific vocabulary, encourage hypothesis thinking
- 9–12: Full scientific terminology, hypothesis-driven inquiry, data analysis

Format with markdown headings. Be specific, practical, and enthusiastic!
```

---

## User Inputs

### Grade Level (dropdown)
| Value | Label |
|-------|-------|
| K | Kindergarten |
| 1 | 1st Grade |
| 2 | 2nd Grade |
| 3 | 3rd Grade |
| 4 | 4th Grade |
| 5 | 5th Grade |
| 6 | 6th Grade |
| 7 | 7th Grade |
| 8 | 8th Grade |
| 9 | 9th Grade (Freshman) |
| 10 | 10th Grade (Sophomore) |
| 11 | 11th Grade (Junior) |
| 12 | 12th Grade (Senior) |

### Available Supplies (textarea)
- Free-form text input
- User types a comma-separated list of materials
- Also populated by clicking quick-select pills
- Ctrl/Cmd+Enter submits the form

### Model (dropdown)
gpt-4o, gpt-4o-mini, gpt-4-turbo, gpt-3.5-turbo

### Temperature (range slider)
- Range: 0.0 – 2.0, step 0.1
- Default: 0.8
- Label updates live as slider moves

### Max Tokens (dropdown)
512 / 1024 / 2000 (default) / 4096

---

## Features

### 1. Core Generation
Sends the grade level and supplies to OpenAI and renders the markdown response as formatted HTML inside the Experiment Plan panel.

### 2. Quick-Select Common Supplies
A panel above the supplies textarea displays 36 common household/classroom materials as clickable pill buttons. Clicking a pill toggles it: active pills are highlighted and the supply is appended to the textarea (comma-separated). Clicking again removes it. Pills and textarea stay in sync.

**Supply list:**
baking soda, vinegar, food coloring, salt, sugar, flour, water, dish soap, vegetable oil, lemon juice, cornstarch, balloons, rubber bands, tape, scissors, paper clips, plastic cups, paper towels, aluminum foil, zip-lock bags, string, straws, popsicle sticks, toothpicks, cotton balls, magnifying glass, ruler, measuring cups, funnel, timer, candle, matches, ice cubes, sand, soil, rocks

### 3. Supply Substitutions
The system prompt instructs the model to include a "Supply Substitutions" section (section 10) in every response, suggesting 1–2 alternatives for each key material in case the original isn't available.

### 4. Save & Display Previous Experiments
- A "Saved Experiments" panel in the left sidebar displays previously saved experiments
- After generating, user clicks **+ Save** in the response meta footer
- Each entry stores: title (extracted from markdown H1), grade label, raw markdown, rendered HTML, and date
- Up to 20 entries stored in `localStorage` under the key `scilab-history`
- Clicking a history entry reloads that experiment into the response panel
- Individual entries can be deleted with the ✕ button (visible on hover)
- "Clear" button in the panel header clears all history (with confirm dialog)

### 5. Difficulty Rating
After generation, a badge appears in the Experiment Plan panel header showing difficulty based on grade:
- **★☆☆ Easy** — Kindergarten through 3rd grade (green)
- **★★☆ Medium** — 4th through 6th grade (blue)
- **★★★ Advanced** — 7th through 12th grade (red)

### 6. Printable Worksheet
After generation, a 🖨 **Worksheet** button appears in the Experiment Plan panel header. Clicking it opens a new tab with a print-ready page containing:
- Grade label and "Generated by Science Lab" attribution
- The full rendered experiment plan
- An **Observation Log** section with fill-in boxes for:
  - What did you notice?
  - What surprised you?
  - Questions I still have
  - Hypothesis supported / not supported

---

## Layout

Two-column CSS grid layout:

| Left column (295px) | Right column (flex) |
|---------------------|---------------------|
| Provider & Credentials panel | Common Household Supplies panel |
| Saved Experiments panel | Available Supplies panel |
| | Experiment Plan panel |

Collapses to single column below 860px viewport width.

---

## Design System

### Fonts
| Role | Font |
|------|------|
| Headings, labels, buttons, UI | Plus Jakarta Sans (400/500/600/700/800) |
| Body text, inputs, monospace content | DM Mono (400/500, italic) |
| Base font size | 15px |

### Color Palette
```
Background layers:   #eef3f7 / #e4ecf3 / #d6e4ef / #c4d8e8
Border:              rgba(40,100,150,0.14) / rgba(40,100,150,0.32)
Text:                #0d1f2d / #1e3a52 / #4e7a9a
Teal (primary):      #1a8fa0 / #0f6e7e
Blue:                #1a6fa8 / #0f5080 / #4499cc
Green:               #1e7d5a / #165e44
Red (error):         #c0454f
Yellow (warning):    #9a7a1e
```

### Background
Layered radial gradients (blue top-left, teal bottom-right, green top-right) over a fine grid pattern using `repeating-linear-gradient`.

### Panels
- White background, `border-radius: 12px`, subtle blue-tinted box shadow
- Panel headers have a colored 3px left-edge accent bar: teal (default), blue, or green
- Colored dot indicators with matching halo glow rings

### Buttons
- Primary: teal-to-blue gradient, white text, drop shadow, hover lifts with `translateY(-1px)`
- Secondary: white background, border, hover teal
- Loading state: grey background, disabled, cursor wait

### Inputs / Selects / Textarea
- Background: `--bg` (#eef3f7)
- Border radius: 8px
- Focus: teal border + `box-shadow: 0 0 0 3px rgba(26,143,160,0.12)`

### Markdown Rendering
| Element | Style |
|---------|-------|
| H1 | Plus Jakarta Sans 800, 1.45rem, bottom border |
| H2 | Plus Jakarta Sans 700, teal color, 3px teal left border |
| H3 | Plus Jakarta Sans 700, blue color |
| p / li | DM Mono 14px, line-height 1.9, dark blue-grey |
| code | Blue-tinted background, blue text |
| blockquote | 3px teal left border, teal-tinted background, rounded right corners |

---

## Error Handling

| Error type | Trigger | Style |
|------------|---------|-------|
| `auth` | HTTP 401, missing key | Red background, `✗` prefix |
| `rate` | HTTP 429 | Blue background, `◷` prefix |
| `generic` | All other errors, network failures | Neutral background, `!` prefix |

Validation errors (no key, no grade, no supplies) show inline before the API call is made.

---

## Keyboard Shortcut

**Ctrl+Enter** / **Cmd+Enter** in the supplies textarea triggers generation.

---

## Bug Fixes Made During Development

**Supplies not reaching the model:** Original implementation embedded supplies only inside the system prompt, then sent a user message saying "using the supplies I provided" with no actual supplies included. The model responded asking for supplies. Fix: supplies are passed directly in the user message body.

---

## Key Decisions & Constraints

- **No backend required** — API key is entered in the UI and held in JS memory only. It is never sent to any server other than `api.openai.com`.
- **OpenAI only** — no Anthropic or other provider support
- **Free-form (unstructured) responses** — no JSON schema mode; model returns markdown prose
- **Client-side markdown rendering** — `marked.js` via CDN, no server-side HTML conversion
- **Single file** — all HTML, CSS, and JS in one `.html` file for maximum portability
