---
title: Payments and POS Strategy — Square vs Stripe
tags: [apex, payments, square, stripe, pos, research, jigsys]
date: 2026-07-31
---

# Payments and POS Strategy — Square vs Stripe

Research + build plan produced 2026-07-31. Implementation plan lives in the repo
at `apex_v2/docs/PAYMENTS_AND_POS_BUILD_PLAN_2026-07-31.md` (33 tasks, acceptance
tests). This note carries the **findings and decisions** — the things that would
be expensive to rediscover.

Related: [[customers/Jigsys Brewpub]], [[Apex v2 — Restaurant OS Build]], [[DECISIONS]]

---

## Why this happened

Jigsy's runs on Square — iPads in a Square Stand, a wired contactless reader, and
a **USB Star TSP143IIU** that prints their kitchen tickets. They said yes to the
website and online ordering. Apex was built on Stripe. The question was whether
to fork Apex for Square, and the answer was no: Square became a second payment
provider behind one interface (decided 2026-07-30).

That made "how does each rail actually work" a question worth answering properly
rather than from memory. Two research agents, one per rail.

---

## Findings that changed the build

Five corrections to what was believed before the research:

### 1. Square hosted checkout supports tipping — it is one boolean

`checkout_options.allow_tipping: true` on `CreatePaymentLink`, and it **defaults
to the merchant's own Square tip configuration**. The venue's existing tip screen
carries over with zero setup.

This had been scoped as an architecture project. It is a flag. Any Square venue
running without it loses **100% of online tips**.

### 2. Stripe has no online tipping at all

Not "hard" — absent. Terminal has native on-reader tipping with structured
`amount_details.tip`, but online the nearest thing is `custom_unit_amount`
("pay what you want"), which **forbids additional line items**. Food plus an
adjustable tip cannot coexist in one Checkout Session.

Every online Stripe tip is our own code: cart selector before redirect,
`tip_cents` column, tip on the ticket, tip in reporting.

### 3. The two rails collect tips at opposite ends of the flow

| | Square | Stripe |
|---|---|---|
| Where | Square's hosted page, after redirect | Our cart, before redirect |
| When we learn it | Webhook, post-payment | At `place_order`, pre-payment |
| Platform fee on tip | Excluded automatically | **Must be excluded explicitly** |

Do not unify them. The cart knows the provider, so it shows a tip selector for
Stripe and none for Square — otherwise Square guests get asked twice.

### 4. `Payment.reference_id` does NOT arrive on Square payment-link webhooks

`Order.reference_id` does not propagate to the Payment. Code comments claimed
Square "echoes this on every event" — false. The integration worked only because
the `square_order_id` fallback was doing all the work. The documented primary
path was dead code that looked alive.

### 5. Square Sandbox cannot test the value proposition

Sandbox does not support **Square POS or Square for Restaurants at all**, nor
application-fee reporting. So three core claims —
*the order reaches the POS*, *the ticket prints*, *the 1.5% lands* — are
**unverifiable until a real merchant connects**.

Consequence: sandbox de-risks the code, not the product. The first production
venue is the first real test.

---

## Two silent-failure traps

Both cost money without erroring:

- **`square_environment` defaults to `'sandbox'`**, and the fee is only attached
  in production. A production venue whose row was never updated takes real money
  while Apex collects **nothing**, with no error anywhere.
- **The Stripe webhook's charged-amount check** must compare against the *tipped*
  total once tips ship. Against a pre-tip figure, every tipped order is charged
  and then never marked paid — guest pays, kitchen never sees it.

---

## Economics — unresolved, and it matters

If WiSense is on Stripe's **"platform sets pricing"** Connect model, we pay
$2/mo per active account + 0.25% + 25¢ per payout + 0.25% of payout volume. On a
venue doing $3k/month paid out daily that plausibly consumes **half** the 1.5% —
net closer to **0.7%**.

Nothing in the code reveals which model we are on. Script written to check it:
`apex_v2/scripts/check_connect_pricing.mjs` (reads `controller.fees.payer`,
read-only). **Do not quote margin to anyone until this is answered.**

Also: Stripe **direct charges do not appear in platform exports** — only Reports,
Sigma, or per-account API reads. A revenue dashboard built on platform exports
will silently show zero.

---

## Where Stripe structurally cannot match Square

- **Printing.** Square's Order *is* the kitchen ticket; it prints on hardware the
  venue already owns with zero Apex code. Stripe has no printing story and never
  will. Realistic answer is Star CloudPRNT / Epson ePOS — printer polls an
  endpoint we host.
- **POS / in-person.** Square venues already have the counter hardware.
- **Order object.** Stripe has none; the legacy Orders API is deprecated. This is
  the one where owning it ourselves is *correct*, not a concession — the same
  staff console then works identically on both rails.
- **Installed hardware and muscle memory.** The deepest gap and the least
  technical.

This is precisely why the dual-rail decision holds: sell Square venues on their
own hardware, sell Stripe to venues that have none.

---

## Strategy — Apex replaces Square, in stages

Nicholas's stated endgame (2026-07-31). The Square rail is the **wedge**, not the
destination. Three walls, all hardware/platform rather than software:

1. **Their printer dies with Square.** The TSP143IIU is USB — only prints for
   whatever is plugged into it. A browser cannot address USB. Replacing Square's
   printing means **replacing the printer** (~$250–350 CloudPRNT). Only
   unavoidable hardware cost in the plan.
2. **Card-present needs a reader we own.** Stripe Terminal (S700 ~$349 /
   WisePOS E ~$249) or a native app. Terminal's SDK does not run in a browser.
3. **Offline.** Square POS keeps taking payments when wifi drops; a browser does
   not. For a Friday night this is what makes Apex a tool rather than a liability.
   This alone forces a native app.

Sequence: ship the Square rail → own the kitchen ticket (CloudPRNT) → own the
front of house (native + Terminal + offline) → the long tail (timeclock,
close-out, tax reports). Each step independently valuable and sellable.

**The recurring decision is when to go native.** Everything past step 2 hits the
same wall. Apple Developer account purchased 2026-07-31, which unblocks it.

---

## Struck from the backlog

**Tap to Pay (the SDK feature) — not deprioritised, removed.** *"Tap to Pay isn't
available on iPads."* Android excludes tablets; under Square for Restaurants it
is restricted to two Samsung phone models.

Jigsy's wired contactless reader is **card-present hardware inside Square POS** —
a different thing entirely, already working, and not ours to touch. Confusing the
two mis-sizes the roadmap by an order of magnitude.

---

## Status at end of day

- **Phase 0 complete** (`e2158e1` + review fixes `e1de685`) — tipping flag,
  webhook correlation, refund fee behaviour, idempotency keys, constant-time
  signature compare, stale-comment removal, dead `reverse_transfer` fallback
  removed.
- **Phase 1 in flight** (Codex) — `acc97a9`, `a207a8d`, `f54c650`, with a
  partial-refund migration mid-flight.
- ⚠️ **`allow_tipping` is live while `tip_cents` does not exist.** A rejected
  Square order currently refunds the food and keeps the customer's tip. No Square
  venue is live so it costs nothing today, but Phase 1 gates every Square venue.
