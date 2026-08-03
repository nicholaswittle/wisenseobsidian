---
type: plan
title: "Apex v2 — go-live sequence"
tags: [go-live, stripe, supabase, checklist, apex-v2]
date: 2026-07-30
status: active
supabase_ref: "pqkremkwfkudrhtxasdj"
---

# Apex v2 — go-live sequence

**As of 2026-07-30 there is no live Stripe account.** Everything built and tested
so far runs in the sandbox (`acct_1TqfSbHeXj7HLVbu`, "WiSense LLC sandbox").

That fact is worth more than it sounds, and it expires: **no venue has ever
onboarded for real, so architecture decisions currently cost nothing to make.**
Every one of them gets expensive the moment Jigsy connects a live account.

---

## Do these BEFORE activating live

### 1. Build direct charges first

The whole reason: see
[[APEX_PLATFORM_FEE_ECONOMICS_2026-07-30]] and
[[APEX_DIRECT_CHARGES_MIGRATION_PLAN_2026-07-30]]. Destination charges make the
platform pay Stripe's processing fees — measured at −51¢ on a $14.82 order and
−67¢ on $26.47, worse as orders grow.

The migration plan's most expensive caveat is *"existing connected accounts
cannot change fee payer and must re-onboard."* **That caveat does not apply
while nothing is live.** Do it now and Jigsy onboards once, onto the right
thing. Do it later and you are asking your first customer to redo bank details
and identity verification because the pricing model was wrong.

Test-mode is also the only safe place to get the deployment order wrong, and the
order matters: **deploy the webhook first and alone** (a Connect-capable handler
receiving platform traffic is harmless; the reverse is a silent payment outage),
then `create-guest-payment`. Roll back only `create-guest-payment`, never the
webhook.

### 2. Confirm the pricing plan on the live account

Under `controller.fees.payer = application_express`, **Interchange Plus fees stay
with the Platform** even on direct charges — so IC+ pricing would defeat the
entire fix. A new US account defaults to flat rate (2.9% + 30¢), which is what
the fix depends on. **Test-mode ledgers cannot answer this** — they simulate
flat-rate regardless of the live plan. Check the Dashboard once the live account
exists, before assuming the economics work.

### 3. Custom SMTP

Auth → Emails → SMTP Settings. Supabase's built-in mailer sends only a handful of
messages per hour — it is a testing service. Invite six staff in one evening and
most never receive the confirmation, and it presents as "the app won't let me
sign in" rather than a mail limit. Resend/Postmark free tiers cover this volume.
Email confirmation is already ON (2026-07-29), which makes this a blocker rather
than a nicety.

### 4. Clear the test data

- Delete the five `@roster.local` placeholder profiles (Kim, Marsha, Dana,
  Courtney, Morgan) — name tags with fake emails, added so the photo reader
  would recognise those names. If a real person signs up while one exists, the
  roster gets two entries with the same name.
- Clear test venues and accounts: `moe's Restaurant`, `test rest`, and the test
  orders under `jigsys`.
- Fix `apex/apex/supabase/config.toml`, which still points at **Horizon's**
  project — a destructive `supabase` command run from that folder hits the wrong
  database.

---

## What does NOT carry over from test mode

Assume nothing transfers. Each of these is a separate live object:

| Thing | Consequence |
|---|---|
| **API keys** | `STRIPE_SECRET_KEY` must be swapped in Supabase secrets |
| **Webhook endpoints** | Live endpoints are separate objects with **different signing secrets**. `STRIPE_WEBHOOK_SECRET` must be re-set, or every event fails signature verification |
| **Connect accounts** | `acct_1TyjMw…` is a test account. **Jigsy onboards again from scratch** — bank details, identity verification, the lot |
| **Payment links** | The three `buy.stripe.com/test_…` links are test-only. Live equivalents needed, then rebuild the Flutter web app with the three `--dart-define` flags from `docs/BILLING_OS.md` |
| **Customers / payment methods** | Not applicable — Apex saves no cards (verified 2026-07-30) |

⚠️ **The live webhook endpoint must include `account.updated` in its enabled
events.** The test endpoint never did, so that branch of `stripe-os-webhook` has
never fired — meaning `stripe_charges_enabled` only ever updates when someone
opens the Stripe screen in the app. If Stripe restricts a venue's account in
live, Apex would not notice, and guests would keep being sent to a checkout that
fails.

⚠️ **Rebuild the web app with every `--dart-define`.** A missing flag silently
disables a feature rather than erroring — Stripe checkout vanished from the live
app on 2026-07-29 purely because three payment-link flags were omitted from a
build command. See `docs/BILLING_OS.md` for the full list.

---

## Verify after going live

Not by reading code — by reading the ledger. Every conclusion that survived
2026-07-29 came from the catalog or the balance sheet; every one that came from
reading a report or a migration was wrong at least once.

1. One real order end to end: guest pays → webhook fires → order reaches the
   staff console showing **PAID ONLINE** → ticket prints **DO NOT COLLECT**.
2. Pull the balance transaction and confirm **the venue was charged the
   processing fee, not the platform**. This is the number the pricing model
   rests on and the only proof that direct charges worked.
3. Reject a paid order and confirm the guest is actually refunded, the venue's
   payout is reversed, and both consoles show **Refunded**.
4. Confirm the founder notification email arrives on a SaaS purchase — the
   webhook sends one and it has never been observed.

---

## Related

- [[APEX_PLATFORM_FEE_ECONOMICS_2026-07-30]] — why 1.5% loses money, measured
- [[STRIPE_FEE_ARCHITECTURE_RESEARCH_2026-07-30]] — fee-payer behaviour by charge
  type, and the service-fee vs surcharge legal distinction
- [[APEX_DIRECT_CHARGES_MIGRATION_PLAN_2026-07-30]] — file-by-file plan
- [[APEX_V2_LIVE_CATALOG_SWEEP_2026-07-29]] — audit the catalog, not the repo
