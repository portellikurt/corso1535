# Spec 05: Two-Tier Settlements, Urban Building & Faction Politics

## 1. Two-Tier Settlement Hierarchy
The Central Mediterranean map contains **32 distinct settlements** split into two gameplay tiers to balance frequent *Corsairs Gold*-style raiding with long-term *Patrician*-style city building:

### Tier 1: Coastal Villages, Watchtowers & *Tonnare* (20 Locations)
* *Examples:* Marsaxlokk, Gozo, Pantelleria, Lampedusa, Cefalù, Augusta, Hammamet, Sfax.
* **Mechanics:** Low garrison (20–60 militia), no heavy coastal artillery.
* **Player Interaction:**
  * Vulnerable to rapid amphibious **Razzia Raids** (Spec 03) to loot stockpiled grain/salt, impress sailors, or demand a one-time tribute.
  * Can be upgraded by an owning faction/player with a **Coastal Watchtower** (extends fog-of-war radar by $350\text{ m}$ and provides early warning against corsair raids).

### Tier 2: Fortified Port Cities & Citadels (12 Locations)
* *Examples:* Birgu (Malta), Palermo, Messina, Syracuse, Naples, Cagliari, Tunis, Algiers, Tripoli.
* **Mechanics:** Full dynamic markets (Spec 04), shipyards, *Tribunale* courts, civic guilds, and multi-bastion harbor defenses.

---

## 2. Urban Construction & Fortifying Malta (1535 → 1565)
In Tier 2 cities where the player holds a **Trading House** (or governs the city), players can invest gold and construction materials (`TIMBER`, `IRON_INGOTS`, `STONE`) into civic infrastructure:

* **Economic Buildings:** Warehouses, Rope-walks (produces `HEMP_SAILCLOTH`), Iron Foundries (produces `MUNITIONS`), and Taverns (increases sailor recruitment pool).
* **The Sciberras / Valletta Mega-Project (Malta Special):** Starting in 1535, Malta's capital is vulnerable Birgu. Players aligned with the Order of St. John can fund the fortification of **Fort St. Elmo** on the Sciberras peninsula and eventually lay the foundations for **Valletta**, transforming Malta from a resource-starved rock into an impregnable naval citadel.

---

## 3. Civic Rank & Political Climbing (*Patrician* Layer)
Within any Tier 2 city, the player climbs five civic ranks by combining trade volume, privateering prestige, civic donations, and guild missions:

1. **Foreign Captain (`0–19%` Rep):** Dock access and basic market trading only.
2. **Licensed Corsair / Merchant (`20–39%` Rep):** Eligible for *Letters of Marque*, *carati* share trading, and warehouse permits.
3. **Guild Alderman (`40–64%` Rep):** $20\%$ shipyard discount, access to municipal trade monopolies, and ability to bribe harbor officials to overlook minor contraband.
4. **Harbor Bailiff / Admiral (`65–84%` Rep):** Commands local city patrol galleys, sets port tariff rates, and unlocks Tier 3 capital ship blueprints.
5. **Governor / Grand Prior (`85–100%` Rep):** Full sovereign control over city tax rates, garrison recruitment, building queues, and diplomatic treaties.

---

## 4. Naval Blockades & City Conquest
Capturing a Tier 2 Fortified Port City is an endgame multi-fleet operation:
1. **Harbor Blockade:** Stationing warships outside the harbor mouth cuts sea trade (`0%` maritime imports/exports). After 14 days of blockade, local food stocks deplete, starving the garrison and lowering defender morale.
2. **Coastal Battery Duel:** Attacking fleets must silence harbor stone bastions (which fire heated shot with high fire chance against wooden hulls) before landing assault troops.
3. **Citadel Assault:** Resolves via a large-scale Spec 03 tactical battle across the harbor mole and citadel gates. Upon victory, the player chooses to **Pillage & Withdraw** (massive immediate gold/loot, destroys $40\%$ of city infrastructure) or **Annex & Govern** (claims the port for their faction or establishes an independent corsair republic).