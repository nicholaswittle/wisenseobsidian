---
title: Apex Hub — Accepted Owner Design
tags: [apex, v3, ui, hub, design, accepted]
aliases: [Owner Hub, Hub Design]
date: 2026-08-08
---

# Apex Hub — Accepted Owner Design (2026-08-08)

The owner home screen, agreed with Nick after several iterations. This is the
spec to build. Mockups: `C:\Users\nikwi\Notes\apex-hub-owner.html` (base),
`C:\Users\nikwi\Notes\apex-hub-widgets.html` (configurable widgets).

> **2026-08-08 (later) — the 3-slot strip is now the owner's choice.** Nick's
> idea: make the health strip configurable like phone widgets, so the owner
> picks which 3 stats matter most to THEIR business. Nick: *"it makes each app
> feel like they are totally in control of it."* This is the accepted version —
> see "The configurable health widgets" section below.

## The governing rules (from Nick)

1. **The hub shows the most-desired buttons for each role + the data they need** —
   NOT capped at 3 buttons. The "3 buttons max" rule lives INSIDE each
   destination screen (see the schedule spec), not on the hub.
2. **The 2-year-old test governs everything** — one obvious named next action, no
   jargon, no blank boxes, no dead ends, do-it-for-them-then-correct. A zero-tech
   user must succeed unaided.
3. **The bottom bar is the navigation. There is NO "Go to" section** — that
   duplicated the bottom bar. The page is just the dashboard data.
4. **The AI agent lives in the bottom bar** (an "Ask" tab, orange-highlighted) and
   has a slim affordance at the top of the page.

## The owner hub layout (top to bottom)

1. **Greeting** — "Good afternoon, Nick" + business name + date.
2. **Assistant affordance** — a slim bar: orange orb ✦ + "What do you need done
   today?" + "Ask it anything" + mic. Tap to talk.
3. **Money hero** — big figure: "Taken through the app today · $412.50".
4. **Health strip** — ONE compact row, three stats (no extra scrolling):
   - **Orders today** (e.g. 7)
   - **On shift** (e.g. 3/4 — orange/warn when a slot is open, green when full)
   - **Labor** (e.g. 18%)
   - Color is meaningful: orange = needs attention, green = healthy.
5. **Needs you** — the real attention items, each with a headline + reason. Shows
   the genuine count (2 shown when 2 exist): e.g. "1 order awaiting payment ·
   waiting over 30 minutes" and "No one on the bar tonight · shift at 6pm not
   covered".
6. **Bottom nav** — Home · Schedule · Orders · Team · **Ask** (the AI agent).

## The bottom nav is the navigation

No "Go to" / "Everything else" section on the page. The bottom bar carries every
destination. The Ask (AI agent) tab is orange-highlighted so it reads as the
"ask it anything" catch-all. Same bottom bar for owner and employee — the
employee's set is leaner (see employee spec when built).

## What the glance answers (why this works)

An owner opens it and in one glance knows three things:
- **Am I making money?** → the money hero.
- **Am I properly staffed right now?** → the "On shift" health stat.
- **Are orders moving?** → the "Orders today" stat + the "Needs you" items.

That's the "at a glance" that was missing. The health strip is the piece that
made it feel complete.

## The configurable health widgets (accepted 2026-08-08)

The 3-slot health strip is NOT fixed — it's the owner's choice, like phone
widgets. This is the accepted version.

**How it works:**
- The health strip always holds **3 slots** (layout stays the 2-year-old shape).
- Each widget has a small **pencil** (✎) to change it.
- Tap a widget → a picker sheet: "What matters most to you. Pick up to 3."
- The owner picks which stats live there. Options include: Labor, Open shifts,
  Sales this week, Orders today, On shift, My tips (grows with the product).
- **Defaults** = Orders today, On shift, Labor — sensible for anyone who never
  touches it.

**EVERY ROLE gets the configurable 3-slot strip** — owner, employee, and
services owner. Same widget mechanism, same pencil, same "pick up to 3" sheet,
same 3-slot cap. Only the DEFAULTS differ by role:
- **Owner** default: Orders today / On shift / Labor
- **Employee** default: My tips / Hours this week / Open shift
- **Services owner** default: Open quotes / Jobs this week / Crew out

Nobody is stuck with the defaults — the pencil lets anyone swap their 3 to what
matters to them.

**Why it works (Nick's words):** *"it makes each app feel like they are totally
in control of it."* What's "most important" differs per owner (labor % vs open
shifts vs tips). Letting them pick means the hub is right for *their* business,
not the designer's guess — and it's a retention lever: the app feels like the
owner runs it, not like a tool they rent.

**Rule:** the strip is 3 slots; the picker is the only way to change what's in
them. No accumulation — one owner can't end up with 8 widgets, or the 2-year-old
test is lost.

## Build notes

- Every number is read or refused — a figure that cannot be computed says which
  input is missing, and a stat with nothing behind it is not rendered (v3's
  hardcoded-22.4% rule).
- Setup cards disappear once there's nothing left to do (no permanent furniture).
- Colors: neutral charcoal surfaces, **coral accents only** (Nick changed this
  from orange to coral on 2026-08-08 — orange kept confusing Claude into making
  everything orange). No warm-tinted dark surfaces; charcoal + coral only.
  Coral accent hexes: `#ff7f50` (primary), `#f0653e` (deep).

Related: [[projects/APEX_V3_SESSION_2026-08-07]] · [[projects/Apex v2 — Restaurant OS Build]] · [[Inbox]] · [[hot]] · [[NOW]] · [[index]]
