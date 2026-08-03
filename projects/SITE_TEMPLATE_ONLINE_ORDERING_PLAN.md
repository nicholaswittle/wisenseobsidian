---
type: plan
title: "Site template — online ordering & card payment built in"
tags: [plan, stripe, site-template, flagship, ordering, apex-v2]
date: 2026-07-30
status: planned
priority: after Apex v2
---

# Site template — online ordering built in

**Decision 2026-07-30:** do not wire Stripe checkout into Jigsy's site as a
one-off. Build it into the **site template**, so every flagship managed site
ships with online ordering already installed and a venue only needs their Stripe
account connected.

**Sequencing: finish Apex v2 first.** The template comes after.

> [!success] Jigsy is taking both — confirmed 2026-07-30
> This plan was written while it was still an open question whether they would
> take the website at all. They are taking the app **and** the site, so the
> template now has a committed first customer rather than a hypothetical one.
> The sequencing is unchanged: build it as a template, not a one-off, so every
> flagship site ships taking money on day one. That is the difference between a
> web project and a product.

---

## Current state

`projects/jigsys_site` is the working reference: a static site on Supabase with
its own guest ordering page (`ordering.js`) and staff console (`staff.js`),
hitting the same `place_order` RPC and the same tables as Apex.

**It has no Stripe at all** — pay-at-pickup only. Online payment currently lives
on the Apex guest link (`apex-v2-ten.vercel.app/?token=…`), which does not look
like the venue's business and is not where their customers are.

Everything the backend needs is already built and proven in production:

- `place_order` already accepts `p_payment_mode` — the site simply never passes it
- `create-guest-payment` takes `{ order_id, public_token }` and returns a
  Checkout URL
- Direct charges, the Connect webhook, refunds and the guest-paid service fee
  are all live and confirmed on the ledger (2026-07-30)

So this is front-end wiring against a tested backend, not new payment work.

---

## Scope

**In `ordering.js` (the guest page):**

1. Pay now / Pay at pickup toggle at checkout — no payment choice exists today
2. Service fee line in the totals, shown only for Pay Now, labelled
   **"Service fee"** (never "card fee" or "processing fee" — see the surcharge
   rules in [[STRIPE_FEE_ARCHITECTURE_RESEARCH_2026-07-30]])
3. Pass `p_payment_mode: 'pay_now'` to `place_order`
4. Call `create-guest-payment`, redirect to the returned URL
5. Handle the customer's return and show the success screen

**In `create-guest-payment` (the one real gotcha):**

`success_url` and `cancel_url` are hardcoded to `APP_URL` — the Apex app. A
customer paying from a venue's own site would finish checkout and land on
`apex-v2-ten.vercel.app`, not back on the venue's page.

Needs a per-venue return URL. **Do not accept one from the client** — that is a
textbook open redirect on a payment flow. Store the venue's site URL in
`restaurant_settings` and let the server decide, or validate against an
allowlist of known origins.

**Estimate:** ~2–3 hours including testing. Low risk; the payment path itself is
already proven.

---

## Why it matters commercially

- The 1.5% only earns anything where customers actually order. That is the
  venue's own site, not a link on our domain.
- It upgrades the flagship web offering from a brochure site to a storefront
  that takes money — a much easier sell at $299 + $99/mo.
- Every future venue gets it by default instead of as custom work.

---

## Related

- [[APEX_PLATFORM_FEE_ECONOMICS_2026-07-30]] — the fee model this earns on
- [[STRIPE_FEE_ARCHITECTURE_RESEARCH_2026-07-30]] — service fee vs surcharge rules
- [[APEX_GO_LIVE_SEQUENCE_2026-07-30]] — nothing from test mode carries to live
- [[APEX_SESSION_2026-07-29_30_FULL_RECORD]]
