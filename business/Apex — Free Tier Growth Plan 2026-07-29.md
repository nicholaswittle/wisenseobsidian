---
title: Apex — Free Tier Growth Plan
tags: [apex, growth, plg, free-tier, strategy, active]
aliases: [Free Tier Plan, Apex PLG, Apex Growth Plan]
date: 2026-07-29
status: active
---

# Apex — Free Tier Growth Plan

> The hook, the retention mechanic, and the free/paid line for Apex as a
> product-led scheduling app. Related: [[Apex v2 — Restaurant OS Build]],
> [[business/WiSense Restaurant OS Master Plan 2026-07-27]], [[NOW]]

**Model:** free scheduling app → habit → gated modules unlock → Pro/OS.
"OS" is Nicholas's internal map of how the pieces fit, **not the pitch.**
Externally this is a scheduling app that grows.

---

## The strategic reframe

**The competitor is the group chat, not Homebase.**

Most small independents use no scheduling software at all — a paper schedule
taped inside the office door plus a staff group text. That is the segment the
photo-import wedge is built for, because *it only works as a hook if they
already have a paper schedule*.

Converting non-consumers beats switching someone off 7shifts: no data to
migrate, no retraining, no contract, no sunk cost. So the free tier should not
be designed to win a feature comparison against Homebase. It should be designed
to beat **paper plus a group text**.

The bar: *be less annoying than a group text on a Tuesday.*

Switchers from real incumbents come later, via word of mouth. Do not design for
them now.

---

## The hook — why they try

**One promise, demoable in 60 seconds:**

> Photograph your schedule. Your staff have it on their phones before you
> finish your coffee.

Not "scheduling that grows with you." No feature list. It works because it
inverts the normal ask — every competitor opens with *"add your 14 employees
and their rates"*, which is an hour of desk work for someone who does not sit
at a desk. **Onboarding is where scheduling apps die.** Apex opens with "point
your camera at the thing already on your wall."

### The ten-minute path

1. Sign up — no card, no org wizard (the `apex_handle_new_user` trigger already
   auto-provisions the org)
2. **Photo import is the first screen after signup**, not buried in a menu
3. They see *their own real week* rendered **before inviting anyone**
4. Then: "Send this to your team" → invite codes

Step 3 is what most apps get wrong. They require invites before showing value,
so the manager must sell it to their team on faith. Let them see their own
schedule first — then they are sharing something that already exists.

### Never dead-end

If photo import fails on a bad picture, fall through to paste-text
(`schedule_text_parser`), then manual entry. A failed import that offers nothing
is the end of the trial.

---

## What keeps them — staff habit, not manager features

**The manager cannot leave once staff are checking the app.** Leaving means
retraining fifteen people. Retention here is staff-side, not manager-side — so
the retention roadmap is employee engagement, not admin tools.

Ranked by strength of dependency:

| Feature | Why it locks | Frequency |
|---|---|---|
| **QR clock-in** | They *must* open the app to get paid | Daily |
| Shift reminders / publish alerts | Pull, not push effort | Weekly+ |
| "Who am I working with tonight" | Genuinely wanted, no substitute | Weekly |
| Swaps | Social loop — needs a reply | Occasional |
| **Chat** | Kills the group text | Daily |

**QR clock-in is the strongest retention feature and is already free.** Daily
forced engagement beats any manager dashboard.

**Chat is the one to fight for.** If staff keep using the group text for
messages, the switch is never finished and the venue is one bad week from
abandoning the app.

---

## Redraw the free/paid line

Two changes to the current split:

**1. Move offline mode to FREE.** Charging for "works when the wifi is bad"
makes the free tier feel broken in exactly the venues being targeted — basement
offices, walk-ins, the dead spot by the dish pit. Paywalling reliability teaches
distrust. Reliability is table stakes, not a feature.

**2. Move basic team chat to FREE.** It is the group-text killer. Monetize
history search or pinned announcements later if needed.

### The resulting line, sayable out loud

> **Free runs your team. Paid runs your books.**

| Tier | Contents |
|---|---|
| **Free** | schedule · swaps · availability · time clock · chat · offline |
| **Pro $25** | tips · labor cost · log book |
| OS $99 | + ordering · capacity · no-show engine |
| Multi $199 | up to 3 locations |

**Tips is the right first gate** — weekly, cash, painful by hand. **Labor cost
is the right second** — owner-facing, money-shaped. People pay to fix money
problems and resent paying for convenience.

**Show locked features greyed out with a real number behind them.** A gate
nobody can see does not convert:
`Tonight's tip split — $482 across 3 servers 🔒`
That beats a pricing page.

---

## The one activation number

> **Schedule published + 3 staff installed, within 7 days of signup.**

That is the stick/don't-stick line. Below it they churn; above it they are
dependent. Judge every onboarding decision on whether it moves this number.
Nothing else in the funnel matters until it is consistently hit.

---

## What would kill this

**Photo import being unreliable.** The entire hook rests on it. If it works on
clean printouts but fails on a handwritten schedule photographed at an angle
under fluorescent light, there is no wedge — only a demo.

**Test it in week one of the pilot, on the real thing**, before anything else.
It is the load-bearing assumption and it is cheap to check.

---

## Pilot design (Melissa's cohort)

Fully unlocked, which is correct for a first pilot — it finds bugs and shows
what people reach for.

**Know what it does and does not test.** Unlocked tests the *features*, not the
*funnel*. It cannot tell you whether the gate entices or annoys, and that is the
assumption the business model rests on. Do not read "they loved it" as "they
would have paid." Save the gate question for a later cohort of real venues
running the actual free tier.

**Day one task:** photo-import the real schedule — handwritten, crumpled,
photographed at an angle in bad kitchen light.

**Instrument usage** (which screens get opened, by whom, how often). With the
free/paid line about to change, that is the evidence that should decide it.
Gate-tap logging is useless while everything is unlocked; screen usage is not.
