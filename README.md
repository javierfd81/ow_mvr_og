# OG mvr — OG Mercy VS Rein

An **Overwatch 2 Workshop** game mode. Team 1 flies in as Reinhardt and hunts Team 2's Mercys, who keep full Guardian Angel mobility and can turn invisible. A revival of the classic **Mercy VS Rein** experience:

> For those who enjoy the original Mercy VS Rein, this is for you! No silenced souls. No broken maps. Flying Reins and Mercy GA galore!

The entire mode lives in a single file, [mvr_og.code](mvr_og.code) — the export produced by the Overwatch Workshop editor's **Copy Code** feature.

## How to play

Asymmetric hunt: **Reinhardts** (Team 1) chase **Mercys** (Team 2) across a rotating map pool. Reins get flight-style mobility, huge health, and Earthshatter; Mercys get supercharged movement, Guardian Angel, and a resurrect on a long, map-scaled cooldown. Supported game modes include **Elimination, Skirmish, Team Deathmatch, and Capture the Flag**, each with its own tuned map pool. Default lobby is up to **2 Reins vs 10 Mercys**.

## Importing the mode

This repository is the source of truth for the mode's code. To load it into Overwatch:

1. Copy the full contents of [mvr_og.code](mvr_og.code).
2. In Overwatch: **Play → Game Browser → Create → Settings → Import** (the reverse of the editor's Copy Code).
3. Paste and confirm, then start a Workshop lobby.

There is no build step or test suite — the file *is* the mode. Editing means changing the DSL text (or round-tripping through the in-game Workshop editor) and re-importing to try it out.

## Features

**Mercy**
- Full-speed Guardian Angel with custom recharge; can sacrifice health for extra GA charge.
- Resurrect on a custom cooldown that scales with map size; booping a Rein with melee shortens it, and rez can be cancelled mid-cast.
- Earn temporary invisibility by booping; back-stab kills on Rein can trigger a resurrect.

**Reinhardt**
- Massive health pool, ready Earthshatter, and enhanced projectile/movement.
- Super Jump (Baptiste-crouch input) and experimental multi-jump mobility.
- Heal-per-elimination, respawn-on-environmental-death, and sacrifice-health-for-ult-charge options.

**Match & quality-of-life**
- Round lifecycle with automatic Rein swapping via a "Seeker Priority List", first-person/third-person perspective toggle, and HUD tips.
- Extensive **per-map fixes**: out-of-map ability restrictions, teleporters, invisible walls/ceilings, and boop-out-of-stuck-spot handling for specific maps.
- AFK-owner vote-kick, server-load display, and other host/mod utilities.

## Customization

Two layers of tuning:

- **Hero stat scalars** live in the `settings` block of [mvr_og.code](mvr_og.code) under `heroes.General` (e.g. Mercy movement speed, Rein health, cooldown scalars).
- **In-lobby toggles** are exposed as Workshop Settings (the `workshop` block) — e.g. GA recharge rate, Super Jump cooldown, multi-jump count, and switches for the heal-per-elim / sacrifice-health / respawn-on-death mechanics. These can be adjusted from the lobby without editing code.

## Repository layout

| File | Purpose |
| --- | --- |
| [mvr_og.code](mvr_og.code) | The complete game mode (Workshop DSL export). |
| [CLAUDE.md](CLAUDE.md) | Guidance for working on the code with Claude Code, including DSL conventions and file structure. |

## Credits

- Credits to all OW players for the reverted changes.
- Original code by **W77BP**, under game mode author **VOZ**.
