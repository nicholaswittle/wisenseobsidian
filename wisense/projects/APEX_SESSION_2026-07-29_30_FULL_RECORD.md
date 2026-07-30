---
type: session-record
title: "Apex v2 — session record 2026-07-29 → 30"
tags: [apex-v2, security, stripe, session-record, audit, pricing]
date: 2026-07-30
status: completed
supabase_ref: "pqkremkwfkudrhtxasdj"
---

# Apex v2 — session record, 2026-07-29 into 07-30

One long session. Three strands: a security sweep that found holes four audits
had missed, a payment path that was failing silently, and a pricing model that
lost money on every order. All three are closed and verified.

**The thread running through all of it:** every wrong conclusion came from
reading something that *described* the system — an audit report, a migration
file, a vault note. Every right one came from measuring the system — the
Postgres catalog, the Stripe ledger, the deployed bundle.

---

## 1. Security — the catalog sweep

Four audits had reviewed this codebase. The three most severe problems in the
system were invisible to all of them, because **they exist in no migration
file**. Found within minutes of querying the live catalog.

| | what it allowed |
|---|---|
| `apex_grant_membership` | `SECURITY DEFINER`, no authorization check, `EXECUTE` granted to `anon`. One call with the publishable key made you **Owner of any venue**. |
| `org invite consume` | UPDATE policy with `WITH CHECK (true)` and no org scoping. One statement could **rewrite every pending invite in the database** — set the role to Owner, set a code you know, redeem it. |
| `apex_set_subscription_status` | No caller check, anon-executable. **Paid tiers for anyone** who knew the function name. |

Also closed: `v_paying_venues` (a `SECURITY DEFINER` view with `SELECT` granted
to `anon` — billing state readable with no account), four more internal helpers
exposed to `anon`, and two functions with mutable `search_path`.

**The `org all` pattern was on six tables, not one.** `shifts org all` had been
fixed earlier; the identical policy was live on `availability`, `sidework`,
`swaps`, `time_entries` and `time_off_requests`. `time_entries` is the one that
matters — clock-in data, which is what people are paid from. Any employee could
edit their own hours upward or delete a colleague's record.

`messages_update` had a `USING` clause and no `WITH CHECK`, so editing your own
message could reassign `user_id` to a colleague. Their name, your words.

Detail: [[APEX_V2_LIVE_CATALOG_SWEEP_2026-07-29]]

### On the audits themselves

Four audit reports were produced. Their accuracy was poor enough to be
dangerous:

- Audit 3 rated the AI functions CRITICAL for "not validating JWTs". The
  Supabase gateway 401s them — verified by probe. Its proposed fix added nothing.
- Audit 4's summary table asserted safety that did not exist, marking functions
  as protected by a gate they had never had. Checking imports instead of the
  table found **a cross-tenant data leak in a function the report listed as
  safe** (`venue-briefing` took `organization_id` from the request body and
  queried with `service_role`).
- Audit 4's CRITICAL cited a line of Apex v1 code that does not exist.
- Three audits said "live Stripe charges not triggered", which was read as
  untested. The Stripe path *had* been tested on 2026-07-28 — the vault held the
  passing result the whole time.

**Treat audit reports as leads to verify, never as conclusions.**

---

## 2. The payment path was failing silently

A guest paid, Stripe took the money, and the order stayed `awaiting_payment`
with nothing on the staff console and no error anywhere.

**Cause was a trigger added that same morning as audit remediation.**
`apex_guard_order_money` exempted the webhook with
`current_user = 'service_role'` — but it is `SECURITY DEFINER`, and inside a
definer function `current_user` is the function's *owner*, never the caller. The
exemption could not fire. The guard raised on every paid order, and the webhook
**discarded the error and returned 200**, so Stripe never retried.

The comment in that function predicted this exact failure and the code did not
implement it. Fixed by making it `SECURITY INVOKER`; the webhook now throws
instead of swallowing.

**Two staff consoles were telling staff to double-charge customers.** Jigsy's
site hardcoded "due at pickup" on the card *and* printed `COLLECT ON SQUARE AT
COUNTER` on the ticket, regardless of payment. A guest who paid online would be
charged again — on paper, which outlives anything on a screen.

**Rejecting a paid order kept the money.** No refund path existed anywhere, and
the console has a Reject button. Built `refund-order`: manager-gated, reverses
the transfer, returns the platform fee, idempotent. The refund runs *before* the
rejection, so a failure leaves the order open rather than rejected-and-unpaid.
Added `charge.refunded` handling so refunds issued from the Stripe dashboard
sync too.

---

## 3. Pricing — losing money on every order

Measured on the ledger, not argued: on a $26.47 order WiSense collected 40¢ and
paid Stripe $1.07. **Net −67¢, and worse as orders grew**, because 1.5% cannot
cover 2.9% + 30¢ at any size.

Cause: destination charges. Stripe documents that the platform always pays
processing on those, and it is not configurable. The 2026-07-28 audit's claim
that `on_behalf_of` moves fees to the venue is **false** — it sets the
settlement merchant only.

Two changes fixed it, each verified on the ledger:

1. **Direct charges.** The charge is created on the venue's account, so Stripe
   bills them the real, method-specific fee — no estimation table to drift.
2. **The 1.5% became a guest-paid service fee** rather than a venue deduction.

Same $32.96 order across all three arrangements:

| | guest pays | venue nets | WiSense |
|---|---|---|---|
| destination charges | $34.94 | $33.11 | **−$0.67** |
| direct charges | $34.94 | $33.11 | +$0.52 |
| **service fee (live)** | **$35.46** | **$33.61** | **+$0.52** |

The venue ends up **50¢ better off than where this started**. The customer
covers the fee — which is what Jigsy was originally pitched.

**Pitch line:** an online order costs the venue card processing and nothing
else — the same thing they pay when a card is tapped at their counter.

Two rules keep this a service fee rather than a card surcharge: never label it a
card or processing fee, and never vary it by payment method. Surcharges carry
Visa/Mastercard caps, receipt requirements, a debit and prepaid ban, and
restrictions in ten states.

Detail: [[APEX_PLATFORM_FEE_ECONOMICS_2026-07-30]],
[[STRIPE_FEE_ARCHITECTURE_RESEARCH_2026-07-30]],
[[APEX_DIRECT_CHARGES_MIGRATION_PLAN_2026-07-30]]

---

## 4. Also shipped

- **Photo import**: names now snap to the roster's spelling. A lowercase "kim"
  learned as an alias had written three shifts to a person who did not exist —
  present on the schedule, absent from her dashboard and her hours. Aliases are
  learned and reused; a correction made once stops the question recurring.
- **Unreadable photos now say why**, using the model's own stated reason
  instead of a generic "try a sharper photo".
- **`scripts/deploy.ps1`** — builds with every `--dart-define`, refuses if one
  is missing, verifies the flags survived into the bundle, and deploys functions
  and app together. Three separate bugs this session came from a forgotten flag
  or a forgotten rebuild.

---

## What is still open

1. **Migration history drift.** The deployed `place_order` matches *neither*
   migration in the repo. This changed how work had to be done three times in
   one session. Until it is reconciled the repo cannot answer "what is running",
   and reviewing it produces false assurance rather than none.
2. **Tier enforcement is UI-only.** Pro features are gated by which buttons
   Flutter renders; the RLS policies check role, not tier.
3. **Go-live checklist** — [[APEX_GO_LIVE_SEQUENCE_2026-07-30]]. Nothing in test
   mode carries over: keys, webhook endpoints and their signing secrets, Connect
   accounts and payment links are all separate live objects.
4. **Partial refunds** are not supported — full refunds only.
5. **`303A2F`** is still rejected-and-paid; it predates the refund path.

---

## The rule worth keeping

> Audit the catalog, not the repo. Measure the ledger, not the documentation.

Every hour lost this session was spent trusting a description of the system.
Every problem solved was solved by querying the system itself.
