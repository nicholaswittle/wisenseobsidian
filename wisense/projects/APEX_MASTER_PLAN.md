---
title: Apex Master Plan — the dispatching document
tags: [apex, master-plan, dispatch, strategy]
date: 2026-08-01
---

# Apex Master Plan

**This is the document Antigravity dispatches from.** It is the single ordered
view of where Apex is and what happens next. Detail lives in the linked notes;
sequence and constraints live here.

Update this file when state changes. If it disagrees with a tool's memory, this
file wins.

---

## 1. The goal

**$300/month by the end of year one from launch — three paying OS venues.**

Not thirty. Three can be onboarded by hand, which means **self-serve is the
product, not an acquisition necessity.** Build it because it is the
differentiator and because venue #4 then costs nothing to add.

Acquisition at this scale is a warm-introduction problem, and Jigsy's standing
with local mom-and-pop owners is the channel. **Retention is the hard part** —
one churn is a third of the business.

The product thesis, in Nicholas's words: *"not I want this — I need this."* A POS
vendor can ship online ordering tomorrow. What they will not ship is an AI that
knows this venue's numbers and says what to do next, at onboarding and at 7pm
when something breaks.

**Pricing** (already in `entitlements.dart`): Free $0 — scheduling, swaps, time
clock, push · Pro $25/mo — manager log, tips, labor cost, offline, chat ·
OS $99/mo — online ordering + KDS, smart capacity, no-show engine, labor vs
revenue.

**The wedge is the free tier, not ordering.** Jigsy's Square report shows labor
tracking at **0.0%** on a $292k business. Ordering is the upsell.

---

## 2. Non-negotiable constraints

Each of these has already caused a production incident. Pass them through on
every dispatched task.

1. **Two-sided changes ship in one commit.** A database change that alters what
   a client reads includes the client change. This has broken guest ordering
   three times.
2. **Every live database change gets a migration file**, and `apply_migration`
   via MCP assigns its own version — it must be corrected to match the file or
   drift returns. This happened three times on 2026-08-01 alone.
3. **Never `ALTER POLICY`.** `DROP POLICY IF EXISTS` then `CREATE POLICY`.
4. **Never `functions deploy --prune`.** It deletes remote functions with no
   local counterpart, including payment webhooks whose IDs are registered with
   Stripe and Square.
5. **`supabase/config.toml` governs per-function `verify_jwt`.** Never drop
   entries. Two payment webhooks run with verification off because Stripe and
   Square do not send Supabase JWTs.
6. **Deploys need PowerShell with Docker running.** Git Bash has a stale PATH
   and the CLI silently falls back to an API path.
7. **Report what you observed, never what you intended.** `cron.job_run_details`
   reports whether the SQL ran, not what the HTTP call returned — it said
   `succeeded` while every request 401'd, and believing it cost hours. Real
   status is in `net._http_response`.
8. **Photos stay owner-supplied.** Never scrape venue images. Guest-uploaded
   Google photos belong to the guests; republishing them under our footer is a
   copyright problem at scale.
9. **No secret material in agent scope.** Configuration and digests only.

### Code that is correct and must not be "tidied"

- `stripe-os-webhook`'s dual signing-secret loop — a Stripe endpoint's `connect`
  flag is immutable, so both scopes need two endpoints and two secrets.
- The `event.account` ownership check on guest orders — metadata on a connected
  account is attacker-controllable.
- The post-create `stripeAccount` assertion in `create-guest-payment`.
- `readVenueAccount()` falling back to Stripe on an unrecognised provider.

---

## 3. Current state

### Live in production

- **Payments, both rails.** Stripe Connect direct charges and Square hosted
  checkout, tips on both, partial refunds, reconciliation cron.
- **Square rails complete through Phase 3** — OAuth, webhook correlation,
  printer onboarding checklist, `square-test-order`.
- **AI support agent, steps 1–3 + UI** — merged to `main`. Health snapshot,
  diagnosis on Haiku, three allowlisted reversible repairs, escalation queue,
  "Get help" in the staff console.
- **Two cron jobs green** — `apex-check-capacity` (*/2),
  `apex-reconcile-pending-payments` (*/15).
- **Migration history reconciled** — local files and remote records match
  exactly. The repo can rebuild a fresh database.
- **Jigsy's site live**, token rotated to `jigsys-enola-7c2a`.
- **wisensellc.com** — case study naming Jigsy's as first client; SMS
  disclosures live on /privacy and /terms.

### Built but NOT merged

⚠️ **`feat/template-to-product` is 18 commits ahead of `main`.** The entire
self-serve build — multi-tenant Next.js renderer under `site/`,
`venue_site_profile`, `get_public_venue_profile`, `enrich-business` onboarding,
launch wizard, photos/branding, growth loops, referral codes — plus the four
plan revisions in `fe275e2`.

Verified good: `venue_profile_isolation.sql` is a real 224-line test shipped
*with* the renderer, and `enrich-business` pulls no photos.

**Merging this is the largest single outstanding integration.**

### Blocked, and on whom

| Blocked on | What |
|---|---|
| **Twilio A2P vetting** | Escalation SMS delivery. Campaign in review 2026-08-01. |
| **Jigsy's owner** (`jigsy895@yahoo.com`) | Square OAuth, owner photos, confirmed menu prices, real domain |
| **Nicholas** | Connect pricing model check; Message Flow screenshot; printer purchase decision |
| **A Mac** | Offline queue, then Terminal. **TestFlight is already done** — the original Apex v2 build is distributed. |

---

## 4. Work queue — dispatch in this order

### Now, unblocked

**A. Merge `feat/template-to-product` to `main`.**
18 commits. Its migrations are partly applied to production already, so `main`
is behind reality. Verify `flutter analyze`, the test suite, and the isolation
test before merging. Do not `--prune`. This gates everything else in self-serve.

**B. Stranger dry-run of the onboarding wizard.**
`apex_v2/docs/DRY_RUN_SCRIPT.md`. Needs a human in a room, not an agent. Nine
timed phases, the operator may only say *"what would you try if I wasn't here?"*.
The friction log becomes the prioritised fix list.

**C. Deploy the consent notice and STOP text.**
Uncommitted in `apex_v2`: the SMS consent line under the alert phone field, and
`Reply STOP to opt out.` in `notify-order-event`, `venue-support-agent`,
`route-callout`. Required before the Message Flow screenshot can be taken, and
before any SMS actually sends.

**D. Push notification delivery (optional — evaluate against Twilio).**
The app is already on TestFlight, but push is NOT wired: nothing captures an
FCM token (`push_token` appears only in `demo_backend.dart`, set to null),
`send-push-notification` is called solely by `route-notification`, and the
escalation path does not push. Wiring it means capturing the token in Flutter,
writing it to `profiles.push_token`, calling the push function from the
escalation, then a rebuild and reinstall. **Slower than A2P vetting, so it is
not the shortcut it appears to be** — worth doing eventually as a second
channel, not as a way to skip Twilio.

**E. Support agent step 4 — proactive watch.**
Notice before staff do: order stuck 10+ min, no successful cron response in 30,
`square_environment` not production on a live venue. Depends on delivery, so it
follows Twilio approval.

### Next, after A

**F. Wizard funnel instrumentation review.** Per-step enter/complete/abandon
landed in `fe275e2`; confirm events actually fire before the dry-run, because
with three venues in year one every abandonment is a large share of all data.

**G. Menu source-of-truth copy.** Apex is authoritative for the online menu.
Confirm the wizard says so plainly — Apex-vs-Square price drift is the failure
that surfaces in month three.

### Deferred, deliberately

- **CloudPRNT printing (~$300).** Off the critical path: Jigsy's printer profile
  already routes "Online & kiosk order tickets", so on the Square rail their
  existing hardware prints Apex orders with no purchase. Build it for the next
  venue that lacks Square.
- **Tap to Pay.** Struck entirely — unavailable on iPads.
- **Square catalog sync**, custom domains, vertical pack #2.

---

## 5. The launch floor — do not gate go-live

`payment_mode` pay-at-pickup needs **no payment integration at all**. Guest
orders online, order lands in Apex, staff read it off the iPad (which *is* the
ticket — no printing dependency), customer pays at the counter as they do today.

Zero OAuth, zero Stripe, zero hardware, zero risk — and **zero revenue**, since
no card online means no 1.5%. Adoption must not be mistaken for revenue.

**Launch on the floor, upgrade in place.** Every step after is additive: same
site, same orders table, same console. Do not gate go-live on OAuth, printing,
or the owner's availability.

---

## 6. Detail notes

- [[wisense/projects/APEX_SELF_SERVE_PLAN_REVISIONS_2026-08-01]] — the $300 target and four revisions
- [[wisense/projects/APEX_V2_TEMPLATE_TO_PRODUCT_GAP_MAP_2026-07-31]] — Builds 1–5
- [[wisense/projects/APEX_V2_SELF_SERVE_OS_GAMEPLAN_2026-07-28]] — tiers, build-free-pay-to-publish
- [[wisense/projects/APEX_AI_SUPPORT_AGENT_PLAN_2026-08-01]] — support agent steps
- [[wisense/projects/JIGSYS_PILOT_LAUNCH_STRATEGY_2026-07-31]] — on-site findings, what can be promised
- [[wisense/projects/JIGSYS_BUSINESS_NUMBERS_AND_REVENUE_MODEL_2026-08-01]] — their numbers, why ordering fees are the wedge not the business
- `apex_v2/docs/PAYMENTS_AND_POS_BUILD_PLAN_2026-07-31.md` — 33 payment tasks
- `apex_v2/docs/SQUARE_DAY_ONE_LIVE_VALIDATION.md` — first-merchant runbook
- `jigsys_site/LAUNCH_CHECKLIST.md` — pre-launch blockers for the pilot
