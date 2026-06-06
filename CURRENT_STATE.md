# Peregrine — Current State

Last updated: June 6, 2026

---

## ✅ What's Built

### Pre-Game Draft Chain
The three draft phases are wired together in sequence and run end-to-end on game start (`Peregrine_Init` → Race → Roster → Advantage → Game).

#### 1. Race Draft (`RaceDraft.galaxy`)
- P1 bans a race, P2 picks from the remaining two, P1 gets the remainder.
- Starting units, workers, and resources are physically swapped out via `SetPlayerRace()`.
- **Live and affects the game.**

#### 2. Roster Draft (`RosterDraft.galaxy`)
- Ban phase + snake-pick phase UI. Each player drafts 4 units from their race's pool of 12 (+ 2 auto-assigned core units).
- Results are stored in `g_roster_p1Units[]` / `g_roster_p2Units[]`.
- **UI works, but the drafted roster is not enforced** — the stored picks are never used to restrict what players can actually build.

#### 3. Advantage Draft (`AdvantageDraft.galaxy`)
- Each player simultaneously picks 1 Zone Advantage: 3 race-specific options + 1 wildcard (Phase Shifting).
- Result feeds directly into `ZoneAdvantage_Init()` after the draft completes.

---

### Zone Advantage Aura System
- All 10 `PeregrineAura_*` behaviors are defined in `BehaviorData.xml`.
- All 10 `PeregrineAuraEffect_*` area-scan effects are defined in `EffectData.xml`.
- After the draft, `ZoneAdvantage_Init()` attaches the chosen aura to the player's zone structures (Pylons / Hatcheries / Sensor Towers). A polling loop re-scans and re-applies every 5 seconds (handles new structures built mid-game).
- **Likely functional**, but untested at scale / unbalanced.

| # | Advantage | Race | Effect |
|---|---|---|---|
| 1 | Feral Reclamation | Zerg | +4 HP/sec regen for allies in zone |
| 2 | Hyper-Metabolic Overheal | Zerg | +100 Max HP + fast regen in zone |
| 3 | Adaptive Camouflage | Zerg | +3 life armor in zone |
| 4 | Quantum Relocation | Protoss | +30% move speed in zone |
| 5 | Harmonic Shielding | Protoss | +4 shield armor in zone |
| 6 | Chronological Override | Protoss | +40% attack speed, +20% move speed |
| 7 | Orbital Munitions | Terran | Auto-damages enemies in zone every 1s |
| 8 | Kinetic Overdrive | Terran | +25% move speed in zone |
| 9 | Scrap Scavenging | Terran | +2 life armor in zone |
| 10 | Phase Shifting | Wildcard | +20% speed, +1 all armor in zone |

---

### Tidal Cycle (`DayNightCycle.galaxy`)
- HUD widget (bottom-left) showing current phase name, countdown timer, next-phase icon, and a minimize button.
- Cycles through 3 phases on a loop: **The Khala Surge** → **The Swarm Tide** → **The Iron Hour**.
- Each phase transition changes the map lighting and plays an ambient sound.
- Phase duration: **10 seconds** (debug value — configured in `GameData.xml` under `PeregrineDayNightConfig > PhaseDuration`).
- **Pure presentation only** — no gameplay effects are tied to the phase yet.

---

## ❌ What's Missing

### Roster Enforcement *(High priority)*
The roster draft picks in `g_roster_p1Units[]` are stored but never consulted. Players can build any unit from the full tech tree regardless of what they drafted. The system needs a production hook or unit-kill trigger to enforce the chosen 6-unit roster.

### Tidal Cycle Gameplay Effects *(High priority)*
The cycle is cosmetic. Nothing changes gameplay-wise when a phase shifts. The intended design (one race gets a buff per phase) has no implementation. Needs:
- A callback or hook in `DayNightCycle_RunPhase()` to fire per-phase logic.
- Per-phase buff application/removal for the favored faction's units.

### Tidal Cycle Phase Duration
10s is a debug value. Real matches need something in the range of **120–180 seconds**. Change `PhaseDuration` in `GameData.xml`.

### Autonomous Macro
Workers exist as normal SC2 workers. The README describes a system where players designate expansion zones and the mod handles harvesting — this layer does not exist.

### Commander / Squad System
No squads, no commander units, no stance directives, no command latency. The "play as a general" core loop described in the README is entirely unimplemented.

### Agitation Mechanic
"Units hate standing still" — not implemented. No idle-detection or drift behavior exists.

### Tidal Cycle Icons / Assets
The HUD references `IconDay`, `IconDusk`, `IconNight` string fields from `PeregrineDayNightConfig` in XML. Whether these point to real assets or empty strings should be verified.

---

## Suggested Next Steps

1. **Bump Tidal Cycle duration** to something playable (quick config change in XML).
2. **Wire Tidal Cycle buffs** — add a phase-change callback in `DayNightCycle_Loop()` that applies a race-specific buff to the favored faction. Self-contained and high-visibility impact.
3. **Enforce the Roster Draft** — add a unit-creation event trigger that removes any unit not in the player's drafted roster, or gates production buildings.
