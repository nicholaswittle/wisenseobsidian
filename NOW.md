---
title: NOW — Weekly Scorecard
tags: [meta, tasks, scorecard, now]
updated: 2026-08-08
---

# NOW — Weekly Scorecard & Task Board

> After [[hot]], read this before planning work. Overwrite "This week" when focus shifts.

## Company truth

| Field | Value |
|-------|-------|
| Stage | REVENUE / PILOT LIVE — FIRST CUSTOMER (Jigsy's Brewpub) |
| Focus | v2 FROZEN (fixes only). v3 Phase 1 — onboarding, orders, schedule, operation contract, and the services pack (schema + quote flow) all built. |
| #1 metric | Three cold-user tests on v3 onboarding (2026-08-07): can a stranger set up a business unaided? |
| Working stack | Claude CLI + Ollama + Cursor + Hermes Agent |

## Scorecard

| Metric | Target | Current | Notes |
|--------|--------|---------|-------|
| First customer | 1 signed | 1 SIGNED | Jigsy's — website + online ordering |
| Security remediation | anon EXECUTE revoked | DONE | 56 functions locked down 2026-08-02 |
| Build on TestFlight | Current | **Build 9 installed + verified** | `build-9` @ `2c448ff`, 141 tests |
| Tip pool | Compliant + live | **LIVE** | Migrations applied, anon revoked, first split reconciled |
| Outside monitor | Running + alerting | RUNNING | GitHub Actions, no SMS (Twilio pending) |
| Tip-pool compliance | Migrations applied | STAGED | Client routed, migrations not applied |
| App icon | Custom 1024x1024 | FLUTTER DEFAULT | One PNG away from fixed |

## This week — next actions

### Human-only (Nicholas)

1. [ ] **Set `NEXT_PUBLIC_SHOWCASE_VENUE=jigsys-enola-7c2a` in Vercel project settings.** Landing page links now read this env var instead of a hardcoded token; unset, they render nothing.
2. [ ] **When Twilio A2P 10DLC campaign clears:** note the approved use case type. Low Volume Mixed or Customer Care covers services vertical too. Anything narrower registers a second campaign against the same brand. Pending campaigns cannot be edited. SHAFT bars firearms content — a gun-shop tenant is email/`tel:` only.
3. [ ] **App icon — still the Flutter default.** One 1024x1024 PNG generates all 15 iOS sizes plus Android. Build 8 reaches seven people at Jigsy's and the first thing each sees is the Flutter logo.
4. [ ] **Apex v3 deposit cap — get a qualified opinion before v3 goes live.** It ships as implemented (one third, on contracts over $1,000 — 73 P.S. 517.7 as v2's code states it). TWO things unverified: the $1,000 floor (v2 caps every job regardless of size; the threshold is my reading of the statute v2 cites), and the jurisdiction — the cap is per-business but nothing asks a new business what law governs it, so every org silently inherits Pennsylvania's. Also unclear whether these trades are "home improvement contractors" under the Act at all; a mobile groomer is currently capped as though they were. The ARITHMETIC is verified and does not need re-checking. Full list: `apex_v3/docs/pre_launch_checklist.md`.
5. [ ] **Payroll Lite: strip the four money columns** before `feat/payroll-export` merges — `base_pay_cents`, `overtime_pay_cents`, `gross_estimate_cents`, `tip_credit_shortfall_cents`. See [[DECISIONS]] 2026-08-02. Branch is stale, needs rebase.
6. [ ] Before Square go-live: obtain owner authorization, complete OAuth, verify `square_environment=production`, POS order creation, printer profile, and fee receipt.
7. [ ] Gate C assets: feature graphic 1024x500, screenshots (both apps)
8. [ ] Host privacy policy URLs (Apex + COMMS LINK)
9. [ ] Create Android upload keystores
10. [ ] Add `com.nicholaswittle.apex://` to Supabase allowed redirect URLs (iOS login)
11. [ ] Buy Google Play ($25) account — Apple Developer ($99) purchased 2026-07-31

### Build 9 — ship window

1. [ ] Archive build 9 with tip-pool screen routed at `/tip-pool`
2. [ ] Upload to TestFlight, wait for processing
3. [ ] Confirm build 8 clears Beta App Review (or withdraw to free the slot)
4. [ ] **In the same window:** apply migrations `20260831000000` + `20260831000001`
5. [ ] Robin confirms the tip-pool roster in the app. Nothing splits until he does.

### Phase A — testing (after build 9 is installed)

1. [ ] Emily's dashboard shows today's shift (`.order()` fix proof)
2. [ ] Clock in as Emily, work a real shift, clock out. Most under-tested path in the app.
3. [ ] Grant Admin to someone from team screen; confirm they lose Labor Cost
4. [ ] Place a pay-now order end to end on a card
5. [ ] Reject an order and confirm the refund returns the platform fee
6. [ ] Run a full pay period through the payroll register and reconcile by hand

### Known issues

1. [~] ~~`shifts.user_id` null on all rows~~ — **LARGELY CLOSED.** 27/27 backfilled. Sidework policy retains name fallback that fails open on ambiguous rows. Dart lookups still match by name in `_WeekSummary`.
2. [ ] `staff_console_screen.dart` is 2001 lines — growing, not shrinking
3. [ ] 4 silent `catch (_) {}` blocks, plus ~8 in admin console that log without surfacing
4. [ ] No payment-path tests. 81 source files, 141 tests, zero covering money or auth.
5. [ ] No router: 29+ imperative `Navigator.push` calls, no deep linking or web back-button
6. [ ] Monitor detection latency is hours, not minutes (GitHub throttles `*/15`, Twilio pending)
7. [ ] **"Square connected" button does nothing — and the state behind it is false.** Diagnosed 2026-08-03. `monetization_upsell_cards.dart:371` renders a `FilledButton.tonalIcon` with `onPressed: () {}` when `squareConnected` — a status badge shaped like a button, so tapping is a no-op by design. It renders because `_squareConnected` reads `restaurant_settings.square_charges_enabled`, which is **`true` in the database despite Square never having been OAuth'd and zero Square payments ever** (`square_payment_id` null on all 45 orders; no `square-connect-oauth` invocation in 24h of edge logs). So the app claims a connection that does not exist *and* hides the "Connect Square" button that would create one. Fix is two parts: set `square_charges_enabled = false` until OAuth actually completes — that alone restores the working button — and give the connected state a real action (Stripe's side offers Disconnect) or make it a non-interactive chip. Note this flag is also the only live-money path in an otherwise all-sandbox setup.

### Parked

- **Apex go-to-market / pricing** — see [[business/Apex Go-To-Market — parked 2026-08-02]]. Three-tier model settled and deliberately parked: not testable without a second customer.
- **Payroll export** — stale branch, needs rebase. DB functions already live.
- **Services vertical — Phase 1 is BUILT, not parked.** Phase 1 was always exempt from the "vertical one pays first" gate because it lives in `site/`, a separate Vercel deployable that cannot reach the Flutter build. Phases 2+ remain gated. Status below; plan: [[projects/APEX_SERVICES_BUILD_PLAN_CANONICAL]].
- iOS / Codemagic (needs Apple account — purchased, not yet activated for store submit)
- New Horizon commercial Duffel/Viator expansion
- COMMS LINK store packaging

## Services vertical — shipped 2026-08-02 (branch `feat/services-vertical`, pushed)

Seven commits on GitHub. **Nothing in it can reach the restaurant product** — verified, not assumed: no existing table altered, no existing function replaced, no existing policy dropped. The one shared file touched is `site/app/[token]/page.tsx`, where a branch reads `profile.vertical`; every org is `restaurant`, so that condition **cannot be true today**.

**Live on the database** (applied through the connector, verified against `pg_catalog`):
`organizations.vertical` · eight additive jsonb columns on `venue_site_profile` · `requests` / `request_payments` / `request_reminders` with RLS and policies · `referrals` + `apex_claim_referral` + `apex_referral_summary` · RPCs `submit_public_request`, `enrich_public_request`, `get_public_services_profile`. Security advisor after: **90 lints, zero ERROR**; guest-callable surface 8 → 11, matching exactly the three added.

**Built and committed, not deployed:** `site/lib/services.ts`, `theme.ts`, `fonts.ts` · `RequestForm`, `ServicesHome`, `ServicesThemeRoot` · `app/services.css` · edge functions `notify-request` and `create-request-payment`. `tsc --noEmit` and a real `next build` both clean; 28 woff2 self-hosted across six families.

**Email is LIVE.** `RESEND_API_KEY` and `RESEND_FROM_ADDRESS` set 2026-08-03 via the Supabase CLI. ⚠️ The verified domain is **`mail.wisensellc.com`, not `wisensellc.com`** — from-address is `Apex Leads <leads@mail.wisensellc.com>`. Sending from the apex domain will be rejected as unverified and will look like a code bug.

**Design decisions worth not re-opening:** `get_public_services_profile` is a *new* function, not four keys added to `get_public_venue_profile`, because that one serves Jigsy's live storefront and carries the whole menu/modifier tree. `requests` is not `online_orders` — that table is sub-hour by construction. Capture-then-qualify is two RPCs on purpose: half of form starters abandon, so name+phone must land before any trade question. `request_payments` and `referrals` both block client INSERT entirely.

**Deposits are capped at one third** of the quote (PA HICPA, work over $1,000), overridable per venue via `legal.deposit_cap_pct`. Over-cap requests are refused *with the reason*, never silently trimmed.

**To make it real:** pick a first services business, set `organizations.vertical = 'services'`, fill `venue_site_profile.services`, and `/{token}` renders. Nothing else blocks.

**Not built, deliberately:** payment webhook (next), reminder cron, GBP onboarding (Flutter — gated), tracked number (needs a provider decision), GPS geofence, recurring schedules, invoices spanning jobs.

> ⚠️ **Two agents were in this repo simultaneously on 2026-08-02.** The other was testing the restaurant on `fix/worked-hours-admin-gates-and-realtime` and switched branches under a live working tree. Nothing was lost, but check `git branch --show-current` before writing.

## Blocked on

| Item | Blocker | Owner |
|------|---------|-------|
| Play Store submit | Graphics + privacy URLs + keystore + $25 account | Nicholas |
| Monitor SMS alerts | Twilio A2P 10DLC campaign approval | Nicholas (wait) |
| Vercel function region | `apex-venue-site` builds in `iad1`; DB is us-west. Must change in Vercel settings | Nicholas |

Related: [[hot]], [[index]], [[log]]