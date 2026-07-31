---
type: meta
title: "Hot Cache"
tags: [meta, hot-cache, context]
updated: 2026-07-31T23:55:00
---

# Recent Context

> ~500-word cache for fast boot. Any agent/session reads this FIRST, then [[NOW]], then [[index]]. Overwrite completely each update — cache, not journal.

## Last Updated
2026-07-31 late. Apex live: [https://apex-v2-ten.vercel.app](https://apex-v2-ten.vercel.app) · HEAD `62f6707`. Jigsy site/staff: [https://jigsyssite.vercel.app](https://jigsyssite.vercel.app) · HEAD `f0dd7b7`.

🟢 **Jigsy Stripe Pay Now is working end-to-end.** Guests see the 1.5% service fee and optional Stripe tips; Jigsy pays Stripe processing; WiSense receives the application fee. Tickets and both staff consoles show tip/fee breakdowns. Full and custom partial refunds work in both staff surfaces and reconcile provider-dashboard refunds.

🟡 **Square is prepared, not live for Jigsy.** OAuth, webhook, hosted checkout, native tips, application fees, and refund code are ready. Do not activate without the owner’s authorization and production/printer validation. The first go-live must verify `square_environment=production` and printer profile routing.

🟡 **One operational payment task remains:** `reconcile-pending-payments` is deployed but must be scheduled in the Supabase dashboard every 15 minutes (authenticated Edge Function schedule). The current browser session is not signed in to Supabase.

🟢 **JIGSY IS TAKING BOTH THE APP AND THE WEBSITE** (2026-07-30). This settles the open question in [[wisense/projects/SITE_TEMPLATE_ONLINE_ORDERING_PLAN]] — the site is no longer speculative. Order of work still stands: finish Apex, then build ordering into the **site template** rather than as a one-off, so every flagship site ships taking money on day one.

💵 **Fee model settled: direct charges + guest-paid service fee.** On a $32.96 order the guest pays $35.46, the venue nets $33.61, WiSense earns 52¢ — versus **losing 67¢** before. Pitch line: an online order costs the venue card processing and nothing else, same as a card tapped at their counter. Two rules keep it a service fee not a card surcharge: never call it a card/processing fee, never vary it by payment method. See [[wisense/projects/APEX_PLATFORM_FEE_ECONOMICS_2026-07-30]].

📸 **Photo import shipped and measured** — `claude-opus-5` at low effort, **~10¢ per import, one-time per venue**. Everything else in Apex runs on Haiku (~$0.001/call). Full record: [[wisense/projects/APEX_PHOTO_IMPORT_BAKEOFF_2026-07-30]]. Three things from it worth carrying:

- 🔴 **`ListModels` advertises Gemini models that `generateContent` 404s.** Every Gemini ID in the deployed function was retired; that branch had never executed in production, it 404'd and fell through to Claude silently. Probe with a real one-token call before trusting any model ID.
- 🔴 **Ranking by "shifts found" picked the worse model** — it had silently shifted a row by one day. And two clean runs got reported as "perfect" when the true rate was ~1 wrong entry per import. Score against a confirmed answer, and run it more than twice.
- ⚠️ **A failed AI call bills like a successful one.** The client giving up does not stop the model. Three retries cost 3×.

🎯 **Endgame: Apex replaces Square as the POS** (2026-07-31) — the Square rail is a **wedge, not the destination**. Stages: ship the Square rail → own the kitchen ticket (CloudPRNT) → own front-of-house (native + Terminal + offline) → the long tail. Three walls, all hardware: their **Star TSP143IIU is USB**, so a browser can never drive it and replacing Square's printing means **replacing the printer** (~$250–350, the only unavoidable hardware cost); card-present needs a reader we own; offline resilience alone forces a native app. Apple Developer purchased 2026-07-31, which unblocks the native path. Full plan: `apex_v2/docs/PAYMENTS_AND_POS_BUILD_PLAN_2026-07-31.md` (33 tasks) + [[wisense/projects/APEX_PAYMENTS_AND_POS_STRATEGY_2026-07-31]].

🔴 **Do not quote platform margin until the Connect pricing model is confirmed.** If WiSense is on Stripe's "platform sets pricing" model we pay $2/mo per active account + 0.25% + 25c per payout + 0.25% of payout volume — plausibly **half the 1.5%** on a venue paid out daily, i.e. net nearer **0.7%**. Nothing in the code reveals which model we are on. Run `node apex_v2/scripts/check_connect_pricing.mjs` (read-only). Related: Stripe **direct charges do not appear in platform exports** — a revenue dashboard built on exports silently shows zero.

🔴 **Square Sandbox cannot test the value proposition.** It does not support Square POS, Square for Restaurants, or application-fee reporting — so *order reaches the POS*, *ticket prints*, and *the 1.5% lands* are **all unverifiable until a real merchant connects**. Sandbox de-risks the code, not the product. Two silent-failure traps recorded: `square_environment` defaults to `'sandbox'` and the fee is production-only (a misrecorded venue takes real money while Apex collects **nothing**, no error); and `Payment.reference_id` **does not arrive** on Square payment-link webhooks — correlation must run on `payment.order_id` → `square_order_id`.

❌ **Tap to Pay (the SDK feature) is struck, not deprioritised** — *"Tap to Pay isn't available on iPads."* Android excludes tablets too. Jigsy's wired contactless reader is **card-present hardware inside Square POS** — a different thing, already working, not ours. Conflating them mis-sizes the roadmap by an order of magnitude.

🟢 **Jigsy's is named publicly as first client** (2026-07-31, with permission). wisensellc.com case study rebuilt with real before/after screenshots; Apex now reads "First Venue Onboarding". **Not claimed**: that the site is live (it has not replaced jigsypizza.com), that Apex is in production, or anything about online ordering. ⚠️ The site's food photos came from **public Google guest uploads** — the hero screenshot was **deleted from the repo, not just unreferenced**, because anything under Next's `public/` is served whether linked or not. Replacing that photography with owner originals is blocker #1 in `jigsys_site/LAUNCH_CHECKLIST.md`.

🟦 **Square becomes a second payment provider, not a fork** (decided 2026-07-30) — [[wisense/projects/APEX_SQUARE_PROVIDER_DECISION_2026-07-30]]. Jigsy prints via a USB Star wired to a Square Stand on iPads, so the staff console can never reach it. Square payments create a Square Order, which prints on the hardware they own — one integration solves money and kitchen. Verified: `app_fee_money` works on Square's **hosted** checkout (`CreatePaymentLink`), so the 1.5% is collectable with no card form of our own. **The owner must authorise either way — Emily is a manager with no Square power.** Real reason to build it is market access, not one venue.

🚦 **[[wisense/projects/APEX_GO_LIVE_SEQUENCE_2026-07-30]] — read before activating a live Stripe account.** No live account yet, so **no venue has onboarded for real and architecture decisions are still free**. That window closes when Jigsy connects. Nothing carries over from test mode: keys, webhook endpoints and signing secrets, Connect accounts, and payment links are all separate live objects.

**BEFORE THE PILOT:** custom SMTP (Supabase's built-in mailer is a testing service — invite six staff in one evening and most never get the mail, and it looks like the app is broken); clear test venues/accounts; delete the five `@roster.local` placeholders (Kim, Marsha, Dana, Courtney, Morgan); set job roles; deactivate 3 orphan Stripe payment links.

⚠️ **Tier enforcement is partial, not absent** (corrected 2026-07-30 — "UI-only" was wrong). Tier writes are `is_super_admin()` only, and **online ordering IS enforced server-side** via `apex_org_has_online_ordering()` inside `place_order`, honouring tier + enabled/disabled overrides. `apex_set_org_module` can only toggle `timeClock` and can never grant a paid module. **The real gap:** every *other* module is UI-only — `tip_pools` / `tip_allocations` / `server_tips` (Pro), `capacity_events` / `call_outs` (OS) all carry plain org-member RLS that never consults tier. A Free venue's manager can reach them straight through PostgREST.
⚠️ **Migration history is unreconciled** — the repo cannot answer what is running in production. Audit the catalog, not the repo ([[wisense/projects/APEX_V2_LIVE_CATALOG_SWEEP_2026-07-29]]).
⚠️ `apex/apex/supabase/config.toml` still points at **Horizon's** project — a `supabase db reset --linked` from that folder hits the wrong database.
⚠️ After any deploy, **hard-refresh** — the Flutter service worker serves the old bundle and it looks exactly like a bug.

## Key Recent Facts
- **Vault Boot Chain:** [[hot]] → [[NOW]] → [[index]].
- **Primary Sales Path:** Apex Restaurant OS ($99/mo) + Flagship Managed Web ($299 + $99/mo).
- **Session record:** [[wisense/projects/APEX_SESSION_2026-07-29_30_FULL_RECORD]]
- **Flagship strategy:** [[wisense/projects/WISENSE_LLC_APEX_FLAGSHIP_PIVOT_PLAN_2026-07-28]]
- **Stripe verified live in sandbox 2026-07-29** (`acct_1TqfSbHeXj7HLVbu`): guest Pay Now works end to end; the SaaS unlock branch of the webhook has **never fired** and must be treated as untested when unlocks are switched back on.
- **COMMS LINK:** ⏸️ PARKED.

## Active Threads
- Next: go-live sequence, then the site template for Jigsy.
- [[NOW]] · [[index]] · [[Apex v2 — Restaurant OS Build]]
