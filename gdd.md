# GDD: DML Breed-on-the-Bench Auto-Battler

A rogue-lite auto-battler prototype inspired by *Dragon Mania Legends* (DML) and *Super Auto Pets* (SAP). Players draft and breed elemental hybrids on their bench to form a 3-unit combat queue. Only the front unit clashes in turn-based combat, while middle and rear units provide class-based passive buffs. Discovered hybrids log in a persistent Codex, converting final scores to EXP to unlock elements and items for future runs.

---

## 1. Pitch
Assemble and breed the ultimate elemental dragon queue! Drag and combine dragons on your bench to synthesize dual and triple-element hybrids. Deploy your squad in a single-file line (Front, Middle, Rear). Only the front dragon engages in physical combat, while the back units project unique class-based elemental passives and cast powerful support Fury skills. Grow your Codex across runs to level up and unlock rarer dragon breeds.

---

## 2. Design Pillars
1. **Tactile Breeding Synthesis:** Players merge base elements on the bench to hatch hybrid species, unlocking multi-class combinations.
2. **Single-File Tactical Combat (SAP-Style):** Simple queue-based clash system where positioning dictates which dragon tanks damage and which dragons provide supporting elemental passives.
3. **Element-Driven Classes:** Active roles (Mage, Berserker, Guardian, Healer) and passive traits are strictly determined by a dragon's element composition.
4. **Permanent Codex Progression:** Final run scores translate to player levels, expanding the available item and dragon pool in the shop.

---

## 3. Core Mechanic: Queue Clashing & Element Classes
* **Queue Combat:** Ally and enemy squads line up in a 3-unit queue (Front $\leftarrow$ Middle $\leftarrow$ Rear). Only the Front unit strikes. Middle and Rear units project passive buffs (e.g., health regen, speed boost) forward.
  * **Allied buffs point rightward:** `[Rear] ➔ [Middle] ➔ [Front]`
  * **Enemy buffs point leftward:** `[Front] ⮜ [Middle] ⮜ [Rear]`
* **Element-Driven Classes:** A dragon's elements dictate its class. A hybrid dragon with multiple elements acts as a multi-class unit, projecting multiple passive buffs simultaneously.
* **Dragon Fury Ultimates:** Deployed units charge individual Fury meters during battle. Tapping a glowing portrait activates their active class skill (e.g., splash attacks, protective shields, or heals).

---

## 4. Core Loop
1. **Preparation Phase (Initial Menu):**
   * Receive Gold from the previous wave.
   * Buy units from the 4-slot Shop (3 slots for dragons, 1 slot on the far right for items). Reroll the shop for 1 Gold.
   * Drag dragons together on the bench to breed. Breeding increases stats and blends classes; leveling via gold is non-existent.
   * Equip items onto dragons (limit: 1 per dragon). Items persist through merges.
   * Deploy up to 3 dragons into the Queue (Front, Middle, Rear slots).
   * Note: Deployed dragon status portraits (HP/Fury) are hidden during this phase since all dragons heal fully between waves and Fury meters start empty.
2. **Auto-Battle Phase:**
   * Shop and bench panels hide; combat status portraits appear.
   * The Front ally and Front enemy attack each other simultaneously.
   * Backline dragons project passive buffs to the Front ally.
   * When a Fury meter is full, the player taps the glowing portrait to trigger its active class skill.
   * When a Front unit dies, the Middle unit steps forward to fight, and the Rear unit slides to Middle.
3. **Resolution Phase:**
   * **Victory:** Earn Gold and Score. Go back to Prep Phase.
   * **Defeat:** Lose 1 Heart. At 0 Hearts, the run ends, and score converts to Player EXP to unlock new elements.

---

## 5. Rules

### A. Elements, Classes, and Breeding
* **Elements:** Fire, Wind, Earth, Water, Plant, Metal.
* **No Gold Leveling:** Dragons have no numeric level. Their power is determined by their Tier:
  * **Tier 1 (Base):** Single-element dragon. Basic stats.
  * **Tier 2 (Hybrid):** Merging two Tier 1s creates a Tier 2 hybrid with both elements (e.g., Fire + Wind = Smoke Dragon). Stat bonus: +30% HP and Attack.
  * **Tier 3 (Advanced Hybrid):** Merging a Tier 2 with a Tier 1 of a third element creates a Tier 3 dragon with three elements (e.g., Smoke + Earth = Agave Dragon). Stat bonus: +60% HP and Attack.
  * **Limits:** Dragons are capped at 3 elements.
* **Class Matrix:**
  * **Fire $\rightarrow$ Berserker:** Passive: Boosts Front unit's attack power by 25% when Front health falls below 50%. Fury: Strikes all 3 enemy queue units with a splash attack.
  * **Wind $\rightarrow$ Ranger:** Passive: Increases Front unit's attack speed (turn rate) by 20%. Fury: Stuns the enemy Front unit for 1 turn.
  * **Earth $\rightarrow$ Guardian:** Passive: Grants 20% flat damage reduction to the Front unit. Fury: Deploys a stone shield that absorbs the next 2 physical hits.
  * **Water $\rightarrow$ Sage:** Passive: Heals the Front unit for 5% max HP at the start of every combat turn. Fury: Instantly restores 30% HP to the Front unit and cures debuffs.
  * **Plant $\rightarrow$ Warden:** Passive: Grants the Front unit a Thorns effect, reflecting 15% of taken damage back to the attacker. Fury: Entangles the enemy front, reducing their attack speed by 50% for 2 turns.
  * **Metal $\rightarrow$ Rogue:** Passive: Grants Front unit's attacks 20% armor pierce. Fury: Executes a critical strike dealing 2.5x damage.
* **Hybrid Synergy:** The dragon's **first element** dictates its active Fury skill. It projects the passive traits of **all** elements it possesses. (E.g., Smoke Dragon [Fire/Wind] acts as a Berserker for Fury, but projects both the Berserker attack buff and the Ranger speed boost).

### B. Item System
* **Equipment Limit:** 1 item per dragon.
* **Merge Rules:** When merging Dragon A into Dragon B:
  * If only one dragon has an item, the hybrid keeps that item.
  * If both dragons have items, the hybrid keeps Dragon A's item. Dragon B's item is detached and placed in the Player Inventory. If the inventory is full, it is automatically sold for 50% gold.

### C. Combat & Queue Mechanics
* **Layout:** A single queue line facing each other:
  * Ally: `[Rear] ➔ [Middle] ➔ [Front]` $\leftarrow$ Clashing $\rightarrow$ Enemy: `[Front] ⮜ [Middle] ⮜ [Rear]`
* **Turn Sequence:** The combat runs on automatic sequential ticks:
  * Tick starts: Passive buffs from backline units are applied to frontline units.
  * Battle tick: Front Ally and Front Enemy attack each other simultaneously.
  * Fury charging: Each attack dealt or taken charges the respective unit's Fury meter by 10%. Backline units charge 5% Fury whenever their frontline ally attacks.
  * Fury Trigger: Clicking a glowing portrait triggers the class-specific active skill immediately, pausing the battle timeline for 1 second.
  * Death transition: If a frontline unit's health hits 0, it is removed. The queue moves up.

### D. Shop & Progression Unlocks
* **Shop Layout:** 4 horizontal slots. Slots 1, 2, and 3 display random dragons. Slot 4 (far right) displays a random equipment item.
* **Costs:** Base Dragon = 3 Gold, Equipment = 5 Gold, Shop Reroll = 1 Gold.
* **Progression Unlocks:** Player Leveling unlocks new basic Tier 1 dragons (new elements) and basic equipment items, adding them directly to the Shop pool. Once a new Tier 1 dragon is unlocked, it can be bred with existing elements, enabling players to discover and log new Tier 2 and Tier 3 hybrids.
* **EXP Unlock Tiers:**
  * **Level 1:** Starts with Fire, Wind, and Earth elements; Iron Sword item.
  * **Level 2:** Unlocks Plant element (Warden class base dragon) & Wooden Shield item.
  * **Level 3:** Unlocks Metal element (Rogue class base dragon) & Spiked Helmet item.

### E. Codex (Dragopedia) Rules
* **Dragon Index:** Logs all discovered hybrid combinations. Unlocked by breeding them at least once.
* **Item Index:** Logs all unlocked equipment items. Unlike elements, an item remains locked in the Codex as a silhouette until the player **purchases and equips it to a dragon at least once** during a run.
* **EXP Conversion:** Score earned at end of run converts 1:1 to Player EXP: `Score = (Waves Cleared * 100) + (Gold Remaining * 10) + (New Codex Discoveries * 200)`.

### F. Leaderboard & Profile Options
* **High Score Tracking:** Local leaderboard tracks player high scores, listing Rank, Player Name, Highest Wave Cleared, and Score.
* **Options Screen:** Allows players to edit their Profile Name (which maps directly to leaderboard entries) and toggle sound/music volume.

---

## 6. Controls
* **Desktop:**
  * **Mouse Drag & Drop:** Move dragons between Bench and Queue slots (Front, Middle, Rear); drag items/gold onto dragons; drag dragons together to breed.
  * **Mouse Click:** Tap buttons (Reroll, Start Wave), buy shop items, click active dragon portraits to trigger Fury skills, and navigate menus.
* **Mobile:**
  * **Touch Drag & Drop:** Same as desktop drag interactions.
  * **Touch Tap:** Tap buttons, purchase shop items, and tap portraits to activate Dragon Fury.

---

## 7. UI Layouts

### A. Main Menu Screen
* **Title Header:** Neon stylized title: "Dragon Mania Legends: Bench Battles".
* **Profile Summary:** Displays customized Player Name (default: "Dragoneer") and EXP progress gauge.
* **Primary Navigation:** Large buttons to "Start Campaign (New Run)", "Dragopedia (Codex)", "Leaderboard", and "Options Screen".

### B. Prep Phase UI (Run Main Menu)
* **Top HUD:** Displays Wave Number, Player HP (Hearts), current Gold, and Score. Contains a button to open the Codex.
* **Battlefield Queue Panel:** A linear layout with 3 interactive slots (Rear $\rightarrow$ Middle $\rightarrow$ Front) facing the enemy slots.
* **Bench Area:** 5 Bench slots for holding and breeding dragons.
* **4-Slot Prep Shop & Inventory:** Displays 3 dragon cards, 1 item card (on the far right), a Reroll button, and 3 slots for detached items.
* **Dragon Portraits:** *Hidden.*

### C. Combat Phase UI
* **Top HUD:** Same as Prep Phase.
* **Battlefield Queue Panel:** Activates automatic combat animations. The backline slots show arrow lines (allies point right, enemies point left) projecting passive buffs.
* **Dragon Portrait Bar:** Deployed dragon status portraits (with current HP bars and circular Fury Meters) appear below the allied battlefield slots. Clicking a portrait triggers Dragon Fury when its meter is full.
* **Bench, Shop, and Inventory Panels:** *Hidden.*

### D. Codex (Dragopedia) Overlay
* **Dragon Grid:** Grid displaying discovered/locked dragon breeds. Locked breeds show as dark silhouettes showing elements and breeding hints.
* **Item Collection Grid:** Grid showing equipment items. Items that have been unlocked but never equipped show as locked silhouettes (e.g. "Equip once to unlock entry").
* **Timeline Map:** Visual progression timeline showing player milestones (Levels 1-5).

### E. Options Screen Layout
* **Profile Configuration:** Text input field to customize Leaderboard Player Name.
* **Settings:** Sliders for Volume and Music Toggle.
* **Leaderboard Display:** Embedded table showing top 5 player high scores.

### F. End Screens (Win / Lose)
* **Victory Screen:** Gold victory banner, scores, remaining hearts, EXP leveling gauges, highlights of newly discovered Codex additions, and a "Menu" button.
* **Defeat Screen:** Red defeat banner, final wave reached, EXP bar updates, and restart prompt.

---

## 8. UX Flow
1. **Main Menu / Options Setup:** Player inputs profile name in Options.
2. **Prep Phase:** Player buys, merges, equips, and positions units in the queue (Front, Middle, Rear).
3. **Combat Phase:** Queue units clash. Player triggers active Fury skills.
4. **Resolution:** All dragons heal to full. Return to Prep with gold.
5. **End Run:** Win/Defeat screen shows up. Profile name is logged in the Leaderboard if it breaks a high score. EXP is added, unlocking base dragons/items for the next run's shop.
