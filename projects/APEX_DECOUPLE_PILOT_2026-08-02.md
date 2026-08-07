---
title: Apex — Decoupling the Pilot Customer
tags: [apex, architecture, multi-tenant, jigsys, build-doc]
date: 2026-08-02
---

# Apex — Decoupling the Pilot Customer

Apex is multi-tenant underneath and single-customer on the surface. This is the
plan to fix the surface. **It is on the critical path to restaurant customer #2
— the actual year-one goal — and has nothing to do with verticals.**

Related: [[projects/Apex v2 — Restaurant OS Build]] · [[projects/APEX_MULTI_VERTICAL_PLAN_2026-08-02]], [[DECISIONS]], [[NOW]]

---

## 0. Two different jobs, do not conflate them

| | What it removes | Needed for | Size |
|---|---|---|---|
| **De-Jigsy** | one customer's specifics | restaurant customer **#2** | hours–days |
| **De-restaurant** | vertical assumptions | landscaping | weeks |

Only the first is urgent. The second is optional and gated on vertical one
paying (Master Plan :156).

**But they compose:** every hardcoded assumption removed in job one is one the
vertical work does not have to remove later. De-Jigsy is roughly the first
third of de-restaurant, and it is worth doing on its own merits regardless.

---

## 1. Tier 1 — Runtime bugs. Fix this week.

These break at customer #2. Verified by grep 2026-08-02.

### 1.1 🔴 `notify-order-event` — five hardcoded venue names

`supabase/functions/notify-order-event/index.ts`, lines **157, 158, 176, 182, 185**:

```
`Jigsy's: NEW order #${code} — ${guestName}, $${totalDollars}...`
`Jigsy's accepted order #${code}. Ready in ~${mins} min...`
`Jigsy's could not accept order #${code}: ${why}...`
`Jigsy's order #${code} is ready for pickup...`
```

Both the staff alert and **the guest-facing SMS**. Restaurant #2's customers
would receive texts naming a different brewpub.

**Fix:** the function already resolves the order → restaurant; pull the venue
name from that row. One lookup, five interpolations. **Hours.**

While in here, take the vertical-neutral phrasing too (`notify-order-event` is
one of only ~6 user-visible SMS bodies in the whole system) — it is the cheapest
moment to do it.

### 1.2 `site/app/page.tsx:10` — root page links to one venue

```tsx
<Link href="/jigsys-enola-7c2a" className="venue-btn venue-btn-ghost">
```

The renderer's root is a hardcoded shortcut to the pilot. Should be a real
index, a redirect, or nothing.

### 1.3 Verify — not yet inspected

- `site/lib/apex-links.ts`
- `site/components/ScrollIn.tsx`
- `scripts/monitor/venues.json` — probably legitimate config (a monitor needs
  targets), but confirm it is a list, not a constant.

---

## 2. Tier 2 — Migration hygiene. Matters the day you need a second environment.

Several migrations **hard-fail** when the pilot org is absent:

- `20260801400000_jigsys_full_board_menu.sql:46` — `raise exception 'jigsys restaurant missing'`
- `20260801410000_jigsys_menu_extras.sql:27` — same
- `20260729000000_ordering_platform.sql:364` — `select id into v_org from organizations where lower(name) = 'jigsys'`
- `20260831000001_seed_jigsys_tip_eligibility.sql:31,49` — `o.name = 'jigsys'`
- `20260814000000_rotate_jigsy_token_strong_tokens.sql` — pilot token rotation

**Consequence:** a clean database — staging, a test environment, a local
`db reset` — cannot migrate. Right now there is exactly one environment, so
this is invisible.

**Do not edit applied migrations.** The fix is forward-only: a follow-up
migration, or converting the hard `raise exception` cases to `raise notice` +
skip in a superseding file. The token-rotation and menu seeds are legitimate
one-time historical data — they should simply **no-op cleanly** when the org
is absent rather than aborting the run.

Worth doing before the first staging environment, not before customer #2.

---

## 3. Tier 3 — Leave alone

Comments naming the pilot as the worked example:
`lib/app.dart:155` · `lib/core/tip_split.dart:19` ·
`lib/features/team/venue_hours_screen.dart:9` ·
`lib/features/tips/tip_management.dart:358` ·
`20260802200000_venue_hours.sql:11` ·
`20260803000000_service_fee_paid_by_guest.sql:6` ·
`20260831000000_tip_pool_eligibility.sql:15` ·
`enrich-business/index.ts:24` (a curl example)

These explain *why* a design exists by citing the real case that produced it —
football season shifting venue hours, an Owner who works the floor and is
tip-eligible. That is good documentation. Keep it.

**Also fine:** `lib/core/name_matcher.dart` holds a **generic** nickname table
(`'sam': ['samuel','samantha']`) plus a never-guess-between-two-people rule.
Earlier notes describing it as "hardcoding the two-Sams case" are wrong — no
pilot staff data is in it.

---

## 4. Tier 4 — Structurally restaurant, not structurally Jigsy

Deeper than de-Jigsy, shallower than a vertical build. Listed for sequencing,
not for this week. All verified in the Fable audit:

- `20260801800000_venue_auto_bootstrap.sql:2` — every new org gets a
  `restaurants` row + `restaurant_settings`
- `apex_onboard_venue` (`20260812000000`) writes only to `restaurants`,
  `venue_site_profile`, `venue_hours`
- `check-capacity/index.ts:52` — sweeps every `restaurant_settings` row,
  service role, ungated by module or tier
- `venue-briefing/index.ts:86` — ungated `menu_items` 86-board query
- `venue-support-agent` SYSTEM_PROMPT (`:77–129`) hardcodes restaurant physics
  — kitchens, printers, iPads, capacity auto-pause

---

## 5. Decision: one app, not a clone — `RECOMMENDED`

The alternative considered was forking the repo into an `apex_services` clone.

**Rejected, for four reasons:**

1. **You have already lived this failure in this workspace.** The vendored
   `wisense_core` / `wisense_ui` divergence needed a dedicated reconciliation
   project, and Apex's `wisense_ui` fork is *still* an open decision
   ([[DECISIONS]] 2026-07-21). A second full application fork is the same bet
   at ten times the size.
2. **Every security fix must then be applied twice, by hand, forever, by one
   person on shift work.** The last week alone: 56 anon `EXECUTE` revokes, the
   identity migration, two tier-gate closures, two AI meters. A codebase in
   this project has already been found carrying a pre-migration function body
   days after the repair was recorded as applied — in *one* codebase.
3. **The shared 45% is precisely the hardened part** — auth, orgs, RLS,
   membership, payments, the module registry. Forking means forking the code
   you most need to be byte-identical across customers.
4. **Two store listings, two release trains, two TestFlight builds, two secret
   sets** — and it breaks the stated UX of picking a vertical at signup.

A clone would be defensible if services were a genuinely different product for
a genuinely different buyer. It is not: same buyer profile — a small local
business with hourly staff that needs a website and a schedule.

**Chosen shape:** one app, where the vertical is **data-driven configuration,
not a code branch.** Which is exactly what the decoupling work produces as a
by-product — remove every hardcoded customer and venue-type assumption and what
remains *is* a config-driven app. `organizations.disabled_modules` already
exists and wins over everything, so a services org is expressible with **zero
schema change** once the copy and onboarding stop assuming.

---

## 6. Order of work

| # | Work | Effort | Gate |
|---|---|---|---|
| 1 | `notify-order-event` venue name from the order row | hours | none — **do now** |
| 2 | `site/app/page.tsx` root link; verify the three unchecked files | hours | none |
| 3 | Vertical-neutral SMS phrasing while in §1.1 | included | none |
| 4 | Migration no-op-when-absent (Tier 2) | half a day | before first staging env |
| 5 | `organizations.vertical` + `shifts.customer_id` / `site_address` | an evening each | none — additive |
| 6 | Tier 4 de-restauranting | days | before any services org exists |
| 7 | Services vertical MVP | 150–250 hrs | vertical one paying |

Items 1–3 are the whole urgent list, and they are hours — smaller than the
"days" first estimated. **The gap to customer #2 is real but shallow.**
