---
title: Apex v3 — Remaining Modules Roadmap
tags: [apex, v3, roadmap, modules, build-plan]
aliases: [Remaining Modules, v3 Gap List, Build List]
date: 2026-08-08
---

# Apex v3 — Remaining Modules Roadmap

What v2 has that v3 has not built yet, in build order with the dependencies
that force the order. Source: `apex_v3/docs/v2_module_inventory.md` (what each
v2 module actually is, from Nick's screenshots).

## Already built in v3

scheduling · shiftSwaps · onlineOrdering (orders) · pushNotifications ·
serviceRequests (the quote/jobs pipeline) · the operations layer · team /
entitlements · onboarding · the hub (owner + employee + services, locked).

## The gap — remaining modules

### Tier 1 — the daily loop (the money/retention core). Build first.

**1. Time clock** — nothing in v3 punches a clock yet. The hub even says "off
the clock" only when there's a clock to ask. **Everything below depends on
this.** Clock in / out, open punches, the punch record.

**2. Labor Cost** — hours and what they cost, scheduled vs actual. The $99/mo
killer feature. v2 does this BEST: it refuses to show a % it can't compute,
explains why, and names what's missing ("Labor cost % needs a sales figure",
"Emily is still clocked in"). **v3 got this wrong** (the hardcoded 22.4%).
Copy v2's behavior exactly.

**3. Labor vs Revenue** — the ratio. Needs a sales figure the app doesn't have
(v2: "Enter today's sales"). Band (Healthy — under 25%). Same refuse/explain
discipline.

**Depends on:** time clock (1) → labor cost (2) → labor vs revenue (3). You
can't do the killer feature without punches first.

### Tier 2 — during service (restaurant)

**4. Smart Capacity** — the most mechanical, clearest module. Status Open/paused,
staff on shift, orders/hour vs max, capacity %, auto-pause toggle, min staff
stepper, max orders/hour stepper, pause button. **The events log is the good
part** — every automatic decision recorded with the numbers that caused it.

**5. Edit Menu / 86 Board** — menu management + marking items unavailable.

**6. QR Wall** — printable QR codes pointing at the guest link.

### Tier 3 — the money loop

**7. Tip Pool** — distribution across staff (v2 has this; PA compliance).
**8. My Tips** — employee's own tips, distinct from Tip Pool.

### Tier 4 — communication

**9. Team chat** — one crew channel, "Message the crew…" (no DMs, no threads).
**10. Call-Outs** — the "I can't make it" engine (`noShowEngine`/`route-callout`).
**11. Log Book** — shift notes (partially in v3's employee hub as "handoff note").

### Tier 5 — restaurant extras / not registry modules

**12. Sidework** — restaurant checklist.

### Deliberately NOT built (do not build)

- **Payroll Export** — never existed in v2 either (confirmed in the inventory
  doc). Skip.
- **Offline Mode / Multi-location** — later phases, not the launch loop.

## The dependency order

**Clock (1) → Labor Cost (2) → Labor vs Revenue (3)** is a hard chain — the
killer feature needs punch data. Capacity (4) is independent and well-defined.
Then money (7-8), then comms (9-11).

## Build discipline (from the locked design system)

- Every screen is an operation caller (D26/D28) — no direct DML.
- The "3 buttons max" rule inside each screen.
- The 2-year-old test: one named next action, no jargon, no dead ends.
- Read-or-refuse: a number that can't be computed says which input is missing.
- Charcoal + coral (#ff7f50 / #f0653e), no warm-tinted darks.
- Design to spec / mockups FIRST, then build — see [[projects/APEX_HUB_DESIGN_2026-08-08]].

Related: [[projects/APEX_HUB_DESIGN_2026-08-08]] · [[projects/APEX_CLAUDE_HANDOFF_PROMPT_2026-08-08]] · [[projects/APEX_V3_SESSION_2026-08-07]] · [[hot]] · [[NOW]] · [[index]]
