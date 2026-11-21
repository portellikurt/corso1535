# Spec 08: User Interface, HUD & Player Experience (UX)

## 1. Design Philosophy & Readability Goals
**Corso 1535** combines dense economic management (*Patrician III*) with real-time naval and tactical action (*Corsairs Gold*). The UI must keep the **map or tactical battlefield unobstructed (at least 75% viewport clear)** while surfacing critical wind, ammunition, morale, and trade alerts at `1x`–`4x` simulation speeds.

---

## 2. Top & Bottom Persistent Strategic HUD

### 2.1 Top Bar (World Clock, Treasury & Wind Compass)
* **Date & Speed Controls:** Displays current campaign date (`DD MMM YYYY`), time-dilation buttons (`|| Pause`, `0.25x`, `1x`, `2x`, `4x`), and quick-pause hotkey (`Space`).
* **Treasury & Reputation Pill:** Player's current **Ducats**, daily net upkeep/income delta (`+/- Ducats/day`), and active *Letters of Marque* / faction standing badges.
* **Wind Rose Widget:** Shows real-time wind arrow ($\theta_w$), wind speed in knots, and seasonal/storm warning icon. Hovering shows optimal sailing angles relative to current heading.

### 2.2 Bottom Context Dock (Selected Fleet or Settlement)
Adapts dynamically based on selection:
* **When a Fleet / Ship is Selected:**
  * Ship portrait, hull class badge, and three color-coded integrity bars (`Green > 60%`, `Amber 25–60%`, `Red < 25%`) for `Hull`, `Sail`, and `Oar`.
  * Crew headcount & `Morale` meter (`0–100`) with hover breakdown (`Provisions`, `Rum`, `Wages`).
  * **Combat & Movement Strip:** Sail level selector (`0% / 50% / 100%`), Oar mode toggle (`Stowed / Cruise / Ram Sprint`), Ammo selector (`Round / Chain / Explosive / Grapeshot`), and `Auto-Reaction Fire` toggles (`Bow / Port / Starboard`).
  * Primary action buttons: `Board / Grapple`, `Raid Village`, `Open Trade Route Editor`, `Disband / Transfer Crew`.
* **When a Settlement (Village or Port City) is Selected:**
  * Settlement tier badge (`Tier 1 Village` vs. `Tier 2 Fortified Port`), controlling faction, and player civic rank.
  * Quick tabs: `Market & Prices`, `Warehouse & Trade Routes`, `Shipyard & Carati`, `Civic & Fortifications`, `Prize Court`.

---

## 3. Strategic Map Overlays & Filters
Players can toggle four map overlays without leaving the main strategic view:
1. **Trade & Price Overlay:** Color-codes ports by surplus/deficit for a selected commodity and draws animated dashed lines for active automated trade convoys.
2. **Wind & Shoal Overlay:** Renders directional wind streamlines and highlights shallow `SHOALS_COASTAL` vs. `DEEP_SEA` boundaries so players don't ground deep-draft carracks/galleons.
3. **Diplomacy & Threat Overlay:** Highlights hostile/neutral/allied territories, active blockades, and recent *razzia* / corsair sightings.
4. **Carati & Opportunity Overlay:** Flags active *carati* share listings, treasure convoy rumor markers, and smuggling/ransom hubs.

---

## 4. Tactical Boarding & *Razzia* HUD (Spec 03 View)
When grappling a ship or assaulting a settlement/fortress:
* **Reinforcement Queue Strip (Top-Center):** Displays incoming friendly or enemy ships with live `T_arrive` countdowns and crew counts.
* **Squad & Archetype Bar (Bottom):** Selectable groups (`All`, `Corsairs/Marines`, `Sailors`, `Crossbow/Arquebus`, `Officers`) with stance buttons (`Advance`, `Hold Chokepoint`, `Fire Pistols/Volley`, `Fallback / Retreat`).
* **Morale & Surrender Banner:** Displays enemy vs. friendly morale bars; when `OFFER_SURRENDER` triggers, presents a clear modal choice (`Accept Surrender & Take Captives` vs. `Refuse Quarter — Fight to Last Man`).

---

## 5. Feedback, Telegraphing & Accessibility
* **Combat Telegraphing:** Incoming cannon salvos display a brief aim/trajectory flash; fire-pots and low-morale/mutiny states trigger distinct status icons on the ship nameplate.
* **Notification Toast Stack:** Non-blocking alerts on the left edge (`Low Provisions`, `Prize Court Ready`, `Convoy Under Attack`, `Historical Event`) with click-to-center-camera support.
* **Colorblind & Clarity Support:** Every state (faction allegiance, hull health, ammo type, wind direction) pairs color with shape/iconography and numeric tooltips rather than relying on color alone.