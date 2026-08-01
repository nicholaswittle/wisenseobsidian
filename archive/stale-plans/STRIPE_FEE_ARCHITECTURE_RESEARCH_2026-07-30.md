---
type: research
date: 2026-07-30
tags: [stripe, connect, fees, pricing, research]
---

# Stripe Connect Fee Architecture — Who Absorbs Processing Cost

Research for WiSense/Apex: restaurant pickup ordering, Express connected accounts,
destination charges, 1.5% platform take rate, measured net loss per order.

**Epistemic note:** every claim below is tagged `[DOC]` (stated explicitly in Stripe
documentation, URL cited), `[DOC-ADJACENT]` (assembled from two or more explicit doc
statements, no leap), or `[INFERENCE]` (my reasoning, not stated anywhere). Section 9
lists what I could not determine at all. This file exists because prior inference
produced wrong answers — treat `[INFERENCE]` as a hypothesis to test, not a finding.

---

## 1. Summary of the mechanics

### 1.1 The measured behavior is documented and correct

The ledger is not a bug. Stripe's destination-charge doc states it directly:

> "Your account balance is debited for the cost of the Stripe fees, refunds, and chargebacks."

> "Your platform pays the Stripe fee after the `application_fee_amount` is transferred
> to your account."

— [docs.stripe.com/connect/destination-charges](https://docs.stripe.com/connect/destination-charges.md?platform=web&ui=stripe-hosted) `[DOC]`

And the settlement sequence, verbatim from the same page:

> "The full charge amount is immediately transferred from the platform to the connected
> account that's specified by `transfer_data[destination]` after the charge is captured.
> The `application_fee_amount` is then transferred back to the platform, and the Stripe
> fee is deducted from the platform's amount."

That is exactly the four-line ledger observed: `+1482 / −73 / −1482 / +22`. The platform
receives only the application fee but is debited the full processing cost. `[DOC]`

At 1.5% take rate against 2.9% + 30¢ cost, the platform loses on every order, and the
per-order loss grows with ticket size because the percentage gap (1.4%) is unbounded
while the 30¢ is fixed. `[DOC-ADJACENT]` — arithmetic over documented behavior.

### 1.2 Fee liability by charge type

| Charge type | Who pays Stripe processing fees | Who is debited for disputes |
|---|---|---|
| **Direct** (created on connected acct) | **Configurable** — connected account by default, or platform | Connected account |
| **Destination** (`transfer_data.destination`) | **Platform, always** | Platform |
| **Separate charges & transfers** | **Platform, always** | Platform |

Source, [docs.stripe.com/connect/charges](https://docs.stripe.com/connect/charges.md) `[DOC]`:

> "When using Direct charges, you can choose how Stripe fees are billed to your
> connected accounts."

> "Stripe debits fees from your platform account." *(stated for both destination charges
> and separate charges and transfers)*

> "Destination charges and separate charges and transfers typically use the platform's
> pricing plan and are assessed on the platform."

> "For disputes on payments created using direct charges, Stripe debits the disputed
> amount from the connected account's balance, not your platform's balance."

> "For disputes where payments were created on your platform using destination charges or
> separate charges and transfers, with or without `on_behalf_of`, your platform balance is
> automatically debited for the disputed amount and fee."

**The critical structural fact:** the fee-payer choice exists *only* for direct charges.
There is no configuration of any kind that shifts processing fees to the connected account
under destination charges. `[DOC]` — this is stated, not inferred.

For direct charges, [docs.stripe.com/connect/direct-charges](https://docs.stripe.com/connect/direct-charges.md?platform=web&ui=stripe-hosted) `[DOC]`:

> "When processing a charge directly on the connected account, the charge amount—less the
> Stripe fees and application fee—is deposited into the connected account."

> "For example, if you make a charge of 10 USD with a 1.23 USD application fee ... 1.23 USD
> is transferred to your platform account. 8.18 USD (10 USD - 0.59 USD - 1.23 USD) is
> netted in the connected account."

Note the platform receives its application fee **gross** under direct charges — the 1.23
arrives whole, the 0.59 comes out of the venue's side. That is the entire difference.

### 1.3 `on_behalf_of` — confirmed: does NOT affect fee liability

Refuted. `on_behalf_of` has no bearing on who pays processing fees. It controls
settlement-merchant presentation only. Verbatim from
[destination-charges](https://docs.stripe.com/connect/destination-charges.md?platform=web&ui=stripe-hosted) `[DOC]`:

> "The `on_behalf_of` parameter determines the settlement merchant, which defaults to your
> platform account. This affects:
> - Whose statement descriptor the customer sees
> - Whose address and phone number the customer sees
> - The settlement currency of the charge
> - The Checkout page branding the customer sees"

And the charges overview explicitly closes the dispute question: disputes hit the platform
"with or without `on_behalf_of`"
([charges](https://docs.stripe.com/connect/charges.md)). `[DOC]`

Removing or keeping `on_behalf_of` in the Apex integration changes nothing about the
economics. It does affect which entity's descriptor appears on the guest's card statement,
which has real chargeback-rate consequences — keep it set to the venue.
`[INFERENCE]` on the chargeback-rate part; the descriptor behavior itself is `[DOC]`.

### 1.4 `application_fee_amount` constraints

- Destination charges: "The `application_fee_amount` is capped at the total transaction
  amount." ([destination-charges](https://docs.stripe.com/connect/destination-charges.md?platform=web&ui=stripe-hosted)) `[DOC]`
- Direct charges: "must be positive and less than the amount of the charge ... capped at
  the captured amount ... There are no additional Stripe fees on the application fee
  itself." ([direct-charges](https://docs.stripe.com/connect/direct-charges.md?platform=web&ui=stripe-hosted)) `[DOC]`
- Refunds: "Application fees aren't automatically refunded when issuing a refund. Your
  platform must explicitly refund the application fee or the connected account ... loses
  that amount." ([direct-charges](https://docs.stripe.com/connect/direct-charges.md?platform=web&ui=stripe-hosted)) `[DOC]`

That last one is a live bug risk in Apex independent of this research: if refunds are
issued without `refund_application_fee`, the venue eats the platform's 1.5% on refunded
orders. Worth checking the current refund path.

---

## 2. Controller configurations and what each costs in onboarding friction

### 2.1 Correcting the parameter values used in the failed attempt

The attempted call used `losses: { payments: "stripe" }` together with
`fees: { payer: "account" }`. Per the API reference
([docs.stripe.com/api/accounts/create](https://docs.stripe.com/api/accounts/create)) `[DOC]`:

| Property | Valid values | Default |
|---|---|---|
| `controller.fees.payer` | `account`, `application` | `account` |
| `controller.losses.payments` | `stripe`, `application` | `stripe` |
| `controller.stripe_dashboard.type` | `full`, `express`, `none` | `full` |
| `controller.requirement_collection` | `stripe`, `application` | `stripe` |

Note `losses.payments` takes `application`, **not** `platform` — the docs' prose says
"platform is liable" but the enum value is `application`. Same for `requirement_collection`.

There are two further read-only values of `fees.payer` documented at
[direct-charges-fee-payer-behavior](https://docs.stripe.com/connect/direct-charges-fee-payer-behavior.md) `[DOC]`:

> "`application_custom` and `application_express` are assigned to accounts created with
> `type=custom` and `type=express`, respectively... You can't set the payer type to
> `application_custom` or `application_express` when you create connected accounts."

This is why the existing Apex accounts read as `application_express`: they were created
with the legacy `type: "express"` shorthand, which implies `fees.payer=application_express`.

### 2.2 The rejection message is a real, documented constraint

Stripe's error — *"When stripe_dashboard[type]=express, your platform must collect fees and
be liable for negative balances or refunds and chargebacks"* — matches the recommendation
table at [integration-recommendations](https://docs.stripe.com/connect/integration-recommendations.md) `[DOC]`:

| | Direct charges | Destination / separate charges |
|---|---|---|
| Dashboard | Full, Custom/Embedded, Express | Custom/Embedded, Express (**not** Full) |
| Negative balance liability | Stripe | Platform |
| Payment fee collector | Platform **or Stripe** | Platform only |

"Payment fee collector: Stripe" in that table corresponds to `fees.payer=account`, and it
appears **only** in the direct-charges column. `[DOC-ADJACENT]` — the mapping between the
table's prose label and the API enum is not spelled out on that page; I am matching them by
meaning. Treat the mapping as high-confidence but unconfirmed.

Also documented, [design-an-integration](https://docs.stripe.com/connect/design-an-integration.md) `[DOC]`:

> "When Stripe is responsible for negative balances on your connected accounts, you must
> integrate embedded components for onboarding, account management, and the notification
> banner."

So `losses.payments=stripe` is not free either — it obligates embedded-component work.

### 2.3 What Express / Standard / Custom mean under the controller model

Legacy `type` values are now shorthand for controller property sets
([migrate-to-controller-properties](https://docs.stripe.com/connect/migrate-to-controller-properties.md)) `[DOC]`:

- **Standard** ≈ `fees.payer=account`, `losses.payments=stripe`, `dashboard=full`,
  `requirement_collection=stripe`. Venue has a full Stripe account, own dashboard, own
  Stripe relationship, pays own processing fees, handles own disputes. Heaviest onboarding
  (full Stripe signup, venue accepts Stripe's ToS directly), lightest platform burden.
- **Express** ≈ `fees.payer=application_express`, `losses.payments=application`,
  `dashboard=express`. Stripe-hosted onboarding, limited co-branded dashboard, **platform
  pays fees and owns disputes**. This is the current Apex config.
- **Custom / none-dashboard** ≈ `dashboard=none`, `requirement_collection=application`.
  Platform builds everything, fully white-label, platform owns fees and losses.

### 2.4 Viable configurations for shifting fees to the venue

`[DOC-ADJACENT]` — assembled from §1.2 + §2.2, no single doc states the full matrix:

To get `fees.payer=account`, you need **direct charges**. Given direct charges, the
dashboard options documented as valid are Full, Express, or Custom/Embedded. The rejection
message says Express specifically forces platform-pays. That leaves:

- **Direct charges + `dashboard=full` (Standard)** — documented, clearly supported.
- **Direct charges + `dashboard=none` + `fees.payer=account`** — *not documented as valid
  and not documented as invalid.* See §9.

---

## 3. Can fee payer be changed after creation?

**No.** [direct-charges-fee-payer-behavior](https://docs.stripe.com/connect/direct-charges-fee-payer-behavior.md) `[DOC]`:

> "You can specify the fee payer only when you create an account."

Dashboard type likewise, [migrate-to-controller-properties](https://docs.stripe.com/connect/migrate-to-controller-properties.md) `[DOC]`:

> "You can't change the `controller.stripe_dashboard.type` of an existing connected
> account. To change a connected account's dashboard, you must create a new `Account`
> object."

**Migration path: there isn't a soft one.** Changing fee payer or dashboard type requires
creating a new `Account` and re-onboarding the venue from scratch — new KYC, new bank
account entry, new ToS acceptance. Existing account IDs must be remapped in the Apex
database and any stored `stripe_account_id` references updated. `[DOC-ADJACENT]`

The docs' reassurance that "you don't have to update any of your connected accounts" refers
only to migrating your *code* from `type` to controller properties — it does not enable
changing behavior on existing accounts. Do not misread it. `[DOC]`

---

## 4. Is `transfer_data.amount` viable?

**Technically yes, and it is documented.** [destination-charges](https://docs.stripe.com/connect/destination-charges.md?platform=web&ui=stripe-hosted) `[DOC]`:

> "The `transfer_data[amount]` is a positive integer reflecting the amount of the charge to
> be transferred to the `transfer_data[destination]`. You subtract your platform's fees
> from the charge amount, then pass the result of this calculation as the
> `transfer_data[amount]`."

Critically, the platform still pays Stripe:

> "Your platform separately pays the Stripe fees on the charge."

So the mechanism is: platform withholds (1.5% + processing cost) from the transfer, keeps
it in platform balance, and pays Stripe out of that. Net effect — venue economically bears
processing cost, Express onboarding preserved, no account migration. **This is the only
option that requires zero re-onboarding.**

### The disclosure problem

> "In Stripe-hosted dashboards or components such as the Stripe Dashboard or Express
> Dashboard, your connected account can't view the total amount of the charge. They only
> see the amount transferred." `[DOC]`

Compare with `application_fee_amount`, where per
[marketplace/tasks/app-fees](https://docs.stripe.com/connect/marketplace/tasks/app-fees)
the connected account **can** see both the total and the fee. `[DOC]`

That asymmetry is the whole risk. With `transfer_data.amount`, the venue's Express
dashboard shows a $22.32 payment on a $26.47 order with **no line item explaining the
$4.15**. The venue cannot reconcile against its own POS. This is not a Stripe violation —
it is a merchant-trust and potentially a contract-disclosure problem.

**Does Stripe consider it acceptable?** Stripe documents the parameter as a first-class
supported pattern with no warning against it. Stripe does *not* document any opinion on
disclosure obligations to the connected account. `[DOC]` on the first half; the second half
is an absence of documentation, not permission. See §9.

`[INFERENCE]` — if you use this, the venue agreement must state the effective deduction in
plain terms ("we deduct 1.5% plus card processing costs of 2.9% + 30¢"), and Apex should
surface a per-order fee breakdown in the venue's own Apex dashboard, since Stripe's won't.
Doing it silently is the version that gets you sued or churned.

---

## 5. The service-fee approach

**Supported, standard, and the least exotic option.** Stripe documents marketplaces taking
a cut via application fee as the core Connect pattern
([connect/marketplace](https://docs.stripe.com/connect/marketplace),
[marketplace/tasks/app-fees](https://docs.stripe.com/connect/marketplace/tasks/app-fees)) `[DOC]`:

> "When a payment is processed, rather than transfer the full amount of the transaction to
> a connected account, your platform can decide to take a portion of the transaction amount
> in the form of fees."

Stripe's own worked example is a food-ordering platform
([saas-platforms-and-marketplaces](https://docs.stripe.com/connect/saas-platforms-and-marketplaces)) `[DOC]`:

> "a food ordering app that splits payments between delivery drivers and restaurants, such
> as DoorDash ... the customer pays a 100 USD charge, Stripe collects a 3.20 USD fee, the
> platform transfers 70 USD to the restaurant and 20 USD to the driver, with a platform fee
> of 6.80 USD remaining."

Note that in Stripe's own example, the platform's 6.80 is stated **after** Stripe's 3.20 —
i.e. the platform's gross margin is sized to absorb processing. Stripe's canonical
marketplace economics assume the platform's take rate exceeds processing cost. A 1.5% take
rate against 2.9% + 30¢ is simply below the floor the pattern assumes. `[INFERENCE]` on the
"floor" framing; the numbers are `[DOC]`.

Apex already has a per-venue `fee_cents` on the order model flowing into the total. Raising
`application_fee_amount` to `1.5% + processing estimate` while adding a matching guest-facing
service fee to the order total is mechanically trivial and requires **no** account changes.

**Stripe-side disclosure constraints:** I found no Stripe documentation imposing checkout
disclosure requirements on a platform *service fee* (as opposed to a card surcharge —
see §6). The constraints that apply are legal and card-network, not Stripe-product.
See §9.

---

## 6. Legal and compliance — Pennsylvania (17025) and card network rules

**This section is the one that should drive the decision.**

### 6.1 The distinction that matters

- A **credit card surcharge** is a fee applied *because the customer paid with a credit
  card* — it varies by tender type. Heavily regulated.
- A **service fee** (or platform/convenience/ordering fee) is applied to *all orders on the
  channel regardless of payment method*. Not a surcharge, and largely outside the card
  network surcharge rules.

`[DOC-ADJACENT]` — this distinction is consistently described across Stripe's own resource
pages ([What is a surcharge fee](https://stripe.com/resources/more/surcharge-fees),
[credit card surcharges guide](https://stripe.com/resources/more/credit-card-surcharges-explained-what-businesses-need-to-know))
and industry sources, but I did not find a single authoritative legal citation that defines
the line crisply. The practical test used across sources: *if a cash-paying customer would
pay the same fee, it is not a surcharge.* Since Apex is online-only with card payment
exclusively, there is no cash comparison — see §9, this is a genuine gray zone.

### 6.2 Surcharge rules if you go that route

- **Pennsylvania has no state statute banning surcharging.** Permitted, subject to federal
  law and network rules. ([FlexPoint PA guide](https://www.getflexpoint.com/credit-card-surcharging-us-states/pennsylvania)) `[DOC]` (non-Stripe source)
- **Debit and prepaid surcharging is prohibited outright**, even when run as credit —
  Durbin Amendment. Stripe: "Surcharges apply only to credit cards, while debit cards,
  including prepaid cards, are excluded."
  ([Stripe surcharge guide](https://stripe.com/resources/more/credit-card-surcharges-explained-what-businesses-need-to-know)) `[DOC]`
- **Caps:** Visa 3%, Mastercard 4%, and in all cases capped at your actual cost of
  acceptance, whichever is lower. `[DOC]` (Stripe resource + industry sources)
- **Network notification:** Visa requires advance notice (commonly cited as 30 days).
  Stripe states: "Visa requires official notice before a business starts surcharging,
  though Mastercard no longer requires this." `[DOC]`
- **Disclosure:** Stripe's own terms —

  > "You must conspicuously disclose surcharge amounts that apply to each transaction and
  > the total costs to the cardholder ahead of purchase, and reflect the surcharge
  > separately on the transaction receipt." `[DOC]`

- Ten states restrict or cap surcharging (CA, CO, CT, FL, KS, ME, MA, NY, OK, TX per
  Stripe). PA is not among them — **but a multi-venue platform will hit those states**. `[DOC]`

### 6.3 The compliance verdict

`[INFERENCE]`, but strongly supported: **do not build a surcharge.** For a small platform
it means per-network registration, per-state rule tracking, debit/credit BIN detection to
avoid surcharging debit, cost-of-acceptance caps, receipt line-item requirements, and an
expansion blocker in ten states. A **flat service fee applied to every online order
regardless of tender** avoids essentially all of it.

Two hard requirements even for a service fee `[INFERENCE]`:

1. **Never label it "credit card fee," "card fee," "processing fee," or "surcharge."** The
   label is evidence of tender-based pricing. Call it a *Service Fee* or *Online Ordering Fee*.
2. **Never vary it by card type or payment method.** The moment it differs for debit vs
   credit, it is a surcharge with all the regulation attached.

Also: FTC junk-fee/drip-pricing enforcement and several state UDAP regimes push toward
showing the all-in total early rather than revealing fees at the final step. Disclose the
service fee on the cart page, not only at payment. `[INFERENCE]` — I did not verify current
FTC rule status for restaurant ordering; see §9.

---

## 7. What comparable platforms do

| Platform | Model | Who bears processing |
|---|---|---|
| **ChowNow** | SaaS subscription $119–$328/mo, 0% commission | **Restaurant** pays 2.95% + 29¢ processing on top of subscription ([Sauce: ChowNow pricing](https://www.getsauce.com/post/chownow-pricing-and-fees)) |
| **Toast** | SaaS + payments; online orders 3.50% + 15¢ | **Restaurant**, bundled into a blended rate above cost ([UpMenu Toast pricing](https://www.upmenu.com/blog/toast-pricing/)) |
| **DoorDash** (per Stripe's own doc example) | Commission ~20%+ | Platform absorbs processing out of a large take rate ([Stripe](https://docs.stripe.com/connect/saas-platforms-and-marketplaces)) |

**The pattern:** none of these run a 1.5% take rate absorbing processing. Every viable model
either (a) charges the merchant processing separately and explicitly on top of software
fees, or (b) prices a blended rate comfortably above cost, or (c) runs a take rate large
enough to swallow it. `[DOC-ADJACENT]`

Notably ChowNow — the closest analogue to Apex (commission-free direct ordering) — charges a
**monthly subscription plus pass-through processing to the restaurant**, and takes no
percentage of the order at all. `[DOC]`

`[INFERENCE]`: the 1.5%-and-absorb-processing model is not one anyone in this market runs,
which is itself the strongest signal.

---

## 8. Comparison of viable options

| | **A. Service fee (guest pays)** | **B. `transfer_data.amount`** | **C. Direct charges + Standard** | **D. Raise take rate to ~4.5%** |
|---|---|---|---|---|
| **Code change** | Raise `application_fee_amount` to 1.5% + processing estimate; add service fee to order total via existing `fee_cents`; show line item at checkout | Replace `application_fee_amount` with computed `transfer_data.amount`; build venue-facing fee breakdown UI (Stripe's won't show it) | Rewrite charge creation to direct charges (`Stripe-Account` header), move `application_fee_amount` semantics, rebuild webhook handling on connected-account events | One-line constant change |
| **Venue onboarding change** | **None** | **None** | **Full re-onboarding.** New `Account` objects, new KYC, new bank details, remap all `stripe_account_id`. Venue signs up with Stripe directly | **None** |
| **What the guest sees** | Subtotal + tax + "Service Fee $0.73" | Nothing — total unchanged | Nothing | Nothing (higher prices if venue passes through) |
| **What the venue sees** | Order total; 1.5%+ application fee deducted, visible in Express dashboard | **Only the net transfer.** No fee explanation anywhere in Stripe UI | Full Stripe dashboard, own fees itemized, own payouts | Application fee visible, but 4.5% is a hard sell |
| **Who bears disputes** | Platform | Platform | **Connected account** | Platform |
| **Existing accounts migrate?** | ✅ Yes, no action | ✅ Yes, no action | ❌ **No** — immutable, requires new accounts | ✅ Yes |
| **Express onboarding preserved?** | ✅ | ✅ | ❌ Standard/full dashboard | ✅ |
| **Legal exposure** | Low **if** flat and tender-neutral (§6) | Low — B2B contract terms, not consumer law | Lowest | Low |
| **Kills the sales pitch?** | Slight — guest sees a fee | No | **Yes** — Express is the selling point | **Yes** — 4.5% vs ChowNow's 0% |

---

## 9. Recommendation

**Primary: Option A — guest-facing service fee, sized to cover processing plus margin.**

Rationale:

1. It is the only option that fixes the economics, preserves Express onboarding, requires no
   account migration, and does not degrade the venue pitch. `[DOC-ADJACENT]`
2. Apex already has `fee_cents` per venue flowing into the order total — the plumbing exists.
3. It is the pattern Stripe documents as canonical for marketplaces
   ([marketplace/tasks/app-fees](https://docs.stripe.com/connect/marketplace/tasks/app-fees)). `[DOC]`
4. Guests already expect a service fee on restaurant ordering apps; venues are highly
   sensitive to their own margin and much less to a guest-side line item. `[INFERENCE]`

**Sizing** `[INFERENCE]`: to net 1.5% after Stripe's 2.9% + 30¢, the service fee needs to be
roughly `0.30 + 0.029 × total`, i.e. ~73¢ on a $14.82 order and ~$1.07 on $26.47. A flat
$0.99–$1.49 online ordering fee is simpler, more legible to guests, and profitable across
typical pickup ticket sizes — and flat is also the safest posture legally, since it can't be
read as proportional to card cost.

**Hard constraints on implementation** (from §6):
- Label it "Service Fee" or "Online Ordering Fee". **Not** "processing fee," not "card fee."
- Flat or order-size-based — **never** varied by card type or payment method.
- Disclose on the cart page, itemized, before the payment step.
- `application_fee_amount` = service fee + 1.5% of subtotal.

**Do not do Option C** (direct charges + Standard). It is the "correct" answer architecturally
and the wrong answer commercially: fee payer and dashboard type are both immutable
([direct-charges-fee-payer-behavior](https://docs.stripe.com/connect/direct-charges-fee-payer-behavior.md),
[migrate-to-controller-properties](https://docs.stripe.com/connect/migrate-to-controller-properties.md)) `[DOC]`,
so every existing venue must re-onboard from zero, and you'd trade away the lightweight
Express onboarding that is the product's differentiator.

**Consider Option B as a supplement, not a substitute.** If some venues refuse a guest-facing
fee, `transfer_data.amount` lets you absorb it on their side per-venue with no re-onboarding —
but only with an explicit contract term and an Apex-side fee breakdown, since Stripe's venue
dashboard will show them nothing.

**Fix independently of all of this:** verify the refund path passes `refund_application_fee`,
or the venue silently eats the platform's cut on every refund
([direct-charges](https://docs.stripe.com/connect/direct-charges.md?platform=web&ui=stripe-hosted)). `[DOC]`

---

## 10. What I could NOT determine from documentation

Listed explicitly because inference here has already cost two deploys.

1. **Whether `fees.payer=account` is permitted with `stripe_dashboard.type=none`.** The
   rejection message covers `express` only. `integration-recommendations` never mentions the
   `none` dashboard in its table. No doc states this combination is either valid or invalid.
   **Only resolvable by attempting the API call in test mode.**

2. **Whether "Payment fee collector: Stripe" in the integration-recommendations table maps
   exactly to `controller.fees.payer=account`.** The prose label and the API enum are never
   connected on a single page. High-confidence match by meaning; unverified.

3. **Whether direct charges + Express dashboard + `fees.payer=application_express` still
   nets the platform its application fee gross.** The direct-charges worked example doesn't
   state which fee-payer setting it assumes. Plausible that direct charges alone fix the
   economics without any account migration — **this would change the recommendation if true
   and is the single highest-value thing to test.** Test it before building anything.

4. **Any Stripe policy on disclosing `transfer_data.amount` deductions to connected accounts.**
   Stripe documents the parameter and documents that the venue can't see the total. It says
   nothing about whether you're obligated to tell them. Silence is not permission.

5. **Whether Stripe's Connected Account Agreement imposes disclosure duties on the platform
   regarding deductions.** I did not read the full CAA text; this should be checked before
   choosing Option B.

6. **Whether an online-only ordering platform's service fee can be legally characterized as
   a surcharge**, given there is no cash-paying customer to compare against. The
   "tender-neutral" test that everyone cites assumes a mixed-tender environment. This is a
   genuine gray zone. If Apex ever adds pay-at-pickup, the answer gets easier (the fee must
   then apply to cash orders too). **Worth a payments-attorney question, not more research.**

7. **Current FTC junk-fee rule applicability to restaurant online ordering.** I did not verify
   the rule's present status, scope, or whether food ordering is covered. Cart-page disclosure
   is prudent regardless, but do not treat my §6.3 note as a compliance determination.

8. **Exact Stripe processing rate on the Apex account.** All arithmetic above assumes standard
   US 2.9% + 30¢, consistent with the observed −73 on $14.82 (2.9% × 14.82 + 0.30 = 0.730).
   Confirm against the actual pricing plan before sizing the service fee.

9. **Whether Accounts v2 (`/v2/core/accounts`) relaxes any of the v1 immutability
   constraints.** The v2 configuration docs surfaced in search but I researched against v1
   throughout, matching the current integration. If migration is ever on the table, re-check
   v2 first.

---

# ADDENDUM (same day) — Direct charges on existing Express accounts

## ⚠️ THIS OVERTURNS THE §9 RECOMMENDATION

**Hypothesis (A) is correct and it is stated explicitly in a Stripe documentation table, not
inferred.** For an account carrying `controller.fees.payer = application_express`, a **direct
charge** debits Stripe's payment processing fee from the **connected account**, not the
platform. `[DOC]`

Open item #3 in §10 — flagged there as "the single highest-value thing to test" — is resolved
by documentation. It did not require an empirical test after all.

**Consequence:** the platform can stop losing money on every order by switching from
destination charges to direct charges, with **no re-onboarding, no new `Account` objects, no
guest-facing service fee, and no fee-estimation matrix.** The service fee recommended in §9 is
no longer necessary to reach breakeven. Read §A.5 before acting — the switch is a real
integration change and carries two specific gotchas.

---

## A.1 The controlling evidence

Source: [docs.stripe.com/connect/direct-charges-fee-payer-behavior](https://docs.stripe.com/connect/direct-charges-fee-payer-behavior)
("Fee behavior on connected accounts"). Reproduced verbatim, abridged to the rows that matter:

| Product category | `account` | `application` | `application_custom` | `application_express` |
|---|---|---|---|---|
| **Stripe payment processing fees** | Connected Account | Platform | Connected Account | **Connected Account** |
| **Interchange Plus Payment Fees** | Connected Account | Platform | Platform | **Platform** |
| **Dispute fees** | Connected Account | Platform | Connected Account | **Connected Account** |
| **LPM Payment Failure Fees** | Connected Account | Platform | Connected Account | Connected Account |
| **Radar** | Connected Account | Platform | Varies | **Varies** |
| **3D Secure** | Connected Account | Platform | Varies | Varies |
| **Stripe Tax** | Connected Account | Platform | Platform | Platform |
| **Instant Payouts** | Connected Account | Platform | Platform | Platform |
| **Card Account Updater** | Connected Account | Platform | Platform | Platform |
| **Stripe currency conversion fee** | Connected Account | Platform | Connected Account | Connected Account |

`[DOC]` — I fetched this page twice with differently-worded extraction prompts to guard
against a summarizer fabricating the table. Both returns agreed on the four column headers and
on every cell in the rows above; the second returned all 17 rows with internally consistent
structure. I am treating it as reliably transcribed. It remains a transcription of a rendered
page, not a direct read — if any single fact in this file deserves a 10-minute empirical
confirmation before a production change, it is the `application_express` × "Stripe payment
processing fees" cell. Test in §A.6.

Supporting statement on the same page, explaining why `application_express` exists as a
distinct value rather than collapsing into `application` `[DOC]`:

> "`application_custom` and `application_express` are assigned to accounts created with
> `type=custom` and `type=express`, respectively. Their billing behaviors for direct charges
> and connected account usage of Stripe products matches the historical behavior of Custom and
> Express accounts."

That sentence is the crux. Legacy Express accounts historically paid their own processing fees
on direct charges, and `application_express` preserves exactly that. It is **not** a synonym
for `application`.

**Hypothesis (B) is refuted.** `controller.fees.payer` is not a global account-level override
that governs every charge. Its scope is the *product categories* in that table — which for
payments means direct charges specifically. Destination charges are governed separately and
unconditionally, per the same page `[DOC]`:

> "Any activity occurring at the platform account level is billed to your platform regardless
> of the entity responsible for fee collection. For example, Stripe charges the platform
> directly for destination charges (with or without `on_behalf_of`)."

So both facts hold simultaneously and without contradiction: destination charge on this account
→ platform pays; direct charge on this same account → connected account pays. The account
property did not change. The charge type did. `[DOC-ADJACENT]` — assembling the two documented
statements; no leap.

## A.2 Are direct charges permitted against a legacy `type: "express"` account?

**Yes.** Found affirmatively while researching the secondary question, on
[platform-pricing-tools](https://docs.stripe.com/connect/platform-pricing-tools) `[DOC]`:

> "If the platform is on IC+ pricing ... configured pricing applies to **direct card charges on
> Custom and Express accounts**."

Stripe would not document pricing behavior for "direct card charges on Custom and Express
accounts" if that combination were prohibited. Corroborated by
[account-capabilities](https://docs.stripe.com/connect/account-capabilities.md) `[DOC]`:

> "The `card_payments` capability applies to all charge types."

The Apex accounts already request `card_payments` and `transfers`, which is the full set needed.
No capability change required. `[DOC-ADJACENT]`

## A.3 GOTCHA 1 — Interchange Plus pricing inverts the answer

Look again at row two. Under `application_express`:

- **Stripe payment processing fees** → Connected Account
- **Interchange Plus Payment Fees** → **Platform**

If the WiSense platform account is on **IC+ / Interchange Plus pricing rather than standard
flat-rate pricing, direct charges do NOT solve the problem** — the fee lands back on the
platform. `[DOC]`

The §1 evidence suggests standard flat-rate: the observed −73 on a $14.82 charge matches
`2.9% × 14.82 + 0.30 = 0.7298` almost exactly, which is a flat-rate signature, not IC+ (IC+
produces ragged per-card amounts varying by card BIN). `[INFERENCE]` — strongly suggestive, not
proof, since two ledger samples can coincide.

**Confirm the platform's pricing plan in the Stripe Dashboard before building anything.** This
single fact decides whether the entire addendum applies.

## A.4 GOTCHA 2 — Dispute fees move to the venue, but the platform still backstops

Under `application_express` direct charges, **dispute fees → Connected Account** `[DOC]`, and
per [charges](https://docs.stripe.com/connect/charges.md) `[DOC]`:

> "For disputes on payments created using direct charges, Stripe debits the disputed amount from
> the connected account's balance, not your platform's balance."

Commercially good — chargeback risk sits with the venue that controls fulfillment. But legacy
`type: "express"` also carries `controller.losses.payments = application`, meaning the
**platform remains liable for negative balances** the connected account cannot cover
([design-an-integration](https://docs.stripe.com/connect/design-an-integration.md)) `[DOC]`.

So: disputes debit the venue first, and the platform is the backstop if the venue's balance
goes negative and stays there. Better than today's "platform pays every dispute unconditionally,"
but not zero risk. `[DOC-ADJACENT]`

This is also a **venue-communication issue**. Today venues see disputes silently absorbed by
the platform. After the switch they'll see chargebacks hit their own balance. That is a
material change to their deal and must be told to them in advance, not discovered.
`[INFERENCE]`

## A.5 What actually changes in the integration

Direct charges are not a one-line change. `[DOC-ADJACENT]` unless noted:

- Charge creation moves onto the connected account via the `Stripe-Account` header. The charge,
  its `PaymentIntent`, and its `BalanceTransaction` all live on the venue's account, not the
  platform's.
- **`on_behalf_of` becomes meaningless and must be removed** — the charge already *is* on the
  venue's account. `[INFERENCE]`, though it follows directly from the definition.
- `application_fee_amount` now arrives **gross**: per the §1.2 worked example, a 1.23 application
  fee on a 10.00 charge transfers 1.23 to the platform, with 0.59 netted off the venue's side.
  1.5% stays 1.5%. `[DOC]`
- **Webhooks change shape.** Events fire on the connected account and arrive with an `account`
  field; any handler keyed to platform-account events needs rework. Verify against the Connect
  webhook docs before implementing — I did not research this path in depth.
- Customers, PaymentMethods and saved cards are per-account objects and do not cross from the
  platform. If Apex saves cards for repeat guests on the platform account, that flow breaks and
  needs a separate design (shared PaymentMethods / cloning). **Check this before committing** —
  it is the most likely hidden blocker.
- Refunds must still explicitly pass `refund_application_fee`, per §1.4. Unchanged. `[DOC]`

## A.6 The empirical test (still worth running, ~10 minutes, no browser)

Confirms the A.1 cell and the A.3 pricing-plan question in one shot. Test mode:

1. Create and confirm a PaymentIntent **on the connected account** — `Stripe-Account: acct_xxx`
   header, `amount=2647`, `currency=usd`, `payment_method=pm_card_visa`,
   `confirm=true`, `application_fee_amount=40`, and `automatic_payment_methods[enabled]=true`
   with `allow_redirects=never` so no browser redirect is needed.
2. Expand the resulting charge's `balance_transaction` **on the connected account**. Read `fee`
   and `fee_details`.
3. List balance transactions on the **platform** account for the same window.

**Read the result as:**
- Connected acct BT shows `fee ≈ 107` and platform shows only a `+40` application fee with no
  processing-fee debit → **hypothesis (A) confirmed, addendum applies, proceed.**
- Connected acct BT shows `fee = 0` and the platform is debited ~107 → hypothesis (B), or the
  platform is on IC+ pricing. **The §9 service-fee recommendation stands unchanged.**

Run this before writing any production code. It costs minutes and settles a question that has
already produced two bad deploys.

## A.7 Secondary question — varying `application_fee_amount` by payment method

**Largely moot if (A) holds.** The entire reason to estimate Stripe's per-method cost was to
deduct it from the venue's payout. Under direct charges the connected account is billed the
actual method-specific fee by Stripe directly, at the real amount, with no estimation anywhere
in the platform's code. The estimation matrix disappears rather than getting solved.

On whether Stripe documents or endorses per-method fee variation:

- **Nothing prohibits it.** `application_fee_amount` is an arbitrary integer per charge, subject
  only to the caps in §1.4. Computing it differently per `payment_method_type` is mechanically
  ordinary. `[DOC]` on the caps; the absence of prohibition is an absence, not an endorsement.
- **Stripe's documented mechanism for this is the Platform Pricing Tool**
  ([platform-pricing-tools](https://docs.stripe.com/connect/platform-pricing-tools)), which lets
  platforms configure pricing rules server-side rather than hand-computing fees per charge.
  I could **not** determine from its docs whether it supports branching on payment method type —
  the page discusses IC+ eligibility and card-charge scope but never states whether per-method
  rules are configurable. `[DOC]` that the tool exists; unresolved that it does this.
- **I found no Stripe warning of any kind** about estimating Stripe's own fees, nor about an
  application fee exceeding actual cost of acceptance. Searched for it specifically; the pricing
  page addresses eligibility and access, not fee-estimation risk. Treat this as *not found*,
  not as *cleared*. `[DOC]`

Agreed that merchant-side deduction is outside the §6 surcharge regime — those rules govern
fees charged to *cardholders* varying by *tender*, and a B2B deduction from a venue's payout is
neither. The exposure here is contract and disclosure with the venue, not card-network or
consumer-protection law. `[INFERENCE]` — reasoning, not a legal opinion.

The practical risk of estimation, which Stripe does not discuss anywhere I could find: an
estimate that runs *over* actual cost silently converts the platform's take rate into something
higher than what the venue agreement says, discoverable by any venue that reconciles carefully.
Under direct charges this risk simply does not arise. `[INFERENCE]`

## A.8 Revised recommendation

**Provisionally: switch to direct charges. Do not build the service fee yet.** Conditional on
two checks, both cheap:

1. Confirm the platform is on **standard flat-rate pricing, not IC+** (§A.3). Dashboard lookup.
2. Run the **§A.6 balance-transaction test**. Ten minutes.

If both pass, direct charges fix the economics with no re-onboarding, no guest-facing fee, no
estimation logic, and 1.5% arriving gross — while moving dispute *fees* to the venue. That is a
materially better outcome than §9's service fee on every axis except integration effort.

If either check fails, **§9 stands exactly as written** and the service fee is the answer.

Independent of which path wins, three items from the main body are unaffected and still live:
the `refund_application_fee` check (§1.4), the saved-card / Customer-object question (§A.5), and
telling venues before their dispute exposure changes (§A.4).

**What I could not determine, addendum:**

- Whether the Platform Pricing Tool supports per-payment-method rules (§A.7).
- Whether the §1.2 direct-charges worked example (10.00 − 0.59 − 1.23 = 8.18) assumes a
  particular `fees.payer` value — the page never states its assumption. The arithmetic is
  *consistent* with connected-account-pays, which corroborates the A.1 table, but the page does
  not say so and I am not treating it as independent confirmation.
- The Connect webhook migration path for direct charges (§A.5) — flagged, not researched.
- Whether saved cards / Customer objects on the platform account can be reused for direct
  charges on connected accounts without a redesign. **Most likely hidden blocker; research
  before committing to the switch.**
