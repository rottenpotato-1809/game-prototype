# Dragon Mania Legends: Bench Battles

A rogue-lite auto-battler prototype inspired by **Dragon Mania Legends** and **Super Auto Pets**.  
Draft, breed, and deploy elemental dragons in a turn-based queue clash against Viking raiders.

---

## Concept

Players build a 3-unit dragon queue from a breeding bench. Dragging two dragons together fuses them into hybrid species with blended elemental classes. Only the frontline dragon fights — backline units project passive buffs forward. Active **Dragon Fury** ultimates let players intervene at critical moments. Between runs, discovered breeds fill a persistent **Dragopedia** codex, and scores convert to EXP that unlocks new elements and items.

**Target audience:** Casual players on desktop and mobile.

---

## Tech Stack

| Layer | Choice |
|-------|--------|
| Language | Vanilla JavaScript (ES modules) |
| Rendering | HTML5 Canvas (2D context) |
| Bundler | Vite |
| Testing | Vitest |
| Persistence | localStorage |

---

## Project Structure

```
├── gdd.md                 # Game Design Document (rules, mechanics, UI)
├── tdd.md                 # Technical Design Document (architecture, config, formulas)
├── gameplay_mockup.html   # Interactive wireframe (open in browser)
│
└── src/                   # Game source code
    ├── main.js            # Entry point
    ├── config.js          # All tunable constants (single source of truth)
    ├── game.js            # State machine
    ├── states/            # One file per screen (MainMenu, Prep, Combat, etc.)
    ├── systems/           # Pure logic (battle, breeding, shop, progression)
    ├── data/              # Static lookup tables (dragons, items, element wheel)
    ├── rendering/         # Canvas draw helpers and reusable UI widgets
    ├── input.js           # Unified mouse + touch input
    ├── storage.js         # localStorage wrapper
    └── logger.js          # Tagged console logging
```

---

## Design Documents

| Document | Purpose |
|----------|---------|
| [gdd.md](gdd.md) | Game rules, mechanics, UI layouts, controls, win/lose conditions |
| [tdd.md](tdd.md) | Architecture, config spec, formulas, testing, git workflow |
| [gameplay_mockup.html](gameplay_mockup.html) | Interactive wireframe — open in any browser to preview all screens |

---

## Getting Started

```bash
# Install dependencies
npm install

# Start dev server
npm run dev

# Run tests
npx vitest run

# Build for production
npm run build
```

---

## Development Workflow

1. Make changes
2. `npm run build` — confirm zero errors
3. `npx vitest run` — confirm all tests pass
4. `git commit` using conventional messages (`feat:`, `fix:`, `chore:`, `test:`, `docs:`)

---

## Vertical Slice Scope

The initial prototype targets:

- **3 elements:** Fire, Wind, Earth
- **7 dragons:** 3 base + 3 dual hybrids + 1 triple hybrid
- **2 items:** Sword, Shield
- **3 combat waves** with a mini-boss
- **8 screens:** Menu, Options, Prep, Combat, Codex, Leaderboard, Win, Lose
- **Persistent storage** for codex, leaderboard, and player profile
