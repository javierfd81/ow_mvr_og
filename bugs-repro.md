# OG mvr — Bug Test Checklist (Community)

Help us confirm some bugs in **OG mvr** (Mercy vs Reinhardt)! Below are simple, step-by-step tests.
No coding needed — just play the steps and tell us what happened.

**How to help**
1. Pick a bug and follow the steps exactly.
2. Compare what you saw to **✅ Should happen** vs **🐞 The bug**.
3. Report back with the info in the template at the bottom.

> Please always include your **player count, map, and game mode** — several bugs only show up in specific setups.

---

## 🔴 Bug 1 — Infinite Resurrect (Mercy rez with no cooldown)

**You'll need:** 2+ Mercy players, at least one of them dead. Any mode.

**Steps**
1. As Mercy, start resurrecting a dead teammate.
2. Let the resurrect bar fill **almost all the way** — do **not** cancel it early.
3. Right as it's about to finish, tap **Resurrect (or Secondary Fire) again**.

**✅ Should happen:** either the rez finishes and your Resurrect goes on its normal long cooldown, **or** it cancels and nobody is revived.

**🐞 The bug:** your teammate **IS** revived **and** your Resurrect cooldown shows **0** — so you can rez again instantly, over and over.

*Tip: the timing is tight (right at the end of the cast) — it may take a few tries to nail.*

---

## 🟡 Bug 2 — End-of-round Rein swap misfires in big lobbies

**You'll need:** a lobby of **8 to 11 players** (2 active Reins). *(Advanced test.)*

**Steps**
1. Get a lobby with **8–11 players**.
2. Make sure **not many people are queued** to be the next Rein (fewer than 2 in the seeker queue).
3. At the end of a round, the **Host holds Ability 1 + Ability 2 + Reload** at the same time to force the Rein swap.

**✅ Should happen:** two new Reins are pulled from the Mercy team to fill **both** Rein slots.

**🐞 The bug:** only **one (or zero)** new Rein gets pulled — both Rein slots aren't filled correctly.

*Please report the exact player count.*

---

## 🟠 Bug 3 — Mercy can't fly after hitting a Skirmish wall with no GA

**You'll need:** **Skirmish** mode on a map with the glowing "wall/fence" that splits the arena — e.g. **Blizzard World, Horizon Lunar Colony, Junkertown, Volskaya, New Junk City, Suravasa**. Play **Mercy**.

**Steps**
1. Use up **all** your Guardian Angel (GA) charges so you have **none** left.
2. Fly or walk into the glowing fence line that splits the map (it knocks you back).

**✅ Should happen:** after the wall knocks you back, your GA works again once it recharges.

**🐞 The bug:** your GA (jump-to-ally) **stays disabled** — you can't fly for a while, maybe the rest of the round.

*Please note: does GA ever come back on its own? If so, roughly how long?*

---

## 🟡 Bug 4 — Death sphere kills you even after you leave

**You'll need:** play **Mercy** on a map with a red "death sphere" marking a no-hiding spot — e.g. **Castillo, Ilios Lighthouse, Ilios Well, King's Row**. *(Paris is the reverse — there you must stay **inside** the sphere.)*

**Steps**
1. Fly into the red sphere. You'll see a warning like *"illegal hiding spot"* / *"Get Back in Sphere!"*.
2. **Leave** the sphere within **3 seconds**.

**✅ Should happen:** leaving in time saves you.

**🐞 The bug:** you still get bounced up and **killed after ~3 seconds**, even though you left in time.

---

## 🟡 Bug 5 — Resurrect refund glitch when a Mercy leaves

**You'll need:** a match with several Mercys. *(Rare timing glitch — hard to force; mostly "report if you see it.")*

**Steps**
1. Have Mercys resurrecting each other during a round.
2. If a **Mercy player leaves the match** while another Mercy is **mid-resurrect**, watch the *"Refunded Resurrect"* message.

**✅ Should happen:** the player who revived the person who left gets a clean *"Refunded Resurrect"*.

**🐞 The bug:** the refund goes to the **wrong** person, or doesn't happen, or the resurrect tracking acts strangely.

*If you ever see a weird or wrong "Refunded Resurrect" message, please screenshot it and report.*

---

## 📋 Report template

Copy, fill in, and send back for each bug you test:

```
Bug #:            (1–5)
Reproduced?       ✅ Yes  /  ❌ No  /  🤔 Sort of
How many tries:
Player count:
Map:
Game mode:
What happened:
Video/screenshot:  (if you have one — hugely helpful!)
```

Thanks for helping test — every confirmation (or "couldn't reproduce") is useful! 🙏
