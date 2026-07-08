<!--
DISCORD COPY-PASTE VERSION
Paste each block below as a SEPARATE Discord message (Discord caps messages at 2000 chars).
The "═══ MESSAGE N ═══" lines are just guides for you — do NOT include them in the posts.
Discord renders the #/##/### headers, **bold**, ✅/🐞 emoji, and -# small text.
Tip: a Forum channel or a pinned thread works great for this.
-->

═══════════════ MESSAGE 1 — Intro ═══════════════

# 🐞 OG mvr — Help us test some bugs!
No coding needed. Pick a bug, follow the steps **exactly**, and tell us what you saw: **✅ what should happen** or **🐞 the bug**. Even a "couldn't reproduce" is useful!

> ⚠️ Always include your **player count**, **map**, and **game mode** — several bugs only show up in specific setups.

Report template is in the last message. Thanks! 🙏

═══════════════ MESSAGE 2 — Bug 1 ═══════════════

## 🔴 Bug 1 — Infinite Resurrect (Mercy rez with no cooldown)
**You'll need:** 2+ Mercy players, at least one dead · any mode

**Steps**
1. As Mercy, start resurrecting a dead teammate.
2. Let the resurrect bar fill **almost all the way** — do **not** cancel it early.
3. Right as it's about to finish, tap **Resurrect (or Secondary Fire) again**.

✅ **Should happen:** the rez finishes and your Resurrect goes on its normal long cooldown, **or** it cancels and nobody is revived.
🐞 **The bug:** your teammate **IS** revived **AND** your Resurrect cooldown shows **0** — so you can rez again instantly, forever.

-# 💡 The timing is tight (right at the end of the cast) — may take a few tries to nail.

═══════════════ MESSAGE 3 — Bug 2 ═══════════════

## 🟡 Bug 2 — End-of-round Rein swap misfires in big lobbies
**You'll need:** a lobby of **8–11 players** (2 active Reins) · *advanced test* · Host required

**Steps**
1. Get a lobby with **8–11 players**.
2. Make sure **not many people are queued** to be the next Rein (fewer than 2 in the seeker queue).
3. At the end of a round, the **Host holds Ability 1 + Ability 2 + Reload** at the same time to force the Rein swap.

✅ **Should happen:** two new Reins are pulled from the Mercy team to fill **both** Rein slots.
🐞 **The bug:** only **one (or zero)** new Rein gets pulled — both Rein slots aren't filled correctly.

-# 💡 Please report the exact player count.

═══════════════ MESSAGE 4 — Bug 3 ═══════════════

## 🟠 Bug 3 — Mercy can't fly after hitting a Skirmish wall with no GA
**You'll need:** **Skirmish** · play **Mercy** · a map with the arena-splitting fence: **Blizzard World, Horizon Lunar Colony, Junkertown, Volskaya, New Junk City, Suravasa**

**Steps**
1. Use up **all** your Guardian Angel (GA) charges so you have **none** left.
2. Fly or walk into the glowing **fence line** that splits the map (it knocks you back).

✅ **Should happen:** after the wall knocks you back, your GA works again once it recharges.
🐞 **The bug:** your GA (jump-to-ally) **stays disabled** — you can't fly for a while, maybe the rest of the round.

-# 💡 Does GA ever come back on its own? If so, roughly how long?

═══════════════ MESSAGE 5 — Bug 4 ═══════════════

## 🟡 Bug 4 — Death sphere kills you even after you leave
**You'll need:** play **Mercy** on a map with a red "death sphere" no-hiding spot: **Castillo, Ilios Lighthouse, Ilios Well, King's Row** *(Paris is the reverse — stay **inside** the sphere)*

**Steps**
1. Fly into the red sphere. You'll see a warning like **"illegal hiding spot"** / **"Get Back in Sphere!"**.
2. **Leave** the sphere within **3 seconds**.

✅ **Should happen:** leaving in time saves you.
🐞 **The bug:** you still get bounced up and **killed after ~3 seconds**, even though you left in time.

═══════════════ MESSAGE 6 — Bug 5 ═══════════════

## 🟡 Bug 5 — Resurrect refund glitch when a Mercy leaves
**You'll need:** a match with several Mercys · *rare timing glitch — mostly "report if you see it"*

**Steps**
1. Have Mercys resurrecting each other during a round.
2. If a **Mercy player leaves the match** while another Mercy is **mid-resurrect**, watch the **"Refunded Resurrect"** message.

✅ **Should happen:** the player who revived the person who left gets a clean **"Refunded Resurrect"**.
🐞 **The bug:** the refund goes to the **wrong** person, or doesn't happen, or resurrect tracking acts strangely.

-# 💡 If you ever see a weird or wrong "Refunded Resurrect" message, screenshot it and report.

═══════════════ MESSAGE 7 — Report template ═══════════════

## 📋 How to report
Copy this, fill it in, and post one per bug you test:
```
Bug #:            (1–5)
Reproduced?       Yes / No / Sort of
How many tries:
Player count:
Map:
Game mode:
What happened:
Video/screenshot:  (attach if you can — hugely helpful!)
```
Thanks for helping test **OG mvr**! 🙏
