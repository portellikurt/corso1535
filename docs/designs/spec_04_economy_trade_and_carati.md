# Spec 04: Localized Economy, Automated Trade & *Carati* Syndicates

## 1. Commodities & Regional Scarcity
The Mediterranean economy simulates **12 core trade goods** categorized into Bulk Staples, Naval Supplies, and Luxury/Contraband:

* **Bulk Staples:** `GRAIN` (Wheat/Barley), `OLIVE_OIL`, `SALT`, `WINE`, `WOOL`
* **Naval & Military Supplies:** `TIMBER`, `HEMP_SAILCLOTH`, `IRON_INGOTS`, `MUNITIONS`, `RUM`
* **High-Margin & Prize Goods:** `SILK_SPICES`, `RANSOM_CAPTIVES`

### 1.1 Dynamic Port Pricing Formula
Every port maintains a local inventory $I_g$ and a target equilibrium stock $E_g$ for each good $g$, updated once per in-game day (`24h` tick). Local price $P_g$ scales non-linearly around base price $P_{\text{base}}$:

$$P_g = P_{\text{base}} \cdot \text{clamp}\left(\left(\frac{E_g}{\max(1, I_g)}\right)^{0.65}, 0.35, 3.20\right)$$

* **Regional Asymmetries (1535 Baseline):**
  * *Sicily (Palermo, Syracuse):* Massive `GRAIN` and `WINE` surplus; high demand for `IRON_INGOTS` and `SILK_SPICES`.
  * *Malta (Birgu):* Severe deficit in `GRAIN`, `TIMBER`, and `HEMP_SAILCLOTH`; high demand for `MUNITIONS` and `RANSOM_CAPTIVES`.
  * *Barbary Coast (Tunis, Algiers, Djerba):* Surplus of `WOOL`, `OLIVE_OIL`, and `SALT`; high payout for unstamped contraband and `TIMBER`.

---

## 2. *Patrician*-Style Warehouses & Automated Trade Convoys
Once a player establishes a **Trading House (Fondaco)** in a port (requiring $\ge 25\%$ faction reputation and $2,500\text{ Ducats}$), they unlock:
1. **Port Warehouses:** Store goods indefinitely off-ship, allowing players to buy during harvest gluts and sell during winter storms or naval blockades.
2. **Automated Convoy Routes:** Players assign a merchant fleet to a multi-stop loop (`Port A -> Port B -> Port C`) with conditional buy/sell price thresholds (e.g., *Buy Grain if $P \le 45\text{ Ducats}$, Sell if $P \ge 75\text{ Ducats}$*) and an escort threat threshold (auto-flee to nearest friendly port if hostile warships are sighted within radar range).

---

## 3. Shareholder Privateering (*Carati* System)
Modeled directly on historical Maltese and Genoese maritime partnerships, ships and expeditions are divided into **24 shares (*carati*)**:

* **Issuing Shares to Fund Expeditions:** Early-game players short on capital can list up to $12/24$ *carati* of a new warship on the Valletta/Birgu or Genoa exchange. Local patricians invest upfront gold in exchange for a proportional cut of net prize proceeds upon return.
* **Investing in AI Corsairs:** Players can purchase *carati* in AI-captained corsair voyages departing friendly ports. Each expedition displays its captain's skill, target hunting zone, and risk rating—paying dividends upon return or defaulting if sunk/captured.

---

## 4. The Prize Court (*Tribunale degli Armamenti*) & Contraband
Every captured ship and cargo crate carries a metadata tag: `victimNation` and `captureLicenseValid` (true only if the player held an active *Letter of Marque* against `victimNation` at the time of capture).

1. **Legal Prize Clearance:** Docking at a licensed port submits captured goods to the *Tribunale*. The state deducts a **$10\%$ Sovereign Tax** and **$3\%$ Admiralty Fee**, issuing a legal bill of sale for the remainder.
2. **Illegal Prizes & Smuggling:** If the player captured a neutral Venetian or allied Spanish vessel without a valid Letter of Marque, submitting it to an official Prize Court results in **confiscation, a heavy fine, and severe reputation loss**.
3. **Renegade Havens (Lampedusa / Djerba):** Players can offload unstamped contraband and unransomed captives at pirate coves at a **$35\%$ fence discount** with zero legal checks—or bribe corrupt port magistrates if their local political influence is high enough.

---

## 5. Crew Upkeep, Morale & Ransom Negotiation
* **Daily Consumption Tick:** Every 24 in-game hours, each ship consumes $1\text{ unit of Provisions}$ per 10 crew and $1\text{ unit of Rum}$ per 20 crew. Running dry drops morale by $-8/\text{day}$; below $15\text{ morale}$, a mutiny check triggers.
* **Ransom Brokers:** Captured enemy officers, merchants, and nobles are held in port *Bagni* (prisons). Ransom offers arrive every 7 days via Jewish, Armenian, or French diplomatic intermediaries, scaling with the captive's rank and social class.