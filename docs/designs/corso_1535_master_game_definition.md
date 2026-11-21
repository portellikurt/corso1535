# CORSO 1535 — Master Game Definition & Architectural Baseline

## 1. High-Concept Summary
**Corso 1535** is a pauseable real-time naval strategy, maritime trade, and privateering simulation set in the **Central Mediterranean starting in Spring 1535**. 

Blending the tactical wind/oar naval combat and isometric deck-boarding of *Corsairs Gold* (1999) with the dynamic localized economy, guild politics, and urban development of *Patrician III* (2003), the game places the player at the historical hinge point between medieval crossbow/galley warfare and early modern gunpowder broadside fleets.

* **Genre:** Real-Time Naval Strategy / Economic Simulation / Tactical Boarding
* **Time Scale:** Pauseable real-time (`0x [Pause]`, `0.25x`, `1x`, `2x`, `4x`) spanning a 30-year dynamic campaign (`1535–1565`).
* **Map Scope:** The Central Mediterranean Triangle (Sicilian Channel, Strait of Messina, Malta, Sicily, Southern Italy, Sardinia, and the Barbary Coast from Algiers to Tripoli).

---

## 2. Core Gameplay Pillars

### Pillar I: Asymmetric Transitional Naval Warfare (*Corsairs Gold* Layer)
* **Wind vs. Oars:** Ships navigate a continuous wind field ($\vec{W}$). Sailing vessels (*Caravels*, *Carracks*, *Galleons*) depend on wind angle of attack and sail integrity, whereas oared vessels (*Fuste*, *Galleys*, *Galleasses*) trade open-ocean endurance for wind-independent burst speed and ramming agility.
* **Hybrid Ballistics & Ammo Matrix:** Combat spans three technological tiers—from steel windlass crossbows, fire-pots, and wrought-iron swivel guns (*versos*) to heavy bronze centerline bow-chaser cannons (*cannone di corsia*) and full multi-deck broadsides firing Round, Bar/Chain, Explosive, and Canister/Grapeshot.
* **Tactical Isometric Boarding & *Razzia* Raids:** Grappling an enemy hull or assaulting a coastal settlement transitions into a real-time isometric tactical skirmish across linked ship decks or village streets, governed by chokepoint positioning, morale, flanking multipliers, and surrender thresholds.

### Pillar II: Dynamic Maritime Economy & Shareholder *Corso* (*Patrician* Layer)
* **Localized Supply & Demand:** Every port produces and consumes goods (Grain, Timber, Hemp, Iron, Olive Oil, Salt, Wine, Silk, Munitions, Rum). Prices fluctuate dynamically based on harvests, blockades, and player-automated trade convoys.
* **The *Carati* (Ship Share) & Prize Court Economy:** Rather than pure piracy, privateering operates through state-issued *Letters of Marque* and investor syndicates (*carati*). Players can buy shares in AI expeditions or sell shares in their own fleet to raise capital, while captured prizes must be cleared through a port's **Prize Court** (*Tribunale degli Armamenti*)—or smuggled into pirate coves if taken illegally from neutral powers.
* **The Ransom & Specialist Loop:** Captured notables, merchants, and enemy captains hold dynamic ransom value negotiated through port brokers, while surviving crew rosters require daily provisions, rum, and plunder shares to prevent mutiny.

### Pillar III: Two-Tier Settlement Raiding & Urban Sovereignty
* **Tier 1 — Coastal Villages & Watchtowers (*Casali* / *Tonnare*):** Lightly fortified fishing, salt, and agricultural settlements. Players can launch rapid amphibious *razzia* raids to seize provisions, recruit mariners, or extract tribute without triggering full-scale imperial war.
* **Tier 2 — Fortified Port Cities & Citadels:** Major harbors (Birgu/Malta, Palermo, Messina, Syracuse, Tunis, Tripoli, Algiers, Cagliari, Naples). Players climb civic ranks via guild reputation and infrastructure investment—constructing warehouses, shipyards, walls, and harbor defenses—eventually governing cities or conquering them via multi-fleet naval blockades and siege assaults.

---

## 3. World Topology & 1535 Factions

Starting in **1535**, the Central Mediterranean is politically fluid: Valletta is not yet built (the Knights operate out of exposed Birgu), Hayreddin Barbarossa has just seized Tunis, and Emperor Charles V is massing an armada across Sicily and Malta.

| Faction | Starting Strongholds | Strategic Identity & Playstyle |
| :--- | :--- | :--- |
| **Order of St. John (Knights of Malta)** | Birgu, Mdina, Gozo, Tripoli | Elite military war galleys, strict *Tribunale* prize laws, scarce raw resources, high starting prestige. |
| **Habsburg Spain & Viceroyalty of Sicily** | Palermo, Messina, Syracuse, Naples | Wealthy grain/silver trade hubs, heavy merchant convoys, bureaucratic trade charters, early galleons. |
| **Ottoman Regencies (Barbary Corsairs)** | Tunis, Algiers, Djerba | High-speed *fuste* and war galleys, aggressive coastal raiding, lucrative ransom and contraband markets. |
| **Republic of Venice** *(Merchant Presence)* | Ionian outposts & Mediterranean trade routes | Neutral high-value silk/spice carriers; dangerous to provoke, ultra-profitable as trade partners or illicit targets. |
| **Genoese & Tuscan Financiers** | Corsican/Sardinian ports & banking houses | Providers of maritime insurance, shipyard upgrades, *carati* venture capital, and mercenary admirals. |
| **Independent Renegades** | Lampedusa, Pantelleria, hidden coves | Lawless pirate havens accepting unstamped contraband and unransomed captives with zero diplomatic standing required. |

---

## 4. Naval Technology & Progression Tree (`1535–1565`)

| Era Tier | Hull Classes | Primary Armament | Tactical Role |
| :--- | :--- | :--- | :--- |
| **Tier 1: Medieval Holdovers** *(1535 Start)* | *Fusta* (Half-Galley), *Brigantino*, Lateen Caravel | Windlass Crossbows, Fire-pots (*Pignatte*), Breech Swivel Guns | Fast coastal *razzia* raiders, blockade runners, and swarm grapplers. |
| **Tier 2: Transitional Galleys** *(1535–1545)* | 24-Bank War Galley, Armed *Polacca*, *Marciliana* | Heavy Bronze Bow-Chaser (*Corsia*), Flanking Sakers, Early Bar Shot | Line-of-battle Mediterranean workhorses; aim prow cannon to cripple target before boarding. |
| **Tier 3: Early Gunpowder Capital Ships** *(1545–1565)* | Venetian/Spanish Carrack (*Nave*), Early Galleon, Heavy Galleass | Multi-Deck Broadside Cannons, Matchlock Arquebusiers, Canister Shot | Open-water fleet anchors capable of devastating broadside duels and port bombardments. |

---

## 5. EGM Spec Decomposition Roadmap

To develop **Corso 1535** using a Spec-Driven / Elephant-Goldfish Model workflow, the codebase is partitioned into six bounded design documents under `docs/designs/`:

1. **`01_world_wind_navigation.md`:** Sea map coordinate system, wind vector field ($\vec{W}$), oar-vs-sail propulsion curves, and fleet pathfinding.
2. **`02_naval_combat_ballistics.md`:** Firing arcs (bow-chaser cones vs. broadside cones), ammo damage matrix, crew-to-cannon staffing ratios, and auto-reaction fire rules.
3. **`03_tactical_boarding_siege.md`:** Grapple triggers, reinforcement countdown timers, deck/village node graphs, flanking DPS multipliers, and surrender state machine.
4. **`04_economy_trade_carati.md`:** Port production/consumption ticks, automated convoy routing, warehouse storage, *carati* investor shares, and Prize Court validation.
5. **`05_settlements_politics.md`:** Two-tier settlement state (villages vs. fortified cities), civic building construction (e.g., fortifying Birgu/Sciberras), faction reputation, and espionage/smuggling.
6. **`06_campaign_progression.md`:** Historical timeline events (`1535` Tunis expedition through `1565` Great Siege era), tech unlocks, and AI fleet director behavior.