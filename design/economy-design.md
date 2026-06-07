# Peregrine -- Economy Design

Last updated: June 7, 2026 (session 2)

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
- **Hive is the only economy sink.** All node transfer routes terminate at Hive.
- **Zerg starts with a Hive** (spawn/setup details deferred; to be wired later).
- **No Lair -> Hive upgrade path** (to be disabled in data when the race package is finalized).
- **Resource Highway is the dedicated node chassis.** It is a custom Zerg economy structure (separate from normal combat/economy buildings).
- **Working direction:** Resource Highways stay visible and spread creep in a compact footprint while acting as pipeline nodes.

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
- Each step picks the best connected frontier node (Hive or connected highway node) toward target.
- Candidate placements are sampled in a forward cone with side-detour angles.
- Candidate validity checks: passable terrain, pathing-connected to anchor, spacing from existing nodes.
- If no candidate is valid, failure count increases and heading bias rotates to search alternate detours.
- Hard stop conditions: no Hive sink, no connected anchor, blocked after max failures, unreachable target, build budget exhausted.
- Completion condition: connected frontier reaches target radius.
- Fast connectivity propagation (one edge per pulse at high pulse rate) keeps the corridor responsive without explicit flush mechanics.

### Current implementation snapshot (June 7, session 2)
- Implemented and working:
	- Hive sink bootstrap + Hive morph blocking.
	- Resource-Highway connectivity propagation with step markers.
	- Connected-only mineral/gas income.
	- One-to-one nearest pairing of mineral patches/extractors to colony nodes.
	- **Directed corridor planner** — correct and tested:
		- Anchor lookup queries each unit type explicitly (`"Hive"`, `"Lair"`, `"Hatchery"`, `PeregrineResourceHighway`) so the nearest structure of any relevant type is always found, including previously placed highways.
		- Step placement uses `c_unitCreateIgnorePlacement` — SC2 footprint snapping is bypassed entirely.
		- `PeregrineResourceHighway` parent is `Hatchery`: generates creep from bare terrain, 2x2 footprint, no placement restriction.
		- Corridor terminates cleanly when the leading anchor reaches within 9 units of the goal (no early jump-to-goal).
		- Goal is the inferred base site from `ExpansionSites_SnapTargetPoint`; beacon stays at raw click.
	- HUD test path: `+ Expand Base` → world click → queue planner.
- Deferred intentionally:
	- Explicit per-node buffer accounting.
	- Transfer-rate caps and stored-resource drop scaling.
	- Non-Zerg expansion automation through the same HUD flow.

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
