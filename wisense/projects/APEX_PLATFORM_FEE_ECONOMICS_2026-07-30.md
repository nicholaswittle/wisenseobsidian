> ## ✅ RESOLVED 2026-07-30 — direct charges, confirmed on the ledger
>
> `create-guest-payment` now creates the Checkout Session on the venue's account
> (`stripeAccount` request option), with no `transfer_data` and no
> `on_behalf_of`. Measured immediately after, on a $34.94 order, the **entire**
> platform-side ledger is:
>
> ```
> application_fee  +52   fee: 0   net: +52
> ```
>
> One entry. No charge, no transfer, no processing fee — the charge lives on the
> connected account, which now bears Stripe's cost. Compare the same ledger for
> a $26.47 destination charge hours earlier: `+2647 / −107 fee / −2647 transfer
> / +40` = **−67¢**.
>
> WiSense now keeps its full 1.5% and pays nothing. Stripe computes the real
> method-specific fee itself, so there is no rate table to keep in step and the
> Klarna/Amex divergence problem disappears rather than being approximated.
>
> Everything below is retained as the record of how it was found and why the
> obvious fixes did not work.

---
type: finding
title: "Apex — the 1.5% platform fee loses money on every order"
tags: [stripe, connect, pricing, economics, apex-v2, finding]
date: 2026-07-30
status: needs-decision
supabase_ref: "pqkremkwfkudrhtxasdj"
stripe_account: "acct_1TqfSbHeXj7HLVbu (sandbox)"
---

# The 1.5% platform fee loses money on every order

Verified against Stripe's settled balance ledger for a real test order, not from
documentation or reasoning.

## The order

$14.82 guest order at Jigsy's, paid online, Connect destination charge to
`acct_1TyjMwQn0WTytQVU`.

| Platform balance entry | cents |
|---|---|
| charge received | +1482 |
| Stripe processing fee | **−73** |
| transfer to venue | −1482 |
| application fee (1.5%) | **+22** |
| **net to WiSense** | **−51** |

The venue receives $14.60. WiSense collects 22¢, pays Stripe 73¢, and is **51¢
down on the order**.

## Reproduced on a second order

| order | platform fee in | Stripe fee out | **platform net** | venue nets |
|---|---|---|---|---|
| $14.82 | +22 | −73 | **−51** | $14.60 |
| $26.47 | +40 | −107 | **−67** | $26.07 |

The formula below predicted −67¢ on $26.47; the ledger says −67¢.

The connected account's own dashboard view of the second order shows **no
processing fee line at all** — $26.47 in, 40¢ application fee out, $26.07 net.
The venue is not being charged for processing, which is the clearest statement
of who is.

**Not a sandbox artifact.** Test mode calculates fees with live rules: 2.9% ×
$26.47 + 30¢ = $1.0677 → the 107 in the ledger. A skipped fee would be 0, not a
precisely correct 107. `controller.fees.payer` is an account property identical
in live, and Stripe's rejection message when the change was attempted quotes a
live API rule.

## Option A as originally written is impossible

Stripe rejects it:

> When `stripe_dashboard[type]=express`, your platform must collect fees and be
> liable for negative balances or refunds and chargebacks.

Express onboarding and venue-pays-processing are **mutually exclusive**. The
light onboarding flow is bought with platform fee liability. So the real choice
is narrower than first written:

- **Full-dashboard (Standard) connected accounts** — venue gets a real Stripe
  login, pays processing, owns disputes. Heavier signup.
- **Direct charges** — charge created on the venue's account with an
  application fee taken off it. Venue pays processing natively, onboarding stays
  light. Changes `create-guest-payment`, not onboarding, and changes the
  customer's statement descriptor.

Attempting the change broke onboarding twice on 2026-07-29 (rejected params, and
a stale pinned `apiVersion`); reverted to Express. **Neither remaining option is
a flag — both change what a venue signs up for.**

## It does not improve with volume

Platform fee is 1.5%. Stripe's is 2.9% + 30¢. Net per order of X cents:

```
0.015X − (0.029X + 30)  =  −0.014X − 30
```

Negative for every X, and **more negative as orders get larger**. There is no
order size at which 1.5% covers 2.9% + 30¢. Busier venue = bigger loss.

## Why

`stripe-connect-onboard` creates accounts with:

```js
stripe.accounts.create({ type: "express", ... })
```

A plain `type: "express"` account defaults to `controller.fees.payer =
"application_express"` — **the platform pays Stripe's fees**. Confirmed on
Jigsy's live account object. It also carries `controller.losses.payments =
"application"`, meaning **WiSense absorbs chargebacks** on food orders it never
touched.

**The 2026-07-28 audit
([[APEX_V2_STRIPE_CONNECT_AUDIT_2026-07-28]]) stated that adding
`on_behalf_of: destination` makes "Stripe processing fees (2.9% + 30¢) and
chargeback liability attach directly to the connected venue account."** The
ledger disproves the fee half outright. `on_behalf_of` sets the settlement
merchant — statement descriptor, country — it does not move who pays.

## Options

**A. Venue pays Stripe's fees** (what most marketplaces do). Create accounts
with `controller: { fees: { payer: 'account' }, losses: { payments: 'stripe' },
stripe_dashboard: { type: 'express' } }` instead of `type: 'express'`. On the
same order the venue nets $13.87 and WiSense keeps the full 22¢.

**B. Raise the platform fee** to cover Stripe — needs ~4.5%+ to earn anything,
which is a very different pitch from "1.5%".

**C. Add a guest-facing service fee** covering processing, keeping the venue's
1.5% story intact. Common in food delivery; visible to the customer.

## Constraint that forces the decision early

**Controller properties are immutable after account creation.** A venue
onboarded under the current settings can never be switched to option A — it
would have to disconnect and re-onboard, redoing bank details and identity
verification. Jigsy's is a test account, so it is free to redo now. **Every
venue onboarded before this is decided is permanently on the losing
arrangement.**

## Recommendation

Option A, decided before any real venue onboards. It keeps the 1.5% pitch
honest, matches how marketplaces normally work, and is a change to one
`accounts.create` call. B and C are pricing changes; A is a configuration
mistake being corrected.

Not yet implemented — this is a business decision, not a bug fix.
