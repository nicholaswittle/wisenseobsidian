---
title: Apex v3 — Fixes List (from live testing)
tags: [apex, v3, fixes, bugs, ui, testing]
aliases: [v3 Fixes, Fixes List, v3 Issues]
date: 2026-08-08
---

# Apex v3 — Fixes List (from live testing)

Found by Nick while testing v3 against v2, 2026-08-08. **The honest headline:
the backend is an A, but several v2 UI/features are better designed than v3's.**
This list is the gap. Fix these, and v3 stops being "better backend, worse UI."

## Website / public site
- [ ] The website won't put what jobs you can do on it (services not shown).
- [ ] The website won't update the quote form.
- [ ] The website builder was NICER in v2 than v3 — v3's is a step back.
- [ ] No AI suggest on blank boxes — e.g. the short introduction in the website
      portion of the app should suggest text, like v2 did.

## AI agent / assistant
- [ ] No AI monitor of the app like v2 had.
- [ ] The AI agent doesn't actually do anything it says it can — it spits out
      generic stuff like "I can't see your apex" instead of acting.
- [ ] Remove the "What do you need done today?" AI agent block on the HOME
      PAGE — it's better on the main menu bar at the bottom as just "Ask".

## Admin / super admin
- [ ] The super admin doesn't have businesses split by restaurant and services —
      so it's not actually being logged what's what.

## Services (the two-portal problem)
- [ ] There are TWO separate portions for services and neither talks to the
      other: one in the app that does nothing, one in the website portion that
      also does nothing. They must be one thing.
- [ ] Pricing for services is only in cents — you can't make it a dollar amount.

## Photos
- [ ] When you upload a photo there's no way to know if it actually worked
      (no success/failure feedback).

## Clock
- [ ] What's the purpose of having the clock-in code right where you have to
      put clock in? v2's version was cleaner.

## Color / theming
- [ ] No customizable color choices like v2 had.

## General
- [ ] v3 UI looks better than v2 overall, but the FEATURES on v2's UI are
      better designed. Port v2's feature design onto v3's cleaner UI.

## The through-line
The backend (operations layer, state machines, trade-neutrality, actor_kind)
is genuinely an A. The gap is that v2's UI was designed around real use and
v3's was designed around the architecture. Fix the features above and v3
becomes the better product, not just the better backend.

Related: [[projects/APEX_V3_REMAINING_MODULES_2026-08-08]] · [[projects/APEX_HUB_DESIGN_2026-08-08]] · [[hot]] · [[NOW]] · [[index]]
