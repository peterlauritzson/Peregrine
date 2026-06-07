# Peregrine -- Economy Design

Last updated: June 7, 2026 (session 3)

---

## Global Rules (all races)

- **Mineral patches and geysers never deplete.** They are auto-topped up every N seconds (exact interval TBD, tuned for desired income rate). This removes the need to handle worker reassignment when patches run dry, and keeps automation logic simple for v1.
- **Killed economy units/structures drop their buffered resources** as a pickup on death. This makes raiding have a concrete payoff beyond just cutting income -- you can steal stored resources.
- Resources are measured in **minerals** and **gas** as standard, but the *source* of those resources is completely race-specific.

---

## Terran -- Industrial Logistics

Standard SC2 harvesting. SCVs mine minerals and gas manually.

- Workers are produced normally and assigned to bases
- Saturation target: 16 mineral workers + 3 gas workers per base
- Automation handles: keeping workers busy, not over-saturating, basic transfer to new bases on expansion
- Player handles: expansion timing, gas vs mineral priority
- Harassment: kill SCVs directly, raid mineral lines

**Rationale:** Terran is the grounded, familiar race. Keeping their economy "normal" makes them the baseline and lets the other two feel more alien by contrast.

---

## Protoss -- Resonance Anchors

Probes are placed at mineral patches as permanent stationary anchors. Once placed, they mine that patch indefinitely without any player input. Assimilators self-harvest gas automatically when built.

- Probes do not move after placement -- they are anchors, not workers
- **Only one Nexus is needed** -- the Nexus is a production building, not an economy building. Mining has no Nexus dependency
- **Probes warp in near Pylons** -- when a probe is produced it instantly relocates to the nearest Pylon to its destination patch, rather than walking across the map from the Nexus. This means Pylon placement has logistical significance beyond power fields
- System auto-produces replacement probes and auto-anchors them to vacant patches when a probe dies (optional: player can disable auto-replace)
- Probes have low HP and are fully visible -- easy to kill, no detection needed
- Harassment: kill individual probes to knock specific patches offline. Requires physically attacking each one

**Rationale:** Fragile, precise, glass network. Raiding a Protoss economy is surgical -- you pick off nodes one by one. Recovery is slow because each replacement has to be produced and placed.

---

## Zerg -- Resource Highway Pipeline

A chain of Resource Highways extends from the main Hive outward across the map. Each highway node auto-mines minerals and gas within its radius. Resources only flow back to the Hive through an **unbroken chain** of connected nodes.

### Structure roles (locked for implementation planning)
- **Hive is the only economy sink.** All node transfer routes terminate at Hive. Hive also mines its own local radius unconditionally — the pipeline does not need to reach back to itself.
- **Zerg starts with a Hive** (spawn/setup details deferred; to be wired later).
- **No Lair -> Hive upgrade path** (to be disabled in data when the race package is finalized).
- **Resource Highway = pipeline node only.** Does not mine. Propagates connectivity outward from the Hive. Compact creep footprint (2x2), placed anywhere passable with `c_unitCreateIgnorePlacement`.
- **Hatchery / Lair = mining node.** Injects mineral/gas income from patches within its harvest radius, but only when the pipeline reaches it (a connected Highway or the Hive itself is within link range).
- **Working direction:** Resource Highways spread creep and mark pipeline depth. Hatcheries sit at base sites and mine. The player sees a clear visual language: highways = veins, Hatcheries = organs.

### Node behavior
Each node has:
- A **local buffer** (e.g. 200 minerals / 50 gas max storage)
- A **mining rate** -- fills buffer from patches in radius
- A **transfer rate** -- pushes buffer toward the Hive through the nearest connected neighbor

### Chain integrity
- While the chain is intact, buffers stay near-empty as resources flow through to the Hive
- **Cut link:** nodes on the Hive-side of the cut drain their buffers through to the Hive naturally (no special logic -- buffers just deplete). Nodes on the far side keep mining into their buffer until full, then stop
- **Repair link:** no special burst logic is required. Reconnected nodes resume transfer, and pipeline throughput is tuned higher than mining input so backlog drains quickly.

### Network topology
- Players can build **branching/redundant paths** -- more expensive but resilient to single-point cuts
- Straight chains are efficient but fragile: one kill can cut off multiple colonies at once
- The Hive itself provides a small base income regardless of network state (Zerg is never completely dead)

With a single guaranteed sink, all connectivity checks reduce to: "is this node connected to the player Hive?" This should simplify graph bookkeeping and reconnect burst handling.

### Directed corridor expansion planner (v1)
- Player intent is directional ("expand toward X"); colony placement is automated.
- Planner runs in bounded steps (no global optimality requirement).
- Each step picks the nearest frontier structure (Hive, Lair, Hatchery, or any Resource Highway) to the goal.
- **Base-site protection rule**: highway candidates are rejected if they would land within 7 units of any inferred base site. Highways must never block where a Hatchery needs to go.
- **Auto-colonise rule**: after placing a highway, any open inferred base site within one step distance (14 units) automatically receives a Hatchery. This fires both mid-corridor (passing by sites) and at the destination.
- **Inferred base sites** are computed at startup by `ExpansionSites.galaxy`:
	- Cluster mineral patches within 14 units of each other.
	- Keep clusters with ≥6 patches or ≥4 patches + a geyser.
	- Refine centroid to the "town-hall pocket": try 8 directions at 7 units, pick the candidate with most clearance from the nearest patch. If a real town hall exists nearby, snap to it directly.
- Corridor completion: anchor within 9 units of goal AND a Hatchery present there → Done. Fallback: try to place Hatchery directly if mineral check passes.
- Hard stop conditions: no Hive sink, no anchor, blocked after 8 fails, build budget (40) exhausted.

### Current implementation snapshot (June 7, session 3)
- Implemented and working:
	- Hive sink bootstrap + Hive morph blocking.
	- Pipeline connectivity propagation: step markers on highways, 0.25s pulse. Hive/Lair are always step 0; highways earn their step through the plan/apply pass.
	- **Role split**: Resource Highways = pipeline only. Hatcheries/Lairs = mining. Hive = unconditional mining + root.
	- **Income model**: Hive mines unconditionally. Hatchery/Lair mines if a connected highway (or Hive) is within 15 units. Gas follows the same gate.
	- **Expansion planner** (working and tested):
		- Anchor = nearest Hive/Lair/Hatchery/Highway to goal (explicit per-type queries).
		- Highways never placed on inferred base sites (7-unit block radius).
		- Any open base site within 14 units of a placed highway auto-receives a Hatchery.
		- Final completion: Hatchery at goal → Done (with mineral-wait fallback).
		- `c_unitCreateIgnorePlacement` throughout — no footprint snapping.
	- **Inferred base sites** (`ExpansionSites.galaxy`): mineral cluster + geyser heuristic, town-hall pocket refinement (8-direction scan for most clearance from mineral line), debug beacons on startup.
	- **Mineral link visuals**: `NeuralParasiteEffect` behavior applied to all claimed patches every economy tick. Glowing = actively mined + pipeline connected. Dark = unclaimed or disconnected.
	- **Pipeline depth visuals**: highways colored green (connected) or red (disconnected) via team color index.
	- HUD wiring: `+ Expand Base` → world click → `ZergEconomy_QueueExpandTowardPoint`.
- Deferred intentionally:
	- Explicit per-node buffer accounting (highway stores resources in transit).
	- Transfer-rate caps and stored-resource death-drop.
	- Non-Zerg expansion automation through the same HUD flow.
	- Remove `ExpansionSites_DebugMarkSites` calls once base-site positions are verified.

### Harassment
- Attack the chain at a bottleneck node to cut off everything beyond it
- Killed highway nodes drop their buffered resources as pickups -- attacker can loot the buffer
- No detection required: highway nodes are visible structures

### Key design decisions (locked)
- Resources drop on highway node death (lootable)
- No explicit reconnect burst mechanic; fast propagation + higher transfer throughput handles recovery
- Directed corridor is primary expansion mode (target-driven, bounded fallback)
- Redundant paths allowed
- Hive has small passive base income (~20% of full income) as a floor
- Hive is the only sink; chain validity is evaluated against Hive connectivity only
- Resource Highway is the dedicated colony node chassis

---

## What is NOT designed yet

- Exact buffer sizes, mining rates, transfer rates (requires playtesting)
- Whether auto-replace for Protoss probes is on by default or opt-in
- Exact top-up interval for patch refresh
- How the Zerg chain "discovers" it is disconnected (polling loop vs. event-based)
- Gas handling specifics for Zerg (does a geyser colony work the same as a mineral colony?)
