# 🦅 Peregrine: The Momentum Doctrine
*"Static defense is a slow death."*

Peregrine is a StarCraft II Extension Mod that fundamentally changes the rules of engagement. It enforces the **Momentum Doctrine**: a set of physics-based mechanics where speed equals power, and stillness equals vulnerability.

In Peregrine, a "deathball" is a liability. Victory belongs to the player who maintains velocity, controls the map, and strikes from multiple angles.

## ⚡ The Philosophy: Momentum Doctrine
Peregrine is built on a simple, brutal truth: **Speed is the only armor.**

This mod strips away the safety nets of standard StarCraft. There are no safe backlines, no impenetrable turtle spots, and no "correct" composition that wins by A-moving.

### Core Concepts

*   **Velocity is Violence:** Units are designed to be lethal when moving and vulnerable when static. The faster you play, the stronger your army becomes.
*   **Death to the Deathball:** Clumping your units is a death sentence. Mechanics are tuned to reward flanking, splitting, and multi-pronged aggression.
*   **Forward Operations:** Your infrastructure should move with your army. Map control isn't just about vision—it's about supply lines and reinforcement speed.
*   **High Risk, High Reward:** We embrace "imbalance" if it promotes action. Defensive play is possible, but it requires active interception, not passive fortification.

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

### Installation for Contributors
1.  Clone this repository into your StarCraft II mods directory (e.g., `C:\Program Files (x86)\StarCraft II\Mods` or your custom development folder).
2.  Open `Peregrine.SC2Mod` in the StarCraft II Editor.

### Directory Structure & Workflow

*   **`Peregrine.SC2Mod/`**: The actual mod source.
    *   **`Base.SC2Data/GameData/`**: XML data files (Units, Effects, etc.). Add new XMLs here and the game will load them automatically.
    *   **`scripts/`**: Custom Galaxy scripts.
        *   `PeregrineLogic.galaxy`: The main entry point for custom logic.
    *   **`Triggers`**: The SC2 Editor's trigger definitions. Includes a bootstrapper (`PeregrineBoot`) to load `PeregrineLogic.galaxy`.
*   **`references/`**: Extracted game data for reference.
    *   `mods/voidmulti.sc2mod`: Tracked in git to monitor ladder balance changes.
    *   Other folders (Core, Liberty, Swarm, Void) are ignored by git but recommended for local development.

### Scripting
All custom logic should be written in `Peregrine.SC2Mod/scripts/PeregrineLogic.galaxy`.
The mod is configured to automatically include and initialize this script on Map Initialization via the `PeregrineBoot` trigger.

### Reference Data
The `references/` folder is designed to hold extracted XML and Galaxy files from the base game.
*   **VoidMulti** is committed to the repo to track balance patches.
*   To get full context (Core, Liberty, etc.), use a tool like **CASCExplorer** to extract `Base.SC2Data` from the SC2 installation into `references/mods/`.

## 🤝 Contributing
We welcome pull requests that align with the **Momentum Doctrine**.

*   **Do:** Suggest mechanics that reward active play, scouting, and splitting armies.
*   **Don't:** Suggest static defensive buffs, shield batteries, or "turtle" mechanics.

## 📜 License
This project is an unofficial mod for StarCraft II. All assets are property of Blizzard Entertainment. Code is provided under the MIT License.
