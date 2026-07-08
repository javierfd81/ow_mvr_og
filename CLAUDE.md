# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

This is a single **Overwatch 2 Workshop** game mode: **"OG mvr"** (OG Mercy VS Rein). Team 1 plays Reinhardt, Team 2 plays Mercy — an asymmetric mode where flying Reins hunt Mercys who keep Guardian Angel and can go invisible. There is no application source code in the conventional sense; the entire mode is one text file written in the Overwatch Workshop scripting DSL.

- [mvr_og.code](mvr_og.code) — the mode, exported via the Workshop editor's **Copy Code** feature (~5,500 lines).
- `mvr_og copy.code` — a byte-identical manual backup (untracked). When you edit the mode, keep these two in sync, or treat `copy` as a disposable checkpoint.

## There is no build / test / lint toolchain

Do not look for `package.json`, a test runner, or a compiler — none exist and none apply. The only way to "run" this code is inside Overwatch:

1. Copy the full contents of `mvr_og.code`.
2. In the Overwatch Workshop editor: **Settings → Import → paste** (the reverse of Copy Code).
3. Play/preview in a Workshop lobby to observe behavior.

Because there is no automated verification, edits must be reasoned about carefully against the DSL's semantics before handing the file back to the user to import and test in-game. Version control (git) is the only tooling; the mode is not yet committed (`master` has no commits).

## File anatomy

The file is a flat, ordered document with four top-level sections, in this order:

- `settings { ... }` — lobby config, enabled game modes + per-mode map pools, enabled heroes per team, and **hero stat scalars** under `heroes.General` (e.g. Mercy `Movement Speed: 300%`, Rein `Health: 500%`). Much of the gameplay "feel" is tuned here rather than in rules. The `workshop { ... }` sub-block lists custom, in-lobby-tunable toggles (e.g. `Super Jump CoolDown`, `GA Recharge Rate`) that rules read at runtime.
- `variables { global: ... player: ... }` — declares state slots. **Each variable is a numbered slot** (`0: TDMSpawnPositions`, `1: MercyRezScalar`, …). Slot numbers are non-contiguous and the names are just labels for those slots.
- `subroutines { ... }` — numbered subroutine slots (e.g. `0: ResetRound`, `14: RespawnRein`), invoked from rules via `Call Subroutine` / `Start Rule`.
- `rule("Name") { event / conditions / actions }` — 177 rules. This is where all custom mechanics live.

## How to navigate and edit the rules

**Section dividers.** Rules are grouped by dummy divider rules whose names are `====` banners and which are marked `disabled` so they never execute. Use them as a table of contents:

`CAMERA` → `MERCY POWER UPS` → `REIN POWER UPS` → `MOD POWERS` → `MISCELLANEOUS` → `TIMERS` → `MAP CHANGES` → `TDM STUFF`

Grep for `disabled rule("===` to jump between sections; grep for `rule(` to list every rule.

**The `disabled` keyword** prefixes any rule (or individual action) that is present in the file but inactive in-game. Many experimental/WIP mechanics (multi-jump, outline reveals, alternate Rein-switching logic) are kept disabled alongside their live counterparts — do not assume a `disabled` rule reflects current behavior.

**Editing conventions that matter:**
- When adding a variable or subroutine, pick an **unused slot number** and add it to the corresponding declaration block. Reusing a slot silently aliases state; skipping the declaration but using the name will not round-trip through the editor.
- Rules run their `actions` top-to-bottom; control flow uses `Wait Until`, `Abort If`, `Loop`/`Skip If`, and `Call Subroutine`. There is no function scope — subroutines share the same global/player variable space.
- Per-map behavior (out-of-map restrictions, boop-from-stuck-spot fixes, teleporters, invisible walls/ceilings) is concentrated under the `MAP CHANGES` section, keyed by map name. Adding map support usually means adding a new rule there, not changing shared logic.
- Round lifecycle flows through `Start Round` → `ResetRound`/`SpawnSetter` subroutines → win-condition rules (`Reins killed all Mercys`, `Mercys killed all Reins`, `Round time ended`) → `EndRound`. Spawn placement differs by game mode (Elimination vs Skirmish vs TDM/CTF), handled by `SpawnSetter`, `SkirmishSpawnSetter`, and `EnsureTDMSpawns`.

## Domain glossary

- **GA** — Mercy's Guardian Angel (her mobility ability); heavily customized (recharge, sacrifice-health-for-charge, out-of-map disabling).
- **Rez** — Mercy Resurrect; gated by a custom cooldown that scales with map size and is reduced by booping Reins.
- **Boop** — knockback; used both as a mechanic (Mercy melee boops Rein → reduces rez CD) and as a bug-fix (boop a player out of a stuck spot).
- **Super Jump / Multi Jump** — custom Rein/Mercy vertical mobility built on Baptiste crouch / Winston jump inputs.
- **Seeker Priority List** — ordering logic for which player becomes Rein each round (`Add/remove player from Seeker Priority List`, `Auto Swap Rein at start of round`).
