# Spec 07: Canonical Data Schemas, Balance Tables & Edge-Case Rules

## 1. Units, Coordinate Conventions & Determinism
To guarantee deterministic execution across `0x` pause, speed multipliers (`0.25x`, `1x`, `2x`, `4x`), and save/load serialization:
* **World Units:** $1\text{ world unit} = 100\text{ meters}$. Strategic map spans $x \in [0, 16000]$, $y \in [0, 12000]$.
* **Angles:** Stored as `headingRad` in radians $[0, 2\pi)$, where $0$ points East ($+X$) and $\pi/2$ points North ($+Y$).
* **Simulation Clock:** `tickIndex` (unsigned 64-bit integer starting at `0`), `fixedDtSec = 0.05` ($20\text{ Hz}$). All in-game timestamps derive from `tickIndex * fixedDtSec`.
* **PRNG:** Seeded Mulberry32 / Xorshift128+ state stored inside `WorldState` per campaign (`rngState: [number, number, number, number]`) so AI actions, weather drift, and combat rolls are replayable.

---

## 2. Canonical Data Schemas

```typescript
export type FactionId =
  | 'KNIGHTS_MALTA'
  | 'HABSBURG_SPAIN'
  | 'OTTOMAN_BARBARY'
  | 'VENICE'
  | 'GENOA_FINANCE'
  | 'RENEGADE';

export type HullClassId =
  | 'FUSTA'
  | 'BRIGANTINO'
  | 'CARAVEL'
  | 'WAR_GALLEY'
  | 'POLACCA'
  | 'MARCILIANA'
  | 'CARRACK'
  | 'GALLEON'
  | 'GALLEASS';

export type CommodityId =
  | 'GRAIN'
  | 'OLIVE_OIL'
  | 'SALT'
  | 'WINE'
  | 'WOOL'
  | 'TIMBER'
  | 'HEMP_SAILCLOTH'
  | 'IRON_INGOTS'
  | 'MUNITIONS'
  | 'RUM'
  | 'SILK_SPICES'
  | 'RANSOM_CAPTIVES';

export type AmmoType = 'ROUND_SHOT' | 'CHAIN_SHOT' | 'EXPLOSIVE_SHOT' | 'GRAPESHOT';
export type GameSpeed = 0 | 0.25 | 1 | 2 | 4;

export interface CrewRoster {
  sailors: number;
  corsairs: number;
  crossbowArquebus: number;
  officers: number;
  morale: number; // [0, 100]
  rowerStamina: number; // [0, 100]
}

export interface ShipEntity {
  id: string;
  name: string;
  ownerFaction: FactionId;
  ownerPlayerId: string | null;
  fleetId: string | null;
  hullClass: HullClassId;
  position: { x: number; y: number };
  headingRad: number;
  sailLevel: 0 | 0.5 | 1;
  oarMode: 'STOWED' | 'CRUISE' | 'RAM_SPRINT';
  hullHp: number;
  maxHullHp: number;
  sailHp: number;
  maxSailHp: number;
  oarHp: number;
  maxOarHp: number;
  activeCannons: number;
  maxCannons: number;
  crew: CrewRoster;
  cargoHoldCapacity: number;
  cargo: Record<CommodityId, number>;
  selectedAmmo: AmmoType;
  autoReactionFire: {
    bow: boolean;
    port: boolean;
    starboard: boolean;
  };
  cooldownsSec: {
    bow: number;
    port: number;
    starboard: number;
    rail: number;
  };
  prizeMeta: {
    isPrize: boolean;
    capturedFromFaction: FactionId | null;
    letterOfMarqueValid: boolean;
    prizeCrewAssigned: number;
  };
}

export interface CaratiStake {
  id: string;
  vesselOrExpeditionId: string;
  holderFaction: FactionId;
  holderIsPlayer: boolean;
  sharesCount: number; // Out of 24 total carati per ship/expedition
  investedDucats: number;
}

export interface ConvoyStop {
  settlementId: string;
  buyRules: Array<{ commodity: CommodityId; maxPrice: number; targetQty: number }>;
  sellRules: Array<{ commodity: CommodityId; minPrice: number; keepQty: number }>;
}

export interface TradeRoute {
  id: string;
  fleetId: string;
  stops: ConvoyStop[];
  currentStopIndex: number;
  minEscortStrength: number;
  fleeOnHostileSighting: boolean;
  active: boolean;
}

export interface WorldState {
  schemaVersion: string;
  campaignSeed: number;
  rngState: [number, number, number, number];
  tickIndex: number;
  gameSpeed: GameSpeed;
  dateIso: string;
  wind: {
    directionRad: number;
    targetDirectionRad: number;
    speedKnots: number;
    targetSpeedKnots: number;
    transitionRemainingSec: number;
  };
  ships: Record<string, ShipEntity>;
  tradeRoutes: Record<string, TradeRoute>;
  caratiStakes: Record<string, CaratiStake>;
}
```

---

## 3. Baseline Hull & Propulsion Balance Table
Speeds are expressed in map units per second at $1\times$ speed ($1\text{ unit/s} \approx 19.4\text{ knots}$ scaled for readable strategic pacing).

| Hull Class | Era Tier | Draft Type | Base Sail Speed | Base Oar Speed | Turn Rate ($\text{rad/s}$) | Hull / Sail / Oar HP | Min / Optimal / Max Crew | Hold Cap | Build Cost (Ducats) |
| :--- | :---: | :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **Fusta** | Tier 1 | Shoal | `7.5` | `9.5` | `1.35` | `320 / 240 / 280` | `18 / 45 / 70` | `80` | `1,400` |
| **Brigantino** | Tier 1 | Shoal | `8.5` | `6.0` | `1.15` | `400 / 320 / 180` | `14 / 35 / 55` | `110` | `1,800` |
| **Caravel** | Tier 1 | Shoal | `9.5` | `0.0` | `0.85` | `520 / 450 / 0` | `12 / 30 / 48` | `180` | `2,200` |
| **War Galley** | Tier 2 | Shoal | `7.0` | `10.5` | `1.20` | `650 / 360 / 420` | `45 / 110 / 160` | `120` | `3,800` |
| **Polacca** | Tier 2 | Shoal | `9.0` | `4.0` | `0.95` | `600 / 480 / 120` | `18 / 45 / 75` | `210` | `3,200` |
| **Marciliana** | Tier 2 | Shoal | `8.0` | `0.0` | `0.75` | `700 / 540 / 0` | `16 / 40 / 65` | `320` | `3,000` |
| **Carrack (Nave)** | Tier 3 | Deep | `7.8` | `0.0` | `0.55` | `1,350 / 850 / 0` | `35 / 90 / 150` | `550` | `6,800` |
| **Early Galleon** | Tier 3 | Deep | `10.2` | `0.0` | `0.68` | `1,200 / 920 / 0` | `32 / 85 / 140` | `440` | `7,500` |
| **Heavy Galleass** | Tier 3 | Deep | `6.5` | `8.5` | `0.72` | `1,600 / 650 / 550` | `70 / 180 / 260` | `260` | `9,200` |

---

## 4. Baseline Commodity & Upkeep Constants

| Commodity | Category | Base Price (Ducats) | Daily Crew Consumption (per 100 crew) | Notes |
| :--- | :--- | :---: | :---: | :--- |
| `GRAIN` | Staple | `35` | `1.5` units (`PROVISIONS` pool) | Primary food staple; spikes heavily in winter/sieges. |
| `OLIVE_OIL` | Staple | `55` | `0.5` units | High demand in northern/central ports. |
| `SALT` | Staple | `28` | `0.0` | Key for preserving fish/meat & curing goods. |
| `WINE` | Staple | `48` | `0.5` units | Morale boost when stocked alongside provisions. |
| `WOOL` | Staple | `52` | `0.0` | Bulk textile input across Mediterranean hubs. |
| `TIMBER` | Naval | `65` | `0.0` | Required for hull repairs & shipyard construction. |
| `HEMP_SAILCLOTH` | Naval | `75` | `0.0` | Required for sail repairs & rigging upgrades. |
| `IRON_INGOTS` | Naval | `90` | `0.0` | Required for cannon casting & fortification upgrades. |
| `MUNITIONS` | Naval | `85` | `0.0` | Consumed per cannon salvo (`1 unit` per `4` shots fired). |
| `RUM` | Naval / Morale | `60` | `1.0` unit | Prevents `-8/day` morale decay; grants `+15` boarding morale. |
| `SILK_SPICES` | Luxury | `180` | `0.0` | Highest-margin trade & contraband cargo. |
| `RANSOM_CAPTIVES` | Prize / Ransom | `120` (base) | `0.5` provisions per 10 captives | Converted to gold via ransom brokers or Prize Court. |

* **Crew Daily Wage Rate:** `2.0` Ducats/day per `Sailor` or `Crossbow/Arquebus`, `4.5` Ducats/day per `Corsair`, `8.0` Ducats/day per `Officer`. Unpaid wages deduct `-10` morale/day.

---

## 5. Mandatory Edge-Case & Deadlock Resolutions
1. **Zero-Sail & Zero-Oar Stalemate:** If a ship has `sailHp == 0` and `oarHp == 0` in open sea and no hostile ships remain within `600` units, a `DISTRESS_TOW` or `SCUTTLE_SALVAGE` action becomes available so the player can either summon an allied/paid tug or scuttle the hulk to recover `25%` of its cargo and surviving crew into boats.
2. **Prize Crew Deficit After Boarding:** If capturing an enemy ship leaves the victor with fewer than `minCrew` on either the flagship or the prize, the player must transfer crew from other friendly ships in the fleet or abandon the prize (`SCUTTLE_PRIZE`) before departing the engagement zone.
3. **Hostile or Blockaded Trade Route Stop:** If a `TradeRoute` stop belongs to a faction at war with the convoy's owner or is under active naval blockade, the convoy automatically skips that stop, logs a `ROUTE_SKIPPED_HOSTILE` alert, and proceeds to the next valid stop.
4. **Faction Collapse Mid-*Carati* Expedition:** If a faction issuing a *Letter of Marque* signs peace or loses its capital while a player's expedition is at sea, `letterOfMarqueValid` is evaluated against active diplomatic state at the moment of **Prize Court submission**, warning the player before docking.