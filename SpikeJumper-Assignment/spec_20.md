# Rainbow Runners — Full Game Design Specification

---

## Overview

**Rainbow Runners** is a side-scrolling endless runner game built as a single HTML file, inspired by Geometry Dash. The world starts completely devoid of color — a cold, grey landscape drained of life by a mysterious shadow. The player controls a magical pony whose rainbow trail restores color to everything she passes. The further you run, the more vibrant the world becomes behind you.

---

## Visual Design

### Core Aesthetic

- The entire world is rendered in greyscale **ahead** of the pony.
- As the pony passes each background element, it transitions to full color in real time.
- The color reveal is **screen-position based**: any background object whose screen X coordinate is to the left of the pony is rendered in color; everything to the right is grey.
- The sky is split the same way — left of the pony is colorized, right remains dark grey.
- The ground is split identically at the pony's screen X position.

### Color Palette per Zone (colored state)

| Zone | Sky (top → bottom) | Ground | Atmosphere |
|---|---|---|---|
| Apple Orchard | `#1a3a08` → `#2a5010` | `#3a5a18` | Warm green dawn, golden sun |
| Spooky Forest | `#0a0820` → `#180830` | `#1a1018` | Deep purple night, pale moon |
| Castle Fields | `#082040` → `#103060` | `#204030` | Rich blue twilight |
| Crystal Caves | `#080520` → `#100a30` | `#141028` | Deep indigo, ceiling stalactites |

### Parallax Layers

Background objects are rendered at varying depth multipliers (0.05–0.85) to create parallax scrolling:

- **Deep background** (depth 0.05–0.15): distant hills, far crystals, castle
- **Mid background** (depth 0.2–0.45): trees, dead trees, medium crystals
- **Near background** (depth 0.55–0.85): foreground crystals, bats

### Canvas

- Canvas size: **1100 × 620 px**
- Ground line: Y = 510
- Pony fixed screen X: 160

---

## Player Character — The Pony

### Pony Selection

Six ponies are selectable from the title screen, each with a unique body color, mane color, and tail color:

| Name | Body | Mane |
|---|---|---|
| Twilight | Purple `#7b3f9e` | Deep purple |
| Pinkie | Pink `#f06fa0` | Hot pink |
| Fluttershy | Pale yellow `#f8f0b0` | Soft pink |
| Applejack | Orange `#e8a832` | Red-orange |
| Rainbow | Sky blue `#60b0ff` | Red/orange |
| Rarity | White `#f0ecff` | Purple |

### Pony Anatomy

- Rounded rectangular body with rounded legs
- Long flowing mane rendered as two bezier-curve strands starting **above the eye**, sweeping back over the head and draping down to the ground
- Long flowing tail rendered as two bezier-curve strands from the back of the body
- Expressive eye with eyelashes
- Snout with nostrils
- Pointed ear with inner color

### Animation States

- **Running**: legs animate with sinusoidal swing (`Math.sin(time * 11) * 6`)
- **Jumping**: standard run animation continues in air
- **Rainbow Mode**: mane and tail become 6 separate rainbow-colored flowing strands, each animated independently
- **Dying**: legs freeze, eyes become X's with a tear drop, head droops, ear wilts, all colors desaturate to grey

### Physics

- Gravity: `0.55` per frame
- Jump force: `-13.5`
- Max jumps: **3** (triple jump)
- In Rainbow Mode: unlimited jumps, reduced jump force (`0.72×`)
- Natural jump arc: accelerates downward then decelerates at peak (standard gravity curve)
- Platform landing resets jump count

---

## Rainbow Trail

### Design

- A horizontal **rainbow flag ribbon** that streams behind the pony as she runs
- 6 bands, one per rainbow color: red, orange, yellow, green, blue, violet
- Each band is **10px tall**, stacked vertically — total ribbon height ~60px
- Bands are perfectly horizontal (fixed Y positions relative to the pony's feet)
- Tiny 2px undulation per band for organic feel
- Ribbon fades from fully transparent at the far left tail end to fully opaque near the pony

### Technical Implementation

- Trail stores **world-x positions** (`scrollX + PONY_X`) so they spread horizontally as the pony runs
- On draw, world-x is converted to screen-x (`wx - scrollX`)
- Up to 120 history points stored
- Gradient: transparent → `aa` at 6% → `ff` at 18% → `ff` at 100%
- Sparkle glow effect at the pony attachment point (radial gradient, pulses with `Math.sin(time * 9)`)

---

## Zones & Procedural Theming

### Zone Structure

The game cycles through **4 zones** continuously. Each full cycle is **5,300 distance units**. After completing Crystal Caves, the cycle resets to Apple Orchard.

| # | Zone | Distance Start | Background Elements |
|---|---|---|---|
| 0 | 🍎 Apple Orchard | 0 | Apple trees with red apples, golden sun |
| 1 | 🌑 Spooky Forest | 900 | Dead bare trees with eerie glow, swooping bats, pale moon |
| 2 | 🏰 Castle Fields | 2,200 | Rolling green hills, grand castle with towers/flags/windows/gate |
| 3 | 💎 Crystal Caves | 3,800 | Dense crystal formations in 3 layers, indigo ceiling stalactites |

### Zone Transitions

- Zone label updates instantly when distance threshold is crossed
- Sky color lerps smoothly between zones using `subT` progress factor
- Ground color lerps between zones
- Background elements are distributed by world-x with parallax depth to ensure visual alignment with zone boundaries

### Crystal Caves — High Density

Three layers of crystals:
- **50 mid-depth crystals** (depth 0.1–0.65), spread every ~90 world units
- **18 large foreground crystals** (depth 0.6–0.9), every ~240 world units
- **30 tiny background crystals** (depth 0.05–0.13), every ~130 world units

---

## Hazards

### Thorny Bushes

- The only obstacle type — procedurally spawned
- Rendered as a cluster of overlapping green ellipses forming a natural bush silhouette
- **6 overlapping circles** at varying positions and sizes
- **Sharp thorns** pointing outward in 9 directions around the bush perimeter
- **Red glowing outline** (`#dd1111`, shadow blur 8) around each foliage circle for immediate danger visibility
- Dark inner shadow for depth
- Drop shadow base ellipse
- Grey until the pony passes them; then full green
- Collision box slightly inset from visual bounds for fair play

### Spawn Behavior

- Spawn timer: 55–130 frames between obstacles
- Height: 38–74px, width: 35–65px
- Positioned at ground level

---

## Platforms

- Floating wooden platforms with mossy green tops and grass tufts
- Width: 80–150px, height: 14px
- Spawn height: 110–230px above ground
- Spawn timer: 95–210 frames
- Land on them to reset jump count
- Grey until pony passes; colored after

---

## Power-Ups

### 🌈 Rainbow Circle

- Visual: spinning ring of 6 rainbow arc segments, white glowing center with a gold star
- Effect when collected:
  - Activates **Rainbow Mode** for ~300 frames (~5 seconds)
  - Pony's mane and tail transform to flowing rainbow strands
  - Movement speed increases to **1.65×**
  - Jump count limit removed (infinite jumps / flight)
  - HUD indicator "⚡ RAINBOW MODE" appears
- Collection particles: 24 rainbow burst particles

### 🛡 Shield

- Visual: hexagonal shield shape in cyan-blue with a sparkle symbol, glowing
- Effect when collected:
  - **7 seconds** of invincibility (shieldTimer = 420 frames at 60fps)
  - Pulsing cyan circle drawn around the pony
  - HUD indicator "🛡 SHIELD" appears
  - Obstacles cannot kill the pony during this period

### Spawn Behavior

- Spawn timer: 180–380 frames between power-ups
- 50/50 chance of rainbow vs. shield
- Spawn height: 70–180px above ground
- Spin continuously on screen before collection

---

## Pattern Library / Obstacle Chunks

The spawner creates varied challenge patterns through random combinations of:

1. **Single low bush** — easiest, one ground-level obstacle
2. **Single tall bush** — requires a higher jump
3. **Wide bush** — wider collision box, needs precise timing
4. **Bush + elevated platform** — jump over bush, optionally land on platform above
5. **Double bush gap** — two bushes with a gap requiring landing between them (implicit from consecutive spawns)
6. **Low bush + high platform** — bush at ground, platform floats above mid-height
7. **Platform chain** — several consecutive platforms at varying heights
8. **Ground-level + mid-air platform** — hop over bush onto elevated platform
9. **Power-up + obstacle** — power-up placed just before or above a hazard
10. **Power-up on elevated platform** — requires jumping to platform to reach reward

---

## Difficulty Progression

- Starting speed: **6 units/frame**
- Speed scaling: `gameSpeed = 6 + distance / 2200`
- At distance 2200: speed ≈ 7
- At distance 5000: speed ≈ 8.27
- Obstacle spawn interval decreases as speed increases (fixed frame count, so objects appear more frequently relative to distance at higher speeds)
- Zone cycling ensures the challenge never fully plateaus — each new cycle is faster than the last

---

## Juice Effects

### Screen Shake
- Triggers on death: `shake = 28`
- Decays at `1.6 per frame`
- Random offset applied to canvas translate: `(random - 0.5) * shake * 1.5`
- Creates impact and weight on collision

### Particles
- **Jump particles**: 8 rainbow particles burst downward from pony feet
- **Death explosion**: 22 rainbow particles burst radially outward
- **Rainbow powerup collect**: 24 particles burst in all directions
- All particles: gravity-affected, fade out over life, radius shrinks proportionally

### Squash & Stretch
- Jump arc is natural parabolic curve (gravity acceleration/deceleration)
- Trail ribbon has gentle per-band wave animation for organic flow

### Death Sequence
- Pony adopts sad pose (X eyes, tear, drooping ear, frozen legs)
- Screen fades to black overlay (opacity animates in over ~65 frames)
- Game Over screen appears after 130 frames

---

## Audio

All audio is procedurally synthesized via the Web Audio API (no external files):

| Event | Sound |
|---|---|
| Jump | Oscillator sweeping 280Hz → 560Hz over 0.22s |
| Rainbow power-up collect | 6-note ascending arpeggio (300→1400Hz, sine) |
| Shield collect | 4-note shimmer sequence (500/750/600/900Hz) |
| Death | Sawtooth oscillator sweeping 350Hz → 70Hz over 0.7s |

---

## UI

### Title / Character Select Screen

Two-column layout:

**Left column — Story & Info panel:**
- "✨ A World Without Color" — lore explaining the grey world and the pony's mission
- "🗺 The Journey" — zone guide with icon, name, and description for all 4 zones
- "🎮 How to Play" — controls reference (Space/click to jump, up to 3 times; rainbow circle = speed + flight; shield = 7s invincibility)

**Right column — Pony Chooser:**
- 3×2 grid of pony cards, each with a mini rendered portrait
- Selected card highlighted in gold
- "Let's Run! 🌈" start button with animated rainbow gradient

### HUD (in-game)

- **Score** — top right, Fredoka One font, shows current distance score
- **Zone label** — bottom center, all-caps, updates on zone change
- **Power-up indicators** — top left: "⚡ RAINBOW MODE" and "🛡 SHIELD", appear/fade with active state

### Game Over Screen

- Full screen dark overlay
- "Game Over" in large Fredoka One
- Final score display
- "Try Again" button

---

## High Score Persistence

- Not yet implemented — to be added
- Planned: `localStorage` to persist best score across sessions, displayed on title screen and game over screen

---

## Technical Notes

- **Single HTML file** — all JS, CSS, and canvas rendering self-contained
- **No external dependencies** except Google Fonts (Fredoka One, Nunito)
- Canvas API for all rendering — no WebGL
- Fonts: Fredoka One (display/UI), Nunito (body text)
- Target: 60fps on modern browsers
- All background colors in zones are intentionally dark so the greyscale-to-color transition is visually dramatic
- Zone/background alignment uses `scrollX % CYCLE_SCROLL` with a two-pass check (current cycle + previous cycle) for seamless looping
