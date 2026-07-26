---
title: Jigsy's Ordering Platform — Square Findings and Business Model
tags: [jigsys, handoff, square, business-model, cloudflare, online-ordering]
date: 2026-07-26
status: active-development
supersedes: Jigsys Ordering Platform — Claude Handoff 2026-07-25
---

# Jigsy's Ordering Platform — Square Findings and Business Model (2026-07-26)

Supersedes the 2026-07-25 handoff. Two things changed materially: we established
what Square will and will not allow, and we settled the business model.

## The headline finding

**Square does not surface unpaid API-created orders to the seller.** From Square's
own Orders API documentation:

> "Orders with fulfillments appear on Square products (such as the Square Dashboard
> and Point of Sale application) **only after they're paid for**."
> — https://developer.squareup.com/docs/orders-api/fulfillments

A Square staff member restated this on the developer forum on **7 May 2026**,
answering a developer describing our exact payload: *"we don't currently support
creating unpaid orders via the API that appear on the POS."* Open Tickets cannot
be created through the API at all. **No product tier or subscription changes
this** — the gate is payment, not plan.

So the orders we had been pushing on Accept were invisible to the restaurant: not
on the register, not in Order Manager, not in the Dashboard order views. They were
verified as reaching Square via the API, which is what made this misleading — the
data was there, the seller just could never see it.

### Corollaries

- **Automatic per-order fee collection is impossible for pay-at-counter.**
  `app_fee_money` requires the payment to run through the Payments API. It does
  work on Terminal API (in-person, card-present) — but that needs a dedicated
  Square Terminal and puts our software in the path of their money.
- **Every competitor sidesteps this by taking the card online.** ChowNow,
  Owner.com, BentoBox, Popmenu, Slice all collect payment on their own platform,
  which is what makes the Square push work for them. Owner.com's help material
  calls online payment "a requirement of the Square API."
- **Square's own products can't do it either.** Square Online has no pay-at-pickup
  option, and Square Kiosk has an open, unimplemented "Pay at Counter" feature
  request. Toast avoids the problem only because Toast *is* the POS.
- Middleware (Chowly, Deliverect, Otter, ItsaCheckmate, Cuboh) only works because
  DoorDash/UberEats already collected the money — they inject already-paid orders.

## Decisions made

### 1. The Square order push was removed

It created records the restaurant could not see or charge against, accumulating
permanently OPEN in their account, with a double-counting risk once the cashier
rang the sale separately. The OAuth connection and token handling remain in the
code, dormant.

Rejected alternatives, recorded so they are not relitigated:

| Option | Why not |
|---|---|
| Take payment online | Solves everything technically. Rejected: this is Nicholas's wife's workplace and we will not put our software in the path of their money |
| Terminal API with `order_id` | Itemized, in-person, and app fees work. Needs a ~$299 Terminal that becomes mode-exclusive, and still means we drive their payments |
| Invoices API | **Verified working** in sandbox: order → customer record → draft invoice → publish → cashier settles via POS More → Invoices → Add payment, with no item re-entry. Rejected for now: writes a customer record per order into their directory and leaves unpaid invoices to manage |
| POS API deep link | Passes a total only — Square logs a "Custom Amount" with no itemization. Deep links are also brittle on iOS Safari (Nicholas had already hit this in a previous project) |
| Cash recording (`source_id: CASH`) | Low risk and itemizes correctly, but only covers cash |

### 2. No customer-facing fee

The $0.99 online ordering fee was removed. Customers pay food + tax exactly.
Charging a fee that funded nothing would only have made Jigsy's prices less
competitive. `feeCents` remains a per-restaurant setting for future clients.

### 3. Commercial model — Jigsy's is free

The pilot is free; cash tips at their discretion. In exchange we ask for social
proof: a testimonial, permission to use the name as a reference, a post at
launch, and ideally a short video of staff using it during a shift.

**The next restaurant pays** — quoted as setup/build (one-time) plus monthly
maintenance. A flat monthly rate fits better than per-order: it guarantees a
revenue floor in a slow month and is easier for an owner to accept than a meter.
Pitch against commission — DoorDash takes 15–30% per ticket; at ~20 orders a day
that is thousands a month.

Infrastructure cost is effectively $0 (Cloudflare Workers + D1 free tiers), so
restaurant #2 onward is close to pure margin. Breakeven for Nicholas is roughly
$100/month against AI/development cost.

## What the product is now

Order intake, staff console, and kitchen ticket. **Square is untouched.** Staff
ring up online orders at the counter exactly as they do phone orders — but the
order arrives legible, itemized, priced, and correct, with no mishearing and no
math. The app removes the phone call, not the ring-up.

This was a deliberate architecture choice and it is the right one: if the app
fails, Jigsy's loses online orders, which is annoying. Nobody's payment fails, no
drawer is off, no customer is stuck at the counter.

### Built since the last handoff

- **Repeating staff alerts** — a waiting order re-alerts every 30s until accepted
  or rejected, escalating after 2 minutes. Screen Wake Lock keeps the kitchen
  tablet awake. Alert tone plays through an `<audio>` element because browsers
  suspend an AudioContext in a hidden tab — which is exactly when the alarm
  matters.
- **Customer notifications** — chime + browser notification when an order is
  accepted or rejected, with the pickup estimate and amount due at the counter.
  Only reaches customers who keep the page open; SMS is the paid follow-on.
- **One-click Accept** — Accept completes the order outright (the owner's wife
  found a second "mark completed" trip too slow) and no longer forces a print
  dialog; staff print on demand.
- **"Didn't pay" no-show marking**, reversible.
- **Monthly totals**, rolled up from a `daily_totals` table rather than summed
  from loaded orders, so history survives however far back it goes.
- **Nightly pruning** — a Cron Trigger deletes orders older than 7 days at 05:00
  UTC (midnight Eastern in winter, 1am in summer, never while open). Order rows
  hold customer names and phone numbers; the restaurant keeps its printed tickets
  and the rollup keeps the numbers.

## Live state

- App: **https://jigsys-ordering-demo.wisense.workers.dev** (public, no login wall)
- `jigsys-ordering-demo.vercel.app` 307-redirects to it
- Cloudflare Worker `jigsys-ordering-demo`, D1 `jigsys-ordering`
- Repo `nicholaswittle/jigsysiteworking`, branch `agent/reusable-ordering-core`
- Docs in-repo: `CLOUDFLARE-DEPLOY.md`, `PRODUCTION-PLAN.md`,
  `OWNER-APPROVAL-CHECKLIST.md`

## Open items

1. **Owner conversation** — menu, prices, tax confirmation, and agreeing the free
   pilot in writing with an end/review date plus the social-proof exchange.
2. **Printer test** on the real Star TSP100 — the app's own ticket, since Square
   is no longer involved.
3. **Regenerate the exposed sandbox Square secrets/tokens** (they appeared in a
   working chat). Sandbox only, no access to any real account.
4. **iPad PWA** install and test on shop Wi-Fi.
5. SMS customer notifications — deferred until there is revenue.

## Related vault notes

- [[business/Jigsys Ordering Platform — Claude Handoff 2026-07-25]]
- [[business/Jigsys Ordering Platform — Claude Handoff 2026-07-24]]
- [[business/Jigsys Website [[Jigsys Website & Direct Ordering Master Plan Direct Ordering Master Plan]]
