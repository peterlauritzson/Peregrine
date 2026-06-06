# Peregrine -- Economy Design

Last updated: June 6, 2026

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

## Zerg -- Creep Colony Pipeline

A chain of Creep Colonies extends from the main Hive outward across the map. Each colony auto-mines minerals and gas within its radius. Resources only flow back to the Hive through an **unbroken chain** of connected colonies.

### Node behavior
Each colony has:
- A **local buffer** (e.g. 200 minerals / 50 gas max storage)
- A **mining rate** -- fills buffer from patches in radius
- A **transfer rate** -- pushes buffer toward the Hive through the nearest connected neighbor

### Chain integrity
- While the chain is intact, buffers stay near-empty as resources flow through to the Hive
- **Cut link:** colonies on the Hive-side of the cut drain their buffers through to the Hive naturally (no special logic -- buffers just deplete). Colonies on the far side keep mining into their buffer until full, then stop
- **Repair link:** isolated colonies immediately start flushing their accumulated buffer toward the Hive -- producing a **burst of income on reconnection**. This rewards fighting to reclaim severed nodes

### Network topology
- Players can build **branching/redundant paths** -- more expensive but resilient to single-point cuts
- Straight chains are efficient but fragile: one kill can cut off multiple colonies at once
- The Hive itself provides a small base income regardless of network state (Zerg is never completely dead)

### Harassment
- Attack the chain at a bottleneck node to cut off everything beyond it
- Killed colonies drop their buffered resources as pickups -- attacker can loot the buffer
- No detection required: colonies are visible structures

### Key design decisions (locked)
- Resources drop on colony death (lootable)
- Burst income on chain repair (strategic recapture incentive)
- Redundant paths allowed
- Hive has small passive base income (~20% of full income) as a floor

---

## What is NOT designed yet

- Exact buffer sizes, mining rates, transfer rates (requires playtesting)
- Whether auto-replace for Protoss probes is on by default or opt-in
- Exact top-up interval for patch refresh
- How the Zerg chain "discovers" it is disconnected (polling loop vs. event-based)
- Gas handling specifics for Zerg (does a geyser colony work the same as a mineral colony?)
