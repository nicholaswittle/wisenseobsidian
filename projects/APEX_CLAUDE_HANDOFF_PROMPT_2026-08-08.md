---
title: Handing Design Work to Claude — Reusable Prompt
tags: [claude, handoff, prompt, v3, design, hub]
aliases: [Claude Prompt, Design Handoff, Handoff to Claude]
date: 2026-08-08
---

# Handing Design Work to Claude — Reusable Prompt

Copy-paste this into Claude when you want it to build from an accepted design.
It points Claude at the spec + mockups and pins the rules so it stops
improvising. Read the notes at the bottom before sending.

---

## The prompt

```
I've got the accepted design for [SCREEN NAME — e.g. the Apex v3 owner home
screen (the hub)]. Build it in v3 to match, using these files as the spec:

- Design spec: C:\Users\nikwi\Notes\projects\APEX_HUB_DESIGN_2026-08-08.md
- Visual mockup: C:\Users\nikwi\Notes\apex-hub-owner.html
- Visual mockup (configurable widgets): C:\Users\nikwi\Notes\apex-hub-widgets.html

Read all of them first, then build the screen to match. The design is already
locked — don't redesign it, implement it exactly.

Implement it under these rules:

1. The hub is: greeting → assistant affordance → money hero → a 3-slot
   configurable health strip → "needs you" items → bottom nav.
2. The 3-slot health strip is the OWNER's choice, like phone widgets. Each
   widget has a pencil to change it; the picker is a sheet "What matters most
   to you — pick up to 3". Defaults: Orders today / On shift / Labor. Never
   more than 3 slots.
3. The bottom nav is the navigation: Home · Schedule · Orders · Team · Ask.
   The Ask tab is the AI agent, orange-highlighted. There is NO "Go to" section
   on the page — don't add one.
4. The AI agent also has a slim affordance bar at the top of the page.
5. Every number is read or refused — if a stat can't be computed, say which
   input is missing; don't render a blank or a hardcoded figure.
6. The 2-year-old test governs: one obvious named next action, no jargon, no
   dead ends. This is Nick's standing bar.
7. Colors: neutral charcoal surfaces, CORAL accents only — NOT orange. Use
   coral #ff7f50 (primary) / #f0653e (deep). Do not make everything coral; it is
   an accent. No warm-tinted dark surfaces — keep surfaces neutral charcoal.

Show me the finished screen when it's done, and confirm it follows the 3-slot +
bottom-nav layout from the mockup.
```

---

## Notes for Nick

### What to change each time
- **SCREEN NAME** — swap in whatever you're handing over (Schedule, Orders,
  Team, etc.).
- **Spec + mockup paths** — point at the matching files. For the hub they're
  the three above. For a new screen, add its own spec/mockup.
- **Rules 1-7** — the hub rules are specific to the hub. For a different
  screen, keep rules 5-7 (read-or-refuse, 2-year-old test, colors) and adjust
  the layout rule (1) and the bottom-nav rule (3) to match that screen.

### Why this works
- **The mockups are the source of truth** — Claude can see exactly what you
  approved, so it stops improvising.
- **The rules are explicit** — especially "no Go-to section" and "3 slots max,"
  the two places it kept drifting.
- **"Don't redesign it, implement it exactly"** — the sentence that stops it
  from "improving" on your accepted design.

### Check these when Claude comes back
1. **The bottom bar is the only navigation** — no Go-to section snuck back in.
2. **The health strip is exactly 3 configurable slots** — not a fixed list, not
   more than 3.
3. **Every number is real** — no hardcoded figure, no blank stat.

### Related
- [[projects/APEX_HUB_DESIGN_2026-08-08]] — the accepted owner hub spec
- [[Inbox]] · [[hot]] · [[NOW]] · [[index]]
