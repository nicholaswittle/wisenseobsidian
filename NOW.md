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
| Stage | REVENUE / PILOT LIVE — FIRST CUSTOMER SIGNED (Jigsy's Brewpub) |
| Focus apps | Apex v2 + Jigsy Online Ordering (Stripe Pay Now live; Square prepared) |
| #1 metric | Onboard Jigsy's online ordering & website · ship Apex v1 |
| Working stack | Claude CLI + Ollama + Cursor |

## Scorecard

| Metric | Target | Current | Notes |
|--------|--------|---------|-------|
| First Customer | 1 signed | 1 SIGNED | Jigsy's — website + online ordering |
| Apex RLS on staging | Applied + smoke-tested | APPLIED TO PROD | Verified in pg_policies, not from files |
| Apex store packaging | Keystore + AAB built | Not started | Gate C note archived |
| Jigsy pilot interviews | ≥1 owner note | DONE | Signed YES — see [[customers/Jigsys Brewpub]] |
| Experiments with outcomes | Keep/kill filled | See [[business/Experiment Log]] | |

## This week — next actions

### Human-only (Nicholas)

1. [ ] When the Jigsy iPad arrives: factory-reset, verify no Activation Lock/MDM, connect to venue Wi-Fi, install TestFlight, accept Apex pilot invitation
2. [ ] Sign into Supabase dashboard and create the authenticated 15-minute schedule for `reconcile-pending-payments`
3. [ ] Before Square go-live: obtain owner authorization, complete OAuth, verify `square_environment=production`, POS order creation, printer profile, and fee receipt
4. [x] ~~Apply Apex RLS migration → smoke-test org isolation → prod~~ — DONE. Applied to prod and verified against the live catalog.
5. [ ] Gate C assets: app icon 512, feature graphic 1024×500, screenshots (both apps)
6. [ ] Host privacy policy URLs (Apex + COMMS LINK)
7. [ ] Create Android upload keystores
8. [ ] Add `com.nicholaswittle.apex://` to Supabase allowed redirect URLs (iOS login)
9. [ ] Buy Google Play ($25) and Apple Developer ($99) accounts

### Known issues (from vault cleanup audit)

1. [x] ~~Apex v2 tier enforcement gap~~ — CLOSED 2026-08-01. `apex_org_has_module()` now gates the write policies on all five tables; verified live in `pg_policies`, not read from a migration file.
2. [ ] **Read-side tier gaps remain**: `server_tips` SELECT and DELETE, and `capacity_events` SELECT, still carry no module check. Low severity. Recommendation on record: close DELETE only — gating the SELECT would hold a downgraded venue's servers' own tip history hostage.
3. [ ] `route-callout` and `venue-support-agent` are the only AI edge functions not metered through `chargeAiCall` — **uncapped per-invocation spend**. Highest-value open item.
4. [ ] `shifts.user_id` is null on all 86 rows; the dashboard matches shifts to users by profile **name string**. A rename, a nickname, or two staff sharing a first name silently empties or crosses a dashboard. Backfill + match on id.
5. [ ] `staff_console_screen.dart` is 2001 lines — growing, not shrinking
6. [ ] 4 silent `catch (_) {}` blocks, plus ~8 in the admin console that log without surfacing to the manager
7. [ ] No payment-path tests. 92 source files, 13 test files, **zero covering money or auth**.
8. [ ] `supabase/keys.txt` is in git history — confirm the repo is private
9. [ ] No router: 29 imperative `Navigator.push` calls, no deep linking or web back-button handling

### Parked

- iOS / Codemagic (needs Apple account)
- New Horizon commercial Duffel/Viator expansion
- COMMS LINK store packaging

## Tomorrow — 2026-08-03

1. [ ] **Read the Fable 5 audit** — `apex_v2/docs/AUDIT_2026-08-02_FABLE.md`. Four sections: security/correctness (weighted to the untested money path), AI feature strategy, competitive positioning vs 7shifts/Homebase/Toast/Square, and a five-item priority list.
2. [ ] **Install build 5** from TestFlight and verify the "viewing this venue" banner — sign in as `apextest@gmail.com`, view Jigsy's, confirm it no longer reads as an unstaffed venue.
3. [ ] **Test clock-in properly** — sign in as `emilyykidman@gmail.com` (she has real shifts on the Jigsy's schedule). This is what could not be tested today.
4. [ ] **Place a pay-at-pickup order** end to end on `apex-venue-site.vercel.app/jigsys-enola-7c2a/order`. The choice is live now. This is the last untested leg of the ordering path and costs nothing to exercise.
5. [ ] **Meter `route-callout` and `venue-support-agent`** through `chargeAiCall` — the only uncapped AI spend left.

## Blocked on

| Item | Blocker | Owner |
|------|---------|-------|
| Play Store submit | Graphics + privacy URLs + keystore | Nicholas |
| Vercel function region | `apex-venue-site` still builds in `iad1`; DB is us-west. `preferredRegion` is Edge-Runtime-only and does not apply — must be changed in Vercel project settings | Nicholas |

Related: [[hot]], [[index]], [[log]]
## Queued — after the AI support agent build

**Restore the Antigravity + headless Claude workflow.** Nicholas's original and
preferred shape: Antigravity holds the tasking and the plan, headless Claude
executes, and the master plan is tracked rather than living in whichever tool is
currently awake. The problem being solved is juggling — when a quota hits,
context fragments across tools and work restarts instead of continuing.

Requirements captured 2026-08-01:
- Headless Claude must run **Opus 5 at medium reasoning effort**
- Antigravity does the tasking and plan-keeping
- The master plan is tracked in this vault, not in a tool session, so a quota
  limit is an inconvenience rather than a reset

Claude is to produce the setup prompt once the AI support agent build is
complete. Do not start it before then.
