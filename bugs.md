# OG mvr — Bug Tracker

Tracks bugs found in `mvr_og.code` (the "OG mvr" — Mercy vs Reinhardt — Overwatch Workshop mode),
their root cause, proposed fix, implementation status, and test status.

- **Source file:** `mvr_og.code` (5,552 lines)
- **Last updated:** 2026-07-08

> Line numbers reference `mvr_og.code`. They shift as the file is edited — re-anchor by rule name
> (shown in each entry) if a number looks stale.

---

## Status legend

| Field | Values |
|---|---|
| **Severity** | 🔴 Critical (game-breaking / exploit) · 🟠 Major (breaks a feature) · 🟡 Minor (edge case / cosmetic) |
| **Discovery** | Player-reported · Code-review · Analysis |
| **Confirmed** | ✅ Repeatable · 🔍 In source (verified by reading code) · ❔ Needs in-game verification |
| **Fix** | Proposed · Designed · — |
| **Implementation** | ⬜ Not started · 🟦 In progress · ✅ Done |
| **Test** | ⬜ Not started · 🧪 In progress · ✅ Passed · ❌ Failed |

---

## Summary

| ID | Title | Severity | Confirmed | Fix | Impl. | Test |
|----|-------|----------|-----------|-----|-------|------|
| [BUG-01](#bug-01--infinite-resurrect-exploit-rez-cancel-timing) | Infinite resurrect exploit (rez-cancel timing) | 🔴 Critical | ✅ Repeatable | Proposed | ⬜ | ⬜ |
| [BUG-02](#bug-02--seeker-switch-misplaced-parenthesis) | Seeker-switch misplaced parenthesis | 🟡 Minor | 🔍 In source | Proposed | ⬜ | ⬜ |
| [BUG-03](#bug-03--mercy-permanently-unable-to-ga-after-fence-wallboop) | Mercy permanently unable to GA after fence WallBoop | 🟠 Major | 🔍 In source | Proposed | ⬜ | ⬜ |
| [BUG-04](#bug-04--deathorbkill-wait-cannot-re-check-position) | DeathOrbKill `Wait` cannot re-check position | 🟡 Minor | ❔ Needs verification | Designed | ⬜ | ⬜ |
| [BUG-05](#bug-05--rezzedmercy-global-clobbered-by-array-write) | `RezzedMercy` global clobbered by array write | 🟡 Minor | ❔ Needs verification | Designed | ⬜ | ⬜ |

---

## BUG-01 — Infinite resurrect exploit (rez-cancel timing)

| | |
|---|---|
| **Severity** | 🔴 Critical |
| **Discovery** | Player-reported |
| **Confirmed** | ✅ Repeatable (reported repeatable by a player; matches code) |
| **Fix** | Proposed |
| **Implementation** | ⬜ Not started |
| **Test** | ⬜ Not started |

**Rules involved:** `Mercy: rez cancel` (L546–570), `Mercy Used Rez` (L572–607), `Set rezzed mercy` (L609–627)
**Supporting:** `Mercy Rez Balancing with Map Size` (L1067) sets `Global.MercyRezScalar` (65–90 s, the intended cooldown).

### Description
A Mercy who resurrects a teammate and taps the cancel input at a precise moment near the **end** of the
cast gets **both** outcomes at once: the target is resurrected **and** the caster's resurrect cooldown is
forced to `0`. Result: unlimited, no-cooldown resurrects.

### Steps to reproduce
**Prerequisites:** 2+ Mercys on Team 2, at least one of them dead.
1. Mercy (player 1) begins resurrecting a dead Mercy (player 2).
2. Let the cast channel **almost** to completion (do not release early).
3. In the narrow window right as the rez commits, press **Ability 2 / Secondary Fire** again to trigger the cancel path.
4. **Expected:** either the rez completes and player 1 pays the `MercyRezScalar` cooldown, **or** it cancels and no one is rezzed.
   **Actual:** player 2 **is** resurrected **and** player 1's rez cooldown reads `0`. Repeat indefinitely.

The timing is a narrow window at the tail of the cast; a player reports it repeatable with practice.

### Root cause
The intended feature is *"cancel early = no cooldown penalty"* (advertised in the tips as
*"Hit Rez While Rezzing To Cancel Early"*). It works like this:

- `Mercy Used Rez` waits 0.4 s into the cast, sets `Player.rezstop = True`, waits for the cast to end, then
  applies the `MercyRezScalar` penalty **only if `rezstop` is still true** (L594), using an
  `If(Ability 2 CD == Secondary Fire CD)` heuristic (L591) to detect a cancel.
- `Mercy: rez cancel` (L562–568), when the player taps cancel while `rezstop` is true, clears `rezstop`,
  calls `Cancel Primary Action`, waits 0.25 s, and **unconditionally forces both cooldowns to `0`.**

The whole design assumes *"if the cancel rule fired, the rez was interrupted, so nobody was resurrected."*
That assumption breaks past the resurrect's point-of-no-return: the rez **completes** (target rezzed) yet the
cancel path still runs, clearing `rezstop` and zeroing the cooldown — so the `MercyRezScalar` penalty at
L594–597 is skipped. The `CD_a2 == CD_sf` detector is also defeated because the cancel forced both to `0`.

**The defect is at L562–568:** the cancel path zeroes the cooldown *without verifying the resurrect was
actually prevented.*

```
// Mercy: rez cancel  (current — L562)
If(Event Player.rezstop);
    Event Player.rezstop = False;
    Cancel Primary Action(Event Player);
    Wait(0.250, Ignore Condition);
    Set Ability Cooldown(Event Player, Button(Ability 2), 0);        // <-- zeroed regardless of rez outcome
    Set Ability Cooldown(Event Player, Button(Secondary Fire), 0);
End;
```

### Proposed fix — per-caster channel timing
Judge each Mercy **only by how long her own resurrect channel ran**, never by the target. This is immune to
the "two Mercys rezzing one soul" case (a target-state check is not — see note below).

- Cancel **before** the resurrect commits → short channel → free (CD = 0). *(feature preserved)*
- Cancel **at/after** the commit point → rez completes → full-length channel → charge `MercyRezScalar`.
  *(exploit closed)*

Two small edits, leaving the completion path untouched:

1. In `Mercy Used Rez`, stamp the channel start:
   ```
   Event Player.<rezStart> = Total Time Elapsed;   // first action when the channel begins
   ```
2. In `Mercy: rez cancel`, gate the free-cooldown branch on elapsed time:
   ```
   If(Event Player.rezstop && Total Time Elapsed - Event Player.<rezStart> < <WINDOW>);
       ... (existing free-cancel body: clear rezstop, cancel, wait, zero both CDs)
   End;
   // a late press now falls through untouched -> Mercy Used Rez applies MercyRezScalar normally
   ```

`<WINDOW>` ≈ **1.4–1.5 s** (Mercy rez cast ≈ 1.75 s). Expose it as a `Workshop Setting Real` under
"Mercy Settings" so it can be tuned live. `<rezStart>` needs a free player-variable index (indices are sparse;
pick an unused one and add it to the `variables player:` table).

**Why not target-state detection ("did a dead ally become alive?"):** it cannot attribute the resurrect to a
specific caster. If Mercy A and Mercy B both rez the same soul and A completes it, *both* see "a dead ally
became alive," so B is wrongly penalized. Overwatch exposes no "who resurrected X," so only per-caster channel
timing is reliable. Edge case under the timing fix: if B channels fully on an already-taken soul she may be
charged for a rez that produced nobody — rare, and degrades safely (an over-charge, not an exploit).

### Implementation notes
- [ ] Add player variable `<rezStart>` to the `variables player:` table.
- [ ] Add `Workshop Setting Real("Mercy Settings", "Rez Cancel Window", 1.4, 0.4, 1.7, …)`.
- [ ] Edit `Mercy Used Rez` (stamp start time).
- [ ] Edit `Mercy: rez cancel` (time-gate the free-cooldown branch).
- [ ] Verify the real commit time in-game and set the default window with margin.

### Test plan
- [ ] Normal completed rez → cooldown = `MercyRezScalar` (per map).
- [ ] Early cancel (well before commit) → cooldown = `0`, target **not** rezzed.
- [ ] **Exploit attempt** (late cancel at end of cast) → target rezzed **and** cooldown = `MercyRezScalar`.
- [ ] Two Mercys rez the same soul → completer pays, interrupted one is free; no cross-charging.
- [ ] Post-rez 20 s floor (`Mercy: change mercy rez cd when they are ressed`, L773) still applies.

---

## BUG-02 — Seeker-switch misplaced parenthesis

| | |
|---|---|
| **Severity** | 🟡 Minor |
| **Discovery** | Code-review |
| **Confirmed** | 🔍 In source (verified) |
| **Fix** | Proposed |
| **Implementation** | ⬜ Not started |
| **Test** | ⬜ Not started |

**Rule:** `Switch Reins at end of round test` (L2545), condition at **L2563**.

### Description
The player-count guard that should read *"more than 7 players"* never evaluates as intended, so the
top-up-from-Team-2 branch of the end-of-round seeker shuffle can behave incorrectly at high player counts.

### Steps to reproduce
1. Fill a lobby with **8–11 players** (enough for 2 active Reins; fewer than 12 so the rule's
   `Number Of Players(All Teams) < 12` gate at L2554 passes).
2. Leave the seeker queue short — **fewer than 2** players in `Global.NextSeekers`.
3. Trigger the end-of-round Rein switch: the **Host holds Ability 1 + Ability 2 + Reload** together (L2555).
4. **Expected:** the queue is topped up to 2 new seekers pulled from Team 2, filling both Rein slots.
   **Actual:** the top-up branch (L2564) never runs — `Count Of(All Players(All Teams) > 7)` collapses to `0`,
   so the condition is always false — and the rule falls through to the single-append `Else If` (L2566),
   filling at most one seeker. Both Rein slots are not rotated as intended.

### Root cause
```
If(Count Of(Global.NextSeekers) < 2 && Count Of(All Players(All Teams) > 7));   // L2563
```
The `> 7` is trapped **inside** `Count Of(...)`. `Count Of` receives `All Players(All Teams) > 7` (a boolean),
not the intended comparison of the count against 7. It should be:
```
If(Count Of(Global.NextSeekers) < 2 && Count Of(All Players(All Teams)) > 7);
```
This runs in the **active** shuffle rule (not the disabled variant at L2490).

### Proposed fix
Move the closing parenthesis so the count is compared to 7:
`Count Of(All Players(All Teams)) > 7`.

### Test plan
- [ ] Lobby with 8+ players → NextSeekers top-up branch fires as intended.
- [ ] Lobby with ≤7 players → falls through to the `Else If` branches unchanged.

---

## BUG-03 — Mercy permanently unable to GA after fence WallBoop

| | |
|---|---|
| **Severity** | 🟠 Major |
| **Discovery** | Code-review |
| **Confirmed** | 🔍 In source (verified) |
| **Fix** | Proposed |
| **Implementation** | ⬜ Not started |
| **Test** | ⬜ Not started |

**Rule:** `WallBoop SUBROUTINE` (L5334), re-enable check at **L5355–5357**.

### Description
When a Mercy is booped by a Skirmish "halve-the-map" fence while she has **0 Guardian Angel charges**, her
Ability 1 (Guardian Angel) is disabled by the WallBoop and never re-enabled — leaving her grounded for the
rest of the round.

### Steps to reproduce
1. Play **Skirmish** on a map with the "halve-the-map" fence — any map that gets a `Global.FencePosition`
   in `SpawnSetter` (e.g. Blizzard World, Horizon Lunar Colony, Junkertown, Volskaya, New Junk City, Suravasa).
2. As **Mercy**, spend Guardian Angel until `MercyGACount == 0` (no charges left).
3. Cross the **active fence line** to trigger `WallBoop` (L5334) — which runs `Set Ability 1 Enabled(False)` (L5350).
4. **Expected:** after the boop's DoT/slow ends, Guardian Angel becomes usable again once a charge recharges.
   **Actual:** the boop-exit re-enable is gated `Team 1 || MercyGACount > 0` (L5355), so with 0 GA it is skipped.
   *Verify in-game* whether the GA-recharge rule (L629) later restores Ability 1 or she stays grounded — this
   determines whether the impact is "briefly stuck" or "stuck for the round."

### Root cause
```
Set Ability 1 Enabled(Event Player, False);          // L5350 — disabled during the boop DoT/slow
...
If(Team Of(Event Player) == Team 1 || Event Player.MercyGACount > 0);   // L5355
    Set Ability 1 Enabled(Event Player, True);
End;
```
Ability 1 is re-enabled only for Team 1 **or** a Mercy with `MercyGACount > 0`. A Mercy with `MercyGACount == 0`
keeps GA disabled. The normal GA recharge rule (`Mercy GA Recharge + Reenable GA`, L629) re-enables Ability 1
*after* a recharge — but if she never regains a charge in a state the recharge rule expects, she can be
stuck.

### Proposed fix
Re-enable Ability 1 unconditionally on WallBoop exit and let the GA-count gate live where it belongs
(`GA use`, L673, already disables Ability 1 when count hits 0). Alternatively, on the `MercyGACount == 0`
branch, hand back at least one charge or trigger the recharge path before exit. Prefer the first: drop the
condition so exit always restores Ability 1.

### Test plan
- [ ] Mercy with 0 GA crosses the fence → after the boop, GA becomes usable again once recharged.
- [ ] Mercy with ≥1 GA crosses the fence → unchanged (still usable).
- [ ] Rein crosses the fence → unchanged.

---

## BUG-04 — DeathOrbKill `Wait` cannot re-check position

| | |
|---|---|
| **Severity** | 🟡 Minor |
| **Discovery** | Analysis |
| **Confirmed** | ❔ Needs in-game verification |
| **Fix** | Designed |
| **Implementation** | ⬜ Not started |
| **Test** | ⬜ Not started |

**Rule:** `Death Orb Kill Subroutine` (L2901).

### Description
The out-of-bounds "death orb" kill uses `Wait(3, Abort When False)`, but inside a subroutine the wait's
condition is evaluated against the subroutine's (static) context rather than continuously re-checking the
player's live position — so the grace/abort behavior may not work as a live check (e.g. a player who leaves
the orb radius within the 3 s may still be killed, or vice-versa).

### Steps to reproduce
1. Play a map with an **"illegal hiding spot" death orb** — the `Else` branch at L2934, i.e. maps **other than**
   Busan / Circuit royal / Lijiang Night Market (e.g. Castillo, Ilios Lighthouse, Ilios Well, King's Row;
   Paris uses the inverted "stay in the sphere" variant).
2. As **Mercy**, enter the orb radius → the message *"illegal hiding spot."* / *"Get Back in Sphere!"* appears
   and the 3 s timer starts (`Execute Death Orbs` L2879 → `DeathOrbKill` L2935).
3. **Leave** the orb radius within those 3 seconds.
4. **Expected:** leaving in time cancels the punishment.
   **Actual (to confirm):** `Wait(3, Abort When False)` in a subroutine has no condition list to re-evaluate,
   so it waits the full 3 s and then applies `Apply Impulse Up` + `Kill` (L2936–2940) regardless of you having left.

### Root cause (to confirm)
`Wait(..., Abort When False)` re-evaluates the **rule/subroutine condition list**, but a `Subroutine` event
has no condition list, so there is nothing to re-check. The intended "still inside the orb after 3 s?" gate
is therefore not actually re-tested.

### Proposed fix
Replace the `Abort When False` wait with an explicit live check, e.g.
`Wait Until(<player left orb radius>, 3)` then `Abort If(<player left orb radius>)`, or move the kill logic
into a real conditional rule whose condition can be re-evaluated.

### Verification / Test plan
- [ ] Confirm the faulty behavior in-game (enter an orb, leave within 3 s — do you die?).
- [ ] After fix: leaving the radius within the grace window cancels the kill; staying kills.

---

## BUG-05 — `RezzedMercy` global clobbered by array write

| | |
|---|---|
| **Severity** | 🟡 Minor |
| **Discovery** | Analysis |
| **Confirmed** | ❔ Needs in-game verification |
| **Fix** | Designed |
| **Implementation** | ⬜ Not started |
| **Test** | ⬜ Not started |

**Rules:** `Set rezzed mercy` (L609), `Mercy Used Rez` (L598), `Player Left Reset Variables` (L2752, write ≈ L2778).

### Description
`RezzedMercy` exists as **both** `global[40]` and `player[13]`. The global normally holds a single player
(`Set rezzed mercy` writes `Global.RezzedMercy = Event Player`), but `Player Left Reset Variables` overwrites
`Global.RezzedMercy` with a **Filtered Array** of players for a resurrect-cooldown refund. During that window,
`Mercy Used Rez` (L598) can stamp the array into a player's `.RezzedMercy`, corrupting the value it expects to
be a single player.

### Steps to reproduce
> Timing-dependent race — the global is reset to a single player every tick by `Set rezzed mercy` (L609), so
> the corrupt window is ~1 frame. Expect several attempts, and inspect variables via the Workshop inspector.
1. During a round, have **Mercy A actively resurrecting** (so `Mercy Used Rez` is about to run
   `Event Player.RezzedMercy = Global.RezzedMercy` at L598).
2. In the same brief window, have a **Team 2 (Mercy) player leave the match** — firing
   `Player Left Reset Variables`, which overwrites `Global.RezzedMercy` with a **Filtered Array** (L2778)
   rather than a single player.
3. **Expected:** the leaver's rezzer(s) get a clean *"Refunded Resurrect"* and `Global.RezzedMercy` remains a
   single player everywhere else.
   **Actual (to confirm):** while the global transiently holds an array, L598 stamps that array into a player's
   `.RezzedMercy`, breaking the later `Current Array Element.RezzedMercy == Event Player` comparison (L2778) and
   mis-refunding — or failing to refund — a resurrect.

### Root cause
Temporal aliasing of one name across scopes and types (single player vs. array), with no guard preventing a
read of the global while it transiently holds an array.

### Proposed fix
Give the player-left refund path its **own** variable (e.g. `Global.RezRefundList`) instead of reusing
`Global.RezzedMercy`, so the single-player global is never transiently an array. Rename to remove the
global/player name collision while at it.

### Verification / Test plan
- [ ] Reproduce: player leaves mid-round while another Mercy is mid-rez; inspect `.RezzedMercy`.
- [ ] After fix: refund path uses a dedicated variable; `Global.RezzedMercy` always holds a single player.

---

## Changelog

- **2026-07-08** — Document created. Logged BUG-01 (player-reported infinite-rez exploit) with proposed
  per-caster-timing fix, plus BUG-02…BUG-05 from code review / analysis. No fixes implemented yet.
- **2026-07-08** — Added explicit **Steps to reproduce** to every bug (expected vs. actual outcome), anchored
  to the relevant rule line numbers.
