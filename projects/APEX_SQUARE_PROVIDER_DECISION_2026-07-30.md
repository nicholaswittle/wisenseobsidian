---
type: decision
title: "Apex will support Square as a second payment provider"
tags: [apex-v2, square, stripe, payments, jigsys, decision]
date: 2026-07-30
status: decided
---

# Square as a second payment provider

**Decided 2026-07-30.** Apex keeps Stripe as the default and adds Square as a
second provider behind one interface. **Not a fork.** A `payment_provider`
column on the venue selects the adapter; everything above it — cart, orders,
staff console, receipts, guest page — stays identical for every customer.

## Why this came up

Jigsy's kitchen printer is a **Star TSP143IIU** — USB, plugged into a Square
Stand. They use **iPads only**. iOS prints via AirPrint, and a USB printer bound
to Square's app is not an AirPrint device, so the staff console's print path
(`printOrder()`, 80mm thermal CSS, "Accept & print") **cannot reach it**. That
work is built and correct; it just has nowhere to send a job at this venue.

So online orders would arrive on a screen and never reach the kitchen.

## The fork was proposed and rejected

The suggestion was to clone Apex for Jigsy and swap Stripe for Square, since
they are the proof-of-concept customer.

Rejected because:

- **It does not solve the problem.** What makes a ticket print is an **Order**
  existing in Square, not a payment. Orders can be created without Square
  touching the money, so a payments rebuild would leave the printing question
  exactly where it was.
- **Everything gets done twice.** In one evening: guest RLS fix, cart
  persistence through checkout, receipt screen, abandoned-order filter, menu
  nav. All of it would have needed doing in both codebases.
- **The proof of concept must run the product being sold.** A Square fork means
  the reference customer runs software no future customer will run, and every
  lesson from them applies to something not for sale.
- **Precedent in this workspace:** `wisense_new_horizon` vendored its own
  `wisense_core` / `wisense_ui`, diverged both directions, and reconciling now
  breaks the build. That fork also started as a sensible exception.

## Why Square is worth supporting anyway

Not for Jigsy's convenience — **for market access**. Square is the default POS
for small independent restaurants. "We work with what you already have" is the
difference between sellable and unsellable to a large share of the venues
within driving distance. As a startup with no leverage, meeting customers where
they are is correct; the mistake would be doing it in a way that costs a second
codebase.

For a Square-native venue it is also *simpler* than the alternative: a Square
payment creates a Square Order, so **payments and kitchen printing are solved by
one integration**. Stripe plus a separate Orders bridge is two integrations for
the same outcome.

## Verified before deciding

| Question | Answer |
|---|---|
| Can a platform take a cut on Square? | Yes — `app_fee_money`, up to 90% of the payment, credited to the developer account. Needs OAuth scope `PAYMENTS_WRITE_ADDITIONAL_RECIPIENTS`; developer account must match the seller's country and currency. |
| Does it work on **hosted** checkout? | Yes — `checkout_options.app_fee_money` on `CreatePaymentLink`. **No embedded card form, no PCI surface of our own**, same redirect-and-return shape the site already handles. |
| Do API-created orders auto-print? | Yes, **but only if configured**: POS → Settings → Hardware → Printers → Profiles → Edit → toggle **"Online & Kiosk order tickets"**, and the fulfillment type (Pickup) must be attached to that profile. Documented failure mode is the order appearing on the POS and never printing. |

## ⚠️ Corrected mid-thread: Square does not avoid the owner

An early argument for Square was that Emily could authorise it herself. **She
cannot — she is a manager, not the owner, and has no power in Square.** The
owner is required either way.

What survives is the *size* of the ask:

- **Stripe** — the owner creates a new financial account: legal name, DOB,
  SSN/EIN, bank details, identity verification. The kind of ask that gets
  postponed for weeks.
- **Square** — the owner approves an app on an account that already exists. No
  new information, no new account to monitor.

Square is still the easier yes. It is not the effortless one.

**Plan one proper conversation with the owner**, with the printer answered, the
fee model clear (the guest pays the 1.5%, the venue pays only card processing —
the same as a card at their counter), and one specific thing to click at the
end. That meeting is where online ordering happens or drifts.

## Build order

1. **Extract the payment provider interface** — start checkout, confirm, refund,
   report platform fee. Four operations. Do it while there is one provider and
   it is freshly tested; retrofitting later costs far more.
2. **Square adapter** — OAuth connect, `CreatePaymentLink` with `app_fee_money`,
   webhook, refund.
3. **`payment_provider` on the venue**, defaulting to `stripe`.
4. **Prove with Jigsy** — printer profile toggle, then a real order end to end,
   including from a signed-out phone (see
   [[projects/APEX_PHOTO_IMPORT_BAKEOFF_2026-07-30]] for why that
   matters).

Several days, not an afternoon. Payment code is where mistakes cost real money.

## Fallback still worth keeping

If Square's auto-print proves flaky at a venue, a **network printer with
CloudPRNT** (Star TSP143IVUE, ~$250–400) lets the server push tickets directly —
no iPad, no Square, no dependency on anyone else's settings. Sell it as the
Apex station rather than a workaround.

## Related

- [[projects/SITE_TEMPLATE_ONLINE_ORDERING_PLAN]]
- [[projects/APEX_PLATFORM_FEE_ECONOMICS_2026-07-30]]
- [[projects/APEX_GO_LIVE_SEQUENCE_2026-07-30]]
