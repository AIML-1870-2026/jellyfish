# Blackjack with AI Agents — Product Spec

## Overview

A fully static, single-file HTML web application that runs a playable Blackjack game where an LLM agent reads the current game state and recommends the optimal action on every turn. No backend, no build step, no data persistence — everything runs in the browser.

---

## Core Requirements

### Delivery Format
- Single `.html` file, deployable by opening locally or hosting as a static page
- No external dependencies beyond Google Fonts and the provider API endpoints
- All game logic, UI, and AI integration contained in one file

### API Key Handling
- User provides their API key by uploading a `.env` file or pasting directly
- `.env` parsing matches `ANTHROPIC_API_KEY`, `OPENAI_API_KEY`, or `API_KEY` variable names, and falls back to scanning lines for any `sk-` prefixed token
- Key is held **in-memory only** — never written to localStorage, cookies, or any persistent storage
- Key is transmitted only to the selected provider's API endpoint, nowhere else
- A privacy badge is displayed confirming in-browser-only handling

### Provider Support
- **Anthropic** — uses `/v1/messages` endpoint with `x-api-key` header and `anthropic-dangerous-direct-browser-access: true`
- **OpenAI** — uses `/v1/chat/completions` endpoint with `Authorization: Bearer` header
- Provider is selected via a toggle before the game starts; switching rebuilds the model family dropdown
- Auto-detects provider from key format (`sk-ant-` → Anthropic, other `sk-` → OpenAI)
- Key format validated live with a status indicator

### Model Selection
- Two-step selection: Model Family → specific Model
- **Anthropic families:** Claude 4 (`claude-opus-4-5`, `claude-sonnet-4-5`), Claude 3.5 (`claude-3-5-sonnet-20241022`, `claude-3-5-haiku-20241022`), Claude 3 (`claude-3-opus-20240229`, `claude-3-haiku-20240307`)
- **OpenAI families:** GPT-4o (`gpt-4o`, `gpt-4o-mini`), GPT-4 (`gpt-4-turbo`, `gpt-4`), GPT-3.5 (`gpt-3.5-turbo`), o-series (`o1`, `o1-mini`, `o3-mini`)
- o-series models omit `temperature` from the request body (not supported)
- Refresh Models button re-populates the model dropdown

---

## Blackjack Rules

- 6-deck shoe, reshuffled automatically when fewer than 15 cards remain
- Dealer stands on soft 17
- Blackjack pays 3:2
- Player actions available: **Hit**, **Stand**, **Double Down**, **Split**, **Insurance**
- Double Down: available on first two cards, doubles the bet, deals one card, then resolves
- Split: available when first two cards have equal value; simplified to playing first hand with a new card
- Insurance: available when dealer shows an Ace; costs half the current bet; pays 2:1 if dealer has blackjack
- Bet amount configurable before each session (min $5, max $500, step $5)
- Starting balance $1,000; reloads to $1,000 automatically if depleted

---

## AI Agent

### Prompt Structure
On every player decision point the agent receives:
- Player hand (rank + suit of each card)
- Player score (with soft-hand annotation if applicable)
- Dealer up card
- List of currently available actions

### Response Format
The agent is instructed to respond **only** with a JSON object — no markdown fences, no preamble:

```json
{
  "action": "<hit|stand|double|split|insurance>",
  "confidence": 0.0,
  "reasoning": "<explanation>",
  "bust_probability": 0.0,
  "ev_note": "<positive|negative|neutral>"
}
```

- Recommended action is validated against the available action list before being surfaced
- Invalid or unavailable actions fall back gracefully (e.g. unavailable `double` → `hit`)
- All API calls and raw responses are logged to the browser console

### Explainability Controls
Three levels of explanation detail, selectable before the game:

| Level | Prompt Instruction |
|---|---|
| **Basic** | One-sentence recommendation only |
| **Statistical** | 2–3 sentences including probability context and EV direction |
| **In-Depth** | 3–5 sentences: full basic strategy reasoning, bust probability, EV comparison of top two actions, card-counting considerations |

### Risk Tolerance Settings
Three risk profiles that modify the agent's system instructions:

| Profile | Behaviour |
|---|---|
| **Conservative** | Prefer standing in borderline cases; avoid doubles unless situation is very strong; minimise variance |
| **Balanced** | Strict basic strategy with no deviation (default) |
| **Aggressive** | Double in more borderline spots; split more liberally; maximise EV over variance reduction |

The active risk profile is shown as a colour-coded badge next to the AI recommendation.

---

## UI — Setup Screen

- Provider toggle (Anthropic / OpenAI)
- API key field (password input with show/hide toggle) + `.env` file upload
- Model Family dropdown → Model dropdown
- Bet Amount number input
- Balance display (carries over between hands within a session)
- Explanation Detail toggle (Basic / Statistical / In-Depth)
- Risk Tolerance toggle (Conservative / Balanced / Aggressive)
- Play button (disabled until key + model are both set)
- Refresh Models button

---

## UI — Game Screen

### Game Table
- Two-column layout: Dealer hand (left) vs Player hand (right)
- Animated card deal with spring physics (`cubic-bezier(0.34, 1.56, 0.64, 1)`)
- Face-down hole card revealed on dealer's turn
- Score badges update live; bust state and blackjack highlighted distinctly
- Current bet displayed beneath player hand

### Action Bar
Five buttons: HIT / STAND / DOUBLE / SPLIT / INSURANCE — each disabled contextually based on game state and balance.

### Status Panel
- "AI recommends: [ACTION]" with active risk badge
- Execute Recommendation button — fires the AI's chosen action automatically; increments the AI agreement counter
- Result banner (win / lose / push) with P&L delta
- Deal New Hand button appears after resolution

### Right Column
1. **AI Analysis panel** — reasoning text from the agent + a colour-coded confidence bar (green ≥80%, amber ≥60%, red <60%)
2. **Strategy Matrix panel** — full basic-strategy hard-total decision table (rows: player total 8–17+, columns: dealer up-card 2–A); current situation highlighted in the accent colour
3. **Hand History panel** — scrollable log of every hand: hand number, player score vs dealer score, result, P&L delta, running balance

### Performance Analytics Panel
Four stat cards:
- **Win Rate** — percentage wins excluding pushes, with W/L/P breakdown
- **P&L** — net gain/loss vs $1,000 starting balance
- **Current Streak** — consecutive wins or losses
- **AI Agreement** — how many times the player followed the AI's recommendation (count and percentage)

Canvas sparkline below the stat cards plots bankroll over time, with a dashed reference line at the $1,000 starting point.

---

## Error Handling

- API errors surface inline in the Analysis panel with the raw error message
- Player can continue playing manually when AI is unavailable
- Invalid JSON responses from the API are caught and handled gracefully
- Markdown code fences in responses are stripped before JSON parsing

---

## Theming

### Current Palette — Green
| Variable | Value | Use |
|---|---|---|
| `--g-dark` | `#1b4332` | Deep forest — primary buttons, table background dark end |
| `--g-mid` | `#2d6a4f` | Forest green — hover states |
| `--g-bright` | `#40916c` | Mid green — borders, button gradients |
| `--g-light` | `#74c69d` | Sage — text3, scrollbars |
| `--g-pale` | `#b7e4c7` | Mint — header text, card border |
| `--g-xpale` | `#d8f3dc` | Very pale mint — page background, panels |
| `--red` | `#bc4749` | Berry red — lose states, bust |
| `--orange` | `#c87941` | Warm amber — push/double states |

Background: diagonal gradient from `#d8f3dc` → `#74c69d`, fixed attachment.

### Fonts
- **Title:** Playfair Display (serif, italic) — `font-family: 'Playfair Display', serif`
- **UI:** Nunito (sans-serif, weights 300–700) — `font-family: 'Nunito', sans-serif`

### Design Tokens
- Border radius: 10–20px (rounded, friendly)
- Borders: 2px solid with low-opacity green
- Cards: frosted-glass panels (`backdrop-filter: blur`) on translucent backgrounds
- Shadows: green-tinted (`rgba(27,67,50,…)`)

### Previous Palette History
The palette went through the following iterations during development:
1. **Dark casino** (original) — dark greens and gold, Cinzel/Inter fonts
2. **Beach/ocean** — sky blue, turquoise, sandy tones, Pacifico/Nunito fonts, wave animations
3. **Neutral ocean** — same palette, beach decorations removed
4. **Green** (current) — full green spectrum, Playfair Display title font

---

## Console Logging

All key interactions are logged with the `[BJ]` prefix:
- App initialisation
- New hand dealt (player cards + score, dealer up-card)
- AI API call (provider, explain level, risk profile, model)
- Raw API response text
- Parsed recommendation (action + confidence %)
- Player action taken
- Hand resolution (result, delta, balance)

---

## File Structure

```
blackjack-ai-agent.html   ← single deliverable file
  ├── <style>             Google Fonts import + all CSS
  ├── <body>
  │   ├── #setup-screen   Provider, key, model, bet, settings
  │   ├── #game-screen
  │   │   ├── .felt-table           Dealer + player hands
  │   │   ├── .action-row           Hit/Stand/Double/Split/Insurance
  │   │   ├── .status-panel         AI rec + result banner
  │   │   ├── Analytics panel       Stats + sparkline
  │   │   └── Right column
  │   │       ├── AI Analysis       Reasoning + confidence
  │   │       ├── Strategy Matrix   Basic strategy table
  │   │       └── Hand History      Per-hand log
  │   └── #aboutModal
  └── <script>            All game logic + AI integration
      ├── Config          Model lists, provider families
      ├── State           Game variables, analytics counters
      ├── Setup           Provider/key/model/settings handlers
      ├── Deck            Build, shuffle, deal, score
      ├── Render          Cards, hands, score badges, buttons
      ├── Strategy Matrix Basic strategy table renderer
      ├── Sparkline       Canvas chart renderer
      ├── Analytics       Stats updater
      ├── Game Flow       newHand(), playerAction(), resolveHand()
      ├── AI Call         getAiRecommendation() — fetch + parse
      ├── History         Hand log renderer
      └── Utils           Modal, toast, sleep
```
