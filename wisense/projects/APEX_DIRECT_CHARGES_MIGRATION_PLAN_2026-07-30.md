---
type: plan
date: 2026-07-30
tags: [stripe, connect, direct-charges, migration, apex-v2]
---

# Apex v2 — Destination → Direct Charges Migration Plan

Source tags used throughout: `[DOC]` = quoted/derived from a cited Stripe doc URL.
`[CODE]` = read from the actual repo file at the cited line. `[INFERENCE]` = my
reasoning, not directly stated by either.

---

## 1. Verdict

**Feasible with caveats.** The code change is small and well-contained. The
*deployment* is where this kills you, and there is exactly one open commercial
question that can invalidate the whole economic rationale.

Why feasible:

- The Express accounts already have exactly the controller configuration that
  moves processing fees to the connected account on direct charges. Legacy
  `type: "express"` is defined as equivalent to `controller.fees.payer =
  application_express`, `controller.losses.payments = application`,
  `controller.requirement_collection = stripe`, `controller.stripe_dashboard.type
  = express`. `[DOC]` https://docs.stripe.com/connect/migrate-to-controller-properties
  And for `application_express`, the fee behaviour table reads:

  | Product Category | account | application | application_custom | application_express |
  | --- | --- | --- | --- | --- |
  | **Stripe payment processing fees** | Connected Account | Platform | Connected Account | Connected Account |
  | **Interchange Plus Payment Fees** | Connected Account | Platform | Platform | Platform |
  | **Dispute fees** | Connected Account | Platform | Connected Account | Connected Account |

  `[DOC]` https://docs.stripe.com/connect/direct-charges-fee-payer-behavior

  So on a direct charge, the venue pays the 2.9% + 30¢ and the dispute fees; the
  platform keeps `application_fee_amount` whole. That is precisely the inversion
  we want — the −51¢ becomes +22¢ on a $14.82 order. `[INFERENCE]`

- No saved cards, so the per-account Customer/PaymentMethod problem does not
  apply (already established, and confirmed by reading
  `create-guest-payment/index.ts` — no `customer`, no `setup_future_usage`). `[CODE]`

- The app never re-reads the Checkout Session or PaymentIntent from Stripe after
  creation, so there is no scattered set of platform-scoped API reads to retrofit
  with `stripeAccount`. `[CODE]` grep across `supabase/functions/` returns Stripe
  API calls only in `create-guest-payment`, `stripe-connect-onboard`, and
  `stripe-os-webhook`.

Caveats, in descending severity:

1. **Webhook routing is a silent-outage trap.** Direct-charge
   `checkout.session.completed` is a connected-account event and will not reach a
   `connect: false` endpoint. And `connect` is **not an updatable parameter** on
   an existing webhook endpoint — the updatable set is `description`, `disabled`,
   `enabled_events`, `metadata`, `url`. `[DOC]`
   https://docs.stripe.com/api/webhook_endpoints/update So `we_1TyJToHeXj7HLVbu9xbVwagT`
   **cannot be flipped**; a second endpoint must be created. See §4.

2. **The IC+ question is genuinely unresolved and it is load-bearing.** If
   WiSense's live platform account is on Interchange Plus rather than flat rate,
   the table above says Interchange Plus payment fees stay with the **Platform**
   even under `application_express`. In that world this migration moves the
   *dispute* fees but not the processing cost, and the −51¢ largely remains.
   I could not determine which plan the account is on (§8). **Resolve this before
   writing code.**

3. **Liability shifts to the venue.** `controller.losses.payments = application`
   means the platform is still liable for negative balances/losses on Express
   accounts, but refunds now debit the *connected account's* balance rather than
   the platform's. "Direct charges: Connected account's balance is debited"
   `[DOC]` https://docs.stripe.com/connect/charges If a venue's balance is empty
   at refund time, the refund can fail or push them negative — and per the
   comment already in `stripe-connect-onboard/index.ts` lines 152-163, Stripe's
   own onboarding copy holds the platform liable for negative balances on Express.
   `[CODE]` This is a real operational change, not a cosmetic one.

4. **The venue becomes the visible merchant.** Statement descriptor already comes
   from the connected account today (destination charge *with* `on_behalf_of`),
   so that specific thing does **not** change — see §7.1. But the payment leaves
   the platform's Payments list. `[DOC]` https://docs.stripe.com/connect/charges

Not advisable *today, without answering (2)*. Advisable the moment (2) comes back
"flat rate".

---

## 2. What the code does now (verified by reading)

### `supabase/functions/create-guest-payment/index.ts` `[CODE]`

- L3 — `import Stripe from "https://esm.sh/stripe@14.21.0?target=deno"`.
- L38-94 — validates `order_id` + `public_token`, requires `payment_mode ==
  'pay_now'`, `status == 'pending_payment'`, `payment_status ==
  'awaiting_payment'`.
- L96-111 — reads `restaurant_settings.stripe_account_id` into a local named
  `destination`, requires `stripe_charges_enabled === true`.
- L113-137 — amount from `order.total_cents`, minimum 50¢ guard, fee taken from
  `order.platform_fee_cents` with fallback `round(amount * 0.015)`, clamped to
  `[0, amount-1]`.
- L146-149 — `new Stripe(stripeKey, { apiVersion: "2023-10-16", httpClient:
  Stripe.createFetchHttpClient() })`.
- L151-189 — `stripe.checkout.sessions.create({...})` with **no** second
  argument. `payment_intent_data` at L167-178 carries `on_behalf_of: destination`
  (L169), `application_fee_amount: fee` (L170), `transfer_data: { destination }`
  (L171), and metadata `{app, type: "guest_order", order_id, organization_id}`.
  Session-level `metadata` duplicates the same four keys at L179-184.
- L191-201 — stores `stripe_payment_intent_id` and `platform_fee_cents` on the
  order row.

### `supabase/functions/stripe-os-webhook/index.ts` `[CODE]`

- L141 — single secret `STRIPE_WEBHOOK_SECRET`.
- L156-172 — `stripe.webhooks.constructEventAsync(body, signature, webhookSecret)`,
  400 on failure.
- L291-321 — `account.updated` branch: writes `stripe_charges_enabled`, and
  back-fills `stripe_account_id` from `account.metadata.restaurant_id` /
  `organization_id` when the row wasn't matched by account id.
- L323-402 — `checkout.session.completed`. L325 reads `session.metadata.type`.
  L328 routes `guest_order`; **the `else` at L371 routes everything else into the
  SaaS SKU activation path** (`activatePro` / `activateOs` / `markFlagshipPaid`).
- L335-369 — re-reads `online_orders.total_cents`, refuses to mark paid unless
  `session.amount_total === total_cents`, then updates `status='waiting',
  payment_status='paid', stripe_payment_intent_id=pi` guarded by
  `.eq("payment_status","awaiting_payment")`, and **throws** on error so Stripe
  retries (the L346-352 comment documents the previous silent failure).

### `supabase/functions/stripe-connect-onboard/index.ts` `[CODE]`

- L164-176 — `stripe.accounts.create({ type: "express", country: "US",
  capabilities: { card_payments: {requested:true}, transfers: {requested:true} },
  metadata: {...} })`.
- L149-163 — an in-code comment already names direct charges as one of the two
  candidate fixes and correctly calls it "a decision about what the venue signs
  up for". This plan is that decision.
- L197-220 — always re-reads `accounts.retrieve(accountId)` and re-syncs
  `stripe_charges_enabled`, which is why the integration works even if the
  `account.updated` webhook never arrives (see §4.5).

### Schema / RPC `[CODE]`

- `supabase/migrations/20260728231446_stripe_connect_platform_fee.sql` L3-16 —
  adds `restaurant_settings.stripe_account_id`, `.stripe_charges_enabled`, and
  `online_orders.stripe_payment_intent_id`, `.platform_fee_cents`.
- Same file, `place_order` L108-117 — `pay_now` requires a connected + enabled
  account, sets `status='pending_payment'`, `payment_status='awaiting_payment'`.
  L196-204 — `platform_fee_cents := round(total * 0.015)`, clamped.
- `20260802250000_fix_order_guard_service_role_exemption.sql` — `apex_guard_order_money()`
  is a `security invoker` BEFORE-UPDATE trigger that **raises
  `order_money_is_immutable` if `payment_status`, `platform_fee_cents`,
  `total_cents`, or `stripe_payment_intent_id` change**, unless the caller is
  `service_role` or a super admin. This is why any refund write must come from an
  edge function with the service key, not from the staff console client. `[CODE]`
  It also documents the exact silent-payment failure mode this plan is trying not
  to repeat.

### Clients `[CODE]`

- `lib/features/ordering/cart_screen.dart` L160-201 — calls `place_order` RPC then
  `functions.invoke('create-guest-payment', {order_id, public_token})` and
  `launchUrl(payMap['url'])`. It consumes only `url`; nothing else in the response
  shape matters to it.
- `lib/features/ordering/staff_console_screen.dart` L392-418 — `_reject()` →
  `_setStatus(order,'rejected', reason:)`; sets `status`, `rejected_at`,
  `reject_reason`. **No money is touched.**
- `C:\development\projects\jigsys_site\staff.js` L311-320 — `rejectOrder()` does
  a PostgREST `patchOrder` setting `status/rejected_at/reject_reason`. Same: no
  refund. L262 and L383 render a "paid" badge from `payment_status === 'paid' ||
  payment_mode === 'pay_now'`.

Confirmed: **there is no refund code anywhere in either client or any edge
function.** `[CODE]`

---

## 3. Q1 — What changes in `create-guest-payment`

### 3.1 The one-line mechanism

stripe-node supports the connected account per-request; the header is
`Stripe-Account`. "All of Stripe's server-side libraries support the
`Stripe-Account` header approach on a **per-request basis**." `[DOC]`
https://docs.stripe.com/connect/authentication In stripe-node (and therefore the
esm.sh Deno build, which is the same package) that surfaces as a second
*request-options* argument on the method call:

```
stripe.checkout.sessions.create({ ...params }, { stripeAccount: destination })
```

`[INFERENCE]` — the docs page rendered only the curl variants for me, so I did not
get the Node snippet verbatim; the `{ stripeAccount }` request-option is the
stripe-node convention and is what the Deno build inherits, but **verify against
the SDK types before shipping** (see §10).

Do **not** create a second `Stripe` client with a global `stripeAccount` — the
same function must still be able to talk to the platform (it doesn't today, but
keeping the scope per-call keeps it obvious which requests are connected-account
requests). `[INFERENCE]`

### 3.2 Parameter-by-parameter

| Param | Now (L#) | After | Why |
| --- | --- | --- | --- |
| `Stripe-Account` | absent | **add** `{ stripeAccount: destination }` as 2nd arg to `sessions.create` | "The `Stripe-Account` header indicates a direct charge for your connected account and uses the connected account's branding in Checkout." `[DOC]` https://docs.stripe.com/connect/direct-charges |
| `payment_intent_data.transfer_data` | L171 | **remove** | Only meaningful for destination charges; the funds already land on the connected account. `[DOC]` charges comparison |
| `payment_intent_data.on_behalf_of` | L169 | **remove** | The connected account *is* the merchant of record on a direct charge; `on_behalf_of` is a destination-charge mechanism for borrowing the connected account's identity. `[INFERENCE]` — I did not find a doc line explicitly forbidding it on direct charges; treat "remove it" as the safe choice and expect an API error if it is in fact rejected. |
| `payment_intent_data.application_fee_amount` | L170 | **keep, unchanged** | Documented direct-charge parameter: `-d "payment_intent_data[application_fee_amount]=123"`; "After payment processing, the `application_fee_amount` is transferred to the platform." `[DOC]` https://docs.stripe.com/connect/direct-charges |
| `payment_intent_data.metadata` | L172-177 | **keep** | Now written on the connected account's PaymentIntent. Fine. |
| session `metadata` | L179-184 | **keep — this is what routes the webhook** | See §4.4. |
| `line_items` | L153-166 | keep; **edit the description string** | L162 currently says "Card processing by Stripe · 1.5% WiSense OS payment rail fee". Under direct charges the venue pays the processing, so this copy is now describing someone else's cost structure to the guest. Reword or drop. `[INFERENCE]` |
| `success_url` / `cancel_url` | L185-188 | keep | Absolute URLs to the platform app; unaffected. |
| `admin.update(...)` L195-201 | keep | but note `stripe_payment_intent_id` now holds a PI **that only exists on the connected account** — see §5 and §7.3. |

Everything else in the file (token validation, the 50¢ minimum, the fee clamp)
stays as-is.

### 3.3 API version

`apiVersion: "2023-10-16"` (L147). Direct charges and `application_fee_amount`
long predate 2023-10-16 and neither appears in any version-change list I read;
the direct-charges doc's examples are version-agnostic. `[INFERENCE]` No pin
change is required. Do **not** bump the version as part of this migration —
that is a separate change with its own blast radius.

---

## 4. Q2 — Webhook routing. **This is the failure mode.**

> If you deploy §3 without §4, guests will pay, Stripe will keep the money on the
> venue's account, and `online_orders` will sit at `awaiting_payment` forever with
> **no error anywhere**. This is the same shape as the bug fixed in
> `20260802250000` — that one cost an hour. This one will not even produce a
> failed webhook delivery to look at, because the event will never be sent to your
> endpoint at all.

### 4.1 The event moves

With a direct charge the Checkout Session belongs to the connected account, so
`checkout.session.completed` is a connected-account event. Endpoint scope is
determined by the `connect` parameter: "`false` for **Your account** scope,
`true` for **Connected accounts** scope." `[DOC]`
https://docs.stripe.com/connect/webhooks

### 4.2 You must create a *second* endpoint — you cannot flip the existing one

`connect` is not in the updatable parameter set for
`POST /v1/webhook_endpoints/:id` (`description`, `disabled`, `enabled_events`,
`metadata`, `url` only). `[DOC]` https://docs.stripe.com/api/webhook_endpoints/update

Therefore `we_1TyJToHeXj7HLVbu9xbVwagT` stays as-is and a **new** endpoint is
created:

```
POST /v1/webhook_endpoints
  url            = https://pqkremkwfkudrhtxasdj.supabase.co/functions/v1/stripe-os-webhook
  connect        = true
  enabled_events[] = checkout.session.completed
  enabled_events[] = account.updated          # see §4.5
  description    = "Apex v2 — Connect (direct charges)"
```

Point it at the **same function URL**. Two endpoints, one handler.

Keep the platform endpoint alive and enabled for `checkout.session.completed`
regardless — it is what activates the SaaS SKUs (`stripe-os-webhook` L371-401),
and those are platform Payment Links, not Connect charges. `[CODE]`

### 4.3 Two endpoints ⇒ two signing secrets ⇒ the handler must try both

Each endpoint has its own `whsec_`. `constructEventAsync` verification "is
identical for both scopes" — same method, but it must be given the *matching*
secret. `[DOC]` https://docs.stripe.com/connect/webhooks

Change at `stripe-os-webhook/index.ts` L141 / L161-172:

- add secret `STRIPE_CONNECT_WEBHOOK_SECRET`
- attempt `constructEventAsync(body, signature, STRIPE_WEBHOOK_SECRET)`; on
  `SignatureVerificationError`, retry with `STRIPE_CONNECT_WEBHOOK_SECRET`; 400
  only if **both** fail.

Order matters only for microseconds. Do not try to guess which endpoint sent it
from the payload — verify, then read. `[INFERENCE]`

### 4.4 What the payload gains, and what the handler must do with it

"Each event for a connected account contains a top-level `account` property
identifying the connected account… Because the connected account owns the object
that triggered the event, **you must make API requests for that object as the
connected account**." `[DOC]` https://docs.stripe.com/connect/webhooks

So `event.account === "acct_..."` on the new path, `undefined` on the old one.

Two things follow for the handler:

1. **Routing still works unchanged.** The `guest_order` branch keys off
   `session.metadata.type` (L325-328), and `create-guest-payment` sets that
   session metadata itself (L179-184). It survives the move verbatim. `[CODE]`
   You do **not** need `event.account` to find the order — `metadata.order_id`
   still does it. This is the single luckiest property of the existing code.

2. **You should nevertheless use `event.account` as a cross-check**, because the
   `else` fall-through at L371 is dangerous once connected-account events reach
   this function: any connected-account `checkout.session.completed` *without*
   `metadata.type === "guest_order"` (a venue taking a payment through their own
   Express dashboard, say) would fall into the SaaS SKU activation path.
   `isOrgUuid()` at L376 currently saves you — it requires
   `client_reference_id`/`metadata.organization_id` to be a UUID — so it is not
   exploitable today, but it is one metadata collision away from being so.
   Harden it: **if `event.account` is present and `metadata.type !==
   "guest_order"`, return 200 and do nothing.** `[INFERENCE]`

3. **Resolving the venue**, if/when you need it: `event.account` is the
   `acct_...`, which maps to `restaurant_settings.stripe_account_id` — the same
   column `create-guest-payment` L96-100 reads and the same one the
   `account.updated` branch matches on at L297-301. A one-row lookup. You do not
   need it for the paid-marking path; you would need it for refunds (§5). `[CODE]`

### 4.5 `account.updated` — check before you assume

The `account.updated` branch (L291-321) handles **connected** account updates, so
it is a Connect-scoped event and should never have been reaching a
`connect: false` endpoint. Two possibilities, and I could not distinguish them
without live Stripe access:

- (a) A Connect endpoint already exists in this Stripe account and I don't know
  about it — in which case §4.2 may reduce to adding
  `checkout.session.completed` to *that* endpoint's `enabled_events` (which *is*
  updatable), and you already have its secret configured.
- (b) That branch is dead code and has never fired — which would be consistent
  with `stripe-connect-onboard` L197-220 defensively re-reading
  `accounts.retrieve` on every call and the comment "webhook can lag". `[CODE]`

**Action: list webhook endpoints (`GET /v1/webhook_endpoints`) and read the
`connect` flag on each before doing anything else.** If (a), this whole section
gets much cheaper. `[INFERENCE]`

Either way, put `account.updated` on the Connect endpoint's `enabled_events` —
it belongs there.

---

## 5. Q3 — Refunds

### 5.1 Current state, restated precisely

There is no refund path. Rejecting a paid order sets `status='rejected'` and
leaves `payment_status='paid'` and the money with the venue
(`staff_console_screen.dart` L392-418; `jigsys_site/staff.js` L311-320). `[CODE]`
Order `8B395F` is the live evidence. **This is already a defect under destination
charges.** Direct charges do not create it — but they change who is out of pocket
when you fix it, so fix it as part of this work.

### 5.2 Under direct charges

- **Where issued:** on the connected account. `POST /v1/refunds` with
  `Stripe-Account: acct_...` and `charge` (or `payment_intent`). `[DOC]`
  https://docs.stripe.com/connect/direct-charges A platform-scoped refund call
  will simply not find the charge — it doesn't exist on the platform. `[INFERENCE]`
- **Who bears the refunded amount:** "Direct charges: Connected account's balance
  is debited." `[DOC]` https://docs.stripe.com/connect/charges
- **The original processing fee:** not returned. The venue paid it (per the
  `application_express` table) and eats it. `[INFERENCE]` — the fee-payer table
  establishes who paid; that Stripe does not return processing fees on refunds is
  general Stripe behaviour I did not re-verify against a doc URL in this pass.
  Flagged in §10.
- **`refund_application_fee`:** "`refund_application_fee=true`: Refunds the
  entire charge and application fee (or proportional amount for partial refunds).
  `refund_application_fee=false`: Refund only the charge; refund the application
  fee separately." `[DOC]` https://docs.stripe.com/connect/direct-charges
  Concretely: `true` means WiSense hands its 1.5% back to the venue; `false`
  means WiSense keeps it while the venue refunds the guest in full and *also*
  ate the processing fee.

### 5.3 Recommendation

**Use `refund_application_fee: true`, always, and full refunds only, for v1.**

Rationale: with `false`, a rejected $26.47 order costs the venue $26.47 + ~$1.07
processing + WiSense's 40¢ — WiSense profits from the venue's cancellation. That
is an indefensible position for a platform whose entire pitch is that it is on
the restaurant's side, and it is the sort of thing that gets noticed in month
three. `[INFERENCE]` Partial refunds add a `platform_fee_cents` proration
question with no business rule behind it yet; don't build it speculatively.

**Implementation shape** (new edge function, e.g. `refund-guest-order`):

1. JWT-authenticated, manager/owner of the org — mirror the `has_role` check in
   `stripe-connect-onboard` L87-104 rather than inventing a new one. `[CODE]`
2. Load the order; require `payment_status='paid'`, `payment_mode='pay_now'`, and
   a non-null `stripe_payment_intent_id`.
3. Load `restaurant_settings.stripe_account_id` for that restaurant.
4. `stripe.refunds.create({ payment_intent, refund_application_fee: true },
   { stripeAccount })`.
5. Write `payment_status='refunded'` **with the service-role client** — the
   `apex_guard_order_money` trigger raises `order_money_is_immutable` on any
   `payment_status` change from a non-service-role caller. `[CODE]` This is why
   the refund cannot be a client-side PostgREST patch from either console.
6. **Check the error on that write and throw**, exactly as L353-362 now does.
7. Idempotency: pass a Stripe `idempotencyKey` derived from the order id, and
   guard the DB update with `.eq('payment_status','paid')`.

**Wiring:** the Reject flow in both consoles should call it. In
`staff_console_screen.dart` `_reject()` (L392-418), and in `jigsys_site/staff.js`
`rejectOrder()` (L311-320) — the latter currently does a raw `patchOrder`, so it
needs a `functions.invoke`-equivalent fetch with the user's JWT.

**Schema:** `payment_status` has no CHECK constraint anywhere in the migrations
(grepped), so `'refunded'` can be written without a migration. `[CODE]` Both
consoles' paid badges (`staff.js` L262/L383) test `payment_status === 'paid' ||
payment_mode === 'pay_now'` — the second clause means a refunded pay-now order
**will still show "paid"**. Fix that in the same change or the console lies. `[CODE]`

---

## 6. Q5 — The existing account (`acct_1TyjMwQn0WTytQVU`)

**Usable immediately, no re-onboarding, no new capabilities** — *provided*
`charges_enabled` and the `card_payments` capability are active, which they must
already be for the current destination charges with `on_behalf_of` to work.
`[INFERENCE]`

- Direct charges need `card_payments` on the connected account; the account was
  created requesting it (`stripe-connect-onboard` L166-169). `[CODE]`
- `transfers` is requested too and is now surplus to the charge flow, but leave it
  — it costs nothing and removing a capability is a state change on a live
  account for no benefit. `[INFERENCE]`
- The fee-payer behaviour is fixed by the controller config that `type: "express"`
  implies, which is already set and which you could not change anyway:
  "You can't change the `controller.stripe_dashboard.type` of an existing
  connected account." `[DOC]`
  https://docs.stripe.com/connect/migrate-to-controller-properties Since
  `application_express` is the behaviour you *want*, immutability is in your
  favour here.
- `stripe-connect-onboard/index.ts` needs **no change at all** for this migration.
  Its L149-163 comment should be updated to record the decision, but that is
  documentation, not behaviour. `[CODE]`

---

## 7. Q4 — What else changes

### 7.1 Statement descriptor — **no change**

Today's charge is a destination charge *with* `on_behalf_of` (L169). Per Stripe:

> The customer's statement uses the connected account's static component for the
> following charge types: Direct charges; Destination charges with
> `on_behalf_of`; Separate charges and transfers with `on_behalf_of`.

`[DOC]` https://docs.stripe.com/connect/statement-descriptors — the guest already
sees the venue's descriptor and will continue to. Removing `on_behalf_of` while
*also* switching to direct is descriptor-neutral. This is a nice property; don't
let anyone "fix" it by keeping `on_behalf_of`.

### 7.2 Platform Dashboard visibility — **changes**

Direct charges land in the connected account's balance, not the platform's.
`[DOC]` https://docs.stripe.com/connect/charges The payments disappear from the
platform's own Payments list; the platform sees them via Connect → the connected
account, and receives `application_fee` objects. `[INFERENCE]` Practically: your
"how much did Jigsy's do today" habit of looking at the platform dashboard stops
working. `daily_revenue` (migration `20260801010000`) reads Supabase, not Stripe,
so in-app reporting is unaffected. `[CODE]`

### 7.3 `application_fee.created` is now asynchronous

"Application fees for direct charges are created asynchronously by default.
Listen for the `application_fee.created` webhook event." `[DOC]`
https://docs.stripe.com/connect/direct-charges You do not need this for order
fulfilment — `checkout.session.completed` still carries everything the handler
uses. But if you ever build platform revenue reporting off Stripe rather than
`platform_fee_cents`, that's the event. Do not add the subscription now.
`[INFERENCE]`

### 7.4 `platform_fee_cents` — still the right value, still the right source

`place_order` computes `round(total * 0.015)` (migration `20260728231446`
L196-204) and `create-guest-payment` L134-137 reads it back with a fallback and
clamp. `application_fee_amount` means the same thing under both charge types —
what the platform takes — and the direct-charge doc's only constraints are
"positive and less than the charge amount", which the existing clamp already
enforces. `[DOC]` + `[CODE]` **No change.** The *stored* number stops being a
lie about profit, though: today it overstates what WiSense keeps (fees eat it);
after the migration it is actually net. `[INFERENCE]`

### 7.5 `account.updated` branch — logically unaffected, operationally see §4.5

The handler body (L291-321) does not care about `event.account`; it reads
`account.id` from `event.data.object`. `[CODE]` No code change needed. It only
matters that the event is subscribed on a Connect-scoped endpoint.

### 7.6 Jigsy static site

`staff.js` reads `online_orders` through PostgREST and never touches Stripe.
`[CODE]` **No change required by the charge-type migration.** It *does* need
changes for §5.3 (refund on reject) and for the misleading paid badge at L262 /
L383. Those are refund work, not direct-charge work — they can ship separately.

### 7.7 Flutter client

`cart_screen.dart` L173-201 reads only `url` off the response. `[CODE]` **No
change.**

---

## 8. Q7 — Determining flat-rate vs Interchange Plus

This is the gate on the whole plan and I could not answer it from here. What I
established:

- The distinction is real and material: `application_express` puts *Stripe
  payment processing fees* on the connected account but *Interchange Plus Payment
  Fees* on the **Platform**. `[DOC]`
  https://docs.stripe.com/connect/direct-charges-fee-payer-behavior
- "Users on network cost plus pricing (also called interchange plus or IC+
  pricing) can access the **Network cost insights** report in the Dashboard."
  `[DOC]` https://support.stripe.com/questions/understanding-ic-fees — **presence
  of that report in the live Dashboard is the cheapest positive signal.**
- IC+ plan-level and transaction-level reports are runnable through the Reporting
  API for accounts on that plan. `[DOC]` (same source) An API probe: attempt to
  create a Report Run of the IC+ report type; availability is itself the answer.
  `[INFERENCE]`
- Your note that **test-mode balance transactions simulate flat-rate regardless
  of the live plan** is consistent with everything I read and I found nothing
  contradicting it. Test data cannot settle this. `[INFERENCE]`

**Recommended:** look for "Network cost insights" under Reports in the **live**
Dashboard, and if there is any ambiguity, ask Stripe support directly — "is
account X on flat rate or IC+" is a one-line support answer and beats another
hour of inference. A negative (no IC+ report) is the green light.

Secondary, weaker signal: pull one **live** balance transaction from an existing
guest order and inspect `fee_details`. Flat-rate shows a single `stripe_fee`
matching 2.9%+30¢ exactly; IC+ shows a different decomposition. On the $14.82
order the platform paid 73¢, and 2.9% + 30¢ on $14.82 = 73.0¢ — an exact
flat-rate match. `[INFERENCE]` That is suggestive of flat rate but **not proof**
(IC+ blended can land near flat rate on a single transaction), so do not ship on
it alone.

---

## 9. Q6 — Smallest test that proves the fee moved

Test mode, API only. Do **not** use the live account for this.

1. Create a test-mode Express connected account and complete the onboarding stub
   (or use an existing test `acct_`). Confirm `charges_enabled: true`.
2. Create a Checkout Session exactly as §3 will:
   ```
   POST /v1/checkout/sessions
   Stripe-Account: acct_TEST
   line_items[0][price_data][currency]=usd
   line_items[0][price_data][unit_amount]=1482
   line_items[0][price_data][product_data][name]=probe
   line_items[0][quantity]=1
   payment_intent_data[application_fee_amount]=22
   mode=payment
   success_url=https://example.com/ok
   ```
3. Pay it with `4242 4242 4242 4242`.
4. **The proof.** Retrieve the charge on the connected account with the
   balance transaction expanded:
   ```
   GET /v1/charges/{ch_...}?expand[]=balance_transaction
   Stripe-Account: acct_TEST
   ```
   Read `balance_transaction.fee_details`. `fee_details[].type` is one of
   `application_fee`, `payment_method_passthrough_fee`, `stripe_fee`, `tax`,
   `withheld_tax`. `[DOC]` https://docs.stripe.com/api/balance_transactions/object

   **Pass condition:** on the *connected account's* balance transaction you see
   **both** a `stripe_fee` entry (~73¢ on $14.82) **and** an `application_fee`
   entry (22¢). That is the whole test — the connected account is being debited
   the processing fee. `amount` 1482, `fee` ≈ 95, `net` ≈ 1387.

   **Fail condition:** `stripe_fee` is absent from the connected account's
   breakdown (i.e. only `application_fee` appears) — meaning the fee stayed with
   the platform and the migration achieved nothing.

5. Cross-check from the platform side: `GET /v1/application_fees` (no
   `Stripe-Account`) should list a 22¢ fee, and its own balance transaction on the
   platform should be +22 with **`fee: 0`**. Platform gross = platform net. That
   is the number that is currently −51¢.

6. Also confirm the webhook: with the test-mode Connect endpoint configured, the
   `checkout.session.completed` delivery should show `"account": "acct_TEST"` at
   the top level of the event. `[DOC]` https://docs.stripe.com/connect/webhooks
   **If it does not arrive at all, you have reproduced the outage in test mode
   instead of production, which is the entire point of doing this first.**

Total cost: one test charge. Do this before touching any repo file.

---

## 10. Unknowns / could not verify

Listed honestly. Several of these are cheap to close and should be closed before
code.

1. **Live pricing plan: flat rate vs IC+.** Unresolved, and it decides whether
   this migration is worth doing (§8). No Stripe API access from this session —
   the Stripe MCP server is unauthenticated here. **Highest priority.**
2. **Whether a Connect-scoped webhook endpoint already exists.** Could not list
   endpoints. Materially changes the size of §4 (§4.5).
3. **stripe-node `{ stripeAccount }` request-option syntax on the esm.sh Deno
   build.** The Stripe doc page rendered only curl for me. The option is the
   standard stripe-node convention and I am confident, but I did not see it in
   this session's fetched text and I did not read the SDK's `.d.ts`. **Verify
   against the type definitions or a one-line test invoke before shipping** — a
   silently-ignored second argument would produce a *platform* charge that looks
   fine and still loses money.
4. **Whether `on_behalf_of` is rejected or merely ignored on a direct-charge
   Checkout Session.** I found no doc line either way. Plan removes it; if it
   turns out to be an API error, removing it was necessary anyway.
5. **Whether refunded processing fees are returned.** I asserted "no" from
   general Stripe behaviour, not from a URL fetched in this session. Confirm on
   the test refund in step 9 (refund the probe charge, read the refund's balance
   transaction on the connected account).
6. **`8B395F` and any other already-paid-then-rejected orders.** Whether they
   should be retro-refunded is a business call, not a technical one, and the
   destination-charge refund for them must be issued from the **platform**
   (they are platform charges) — the new direct-charge refund function will not
   find them. If you build the refund path, it must handle both eras, or you
   need a one-off manual refund for the historical rows. **This is a real
   forward-compatibility trap.** `[INFERENCE]`
7. **Negative-balance behaviour** when a venue with an empty Stripe balance is
   refunded. `controller.losses.payments = application` says the platform is
   liable, but I did not trace the exact mechanics of a refund against an
   insufficient connected-account balance.
8. **Express Dashboard behaviour for the venue** once payments are direct — the
   venue will start seeing gross charges and Stripe fees in their Express
   dashboard where previously they saw net transfers. That is arguably better,
   but it is a visible change to a live customer and someone should tell Jigsy's
   before it happens, not after. `[INFERENCE]`

---

## 11. Deployment sequence

### Gate 0 — before any code

- [ ] Resolve IC+ vs flat rate (§8). **If IC+, stop.** The rest of this plan
      does not pay for itself.
- [ ] `GET /v1/webhook_endpoints` — record `connect` on each (§4.5).
- [ ] Confirm the `{ stripeAccount }` SDK syntax (§10.3).
- [ ] Run the test-mode probe (§9) end to end, including the webhook delivery.

### Ship 1 — **must ship as one atomic deployment**

Nothing here is separable. Shipping any subset produces either lost money or
silent unpaid orders.

1. Create the Connect webhook endpoint (`connect: true`, same URL,
   `checkout.session.completed` + `account.updated`). Record its `whsec_`.
2. `supabase secrets set STRIPE_CONNECT_WEBHOOK_SECRET=whsec_...`
3. Deploy `stripe-os-webhook` with dual-secret verification (§4.3) **and** the
   `event.account` fall-through guard (§4.4.2). **This function goes first** —
   it is backward compatible (still verifies and handles platform events
   identically) and must be live and proven before any direct charge exists.
4. *Only then* deploy `create-guest-payment` with the direct-charge change (§3).
5. Place one real $0.50-ish live order. Verify: session created on the connected
   account, webhook delivered with `account` set, row flips to
   `status='waiting' / payment_status='paid'`, and the connected account's
   balance transaction shows `stripe_fee` (§9 step 4).

The ordering of 3 before 4 is the whole safety property: a Connect-capable
handler receiving platform-only traffic is harmless; a platform-only handler
receiving Connect traffic is the outage.

### Ship 2 — separate, and can wait

- `refund-guest-order` edge function (§5.3).
- Wire Reject → refund in `staff_console_screen.dart` and `jigsys_site/staff.js`.
- Fix the paid-badge logic at `staff.js` L262/L383 and its Flutter equivalent.
- Handle the pre-migration destination-charge orders in the refund path (§10.6).

### Ship 3 — cosmetic

- Update the L149-163 comment in `stripe-connect-onboard/index.ts` to record the
  decision instead of the open question.
- Update the guest-facing line-item description (L162 of `create-guest-payment`).

---

## 12. Rollback

The rollback is cheap in one direction only, and there is a window where it is
not clean. Know which one you are in.

**Rolling back `create-guest-payment`** (redeploy the previous version, restoring
`on_behalf_of` + `transfer_data` and dropping the `stripeAccount` argument):
new orders immediately go back to destination charges on the platform. Losing
money again, but working. This is a single `supabase functions deploy` and takes
under a minute.

**Do not roll back `stripe-os-webhook`.** The dual-secret version handles both
scopes; the old version handles only one. Leaving the new webhook deployed is
strictly safer under every rollback scenario. Leave the Connect endpoint
registered too — an extra subscribed endpoint costs nothing.

**The dirty window:** any guest who has an open Checkout Session created as a
direct charge at the moment you roll back will still complete on the connected
account, and their `checkout.session.completed` still needs the Connect endpoint
and the dual-secret handler to be alive. Hence the rule above. Give it 30 minutes
after rollback before considering the Connect path cold.

**If a payment lands and the order does not flip:** the money is on the connected
account, not lost. Repair as `20260802250000` did — `set local role service_role`
then update the row with the real `pi_...` (readable from the Stripe Dashboard
under the connected account). Keep that migration open as the template.

**Not rollback-able:** any refund already issued. Refunds are terminal.

---

## 13. One-paragraph summary for the impatient

Direct charges will fix the negative unit economics **if and only if** the
platform is on flat-rate pricing, because `application_express` moves Stripe
processing and dispute fees to the connected account but leaves Interchange Plus
fees with the platform. The code change is small: add `{ stripeAccount }` to the
`sessions.create` call, delete `on_behalf_of` and `transfer_data`, keep
`application_fee_amount` exactly as-is. The dangerous part is that
`checkout.session.completed` becomes a connected-account event, the existing
webhook endpoint cannot be converted (`connect` is not updatable), and a new
Connect endpoint with its own signing secret must be live *before* the first
direct charge — otherwise guests pay and orders never mark paid, with no error
anywhere. Ship the webhook first, the charge second, and prove both in test mode
first. Refunds are a real gap that already exists and gets more urgent under
direct charges: build `refund-guest-order` with `refund_application_fee: true`,
issued on the connected account, writing `payment_status` with the service role
because the money-immutability trigger blocks anyone else.
