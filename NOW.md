---
title: NOW — Weekly Scorecard
tags: [meta, tasks, scorecard, now]
updated: 2026-08-01
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
| Apex RLS on staging | Applied + smoke-tested | Pending (human) | Migration in repo |
| Apex store packaging | Keystore + AAB built | Not started | Gate C note archived |
| Jigsy pilot interviews | ≥1 owner note | DONE | Signed YES — see [[customers/Jigsys Brewpub]] |
| Experiments with outcomes | Keep/kill filled | See [[business/Experiment Log]] | |

## This week — next actions

### Human-only (Nicholas)

1. [ ] When the Jigsy iPad arrives: factory-reset, verify no Activation Lock/MDM, connect to venue Wi-Fi, install TestFlight, accept Apex pilot invitation
2. [ ] Sign into Supabase dashboard and create the authenticated 15-minute schedule for `reconcile-pending-payments`
3. [ ] Before Square go-live: obtain owner authorization, complete OAuth, verify `square_environment=production`, POS order creation, printer profile, and fee receipt
4. [ ] Apply Apex RLS migration on Supabase staging → smoke-test org isolation → prod
5. [ ] Gate C assets: app icon 512, feature graphic 1024×500, screenshots (both apps)
6. [ ] Host privacy policy URLs (Apex + COMMS LINK)
7. [ ] Create Android upload keystores
8. [ ] Add `com.nicholaswittle.apex://` to Supabase allowed redirect URLs (iOS login)
9. [ ] Buy Google Play ($25) and Apple Developer ($99) accounts

### Known issues (from vault cleanup audit)

1. [ ] Apex v2 tier enforcement gap: tip_pools, tip_allocations, server_tips, capacity_events, call_outs have plain org-member RLS — no tier check. Free venues can access Pro/OS features via PostgREST
2. [ ] Apex v2 deploy is stale (5 commits behind HEAD, including Square safeguards)
3. [ ] staff_console_screen.dart is 1952 lines — needs decomposition
4. [ ] 4 silent `catch (_) {}` blocks in production code paths
5. [ ] No payment-path tests (ordering module has 0 dedicated tests)

### Parked

- iOS / Codemagic (needs Apple account)
- New Horizon commercial Duffel/Viator expansion
- COMMS LINK store packaging

## Blocked on

| Item | Blocker | Owner |
|------|---------|-------|
| Apex launch security | DB access to apply RLS | Nicholas |
| Play Store submit | Graphics + privacy URLs + keystore | Nicholas |

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
