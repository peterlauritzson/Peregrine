# Peregrine -- Current State

Last updated: June 7, 2026 (session 3)

---

## What's Built

### Pre-Game Draft Chain
Wired end-to-end on game start: Race -> Roster -> Advantage -> Game.

#### 1. Race Draft (`RaceDraft.galaxy`)
- P1 bans a race, P2 picks, P1 gets the remainder.
- Starting units, workers, and resources physically swapped via `SetPlayerRace()`.
- **Live and affects the game.**

#### 2. Roster Draft (`RosterDraft.galaxy`)
- Ban phase + snake-pick UI. Each player drafts 4 units from pool of 12 (+ 2 core units auto-assigned).
- Results stored in `g_roster_p1Units[]` / `g_roster_p2Units[]`.
- **UI works -- roster is NOT enforced.** Players can still build any unit.

#### 3. Advantage Draft (`AdvantageDraft.galaxy`)
- Each player picks 1 Zone Advantage (3 race-specific + 1 wildcard).
- Result feeds into `ZoneAdvantage_Init()` which attaches auras to zone structures.

---

### Zone Advantage Aura System
- All 10 `PeregrineAura_*` behaviors defined in `BehaviorData.xml`.
- All 10 `PeregrineAuraEffect_*` area-scan effects defined in `EffectData.xml`.
- `ZoneAdvantage_Init()` attaches chosen aura to zone structures (Pylons / Hatcheries / Sensor Towers). Polling loop re-applies every 5 seconds to catch new structures.
- Likely functional, untested at scale / unbalanced.

---

### Tidal Cycle (`DayNightCycle.galaxy`)
- HUD widget bottom-left: current phase, countdown, next-phase icon, minimize button.
- 3-phase loop: **The Khala Surge** (Protoss/yellow) -> **The Swarm Tide** (Zerg/red) -> **The Iron Hour** (Terran/blue).
- Changes map lighting and plays ambient sound per phase.
- Phase duration: **10 seconds** (debug value -- `PeregrineDayNightConfig > PhaseDuration` in `GameData.xml`).
- **Cosmetic only** -- no gameplay buffs wired yet.

---

### Strategic HUD (`StrategicHUD.galaxy`)
Persistent overlay UI with three panels.

**1. Status Strip** (top-left, below resource bar)
- Single label: `[GLOBAL] STANCE: x  TARGET: x  RETREAT: x`
- Switches to `[Cmd #N]` context when a commander is selected (commander selection is a stub).

**2. Command Panel** (top-right vertical)
- STANCE section: Aggressive / Defensive / Flanking / Hold Line
- ORDERS section: Attack Zone / Reinforce Zone / Fall Back / Retreat threshold (cycles Never/20%/40%/60%)
- ECONOMY section: + Expand Base / ~ Army Comp / ^ Tech Tier Up
- All buttons write to either the active commander slot OR global player state.
- Stance/order buttons still print `(not yet wired)`.
- `+ Expand Base` is now wired for target selection: left-click world to set target, right-click to cancel.
- For Zerg players, target selection queues directed corridor expansion in `ZergEconomy_QueueExpandTowardPoint()`.

**3. Economy Info Panel** (right, below command panel)
- Live test info updates every second.
- Shows base count for all races.
- For Zerg, shows expansion planner state (`Idle`, `Active`, `Done`, or `Fail(code)`).

**Commander slot architecture** (data live, behavior stubbed)
- Up to 8 commander slots per player, indexed `player * 8 + slot`.
- `Commander_RegisterSlot()`, `Commander_ReleaseSlot()`, `Commander_FindSlot()` implemented.
- `HUD_GetActiveCommanderSlot()` is a **stub returning -1** -- all HUD actions fall through to global stance until filled in.

---

### Resource Refresh (`ResourceRefresh.galaxy`)
Prevents all mineral and gas sources from ever depleting.
- 30-second polling loop.
- Tops up all mineral field variants, neutral geysers, and owned gas buildings (Refinery / Assimilator / Extractor and Rich variants).

---

### Economy Systems (new this session)
Fully automated economy, always running. 8-second tick dispatches to race-specific handler.

**Terran** (`TerranEconomy.galaxy`)
- `PeregrineAutoTrain_SCV` behavior on CC/OC/PF auto-trains SCVs via data (validator-gated at <16 local workers).
- `TerranEconomy_OnSCVCreated` immediately issues `Harvest_Gather` to the nearest mineral patch when a new SCV is created.
- `TerranEconomy_BuildRefineries` periodically scans geysers near each town hall and orders a hidden `PeregrineGhostBuilder` to build missing Refineries.
- Gas-worker saturation and smart base transfer are still TODO.

**Protoss** (`ProtossEconomy.galaxy`)
- `ProtossEconomy_TrainProbes` -- produces probes from idle Nexus when unanchored patches exist and no idle probes are waiting.
- `ProtossEconomy_AnchorIdleProbes` -- sends idle probes to the first unanchored mineral patch.
- Pylon-warp mechanic is TODO.

**Zerg** (`ZergEconomy.galaxy`)
- No workers. Income injected directly per tick.
- Hive is enforced as sink:
	- startup bootstrap converts one Hatchery to Hive if needed,
	- Hive morph attempts are blocked for managed players.
- Larva output is suppressed for managed Zerg players.

**Structure roles (split):**
- `PeregrineResourceHighway` = **pipeline only**. Propagates connectivity step markers outward from the Hive. Does NOT mine.
- `Hatchery` / `Lair` = **mining nodes**. Inject mineral/gas income from patches within radius, but only if connected to the pipeline.
- `Hive` = **sink + miner**. Always mines its local radius unconditionally; is the graph root (step 0).

**Connectivity system:**
- Behavior markers `Connected/Disconnected` + `PeregrineCreepNode_Step0..12` on highway nodes.
- Propagation pulse every 0.25s: Hive/Lair always step 0; highways earn their step through the plan/apply pass.
- Highways display green (connected) or red (disconnected) team color.

**Income model:**
- Hive: mines all patches within `c_zergSinkHarvestRadius` unconditionally each tick.
- Hatchery/Lair: mines their local patch radius only if a connected highway or the Hive is within `c_zergNodeLinkRadius` (15 units).
- Hive passive floor income injected each tick regardless of network state.
- Gas income: same connectivity gate, scans Extractors near each mining building.

**Visual feedback:**
- Mineral/geyser patches claimed by an active mining building glow with `NeuralParasiteEffect` (purple tendrils). Refreshed every economy tick. Dark = unclaimed or disconnected.

**Inferred base sites** (`ExpansionSites.galaxy`):
- Startup scan clusters mineral patches and geysers into likely base locations.
- `RefinePositions` pass: if a real town hall is present → snap to its exact position. Otherwise try 8 directions at 7 units from the centroid and pick the candidate with most clearance from the mineral line (the open "town-hall pocket").
- `DebugMarkSites`: places a `BeaconExpand` marker at every inferred site on game start for visual verification. **Remove this call once sites look correct.**

**Directed corridor expansion planner:**
- `FindAnchorToward`: explicit per-type queries (`"Hive"`, `"Lair"`, `"Hatchery"`, `PeregrineResourceHighway`) — nearest structure of any type wins. Highways are always considered, never skipped.
- `TryPlaceExpansionColony` rules:
	1. **Never block a base site**: highway candidates within `c_zergExpandBaseBlockRadius` (7 units) of an inferred site are skipped.
	2. **Auto-colonise**: after placing a highway, any open inferred base site within `c_zergHatcherySnapRadius` (14 units) automatically gets a Hatchery.
	3. **Completion**: once the nearest anchor is within `c_zergExpandTargetReach` (9 units) of goal AND a Hatchery exists there → Done. Fallback: if Hatchery not yet placed, try directly (waiting for minerals if needed).
- All placement uses `c_unitCreateIgnorePlacement`; no SC2 footprint snapping.
- Failure states: `FailNoAnchor`, `FailBlocked` (8 fails), `FailBudget` (40 builds).
- Queue API: `ZergEconomy_QueueExpandTowardPoint(player, point)` — snaps click to nearest inferred base site.

**`PeregrineResourceHighway` unit data:**
- `parent="Hatchery"` — generates creep from bare terrain.
- `Footprint = Footprint2x2Contour`, `PlacementFootprint = Footprint2x2` — small, unrestricted.

**EconomyManager** (`EconomyManager.galaxy`)
- Includes all three race files. Single dispatcher; 8-second loop.
- Wired into `PeregrineLogic.galaxy` via `EconomyManager_Init()` call in `Peregrine_OnAdvantageFinished`.
- `EconomyManager_TickPlayer(player, race)` can also be called directly for on-demand triggers.

---

## What's Missing / TODO

### High priority

| Item | Notes |
|---|---|
| **Harvest_Gather abilcmd** | String used in Terran/Protoss gather orders. If workers sit idle in-game, this string is wrong -- check first. |
| **Roster enforcement** | `g_roster_p1Units[]` stored but never consulted. Players can build any unit. Needs unit-creation event trigger or production gate. |
| **Tidal Cycle gameplay effects** | Cycle is cosmetic. Needs phase-change callback in `DayNightCycle_Loop()` and per-phase buff/debuff per favored race. |
| **Tidal Cycle phase duration** | 10s is debug. Change `PhaseDuration` in `GameData.xml` to 120-180s for real play. |

### Medium priority

| Item | Notes |
|---|---|
| **Zerg colony buffering/transfer model** | Connectivity + directed expansion are implemented. Missing: explicit per-node buffer storage, transfer-rate caps, and death-drop payout based on stored buffer. See `design/economy-design.md`. |
| **Pylon-warp for Protoss probes** | On probe production, teleport to nearest Pylon before issuing anchor order. |
| **Terran gas saturation** | Auto-assign 3 SCVs per Refinery. Only mineral side is automated now. |
| **Commander unit types** | No commander unit type defined in XML. `HUD_GetActiveCommanderSlot()` is a stub. Whole commander/squad system waits on this. |
| **Economy panel live data depth** | Panel now shows live base count + Zerg planner status. Still missing broader economy telemetry (income rates, saturation, per-race detail). |
| **Player-triggered economy (non-Zerg)** | Zerg expansion trigger path is wired via HUD + click-targeting. Protoss/Terran economy direction controls remain unwired. |

### Low priority / future

| Item | Notes |
|---|---|
| **Commander / Squad system** | No squads, no stance execution, no command latency. General gameplay loop fully unimplemented beyond data structures and HUD buttons. |
| **Agitation mechanic** | Units hate standing still -- not implemented. |
| **Tidal Cycle assets** | `IconDay/Dusk/Night` fields in XML may point to empty strings. |
| **Army Comp targeting** | `~ Army Comp` button cycles a label but no production ratio logic exists. |
| **Expand logic (non-Zerg)** | Zerg directed expansion is implemented. Terran/Protoss expand automation via HUD is still pending. |