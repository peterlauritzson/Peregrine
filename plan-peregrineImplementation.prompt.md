# 🦅 Peregrine: Implementation Plan

## Overview

The Faction Draft is complete. This plan implements all remaining features in discrete, testable phases. Each phase ends with a clear verification checkpoint before the next begins.

Core philosophy: **behaviors and data do the heavy lifting**. Triggers are used only for discrete events (unit created, game start, button clicked) — never for scanning/polling units.

---

## Phase 1: Roster Draft

**What:** After the Faction Draft resolves, players snake-draft 4 units from their race's pool. Each race also has 2 hardcoded "Core" units auto-assigned.

**Core Units (non-draftable, always included):**
| Race | Core Unit 1 | Core Unit 2 |
|---|---|---|
| Terran | Marine | Medivac |
| Protoss | Stalker | Observer |
| Zerg | Zergling | Queen |

**Implementation:**

1. **Data:** Add a new `CUser id="PeregrineRosterConfig"` block in `GameData.xml` with fields for each race's draftable unit pool (up to ~8 units per race). This keeps the pool data-driven and easy to adjust.

2. **Script:** New file `scripts/RosterDraft.galaxy`:
   - `RosterDraft_Start(trigger callback)` — called from `Peregrine_OnDraftFinished`
   - Snake order: P2 picks 1 → P1 picks 2 → P2 picks 1 (1-2-2-1 format since P1 already got last pick in faction draft)
   - Dialog UI shows the available units as buttons with portraits/names; already-picked units are greyed out
   - Selections stored in `int g_roster_p1[6]` and `int g_roster_p2[6]` global arrays (unit type IDs)
   - Debug mode: if single player, auto-pick for P2 using a random valid selection

3. **Wire-up:** `Peregrine_OnDraftFinished` chains to `RosterDraft_Start(rosterCallback)`.

**Verification checkpoint:** Both players see the draft UI. After picks, `DebugLog` prints each player's 6-unit roster correctly. Single-player auto-picks work.

---

## Phase 2: Advantage Draft

**What:** After the Roster Draft, each player picks one Zone Advantage — a powerful buff that applies only while fighting inside their own territory.

**Available Advantages:**
| Race | Option A | Option B | Option C |
|---|---|---|---|
| Zerg | Feral Reclamation | Hyper-Metabolic Overheal | Adaptive Camouflage |
| Protoss | Quantum Relocation | Harmonic Shielding | Chronological Override |
| Terran | Orbital Munitions | Kinetic Overdrive | Scrap Scavenging |
| Wildcard | Phase Shifting | *(any race)* | |

**Implementation:**

1. **Script:** New file `scripts/AdvantageDraft.galaxy`:
   - `AdvantageDraft_Start(trigger callback)` — called from Roster Draft callback
   - Each player simultaneously sees their own race-specific dialog (3 buttons + Wildcard button)
   - Selection stored as `int g_advantage_p1` and `int g_advantage_p2` (constants defined for each advantage)
   - Simultaneous pick (both players lock in, dialog closes when both have chosen)
   - Debug mode: auto-pick for P2

2. **Wire-up:** Roster Draft callback chains to `AdvantageDraft_Start(advantageCallback)`. `advantageCallback` calls `Peregrine_OnAllDraftsFinished()` which fires `DayNightCycle_Start()` and the game-start setup.

**Verification checkpoint:** Both players see correct race-filtered options. Selections are logged. Wildcard appears for both races. Game starts after both pick.

---

## Phase 3: Territory Zone System

**What:** Each race projects a "home territory" from specific structures. Units inside friendly territory receive an `InZone` marker buff. This is **pure data** — no scanning triggers.

**Territory Structures:**
| Race | Structure | Notes |
|---|---|---|
| Zerg | Creep Tumor / Hatchery | Uses native creep; we attach our search to Hatchery + Tumors |
| Protoss | Pylon | Existing power field range |
| Terran | New "Sensor Relay" building | Small, cheap, buildable structure |

**Implementation:**

1. **New behavior:** `CBehaviorBuff id="PeregrineInZone"` — a marker buff with no stat changes. Duration refreshes continuously while the source search is active.

2. **Per-race search effect:** For each territory structure, add a `CEffectSearchArea` that:
   - Fires periodically (every ~0.5s via a `CBehaviorBuff` with a `CEffectPeriodic` on the structure)
   - Filters: owner = same player, unit category = not structure
   - Applies `PeregrineInZone` buff to all matched units

3. **Terran Sensor Relay:** New `CUnit` entry in `UnitData.xml` — inexpensive, no attack, small footprint, projects zone like a Pylon. Add to buildable structures for Terran in `GameData.xml`.

4. **Enemy zone detection (for future use):** A second search effect on the same structures applies `PeregrineEnemyPresent` buff to the structure itself when enemies are nearby — used later for Agitation.

**Verification checkpoint:** Place a Pylon/Hatchery/Sensor Relay on a test map. Friendly units walking into radius visibly receive `PeregrineInZone` buff (check via unit tooltip or behavior list). Buff disappears when unit leaves radius.

---

## Phase 4: Zone Advantage Buffs

**What:** Implement all 10 advantages as `CBehaviorBuff` entries. Each is only active on a unit when `PeregrineInZone` is also present (via a behavior requirement).

**Buff Designs:**
| Advantage | Effect |
|---|---|
| **Feral Reclamation** | +3 HP/sec regen while in zone |
| **Hyper-Metabolic Overheal** | Can overheal to 125% max HP while in zone |
| **Adaptive Camouflage** | Units gain cloak while in zone and not attacking |
| **Quantum Relocation** | Short-range blink active ability while in zone |
| **Harmonic Shielding** | +2 shield armor and +2 shield regen/sec while in zone |
| **Chronological Override** | +20% attack and movement speed while in zone |
| **Orbital Munitions** | Ranged units gain +2 damage while in zone |
| **Kinetic Overdrive** | +25% movement speed and +10% acceleration while in zone |
| **Scrap Scavenging** | Units killed in zone yield +5 minerals to the owner |
| **Phase Shifting** | Unit becomes briefly invulnerable on taking lethal damage (1 charge, refreshes every 60s) |

**Implementation:**

1. **XML data:** One `CBehaviorBuff` per advantage in a new `BehaviorData.xml` under `Base.SC2Data/GameData/`. Each buff uses a `CValidatorUnitBehaviorStackCount` or `CValidatorUnitCompareBehavior` requirement: *only active when `PeregrineInZone` buff is present*.

2. **Game-start trigger:** A short trigger in `PeregrineLogic.galaxy` fires once on `Peregrine_OnAllDraftsFinished`. It reads `g_advantage_p1` / `g_advantage_p2` and uses `UnitBehaviorAdd` on all current and future units belonging to each player — applying the correct always-present "container" behavior that carries the conditional advantage buff.

3. **On-unit-created trigger:** `TriggerAddEventUnitCreated` per player — when a new unit is created for that player, immediately apply their advantage container behavior. This is a discrete event trigger, not a scan.

**Verification checkpoint:** Test each advantage by entering/leaving Pylon range. Buff icon appears in zone, disappears outside. Stat changes are measurable (e.g., Kinetic Overdrive visibly speeds units).

---

## Phase 5: Roster Enforcement

**What:** Players can only build and use units from their drafted 6-unit roster. Non-roster units are blocked.

**Implementation:**

1. **Command card lockout (game start):** In `Peregrine_OnAllDraftsFinished`, iterate over all production structures for each player. Use `SyncMethodCall` / actor messages or `GameCommandSetState` to disable production buttons for non-roster units on that player's structures. This is a one-time trigger at game start, not a scan.

2. **Creation failsafe (discrete trigger):** `TriggerAddEventUnitCreated` with a filter for each non-roster unit type. If a non-roster unit is somehow created (e.g., from abilities), immediately remove it and refund cost via `PlayerModifyPropertyInt`. This is a discrete event, not a scan.

3. **Tech tree restriction:** Non-Core structures that exclusively produce non-roster units can be removed from the build menu entirely at game start via the same `GameCommandSetState` pass.

**Verification checkpoint:** Attempt to build a non-roster unit — button should be greyed out/hidden. If exploited via hotkey, unit is immediately removed and cost refunded (check mineral count).

---

## Phase 6: Command Latency

**What:** All player-issued unit orders are delayed by 2 seconds before execution.

**Implementation:**

1. In `PeregrineLogic.galaxy`, create an order queue system:
   - `TriggerAddEventUnitOrder` captures every issued order with `EventUnitOrderId()`, `EventUnit()`, and target info
   - Order is cancelled immediately (`UnitOrderIssueStop`)
   - A `TimerStart` of 2.0s is created; on expiry, the original order is re-issued via `UnitOrderIssue`
   - Queue stored as parallel arrays: `unit g_latency_unit[]`, `int g_latency_order[]`, `point g_latency_target[]`, `timer g_latency_timer[]`

2. This uses discrete event triggers (unit order issued) — no scanning.

**Verification checkpoint:** Issue a move order. Unit pauses 2 seconds, then executes. Attack orders, ability orders all delayed equally. Verify queued orders don't stack incorrectly if unit is ordered multiple times.

---

## Phase 7: Stance Programming

**What:** Players assign one of three stances to units/squads. Stance affects auto-targeting and movement behavior.

**Stances:**
| Stance | Behavior |
|---|---|
| **Aggressive** | Unit auto-acquires targets at max range, pursues fleeing enemies further |
| **Defensive** | Unit holds position, only attacks enemies that enter a short radius |
| **Hold** | Unit does not auto-attack; only attacks on explicit order |

**Implementation:**

1. **UI:** A dialog radial or right-click context menu added to the command card — 3 stance buttons, visible for all combat units. Implemented in a new `scripts/StanceProgramming.galaxy`.

2. **Data:** Three `CBehaviorBuff` entries — `PeregrineStanceAggressive`, `PeregrineStanceDefensive`, `PeregrineStanceHold` — each modifying `AcquireRange`, `ReturnRange`, and `AutoAcquireTargetFilters` via behavior XML fields.

3. **Trigger:** On stance button click (`TriggerAddEventDialogControl`), remove old stance behavior and apply new one via `UnitBehaviorAdd/Remove`. Works on a selection group by iterating `UnitGroupSelected()`.

4. **Default stance:** Defensive applied to all units on creation via the `TriggerAddEventUnitCreated` handler already established in Phase 5.

**Verification checkpoint:** Select a unit, switch stance. Aggressive unit pursues enemies past normal leash range. Hold unit stands still when attacked from range. Stance icon visible in UI.

---

## Phase 8: Commanders & Squads

**What:** Players designate a "Commander" unit per squad. Squad members auto-follow their Commander and receive shared vision.

**Implementation:**

1. **Designation:** Right-click context menu button "Set as Commander" on a unit. Stored in `unit g_squad_commander[MAX_SQUADS]` and `unitgroup g_squad_members[MAX_SQUADS]` per player. New file `scripts/Squads.galaxy`.

2. **Auto-follow:** `TriggerAddEventUnitMoved` on the Commander unit — when Commander moves more than X distance from squad centroid, issue a follow order to all squad members via `UnitOrderIssue(member, c_orderIdFollow, commander)`. Discrete event, not a scan.

3. **Shared vision:** `VisibilityRevealArea` / `UnitMakeDetectable` as appropriate for squad members around the Commander's position, updated on the same move event.

**Verification checkpoint:** Designate a Commander. Move it — squad follows. Commander dies — squad members revert to individual stance behavior. Squad UI shows Commander portrait.

---

## Phase 9: Battlefield Ecosystem

### 9A: Emergent Flanking

**What:** Units attacked from their rear arc receive a "Flanked" debuff (reduced armor, movement speed).

**Implementation — pure data:**

1. `CBehaviorBuff id="PeregrineFlanked"` — -1 armor, -15% movement speed, 3s duration.

2. On each attack effect in the game, a `CEffectSet` chains to a `CValidatorLocationInArc` check (rear 120° arc of target). If valid, applies `PeregrineFlanked` to the target. This requires patching relevant weapon `CWeapon` entries to add the effect chain — done in `EffectData.xml` / `WeaponData.xml` overrides.

3. No triggers involved — entirely effect/validator chain.

### 9B: Agitation Mechanic

**What:** Units that are repeatedly attacked without retaliating accumulate "Agitation" stacks. At max stacks, they are forced to reposition.

**Implementation:**

1. `CBehaviorBuff id="PeregrineAgitation"` — stackable (max 5), applies via the same attack effect chain as Flanking (added to `CEffectSet` on weapon hit, filtered to target that is not currently attacking).

2. At stack 5: a `CEffectApplyBehavior` triggers a `PeregrineAgitationBreak` buff that issues a random-direction move order. Since issuing movement from a buff isn't natively supported, a small **discrete trigger** watches for `PeregrineAgitationBreak` buff application via `TriggerAddEventUnitBehavior` — on detection, issues a repositioning move order to the unit. Stacks then reset.

**Verification checkpoint:** Attack a passive unit repeatedly — Agitation stacks visible. At 5 stacks, unit moves to reposition. Attacking unit that retaliates does not accumulate stacks.

---

## File Structure Summary

```
Peregrine.SC2Mod/
  scripts/
    PeregrineLogic.galaxy       ← Modified: chains all drafts, game-start setup
    RaceDraft.galaxy            ← Existing
    RosterDraft.galaxy          ← NEW (Phase 1)
    AdvantageDraft.galaxy       ← NEW (Phase 2)
    StanceProgramming.galaxy    ← NEW (Phase 7)
    Squads.galaxy               ← NEW (Phase 8)
    DayNightCycle.galaxy        ← Existing (unchanged)
    Debug.galaxy                ← Existing (unchanged)
  Base.SC2Data/GameData/
    GameData.xml                ← Modified: zone search effects, Sensor Relay, roster config
    UnitData.xml                ← Modified: Sensor Relay unit, stat tweaks
    BehaviorData.xml            ← NEW: all advantage buffs, InZone marker, stances, flanked, agitation
    EffectData.xml              ← NEW: flanking/agitation effect chains
    WeaponData.xml              ← NEW: weapon overrides to add flanking effect sets
```

---

## Dependency Order

```
Phase 1 (Roster Draft)
  └─ Phase 2 (Advantage Draft)
       └─ Phase 3 (Territory Zones)
            └─ Phase 4 (Advantage Buffs)
                 └─ Phase 5 (Roster Enforcement)
                      └─ Phase 6 (Command Latency)
                           └─ Phase 7 (Stances)
                                └─ Phase 8 (Squads)
                                     └─ Phase 9 (Battlefield Ecosystem)
```

Each phase is independently testable on a standard melee map using the existing single-player debug mode from the Faction Draft.
