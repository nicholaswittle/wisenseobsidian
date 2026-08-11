---
title: Jigsy's Pilot — Launch Strategy and On-Site Findings
tags: [jigsys, apex, square, pilot, launch, strategy]
date: 2026-07-31
---

# Jigsy's Pilot — Launch Strategy and On-Site Findings

On-site reconnaissance of Jigsy's live Square POS, 2026-07-31 evening, and the
launch sequencing it changes.

Related: [[customers/Jigsys Brewpub]], [[projects/APEX_PAYMENTS_AND_POS_STRATEGY_2026-07-31]]

---

## What was observed on their POS

**The printer profile is already correct.** "Lounge Printer" — a Star TSP143 on
USB, wired to the Square Stand — runs a profile that already includes
**"Online & kiosk order tickets"**, alongside in-person tickets, receipts and
void tickets. Status: Ready.

This was the single largest unknown in the Square rail. Orders Apex creates
through Square should route to that printer **with no change by the venue**.

**"Enable order creation in checkout" is OFF, and must stay off.** Earlier
guidance to enable it was wrong. Square's own description: *"Create orders for
in-store and curbside pickup. All fulfillment methods must be turned off in
Square Dashboard to enable this feature."* It governs the POS creating its own
orders, and turning it on requires disabling fulfilment methods — which is
exactly what Apex's PICKUP orders depend on.

**`Checkout → Order tickets` is set to `Manual`.** Unverified. This is the
closest thing to an auto-print control, and if it means a human taps to print
each ticket, the "it just prints" claim has an asterisk. Must be checked before
promising unattended printing.

**Square account owner: `jigsy895@yahoo.com`** — account "Jigsy's Inc", business
"Jigsy's Old Forge Pizza & Brewpub". OAuth must be authorised by this login.
Emily is a trusted manager with no Square authority.

Also present: sales tax configured (1 active), tipping configured under
Checkout, and a full beer/wine/liquor catalogue in Square — the online menu
deliberately excludes alcohol.

---

## What it changes

### Printing is free on Square, expensive on Stripe

On the **Square rail**, an Apex order prints on their existing hardware with no
purchase and no setup. On **Stripe** there is no printing path at all until a
~$300 CloudPRNT printer replaces the USB unit and the endpoint is built (2–3
days). Same product, same venue, wildly different cost.

### Phase 4 leaves the critical path

CloudPRNT was sequenced as a launch dependency. If the pilot runs on Square it
becomes work for the *next* venue — one that does not already have Square —
which is the actual reason to own printing. Weeks deferred, nothing lost.

### The ask to the owner gets much smaller

Not "adopt online ordering and change your setup" but "nothing changes on your
side; orders just start arriving from the website too."

---

## The floor: pay-at-pickup needs no integration at all

**This is the important strategic point.** `payment_mode` already supports
pay-at-pickup. Guest orders online → order lands in Apex → staff read it off the
new iPad → food gets made → customer pays at the counter on the Square setup they
already use.

No OAuth, no Stripe, no printer, no fee, nothing touching their money. The iPad
purchased 2026-07-31 **is** the ticket — it functions as a KDS, so there is no
printing dependency whatsoever.

What the floor buys: the only restaurant taking online orders within five miles,
zero risk to their existing operation, one click to stop, and a live reference
customer.

What it costs: **no platform revenue.** No card processed online means no 1.5%.
Worth naming so adoption is not mistaken for revenue — Jigsy's value right now is
being a real venue that says yes.

### Every upgrade is additive, not a redo

1. Pay at pickup, iPad as ticket ← **works today**
2. Add Square OAuth → their printer prints, cards online, fee starts
3. Later: Stripe or CloudPRNT, only if they ever leave Square

Nothing in step 1 is discarded at step 2 — same site, same orders table, same
staff console. It is flipping `payment_mode` and connecting an account.

**Therefore: launch on the floor, upgrade in place.** Do not gate going live on
OAuth, printing, or the owner's availability. This also inverts the sales
conversation — instead of asking permission for an integration, the ask becomes
"orders are coming in, want them to print automatically and let people pay
online?"

---

## What can honestly be promised

**Stays the same:** their printer, card reader, in-person checkout, POS, money
and deposits, tip settings (the hosted checkout inherits their config), sales
tax. No new hardware, no new Square fees, no migration.

**Genuinely new, and must be said:**

- **A new order queue** someone has to watch.
- **The menu lives in two places.** Apex holds its own copy of menu and prices;
  a change in Square does not reach the website. This is the one that quietly
  goes wrong in month three when a price drifts.
- **A one-time OAuth authorisation** by the owner.
- **The 1.5% platform fee** on their side of online orders, funded by the guest's
  service fee. They should hear it from us rather than notice it.

**Cannot be promised yet:** unattended printing (pending the `Order tickets`
check), and *anything* until OAuth — Square Sandbox structurally cannot test POS
delivery, printing, or application fees.

### The pitch that protects both sides

Do not promise it works. Promise it is reversible and cheap to find out:

> Let me connect it and place one test order while you watch. If the ticket
> prints and you like it, we keep going. If not, you disconnect it in one click
> and nothing about your setup has changed.

Verifiable in ten minutes, near-zero cost to say yes to, and it means not having
promised something that did not hold.

---

## Open items

- [ ] Capture `Checkout → Order tickets` submenu options
- [ ] Capture `Checkout → Tipping` configuration
- [ ] Owner authorises Square OAuth (`jigsy895@yahoo.com`)
- [ ] Verify `square_environment = production` on their settings row the day they
      connect — it defaults to `sandbox`, and the platform fee is production-only,
      so a misconfigured venue takes real money while Apex collects nothing

## Strategic tension worth holding

Leaning on their Square hardware makes the pilot nearly free and makes eventual
replacement harder — it deepens the dependency the long-term plan wants to
remove. Still the right call: getting a real venue live and proving the product
beats optimising for a migration not yet earned. Go in knowing it is the trade.

---

## Why Jigsy's is free — and why that is not the pricing model

Jigsy's pays nothing because they are the **proof of concept**. Free is the cost
of proving the stack works end to end in a real venue, not a trial, not a
discount, and not a precedent.

This matters because [[projects/APEX_V2_SELF_SERVE_OS_GAMEPLAN_2026-07-28]]
recommends **strict build-free / pay-to-publish** and explicitly rejects a
free-trial model as trial-abuse surface. Those two facts are compatible: Jigsy's
is a one-off outside the funnel. Venue #2 meets the paywall at Publish.

For reference, the tiers that already exist in `entitlements.dart` and which
Jigsy's is receiving at no charge:

- **Free ($0)** — scheduling, shift swaps, time clock, push
- **Pro ($25/mo)** — manager log, tips, **labor cost**, offline, chat
- **OS ($99/mo)** — **online ordering + KDS**, smart capacity, no-show engine,
  labor vs revenue analytics

Jigsy's is being given the OS and Pro tiers — $124/month of product — for the
duration of the proof. Emily has indicated she would advocate for payment to the
owner if it demonstrably helps the business; nothing is committed, and she has
no authority to commit it.

**The funnel fit is worth noting:** their Square report shows labor tracking at
**0.0%** and orders **−14%**. The free tier addresses the first with something
they have no substitute for; the OS tier addresses the second. The wedge is the
The free scheduling, not the ordering — ordering is the upsell.

---

## Update 2026-08-11 — separate Jigsy's website, launch prep underway

Jigsy's brought up the website and wants to start getting it ready for launch —
this is Nicholas's current priority and the driver behind getting v3 correct.

**Setup:** there is a **separate Jigsy's site on Vercel** that **v3 can also
control** — i.e. two coordinated surfaces: the Jigsy's storefront (being built
for launch) + v3 behind it (scheduling/timeclock/labor) once the pilot is live.

Known launch blocker (from NOW.md): `NEXT_PUBLIC_SHOWCASE_VENUE=jigsys-enola-7c2a`
not yet set in Vercel — unset, landing-page links render nothing. Set it before
launch.
