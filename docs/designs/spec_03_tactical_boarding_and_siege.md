# Spec 03: Tactical Isometric Deck Boarding & *Razzia* Assaults

## 1. Transition Trigger & Reinforcement Window
Tactical combat triggers when:
1. Two ships drop below $3.0\text{ knots}$ relative velocity within $15\text{ meters}$ with grappling hooks engaged.
2. A fleet initiates an amphibious **Coastal *Razzia*** against a Tier 1 village or a **Port Siege Assault** against a Tier 2 fortress.

### 1.1 World Freeze & Deterministic Reinforcement Timer
* Upon entering Tactical Boarding, the Strategic Ocean Layer pauses except for a deterministic **Reinforcement Queue** driven by tactical simulation ticks (scaled identically to the player's chosen game speed `0.25x`–`4x`, avoiding real-time wall-clock exploits).
* Any ship within $400\text{ meters}$ on the Strategic Layer at the moment of grappling is queued with an arrival countdown:
  $$T_{\text{arrive}} = \frac{\text{distance}}{V_{\text{approach}}}$$
* When $T_{\text{arrive}}$ reaches $0$, the reinforcing vessel docks alongside an open flank node on the deck graph and deploys its boarding party.

---

## 2. Deck & Settlement Node Graph
Each hull class and settlement layout is defined as a 2D isometric navigation mesh partitioned into tactical zones and chokepoint nodes:

* **Ship Deck Zones:** `FORECASTLE`, `WAIST_MAIN_DECK`, `QUARTERDECK`, `GUN_DECK` (on Tier 3 vessels).
* **Grapnel Bridges (`GANGLWAY_NODE`):** Narrow $1.5\text{ m}$ chokepoints linking grappled hulls. Only 2 units may traverse abreast simultaneously.
* **Village / Fortress Zones:** `BEACH_LANDING`, `MARKET_SQUARE`, `WATCHTOWER`, `CITADEL_GATE`. Defenders gain $+25\%$ armor and ranged accuracy behind stone walls or barricades.

---

## 3. Unit Archetypes & Asymmetric Melee Math

Every crew member belongs to one of four combat roles:

| Unit Role | Weapon Loadout | HP | Melee DPS | Ranged Capability | Tactical Specialty |
| :--- | :--- | :---: | :---: | :---: | :--- |
| **Sailor / Oarsman** | Cutlass / Boarding Axe | `60` | `8` | None | High numbers; captures rigging/helm nodes fastest. |
| **Corsair / Marine** | Rapier + Wheellock Pistol | `95` | `15` | 2 high-damage pistol shots (`35 dmg`) | Shock troops; breaks chokepoints with opening pistol volley. |
| **Crossbowman / Arquebusier** | Windlass Crossbow or Matchlock | `70` | `5` | Continuous ranged fire (`18 dmg / 4s`) | Holds elevated forecastles and fortress walls. |
| **Officer / Knight / Rais** | Estoc + Buckler + Pistol | `140` | `22` | 3 pistol shots (`40 dmg`) | Projects a $6\text{ m}$ Morale Aura ($+20\%$ ally attack speed). |

### 3.1 Chokepoint Flanking & Overpowering Rule
Combat resolves on a $0.5\text{ second}$ ($10\text{ tick}$) melee cycle. When $N$ attackers engage a single defender within melee radius ($1.8\text{ m}$):
* The defender can only block/counter-attack **1 primary target** in their front $120^\circ$ arc.
* Flanking attackers ($N \ge 2$) deal $+35\%$ backstab/flank damage and take **zero counter-attack damage**—rewarding players for piercing a gangplank chokepoint and enveloping defenders.

---

## 4. Morale, Surrender & Last-Stand State Machine

Each side tracks a continuous `moraleScore` $\in [0, 100]$, modified every second by:
* Friendly casualties ($-1.5$ per death)
* Loss of Captain / Officer ($-25$ immediate shock)
* Rum ration bonus prior to battle ($+15$ starting buffer)
* Outnumbered ratio in active deck zone ($-2.0/\text{sec}$ if outnumbered $> 2:1$)

### 4.1 Surrender Evaluation
* **Trigger Threshold:** Evaluated every $2\text{ seconds}$. If an AI defender's surviving headcount drops below their hull class's `surrenderCrewThreshold` (e.g., `5` on a *Fusta*, `10` on a *War Galley*, `18` on a *Galleon*) AND `moraleScore < 30`, the AI triggers `OFFER_SURRENDER`.
* **Player Choice (`ACCEPT` vs. `REFUSE_QUARTER`):**
  * `ACCEPT`: Combat ends immediately; surviving enemy crew become prisoners/ransom captives, and ship cargo/hull remain intact.
  * `REFUSE_QUARTER`: Enemy enters `DESPERATE_LAST_STAND` ($+30\%$ melee DPS, immune to surrender checks, must be eliminated to the last man) and the player suffers a $-10$ global **Honor/Chivalry** reputation penalty with Christian and Ottoman Prize Courts alike.