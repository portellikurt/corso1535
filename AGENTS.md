# AGENTS.md — CORSO 1535 Architectural Baseline & EGM Rules

## 1. Project Identity & Vision
**Corso 1535** is a pauseable real-time naval strategy, maritime trade, and privateering simulation set in the **Central Mediterranean (`1535–1565`)**. It combines wind/oar naval combat and isometric deck boarding (*Corsairs Gold*) with localized supply/demand markets, *carati* shareholder syndicates, and civic urban development (*Patrician III*).

* **Master Architecture Spec:** [docs/designs/corso_1535_master_game_definition.md](docs/designs/corso_1535_master_game_definition.md)
* **Subsystem Specs Directory:** `docs/designs/`

---

## 2. Core Simulation Invariants (Non-Negotiable)
Any agent drafting specs or implementing code **MUST** obey these invariants:

1. **Deterministic Fixed-Timestep Simulation:**
   * The Strategic Ocean Layer runs at a fixed **$20\text{ Hz}$** ($\Delta t = 0.05\text{ s}$).
   * Player time-dilation speeds (`0x [Pause]`, `0.25x`, `1x`, `2x`, `4x`) scale the number of ticks executed per frame—**never** alter $\Delta t$.
   * Tactical Boarding timers (such as the `T_arrive` reinforcement queue) run on deterministic tactical ticks scaled identically to game speed, never wall-clock real time.
2. **Strict Separation of Simulation & Presentation:**
   * All core gameplay state (wind vectors, ballistics, boarding graphs, market prices, AI fleet decisions) must reside in pure, serializable data structures decoupled from rendering/UI code so save/load serialization and headless simulation tests remain deterministic.
3. **Data-Driven Configuration:**
   * Hull classes, weapon tiers, ammunition multipliers, commodity base prices, and historical timeline events must be loaded from declarative schemas (JSON/YAML) rather than hardcoded magic numbers.
4. **No AI Fog-of-War Cheating:**
   * AI merchants, state patrols, and rival corsairs operate strictly on visible radar/watchtower intel, port tavern rumors, and known market price histories.

---

## 3. Specification Map (`docs/designs/`)
Before modifying or implementing any subsystem, consult its authoritative spec:

| File | Domain & Responsibility |
| :--- | :--- |
| [docs/designs/corso_1535_master_game_definition.md](docs/designs/corso_1535_master_game_definition.md) | High-concept pillars, 1535 faction matrix, and overall roadmap. |
| [docs/designs/spec_01_world_wind_and_navigation.md](docs/designs/spec_01_world_wind_and_navigation.md) | Map coordinate bounds, terrain draft rules, wind vector field ($\vec{W}$), and hybrid sail/oar propulsion. |
| [docs/designs/spec_02_naval_combat_and_ballistics.md](docs/designs/spec_02_naval_combat_and_ballistics.md) | Bow-chaser vs. broadside arcs, era weapon tiers, ammo damage matrix, ramming, and auto-reaction fire. |
| [docs/designs/spec_03_tactical_boarding_and_siege.md](docs/designs/spec_03_tactical_boarding_and_siege.md) | Deck/village node graphs, reinforcement queue, unit roles, flanking DPS multipliers, and surrender checks. |
| [docs/designs/spec_04_economy_trade_and_carati.md](docs/designs/spec_04_economy_trade_and_carati.md) | 12-commodity pricing formula, *Fondaco* warehouses, automated trade loops, *carati* shares, and Prize Courts. |
| [docs/designs/spec_05_settlements_and_politics.md](docs/designs/spec_05_settlements_and_politics.md) | Two-tier settlements (villages vs. fortified cities), civic ranks, Valletta/Birgu upgrades, and blockades. |
| [docs/designs/spec_06_campaign_progression_and_ai_director.md](docs/designs/spec_06_campaign_progression_and_ai_director.md) | `1535–1565` historical event triggers, 3-vector tech unlocks, and strategic AI Fleet Director. |
| [docs/designs/spec_07_data_schemas_and_balance_baseline.md](docs/designs/spec_07_data_schemas_and_balance_baseline.md) | Canonical data schemas, command/event contracts, save/load serialization, edge-case state machines, and baseline balance tables. |
| [docs/designs/spec_08_ui_hud_and_ux_specification.md](docs/designs/spec_08_ui_hud_and_ux_specification.md) | Strategic & tactical HUD layout, trade/economy/carati screens, boarding/siege controls, and visual/audio feedback cues. |

### Implementation Rules for Schemas, Balance & UI
- **Canonical Types & Constants:** Follow [docs/designs/spec_07_data_schemas_and_balance_baseline.md](docs/designs/spec_07_data_schemas_and_balance_baseline.md) for entity structures, units, state transitions, and balance constants; avoid inventing conflicting structs or hardcoding one-off tuning values in gameplay code.
- **Edge-Case First:** Explicitly cover boarding handoffs/reinforcements/prize splits, stalemates, trade-route disruptions, and diplomatic/carati/prize-court edge cases specified in [spec_07_data_schemas_and_balance_baseline.md](docs/designs/spec_07_data_schemas_and_balance_baseline.md).
- **Decoupled UI ([spec_08_ui_hud_and_ux_specification.md](docs/designs/spec_08_ui_hud_and_ux_specification.md)):** Keep HUD and presentation layers driven by simulation state/events and player commands, ensuring key states and warnings remain legible across simulation speeds (`0x`–`4x`).

---

## 4. Elephant-Goldfish Model (EGM) Workflow Rules
1. **Spec Before Code:** Do not implement a subsystem until its corresponding `docs/designs/*.md` file has passed `/egm:egm-review` and `/egm:egm-resolve`.
2. **Bounded Contexts:** Keep spec changes scoped to a single subsystem file per review cycle so Goldfish reviewers can audit edge cases thoroughly without context dilution.
3. **Drift Prevention:** Whenever gameplay balancing or mechanic tweaks occur directly in code, run `/egm:egm-drift-report` to reconcile the implementation with `docs/designs/` before starting the next feature.
4. **Explicit Edge-Case Coverage:** Every subsystem spec must explicitly define:
   * State transitions and failure/deadlock recovery (e.g., zero-sail immobilization, minimum prize crew splits, bankruptcy/mutiny thresholds).
   * Exact formulas for rates, multipliers, and clamping bounds.