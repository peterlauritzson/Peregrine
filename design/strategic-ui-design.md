# Peregrine — Strategic UI & Automation Design

*The goal: eliminate mechanical chore APM entirely, leaving only decisions that require strategic judgment.*

---

## The Core Question: What Should Be Automated?

A useful filter: **"Is the player deciding, or just executing?"**

- If the answer is *just executing* — automate it.
- If the answer is *deciding* — it stays a player input.

---

## Automation: What the System Handles

These should run silently in the background with zero player interaction required.

### Economy
| Action | Why Automate |
|---|---|
| Worker mineral/gas harvesting | No decision involved — optimal saturation is always correct |
| Worker production | Always produce workers up to saturation; no player judgment needed |
| Supply building construction | Pure upkeep — automatic response to supply cap proximity |
| Expansion construction (once flagged) | *Triggering* the expansion is a player decision; actually placing and saturating it is not |
| Mule/Chrono/Inject equivalents | These are chore reminders, not decisions |

### Combat Micro
| Action | Why Automate |
|---|---|
| Unit attack-move execution | Player issues a directional order; units pathfind and engage autonomously |
| Retreating wounded units | Tactical reflex, not strategy — auto-pull units below ~20% HP |
| Targeting priority within a fight | Units follow preset priority rules (anti-air > casters > front-line) |
| Spreading vs. splash | Units auto-spread when near AoE sources they can detect |

### Logistics
| Action | Why Automate |
|---|---|
| Unit production queue | Once player sets a "stance" or army composition target, production is continuous |
| Sending reinforcements toward active battles | Units produced mid-fight auto-walk toward the active frontline marker |

---

## Player Decisions: What Stays Manual

These are the inputs that actually constitute strategy. All of them require judgment, timing, or information asymmetry.

### Economy
| Decision | Why It Stays Manual |
|---|---|
| **Declare an expansion** | *Where* and *when* to expand is a strategic commitment with risk |
| **Gas vs. mineral focus** | Affects tech path and army composition — a meaningful tradeoff |
| **Tech path selection** | Which upgrade tier to pursue first |

### Army
| Decision | Why It Stays Manual |
|---|---|
| **Set Army Stance** (Aggressive / Defensive / Flanking / Holding) | The *character* of how your army behaves |
| **Designate Attack Vector** (click a target zone on the map) | Where to apply pressure — the core strategic action |
| **Set Retreat Threshold** (slider: Never / 40% / 60% / Always) | How aggressive your units fight before pulling back |
| **Activate a Commander Ability** (special cooldown skills per drafted unit type) | High-stakes decisions with cooldowns |

### Doctrine
| Decision | Why It Stays Manual |
|---|---|
| **Army Composition Target** (set ratios: % frontline / % ranged / % support) | Defines what gets produced — production then auto-executes to that target |
| **Zone Priority** (which zone to reinforce / contest) | Determines where autonomous reinforcements flow |
| **Offensive Timing** (when to commit to a major push) | This is the core skill expression — recognizing the right moment |

---

## UI Design

### The Problem with the SC2 Default HUD

The native SC2 UI is designed for high-APM unit micro:
- The command card (bottom center) shows individual unit orders
- The minimap and unit panel are fine and reusable
- The resource bar (top) works fine

The things we *cannot* easily change in a mod (without custom layouts, which require `.SC2Layout` XML files in the UI folder):
- The command card itself is hard to fully rework
- The unit info panel at the bottom is tied to the selected unit

The things we *can* do freely via Galaxy `DialogCreate`:
- Overlay panels at any anchor point, any size
- Buttons, labels, images, progress bars, sliders (limited — via workaround)
- Per-player visibility

**Verdict:** Keep the default resource bar and minimap. Replace or supplement the command card interaction with a persistent overlay panel. Use a hidden "Command Unit" as the anchor for the SC2 command card buttons (so the card feels native when selected).

---

### The Command Unit Approach

Spawn one invisible, indestructible unit per player called the **Strategic Command Post** (or "HQ"). It:
- Is permanently selected (or re-selected when nothing else is selected)
- Has a custom command card defined in XML with Peregrine's strategic buttons
- Clicking a button on it fires a Galaxy trigger

This is how many campaign maps implement "global abilities" — it's a well-worn pattern in SC2 modding.

The HQ command card would have 3 rows of buttons (the standard SC2 card is 4×3):

```
Row 1 — Army Stance
[ Aggressive ]  [ Defensive ]  [ Flanking ]  [ Hold Position ]

Row 2 — Operational Orders  
[ Attack Zone ]  [ Reinforce Zone ]  [ Fall Back ]  [ (empty) ]

Row 3 — Strategic
[ Declare Expansion ]  [ Shift Army Comp ]  [ Tech Tier Up ]  [ (empty) ]
```

When a real unit is selected, its normal command card shows instead (standard SC2 behavior). When the player clicks empty space or presses Escape, the HQ auto-reselects.

---

### Persistent HUD Overlays (Dialog-based)

Supplement the command card with 2-3 small persistent panels built with `DialogCreate`. These stay on screen at all times.

#### Panel 1 — Army Status Bar (Top-Left, below resource bar)
A compact single-row strip showing:
```
  Stance: [AGGRESSIVE]   Vector: [Enemy Natural]   Threshold: [40%]   Supply: 34/66
```
- Text labels updated by trigger whenever a stance/vector changes.
- Gives instant readability of your current doctrine without selecting anything.

#### Panel 2 — Tidal Cycle Widget (Bottom-Left — already exists)
Already implemented. Keep as-is. Phase name + countdown + next phase icon.

#### Panel 3 — Economy Overview (Bottom-Right, above supply)
A small panel, maybe 200×80px:
```
  Bases: 1 active  |  Workers: 24/32 (75%)
  [ + Expand ]   (lights up when you can afford it)
```
The `[ + Expand ]` button is a shortcut — clicking it puts you in "place expansion" mode (or simply fires the order to build at the next safe expansion location automatically).

---

### Handling the Side Panel Fantasy (C&C-Style)

A full side panel replacing the SC2 bottom bar would require editing `.SC2Layout` files — these are XML UI layout definitions that ship with the game. It *is* possible to override them in a mod by placing a modified layout file at the right path in the mod's UI folder. This is advanced and fragile (breaks across patches), but not impossible.

**Recommendation: don't pursue this now.** The Command Unit + Dialog overlay approach gives ~80% of the benefit with ~10% of the complexity. Revisit if the overlay approach feels cluttered after playtesting.

---

## Interaction Flow Example (Full Turn of Play)

1. Game starts. Your **HQ unit** is auto-selected. Command card shows your 3 rows of strategic buttons.
2. You click **"Aggressive"** stance. Your army begins advancing toward contested zones autonomously.
3. You click **"Attack Zone"** and click the enemy's natural expansion on the minimap. Your army receives a target vector. Units begin routing toward it.
4. You watch the fight. The system handles micro. You notice your army is taking heavy losses.
5. You click **"Fall Back"**. Units retreat. The 2-second command latency means you need to anticipate this — not twitch-react.
6. The Tidal Cycle ticks over to **The Khala Surge** (Protoss phase). If you're Protoss, your passive shield regen kicks in — good timing to re-engage.
7. You see you can afford a second base. You click **"+ Expand"** in the economy panel. A base starts construction automatically at the next viable location.
8. You adjust your **Army Comp target** to 60% ranged, 40% frontline — production automatically shifts to fill that ratio.

---

## What Still Needs Design Work

- **The Flanking stance specifically** — needs a definition of how "autonomous flanking" is calculated (does it always try to hit the same target from a perpendicular angle? Does the player paint an arc?)
- **Commander Abilities** — each drafted unit type could grant 1 special cooldown ability to the player. Needs a design pass per unit.
- **The "Declare Expansion" flow** — does the system auto-pick the safest open expansion? Or does the player click a location? Auto-pick is simpler; player-click preserves more strategy.
- **Army Comp target UI** — a simple 3-button toggle (Frontline-heavy / Balanced / Ranged-heavy) is more practical than a slider given Galaxy's UI limits.
