# 🦅 Peregrine: The Momentum Doctrine
*"Static defense is a slow death."*

Peregrine is a StarCraft II Extension Mod that fundamentally changes the rules of engagement. It enforces the **Momentum Doctrine**: a set of systemic mechanics where speed equals power, stillness equals vulnerability, and map control is paramount. 

In Peregrine, mechanical execution (chore APM) is entirely systemic. Victory belongs to the player who out-drafts their opponent, curates tactical behaviors, and strikes from multiple angles.

## 🎲 Phase 1: The Pre-Game Drafts
Before the momentum begins, players engage in a three-part strategic drafting phase. This establishes the asymmetrical matchup, the unit rosters, and the territorial rules of engagement.

### 1. The Faction Draft
You don't just blindly pick your race. Factions are drafted:
*   Player 1 bans one of the three races.
*   Player 2 selects their race from the remaining two.
*   Player 1 is assigned the final remaining race.

### 2. The Roster Draft (Deckbuilding)
You do not have access to the full StarCraft II tech tree. Instead, you build a custom 6-unit "deck" for the match.
*   **The Core:** Each player automatically receives a foundational "Core" of 2 essential units ensuring basic anti-ground and anti-air capabilities (e.g., Terran always gets Marines and Medivacs).
*   **The Snake Draft:** Players alternate drafting 4 specialized units from their faction's roster in a 1-2-2-1 snake format, allowing for active counter-picking. (If your opponent drafts heavy armor early, you can immediately pivot to armor-piercing or high-mobility units).

### 3. The Advantage Draft
Finally, players draft their "Zone Buff." Map control in Peregrine projects a faction-specific zone (Zerg Creep, Protoss Power Fields, Terran Sensor Networks). The Advantage you draft is a game-changing mechanical buff your units receive *only* when fighting inside your territory. 

**Zerg (The Biomass Engine)**
*   **Feral Reclamation:** Units dying on the zone refund 30% of their cost or spawn autonomous broodlings. Trades fuel the frontline.
*   **Hyper-Metabolic Overheal:** Units regenerate rapidly on the zone, over-healing up to 150% max HP as a decaying "blood shield."
*   **Adaptive Camouflage:** Units standing perfectly still on the zone automatically burrow/cloak, turning defensive territory into a minefield.

**Protoss (The Psionic Matrix)**
*   **Quantum Relocation:** Squads can instantly teleport to any other point within the same continuous power field. 
*   **Harmonic Shielding:** Individual unit shields are disabled. The zone projects a massive, collective shield pool that absorbs damage for all friendly units inside it.
*   **Chronological Override:** Local time is sped up. Attack cooldowns, turn rates, and spell casting times are reduced by 40%.

**Terran (The Industrial War Machine)**
*   **Orbital Munitions:** The zone automatically calls down artillery strikes on spotted enemy units.
*   **Kinetic Overdrive:** Moving in a straight line builds momentum stacks, granting bonus speed and kinetic knockback damage on the next impact.
*   **Scrap Scavenging:** Enemy units killed inside the zone instantly convert into stationary, automated auto-turrets lasting 30 seconds.

**Wildcard (Draftable by any Faction)**
*   **Phase Shifting:** Units in the zone ignore collision, allowing bruisers to march directly through the enemy vanguard to crush the backline.

---

## ⚔️ Phase 2: Tactical Orchestration (The Game Loop)
Once the game loads, standard "chore APM" vanishes. You play as the general, not the foot soldier.

*   **Autonomous Macro:** You do not inject larva, drop mules, or babysit worker lines. You designate zones for expansion, and the underlying systems handle the saturation and harvesting. Your production structures only build the units you drafted, allowing you to focus entirely on logistics and frontlines.
*   **Commanders & Squads:** You do not lasso-select units and right-click enemies. Units are permanently leashed to "Commanders" on the field. 
*   **Stance Programming:** You issue high-level directives to your Commanders. If you order a Commander to execute an "Aggressive Flank," the attached squad autonomously calculates wide pathing arcs, skirts the enemy frontline, and dives the backline. If you order a "Phalanx," the units find the nearest choke point, root themselves, and brace for impact.
*   **Command Latency:** Commands take 2 seconds to execute. You cannot twitch-react or stutter-step out of bad positioning. You must anticipate where the enemy's momentum is carrying them and commit to your orders ahead of time.

## 🌍 The Battlefield Ecosystem
The map itself is a weapon. The physics and geometry of Peregrine naturally destroy the traditional SC2 "Deathball."

*   **Emergent Flanking:** Because your units are autonomous, they have distinct target priorities. When your army meets the enemy, it instantly shatters. Bruisers lock onto the frontline, assassins dive the casters, and skirmishers orbit the edges. The battle naturally fans out into multi-pronged skirmishes.
*   **The Agitation Mechanic:** Units hate standing still. If an army idles for too long, they become "Agitated," naturally drifting toward contested zones or objectives. To stop moving is to invite death. 

---

## 🎮 How to Play
Peregrine is an Extension Mod, meaning it can be played on any standard Melee Map.

1.  Open StarCraft II.
2.  Go to **Custom > Melee**.
3.  Select a map (e.g., "Site Delta").
4.  Click **"Create with Mod"** in the bottom right.
5.  Search for **"Peregrine"**.
6.  Launch the lobby!

## 🛠️ Development Setup
This project uses a **Hybrid Workflow** combining the SC2 Editor for data discovery and VS Code for mass editing and scripting.

### Prerequisites
*   StarCraft II (installed via Battle.net)
*   Visual Studio Code
*   **Recommended Extensions:**
    *   *XML* by Red Hat
    *   *SC2 Galaxy* by Talv

### Directory Structure & Workflow
*   **`Peregrine.SC2Mod/`**: The actual mod source.
    *   **`Base.SC2Data/GameData/`**: XML data files for custom behaviors, autonomous logic, and Commander auras.
    *   **`scripts/`**: Custom Galaxy scripts. `PeregrineLogic.galaxy` handles the UI for the Draft Phase and macroscopic territorial logic.
*   **`references/`**: Extracted game data for reference.

## 🤝 Contributing
We welcome pull requests that align with the **Momentum Doctrine**.
*   **Do:** Suggest autonomous behaviors, emergent pathing ideas, and draftable Territorial Advantages.
*   **Don't:** Suggest static defensive buffs, micro-intensive abilities, or mechanics that reward extreme APM.

## 📜 License
This project is an unofficial mod for StarCraft II. All assets are property of Blizzard Entertainment. Code is provided under the MIT License.