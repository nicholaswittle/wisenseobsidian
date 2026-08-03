---
title: Apex Multi-Vertical — Implementation Plan
tags: [apex, architecture, verticals, landscaping, build-doc, strategy]
date: 2026-08-02
---

# Apex Multi-Vertical — Implementation Plan

**Revised 2026-08-02 after a Fable code audit.** Every claim below is either
cited to a file the auditor opened or explicitly marked unverified. The first
draft of this plan was wrong in four material ways; the corrections are marked
⚠️ and the original estimates are struck rather than deleted.

Related: [[DECISIONS]], [[projects/APEX_V2_TEMPLATE_TO_PRODUCT_GAP_MAP_2026-07-31]],
[[NOW]], [[hot]]

---

## 0. Headline: your own Master Plan already said this

`docs/WiSense Restaurant OS Master Plan 2026-07-27.md` lines 108–160 already
prescribes vertical packs on a shared core, and states discipline #1:

> **"Terminology comes from venue config, never hardcoded"** (:148)

The shipped code violates it. It also states:

> **"Do not build vertical two until vertical one is paying"** (:156)

Vertical one has **one** customer against a target of three. That line was
written by you, before any of this conversation, and it is the strongest
argument in the document.

---

## 1. What the audit changed

### ⚠️ 1.1 The geofence does not exist

The single largest correction. Clock-in is a **daily-rotating wall QR** —
`lib/core/clock_qr.dart` (payload `APEX|<orgId>|<yyyy-mm-dd>|<digest>`),
consumed by `qr_scan_screen.dart` and `qr_wall_screen.dart`. A grep of all of
`lib/` for geofence / haversine / Geolocator / location found **no geolocation
code anywhere in the app**.

"Geofenced time clock" appears in the Master Plan (:122) and in three external
audits. It is aspiration, none of them checked, and this plan repeated it.

Consequences:
- The mapping row "geofenced clock-in → clock in at client property" **breaks**.
- Proposed schema change #3 was mislabeled. It is not a schema change, it is a
  **from-scratch feature**: background location permission, geocoding
  `site_address`, distance math, iOS review risk. 30–50 hours.
- Worse for landscaping specifically: **a wall QR is venue-physical by design.
  There is no wall at a client's property.**

### ⚠️ 1.2 Sizing was ~5x optimistic

| | First draft | Audited |
|---|---|---|
| Landscaping site only | ~1 week | **15–30 hrs (2–4 weekends)** ✅ close |
| Services vertical MVP | ~~3–4 weeks of evenings~~ | **150–250 hrs = 4–7 months of evenings** |
| Four schema changes | ~1 day | #1 and #2 an evening each ✅; #3 is the geofence feature; #4 depends on scope |

### ⚠️ 1.3 Payroll is **not** a vertical blocker — this inverts the first draft

The 2026-08-02 decision (hours and tips only, no computed pay) already solved
it. **An hours report reskins fine.** Tip credit and tipped OT only return as a
blocker if the money columns return. See [[DECISIONS]] and
`docs/PAYROLL_LITE_SCOPE_2026-08-02.md`.

Related drift worth knowing: `apex_payroll_register` **is not in any migration
on `main`** — it exists only on the live DB and `feat/payroll-export`.

### ⚠️ 1.4 The quote lifecycle cannot ride `online_orders`

The first draft proposed `payment_mode = 'quote'` on the existing order path.
It doesn't work. `online_orders` is sub-hour by construction:
`status default 'waiting'`, `pickup_minutes`, `accepted_at/rejected_at/completed_at`
(`20260729000000:114–134`). Escalation fires on "still WAITING"
(`notify-order-event:157`); support-agent thresholds are 10m/30m
(`venue-support-agent:102–105`); capacity pauses on orders/hour.

A requested → quoted → scheduled → invoiced machine over weeks shares **no
states, no timings, no payment shape**. It is a new table and a new state
machine. 40–70 hours with RLS, staff UI and notifications.

### 1.5 `restaurants` is the skeleton, not the costume — the deepest finding

`20260801800000_venue_auto_bootstrap.sql:2` — *"Every new organization gets one
restaurants row + restaurant_settings."* `venue_site_profile.venue_id` FKs
`restaurants(id)` (`20260809000000:9`). The public site RPC builds a `menus` key
unconditionally into the anon payload
(`20260810000001_get_public_venue_profile_highlights.sql:56`). The
`check-capacity` cron sweeps **every** `restaurant_settings` row with the
service role, ungated (`check-capacity/index.ts:52`).

**A landscaping org created today is a row in a table called `restaurants`,
with restaurant settings, swept hourly by a restaurant cron, serving an empty
menu in its public payload.** None of that is behind the module registry.

### 1.6 Half of change #1 already exists, at the wrong level

`venue_site_profile.vertical text not null default 'restaurant'` has existed
since `20260809000000:20`, and `site/lib/types.ts:85` carries it — consumed
only as **marquee filler text** (`site/components/Marquee.tsx:23`). There is no
`organizations.vertical`. So the column is right, the level is wrong.

### 1.7 `disabled_modules` may make the registry change unnecessary

`organizations.disabled_modules` already exists and **wins over everything**
(`20260730210000:41–44`). A services org with
`disabled_modules = {onlineOrdering, tipManagement, smartCapacity, noShowEngine}`
is expressible **today, with zero schema change.** `vertical` then only needs
to drive *defaults and labels*, not gating.

Favourable structure confirmed: `apex_org_has_module()` is one function with a
numeric tier ladder (`20260730210000:12–81`), mirrored to `_tierGrants`
(`lib/core/entitlements.dart:76`), and Flutter nav is a single declarative
`moduleRoutes` table (`lib/app.dart:218`).

What the registry **cannot** reach: enforcement is writes-only by design
(:8–10), and service-role paths never consult it — the capacity cron,
`venue-briefing`'s ungated `menu_items` 86-board query (`:86`), the anon public
profile RPC.

### 1.8 The label leak is smaller than feared

Raw count ~700–800 literals in `lib/` (order 272, restaurant 126, venue 122,
tip 117, menu 96, guest 45, kitchen 26), concentrated in
`lib/features/ordering/*` (243 alone). **But most of it sits inside modules
that would be OFF for services.** The shared core a services user actually sees
— Schedule, Team, Chat, Time Clock, Log Book, nav labels, auth/onboarding — is
plausibly **100–150 strings: days, not weeks.**

A noun map covering only what a services org can see is cheap. Covering the
whole app is weeks and mostly wasted.

### 1.9 A shift is one person; there is no job entity

Shifts are written as `{organization_id, shift_date, staff, start_time,
end_time, role, zone, title, day_num}` (`schedule_screen.dart:525–535`). Adding
`customer_id` / `site_address` is additive and fine — but **"a crew at a job" is
N rows sharing a job, and no job entity exists to share.** `zone` (free text) is
the nearest existing concept.

### 1.10 `menu_items.price_cents` is NOT NULL

`check (price_cents >= 0)` (`20260729000000:71`). "Call for estimate" is
literally unrepresentable. Needs a nullable price or a `price_type`.

---

## 2. 🔴 Fix now, regardless of verticals

**`notify-order-event` hardcodes the literal string "Jigsy's" in every SMS
body** — `index.ts:157, 158, 176, 182, 185`, e.g.
`` `Jigsy's order #${code} is ready for pickup...` ``.

The copy is not venue-parameterized, let alone vertical-parameterized. **This
breaks at restaurant customer #2** — which is the actual year-one goal — and it
has nothing to do with landscaping. The user-visible SMS surface is small
(this file plus the escalation SMS in `venue-support-agent/index.ts:219`), so
it is a cheap fix with a hard deadline.

---

## 3. The justified scope right now

Everything else is deferred until vertical one is paying.

| Do | Effort | Why it survives |
|---|---|---|
| **De-hardcode "Jigsy's" from SMS** | hours | Blocks restaurant customer #2. Nothing to do with verticals |
| **`organizations.vertical`** | an evening | Additive; also backfill `venue_site_profile.vertical` for consistency |
| **`shifts.customer_id` + `site_address`** | an evening | Additive, nullable, forecloses nothing |
| **Landscaping site for the brother-in-law** | 15–30 hrs | Exercises the `vertical` field that already exists; the renderer already has section toggles (`20260809000000:19` defaults `{"menu":true,...,"order_cta":true}`) |

The site work: a services theme (new hero/about/service-list treatment, hide
`PickupInfo` and `OrderCta`), a **dumb quote-request contact form** — not the
quote lifecycle — and onboarding his org through `apex_onboard_venue` or by
hand.

A static one-pager in an evening also works and is cheaper. It buys none of the
platform learning, which is the only reason to prefer the renderer.

---

## 4. Deferred

Quote-request state machine · geofenced clock-in (a new feature, not an
adaptation) · job entity for crews · customer/client records · recurring jobs ·
estimate → invoice · materials and markup · route optimisation · HVAC entirely
(dispatch, parts inventory, service agreements ≈ 0% built — **landscaping is
dramatically closer than HVAC; do not treat "field services" as one market**).

---

## 5. What would make this fail

1. **The trigger customer is a $0 website customer, not an OS customer.** A
   landscaping *site* validates nothing about a services *vertical* — nobody is
   asking for crew scheduling, quotes or clock-in. Building the MVP for him is
   building a platform for zero paying users, which Master Plan :158 names as
   *"exactly how the dead projects died."*
2. **The quote lifecycle is a second product wearing the first one's tables.**
   Every reuse shortcut — quotes into `online_orders`, jobs into `shifts` —
   recreates the identity-by-name pattern the `20260802*` migration series just
   paid down. Semantic debt, with triggers and fail-open policies as interest.
3. **Solo maintenance fork.** Crons, webhooks, the support-agent prompt, SMS
   copy and onboarding all become two-branch code. Every restaurant fix at 7pm
   carries a "did this break services?" tax, with no second engineer.
4. **Safety debt compounds.** The fail-open sidework policy, name-keyed shifts,
   and `time_entries.shift_id ... ON DELETE CASCADE` (*unverified in repo — the
   table predates these migrations*) are all worse in a vertical where **the
   shift row is the billing record**.
5. **He is family and will not tell you it is not landing.** Measure logins and
   quote requests, not enthusiasm.

---

## 6. Restaurant assumptions that are not behind any button

Beyond §1.5 — verified, and each needs a vertical variant or a gate before a
services org exists:

- **`venue-support-agent` SYSTEM_PROMPT** (`index.ts:77–129`): *"restaurant
  operations app"*, *"standing in the kitchen at 7pm"*, printers, iPads, Square,
  capacity auto-pause, `resume_ordering`. A landscaper asking it anything gets
  restaurant physics back. Its allowlisted repairs (`apex_support_action`) are
  all ordering repairs.
- **`venue-briefing`** composes the 86 list from `menu_items` (`:70–101`),
  service-role, ungated.
- **Ordering payment rail** — `square-webhook`, `stripe-os-webhook`,
  `reconcile-pending-payments`, `refund-order`, `create-guest-payment` and their
  pg_cron schedules (`20260808000001`) — all keyed to
  `online_orders` / `restaurant_settings`. Dormant for a services org, but running.
- **`parse-menu`** — restaurant-only by name and purpose.
- **`apex_onboard_venue`** (`20260812000000`) writes exclusively to
  `restaurants` + `venue_site_profile` + `venue_hours`. **The onboarding path
  itself assumes the vertical.**

---

## 7. Branding

`apex_v2` is a repo name; the product is **Apex**. One product, one codebase,
one price sheet — **two landing pages**, because "software for landscapers"
converts and "software for independent businesses" does not. Copy currently
says "independent venues" throughout, and a landscaper is not a venue; the
pitch needs its own neutral noun before anything is printed.
