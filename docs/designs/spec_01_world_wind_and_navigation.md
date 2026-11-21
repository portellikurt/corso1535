# Spec 01: World Map, Wind Physics & Naval Navigation

## 1. Overview & Simulation Tick
The Strategic Ocean Layer operates on a deterministic fixed simulation tick of **$20\text{ Hz}$** ($\Delta t = 0.05\text{ s}$ at $1\times$ speed). Player time-dilation multipliers (`0x`, `0.25x`, `1x`, `2x`, `4x`) scale the number of ticks processed per frame rather than altering $\Delta t$, preserving deterministic physics across speeds.

---

## 2. World Coordinate System & Navigation Mesh
* **Map Bounds:** Cartesian 2D projection covering the Central Mediterranean ($32^\circ\text{N}$ to $41^\circ\text{N}$, $3^\circ\text{E}$ to $18^\circ\text{E}$), normalized to $16,000 \times 12,000$ world units ($1\text{ unit} \approx 100\text{ meters}$).
* **Terrain Layers:**
  1. `DEEP_SEA`: Accessible to all hull classes.
  2. `SHOALS_COASTAL`: Accessible only to shallow-draft vessels (*Fusta*, *Brigantino*, *Caravel*, *War Galley*). Deep-draft vessels (*Carrack*, *Galleon*) take grounding hull damage and speed penalties ($0.25\times$) if forced into shoals.
  3. `LAND_IMPASSABLE`: Islands and mainland coastlines.
* **Choke Points:** Narrow straits (Strait of Messina, Sicilian Channel between Marsala and Cape Bon, Gozo Channel) use higher-resolution navigation polygons and generate localized wind funnels.

---

## 3. Wind Vector Field ($\vec{W}$)
The wind is modeled as a continuous, smoothly interpolating vector field $\vec{W}(x, y, t) = (\theta_w, v_w)$, where $\theta_w \in [0, 2\pi)$ is wind direction and $v_w \in [0, 35]\text{ knots}$ is wind speed.

* **Seasonal Bias:**
  * *Summer (May–Sept):* Predominant *Maestrale* (NW) and *Levante* (E) breezes, with frequent dead-calm zones ($v_w < 3\text{ knots}$) in the central basin.
  * *Winter (Nov–Mar):* Heavy *Sirocco* (SE) and *Tramontana* (N) gales ($v_w > 25\text{ knots}$), inflicting storm attrition on low-freeboard oared vessels in open deep sea.
* **Macro Update:** Wind targets update every $6\text{ in-game hours}$ and linearly interpolate over $30\text{ in-game minutes}$ to prevent sudden vector snapping.

---

## 4. Hybrid Propulsion Model (Sails vs. Oars)

Every hull possesses both a **Sail Propulsion Profile** and an **Oar Propulsion Profile** (where pure sailing ships have `maxOarSpeed = 0`).

### 4.1 Sail Velocity ($V_{\text{sail}}$)
Let $\Delta\theta = |\text{headingRad} - \theta_w|$ normalized to $[0, \pi]$ be the angle off the wind:
* **In Irons ($\Delta\theta < 35^\circ$ for square-rigged; $\Delta\theta < 25^\circ$ for lateen-rigged):** $C_{\text{wind}}(\Delta\theta) = 0$.
* **Close-Hauled to Broad Reach ($35^\circ \le \Delta\theta \le 135^\circ$):** $C_{\text{wind}}(\Delta\theta)$ peaks at $1.0$ around $110^\circ$ (broad reach).
* **Running ($135^\circ < \Delta\theta \le 180^\circ$):** $C_{\text{wind}}(\Delta\theta) = 0.85$ for square rigs, $0.70$ for lateen rigs.

$$V_{\text{sail}} = V_{\text{baseSail}} \cdot \text{sailLevel} \cdot \left(\frac{\text{sailHp}}{\text{maxSailHp}}\right) \cdot C_{\text{wind}}(\Delta\theta) \cdot \min\left(1.0, \frac{v_w}{18.0}\right)$$

### 4.2 Oar Velocity ($V_{\text{oar}}$)
Oared ships (*Fusta*, *Galley*, *Galleass*) operate in three oar states: `STOWED`, `CRUISE`, and `RAM_SPRINT`.
* `CRUISE`: Constant wind-independent speed scaled by surviving rower/sailor headcount ratio ($\text{activeRowers} / \text{optimalRowers}$).
* `RAM_SPRINT`: Grants $+45\%$ burst speed for $15\text{ seconds}$, consuming `rowerStamina` ($100 \rightarrow 0$). Once exhausted, oars drop to $0.5\times$ efficiency until stamina recovers.
* **Open-Sea Gale Penalty:** In `DEEP_SEA` when $v_w > 24\text{ knots}$, low-freeboard galleys suffer $-50\%$ $V_{\text{oar}}$ and take gradual hull flooding damage unless sheltering near coastlines.

### 4.3 Effective Speed & Turning Rate
$$V_{\text{total}} = \max(V_{\text{sail}}, V_{\text{oar}}) + 0.25 \cdot \min(V_{\text{sail}}, V_{\text{oar}})$$
* **Turn Rate ($\omega$):** Galleys pivot nearly in place using differential oar banks ($\omega_{\text{base}}$ remains $60\%$ active even at $V_{\text{total}} = 0$). Pure sailing hulls require forward momentum ($V_{\text{total}} > 0.15 \cdot V_{\text{max}}$) to turn effectively via rudder authority.

---

## 5. Fleet Formation & Cursor Intent State Machine
When controlling a multi-ship squadron (up to 6 ships per fleet), commands resolve through a deterministic state machine:
1. `MOVE_TO_POINT(x, y)`: Fleet holds `LINE_AHEAD` or `LINE_ABREAST` formation, throttling faster vessels to match the slowest hull unless `BREAK_FORMATION` is toggled.
2. `INTERCEPT_TARGET(shipId)`: Computes lead-pursuit vector using target's current velocity $\vec{V}_{\text{target}}$ and own wind-optimal tacking angle if sailing directly into irons.
3. `BROADSIDE_ORBIT(shipId, preferredRange)`: Steers parallel to target heading at optimal gun range, alternating port/starboard arcs.
4. `GRAPPLE_APPROACH(shipId)`: Closes relative velocity below $3.0\text{ knots}$ within $15\text{ meters}$ to trigger Spec 03 boarding.