---
title: NOW — Weekly Scorecard
tags: [meta, tasks, scorecard, now]
updated: 2026-08-02
---

# NOW — Weekly Scorecard & Task Board

> After [[hot]], read this before planning work. Overwrite "This week" when focus shifts.

## Company truth

| Field | Value |
|-------|-------|
| Stage | REVENUE / PILOT LIVE — FIRST CUSTOMER (Jigsy's Brewpub) |
| Focus | Build 9 SHIPPED + tip pool LIVE — next is Phase A testing |
| #1 metric | Run Phase A testing; strip payroll money columns; `time_entries.shift_id NOT NULL` |
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
4. [ ] **Payroll Lite: strip the four money columns** before `feat/payroll-export` merges — `base_pay_cents`, `overtime_pay_cents`, `gross_estimate_cents`, `tip_credit_shortfall_cents`. See [[DECISIONS]] 2026-08-02. Branch is stale, needs rebase.
5. [ ] Before Square go-live: obtain owner authorization, complete OAuth, verify `square_environment=production`, POS order creation, printer profile, and fee receipt.
6. [ ] Gate C assets: feature graphic 1024x500, screenshots (both apps)
7. [ ] Host privacy policy URLs (Apex + COMMS LINK)
8. [ ] Create Android upload keystores
9. [ ] Add `com.nicholaswittle.apex://` to Supabase allowed redirect URLs (iOS login)
10. [ ] Buy Google Play ($25) account — Apple Developer ($99) purchased 2026-07-31

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

### Parked

- **Apex go-to-market / pricing** — see [[business/Apex Go-To-Market — parked 2026-08-02]]. Three-tier model settled and deliberately parked: not testable without a second customer.
- **Payroll export** — stale branch, needs rebase. DB functions already live.
- **Services vertical** — gated on vertical one paying. Plan: [[projects/APEX_SERVICES_BUILD_PLAN_CANONICAL]].
- iOS / Codemagic (needs Apple account — purchased, not yet activated for store submit)
- New Horizon commercial Duffel/Viator expansion
- COMMS LINK store packaging

## Blocked on

| Item | Blocker | Owner |
|------|---------|-------|
| Play Store submit | Graphics + privacy URLs + keystore + $25 account | Nicholas |
| Monitor SMS alerts | Twilio A2P 10DLC campaign approval | Nicholas (wait) |
| Vercel function region | `apex-venue-site` builds in `iad1`; DB is us-west. Must change in Vercel settings | Nicholas |

Related: [[hot]], [[index]], [[log]]