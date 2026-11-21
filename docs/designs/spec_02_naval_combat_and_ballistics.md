# Spec 02: Naval Combat, Weapon Tiers & Ballistics Matrix

## 1. Firing Arcs & Weapon Mount Architecture
Unlike pure Age-of-Sail games where all ships fire broadsides, **Corso 1535** distinguishes three directional weapon mount types:

1. **Prow Centerline Mount (`BOW_CHASER`):** Narrow $18^\circ$ forward cone. Houses heavy bronze *cannone di corsia* and flanking sakers on galleys. Requires steering the entire ship to aim.
2. **Broadside Batteries (`PORT` / `STARBOARD`):** Wide $70^\circ$ lateral cones per side. Houses multi-deck cannons on *Caravels*, *Carracks*, *Galleons*, and *Galleasses*.
3. **Bulwark & Fighting-Top Skirmishers (`RAIL_360`):** Short-range $360^\circ$ aura ($80\text{ m}$ radius) for windlass crossbows, matchlock arquebuses, fire-pots (*pignatte*), and breech-loading swivel guns (*versos*).

---

## 2. Weapon Era Tiers & Reload Mechanics

### 2.1 Crew Staffing & Reload Formula
Each gun battery or crossbow detachment requires an optimal crew count $C_{\text{opt}}$ (`sailors + corsairs`). Effective reload time $T_{\text{reload}}$ scales with staffing and morale ($M \in [0.0, 1.0]$):

$$T_{\text{reload}} = T_{\text{base}} \cdot \max\left(1.0, \frac{C_{\text{opt}}}{C_{\text{assigned}}}\right) \cdot \left(1.4 - 0.5 \cdot M\right)$$

### 2.2 Weapon Archetypes
| Weapon Class | Era Tier | Mount Type | Max Range | Base Reload | Special Mechanic |
| :--- | :---: | :--- | :---: | :---: | :--- |
| **Windlass Crossbows** | Tier 1 | `RAIL_360` | $90\text{ m}$ | $6.0\text{ s}$ | Ignores gunport armor; targets exposed deck crew directly. |
| **Swivel Guns (*Versos*)** | Tier 1 | `RAIL_360` | $110\text{ m}$ | $8.0\text{ s}$ | High anti-personnel scatter; stuns enemy reload timers for $1.5\text{ s}$. |
| **Fire-Pots (*Pignatte*)** | Tier 1–2 | `RAIL_360` | $45\text{ m}$ | $14.0\text{ s}$ | Ignites enemy rigging/deck; deals DoT until crew spends ticks extinguishing. |
| **Bronze Bow-Chaser** | Tier 2 | `BOW_CHASER` | $260\text{ m}$ | $18.0\text{ s}$ | Heavy single-salvo impact; high chance to shatter enemy rudder or prow. |
| **Broadside Culverins** | Tier 2–3 | `PORT` / `STARBOARD` | $240\text{ m}$ | $15.0\text{ s}$ | Sequential rolling broadside; supports all four ammo types. |

---

## 3. Ammunition Damage Matrix
For gunpowder cannons (`BOW_CHASER` and `BROADSIDE`), players toggle between four munitions. Damage applies to four distinct hit pools: `hullHp`, `sailHp` (or `oarHp` on galleys), `activeCannons`, and `crew`.

| Ammo Type | Hull Multiplier | Sail / Oar Multiplier | Cannon Destruction | Crew Casualties | Range Modifier |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **Round Shot** *(Standard)* | `1.0x` | `0.8x` | `1.0x` | `0.9x` | `1.0x` |
| **Bar / Chain Shot** | `0.2x` | `2.6x` | `0.1x` | `0.3x` | `0.75x` |
| **Explosive / Carcass Shot** | `1.9x` | `0.6x` | `1.8x` | `1.1x` | `0.85x` |
| **Canister / Grapeshot** | `0.1x` | `0.2x` | `0.0x` | `2.8x` | `0.50x` |

*Note:* Changing ammo types while a battery is mid-reload resets progress by $50\%$ of $T_{\text{reload}}$ (preventing instant zero-cost ammo swapping right before impact).

---

## 4. Ramming & Auto-Reaction Fire Rules

### 4.1 Galley Prow Ramming
Oared vessels equipped with a reinforced spur (*rostro*) deal kinetic collision damage when striking an enemy hull at an angle between $45^\circ$ and $135^\circ$ while `RAM_SPRINT` is active:
$$D_{\text{ram}} = K_{\text{hullClass}} \cdot V_{\text{rel}}^2 \cdot \sin(\theta_{\text{impact}})$$
* Self-damage to the ramming galley's prow is $25\%$ of $D_{\text{ram}}$.
* Immediately after ramming impact, the galley automatically deploys grappling hooks if `autoGrappleOnRam == true`.

### 4.2 Auto-Reaction Fire State
Each firing arc (`BOW`, `PORT`, `STARBOARD`) has an independent boolean toggle `autoReactionFire`:
* When `true` and the arc's cooldown is $0$, the simulation checks every $4\text{ ticks}$ ($0.2\text{ s}$) whether any hostile vessel's bounding box intersects the firing cone within $85\%$ of maximum effective range.
* If valid and unobstructed by friendly hulls (`friendlyFireCheck == true`), the battery fires automatically, consuming `munitions` from ship cargo.

---

## 5. Ship Disable & Surrender Triggers at Sea
1. **Sinking (`hullHp <= 0`):** Ship enters a $20\text{ second}$ sinking animation; surviving crew launch lifeboats (can be rescued as prisoners or recruits).
2. **Immobilized (`sailHp == 0` AND `oarHp == 0`):** Ship drifts dead in the water. If all friendly ships in the battle are immobilized and outgunned $3:1$, AI captains strike their colors automatically.
3. **Prize Capture:** Captured vessels do not vanish—they require assigning a **Prize Crew** (minimum $20\%$ of the target's `optimalCrew`) transferred from the player's fleet before the prize can navigate to port.