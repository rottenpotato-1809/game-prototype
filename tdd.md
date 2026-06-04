# TDD: DML Breed-on-the-Bench Auto-Battler

Technical implementation guidance for the prototype defined in `gdd.md`.  
Audience: developers implementing the game. All design rationale lives in the GDD; this document covers *how* to build it.

---

## 1. Tech Stack

| Layer | Choice |
|-------|--------|
| Language | Vanilla JavaScript (ES modules) |
| Rendering | Single HTML5 `<canvas>`, 2D context |
| Bundler | Vite |
| Testing | Vitest |
| Persistence | `localStorage` |
| Version control | Git — conventional commits |

**Canvas-only rule:** The HTML file is a dumb shell containing one `<canvas>` tag and one `<script type="module">` entry point. All UI (buttons, text fields, grids, bars, tooltips) is drawn on the canvas and hit-tested in code. No DOM elements for gameplay.

---

## 2. Project Structure

```
src/
├── main.js              # Entry point — creates canvas, boots Game
├── config.js            # THE single config file (see §3)
├── game.js              # State machine — owns the current screen
│
├── states/              # One file per game screen
│   ├── MainMenuState.js
│   ├── OptionsState.js
│   ├── PrepState.js
│   ├── CombatState.js
│   ├── CodexState.js
│   ├── LeaderboardState.js
│   ├── WinState.js
│   └── LoseState.js
│
├── systems/             # Pure logic — no draw calls
│   ├── battleEngine.js  # Queue combat tick loop, damage calc
│   ├── breedingEngine.js# Merge rules, tier promotion, item detach
│   ├── shopEngine.js    # Roll pool, buy/reroll, gold math
│   └── progressionEngine.js # EXP, level-ups, codex unlock checks
│
├── data/
│   ├── dragons.js       # Dragon breed registry (names, elements, base stats)
│   ├── items.js         # Equipment definitions (name, stat effect)
│   └── elementWheel.js  # Element counter/weakness lookup table
│
├── rendering/
│   ├── renderer.js      # Core canvas draw helpers (rect, text, bar, circle, image)
│   └── uiComponents.js  # Reusable canvas widgets (Button, Slot, Card, Bar, TextInput)
│
├── input.js             # Unified mouse + touch → canvas-relative coords, drag state
├── storage.js           # localStorage wrapper — codex, leaderboard, profile, level
└── logger.js            # Prefixed console.log helper ([Shop], [Combat], [Breeding], etc.)

tests/
├── battleEngine.test.js
├── breedingEngine.test.js
├── shopEngine.test.js
├── progressionEngine.test.js
└── elementWheel.test.js
```

### Separation rules

* **`systems/`** files are pure functions/classes. They import `config.js` and `data/` only. They never import canvas, rendering, or input code. This makes them fully unit-testable.
* **`states/`** files own screen-specific layout, drawing, and input wiring. They call into `systems/` for logic and `rendering/` for drawing.
* **`data/`** files export static lookup tables. They import `config.js` for tunable stat values.

---

## 3. Centralized Config (`src/config.js`)

Every tunable number lives here. Other files import from config — never hardcode a value.

The config file must follow this format — every key has a comment with plain-language explanation and recommended range:

```js
// ============================================================
// CANVAS & DISPLAY
// ============================================================
CANVAS_WIDTH: 1024,        // Render width in pixels
CANVAS_HEIGHT: 576,        // Render height in pixels (16:9)
BG_COLOR: '#0b132b',       // Main background color
BORDER_COLOR: '#5ed3f3',   // UI outline/accent color
ACCENT_COLOR: '#ff007f',   // Enemy/danger highlight color
FURY_READY_COLOR: '#ffeb3b', // Fury meter full glow

// ============================================================
// GAME RULES
// ============================================================
STARTING_HEARTS: 3,        // Lives per run (1–5)
TOTAL_WAVES: 10,           // Waves to win a run (3 for vertical slice)
BENCH_SLOTS: 5,            // Breeding bench capacity (3–8)
QUEUE_SIZE: 3,             // Battle queue slots: Front, Middle, Rear
INVENTORY_SLOTS: 3,        // Detached-item overflow slots

// ============================================================
// ECONOMY
// ============================================================
DRAGON_COST: 3,            // Gold to buy a base dragon (1–10)
ITEM_COST: 5,              // Gold to buy equipment (1–15)
REROLL_COST: 1,            // Gold to refresh shop (1–3)
SELL_REFUND_RATE: 0.5,     // Fraction of buy price returned on auto-sell (0–1)
WAVE_GOLD_REWARD: 5,       // Base gold earned per wave win (1–15)
INTEREST_PER_10G: 1,       // Bonus gold per 10g saved (0–3)
MAX_INTEREST: 3,           // Interest cap (0–5)

// ============================================================
// SHOP
// ============================================================
SHOP_DRAGON_SLOTS: 3,      // Number of dragon cards in the shop
SHOP_ITEM_SLOTS: 1,        // Number of item cards (rightmost)

// ============================================================
// COMBAT TUNING
// ============================================================
TICK_DELAY_MS: 800,        // Milliseconds between combat ticks (200–2000)
FURY_PER_HIT: 10,          // % Fury gained per attack dealt or taken (1–25)
FURY_PER_BACKLINE_TICK: 5, // % Fury gained by mid/rear per frontline hit (1–15)
FURY_THRESHOLD: 100,       // % needed to trigger Fury skill
FURY_PAUSE_MS: 1000,       // Combat pause duration when Fury triggers (500–2000)

// ============================================================
// STAT SCALING
// ============================================================
TIER2_STAT_MULT: 1.3,      // T2 hybrid stat multiplier (1.1–1.5)
TIER3_STAT_MULT: 1.6,      // T3 hybrid stat multiplier (1.3–2.0)
ELEMENT_STRONG_MULT: 2.0,  // Damage multiplier when hitting a weak element
ELEMENT_WEAK_MULT: 0.5,    // Damage multiplier when hitting a strong element

// ============================================================
// CLASS PASSIVES (see GDD §5.A for class descriptions)
// ============================================================
BERSERKER_ATK_BUFF: 0.25,  // +ATK% when frontline HP < 50% (0.1–0.5)
BERSERKER_HP_THRESHOLD: 0.5,
RANGER_SPEED_BUFF: 0.20,   // +speed% to frontline (0.1–0.4)
GUARDIAN_DMG_REDUCTION: 0.20, // Flat damage reduction (0.1–0.4)
GUARDIAN_SHIELD_HITS: 2,    // Hits absorbed by Fury shield (1–5)
SAGE_HEAL_PER_TICK: 0.05,  // % max HP healed per tick (0.01–0.10)
SAGE_FURY_HEAL: 0.30,      // % max HP restored by Fury (0.1–0.5)
WARDEN_THORNS: 0.15,       // % damage reflected (0.05–0.30)
WARDEN_SLOW_AMOUNT: 0.50,  // Speed reduction on enemy (0.2–0.8)
WARDEN_SLOW_TURNS: 2,      // Duration of slow (1–4)
ROGUE_ARMOR_PIERCE: 0.20,  // Ignore % of damage reduction (0.1–0.4)
ROGUE_CRIT_MULT: 2.5,      // Critical strike multiplier (1.5–4.0)

// ============================================================
// PROGRESSION & SCORING
// ============================================================
EXP_PER_WAVE: 100,         // EXP earned per wave survived
EXP_PER_GOLD: 10,          // EXP earned per gold remaining
EXP_PER_DISCOVERY: 200,    // EXP earned per new codex entry
EXP_PER_LEVEL: 1000,       // EXP needed = level * this value
LEADERBOARD_MAX_ENTRIES: 5, // High scores shown

// ============================================================
// FONT & UI SIZING
// ============================================================
FONT_FAMILY: 'monospace',  // Fallback font
FONT_SIZE_TITLE: 28,
FONT_SIZE_HEADING: 16,
FONT_SIZE_BODY: 12,
FONT_SIZE_SMALL: 10,
BUTTON_HEIGHT: 36,
SLOT_SIZE: 80,             // Width/height of bench and queue slots
PORTRAIT_HEIGHT: 60,       // Height of combat status portrait cards
BAR_HEIGHT: 6,             // HP/Fury bar thickness
```

---

## 4. Key Formulas

All formulas reference config keys — never literal numbers in code.

| Formula | Expression |
|---------|------------|
| **Combat damage** | `baseDmg × itemMod × elementMod` where `elementMod` is `ELEMENT_STRONG_MULT`, `ELEMENT_WEAK_MULT`, or `1.0` |
| **Tier stat scaling** | `baseHP × TIER{N}_STAT_MULT`, same for ATK |
| **Wave gold income** | `WAVE_GOLD_REWARD + min(floor(currentGold / 10) × INTEREST_PER_10G, MAX_INTEREST)` |
| **End-of-run EXP** | `(wavesCleared × EXP_PER_WAVE) + (goldLeft × EXP_PER_GOLD) + (newDiscoveries × EXP_PER_DISCOVERY)` |
| **Level-up threshold** | `playerLevel × EXP_PER_LEVEL` |
| **Item auto-sell value** | `floor(itemCost × SELL_REFUND_RATE)` |

---

## 5. State Machine

`game.js` holds a simple state machine. Each state is an object implementing three methods:

```
enter()    — called once when transitioning into this state
update(dt) — called every frame with delta time
render(ctx)— called every frame with the canvas context
```

State transitions are triggered by calling `game.setState('combat')` etc. The state list mirrors the GDD screens: `mainMenu`, `options`, `prep`, `combat`, `codex`, `leaderboard`, `win`, `lose`.

---

## 6. Input System

`input.js` exposes a shared singleton tracking:

* **`pointer`** — `{ x, y, isDown, isDragging, dragStart, dragPayload }`
* **`keyboard`** — active key buffer for text input fields

It converts both `mouse` and `touch` events to canvas-relative coordinates. States register clickable regions as `{ id, x, y, w, h, onClick }` hitboxes each frame. The input system tests pointer-up against registered hitboxes and fires the matching callback.

For drag-and-drop (breeding, equipping, deploying): the state sets `dragPayload` on pointer-down over a draggable entity and checks drop-target overlap on pointer-up.

---

## 7. Logging

`logger.js` exports tagged log functions. Use them at every key interaction:

| Tag | When |
|-----|------|
| `[Game]` | State transitions |
| `[Shop]` | Buy, reroll, auto-sell |
| `[Breeding]` | Merge attempts, successes, tier-ups |
| `[Combat]` | Each tick: attacker, target, damage, HP remaining |
| `[Fury]` | Charge updates, activation, skill effect |
| `[Progress]` | EXP gain, level-up, codex unlock |
| `[Storage]` | Save/load operations |

Example: `[Breeding] Merged Fire(T1) + Wind(T1) → Smoke(T2). Stats: HP=130, ATK=65.`

---

## 8. Testing Requirements

* **What to test:** All `systems/` modules and `data/` lookup tables. Tests must cover:
  - Element wheel lookups (strong/weak/neutral)
  - Damage calculation with all modifier combos
  - Breeding merge rules (valid, invalid, tier promotion, item detach)
  - Shop roll filtering based on unlock level
  - EXP calculation and level-up thresholds
  - Fury charge accumulation and threshold checks
* **When to run:** Every development turn before committing.
* **Command:** `npx vitest run`
* **Coverage target:** 100% of system functions, edge cases for invalid merges and boundary values.

---

## 9. Git & Build Workflow

Every completed work unit follows this sequence:

1. `npm run build` — confirm zero errors/warnings.
2. `npx vitest run` — confirm all tests pass.
3. `git add -A` then `git commit -m "<type>: <description>"` using conventional types: `feat:`, `fix:`, `chore:`, `test:`, `docs:`.
4. Verify `npm run dev` starts cleanly. If sandboxed, note for manual verification.

---

## 10. Vertical Slice Scope

For the initial prototype, implement only the subset defined in GDD §10:

* **3 elements:** Fire, Wind, Earth (3 base dragons, 3 T2 hybrids, 1 T3 hybrid)
* **2 items:** Sword (ATK boost), Shield (DMG reduction)
* **3 combat waves** ending with a mini-boss
* **All 8 screens** functional (menu, options, prep, combat, codex, leaderboard, win, lose)
* **localStorage persistence** for codex, leaderboard, and player profile
