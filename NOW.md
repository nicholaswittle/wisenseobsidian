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
2. [x] ~~Read-side tier gaps~~ — CLOSED 2026-08-02. `server_tips` DELETE and `capacity_events` SELECT now gated. `server_tips` SELECT left open on purpose (see Today, item 4).
3. [x] ~~`route-callout` / `venue-support-agent` unmetered~~ — CLOSED 2026-08-02. Both now go through `chargeAiCall`. No uncapped AI spend remains.
4. [ ] `shifts.user_id` is null on all 91 rows; the dashboard matches shifts to users by profile **name string**. A rename, a nickname, or two staff sharing a first name silently empties or crosses a dashboard. Backfill + match on id.
5. [ ] `staff_console_screen.dart` is 2001 lines — growing, not shrinking
6. [ ] 4 silent `catch (_) {}` blocks, plus ~8 in the admin console that log without surfacing to the manager
7. [ ] No payment-path tests. 92 source files, 13 test files, **zero covering money or auth**.
8. [ ] `supabase/keys.txt` is in git history — confirm the repo is private
9. [ ] No router: 29 imperative `Navigator.push` calls, no deep linking or web back-button handling

### Parked

- **Apex go-to-market / pricing** — see [[business/Apex Go-To-Market — parked 2026-08-02]]. Three-tier model settled (free generic site · $499–799 un-generic site · $99/mo + 1.5%), and deliberately parked: none of it is testable without a second customer. Two things in it are not business decisions and should not wait — **five parameterised themes** (2–4 days; converts the $499 deliverable from days of hand-written code into ~90 min of config) and the **referral table** (2 hrs; there is no referrals table, so the referral tile records nothing).
- iOS / Codemagic (needs Apple account)
- New Horizon commercial Duffel/Viator expansion
- COMMS LINK store packaging

## Today — Sunday 2026-08-02

> **Build 6 has an ordered runbook: `apex_v2/docs/BUILD_6_RUNBOOK.md`. Follow it
> in order.** Two near-misses today came from a server-side change landing
> before the client that could handle it — assigning `Admin` stripped Emily's
> manager UI on the shipped build (reverted), and the tip-pool migration would
> have blocked tip splitting with no in-app recovery. Ship the client, confirm
> it is installed, *then* change the data.

Fable 5 audit landed: `apex_v2/docs/AUDIT_2026-08-02_FABLE.md`. Its headline is
worth recording — **it went looking for a way to mark an order paid without
paying, or route a payment to the wrong account, and found none.** Two live
`BEFORE UPDATE` triggers on `online_orders` block client mutation of amounts and
payment status, which is why the permissive `online_orders_member_update` policy
is not the hole it appears to be. Both webhooks verify the connected account
owns the order *and* verify the amount before marking paid.

1. [ ] **Install build 5**; verify the "viewing this venue" banner as `apextest@gmail.com`, then test clock-in as `emilyykidman@gmail.com` (she has real shifts). Clock-in is still completely untested.
2. [ ] **Backfill `shifts.user_id` and move lookups off name equality** — the audit's top finding and it is worse than assessed here yesterday: the name string is load-bearing *inside Postgres*, not just in Dart. The `sidework` "staff complete" policy resolves `assigned_to` against `profiles.name`, so two staff sharing a first name is an **RLS-enforced** cross-user leak — one can complete the other's sidework. 91/91 rows NULL. ~0.5–1 day.
3. [ ] **Place a pay-at-pickup order** end to end. Last untested leg of the ordering path, costs nothing.
4. [x] ~~Tier-gate `server_tips` DELETE and `capacity_events` SELECT~~ — DONE overnight, verified in `pg_policies`. `server_tips` SELECT deliberately left open: gating it would hold a downgraded venue's servers' own tip history hostage to the venue's billing. The migration says so.
5. [x] ~~Meter `route-callout` + `venue-support-agent`~~ — DONE overnight, both deployed. Metered differently on purpose: the support agent returns the gate's refusal (the model *is* the feature); `route-callout` degrades to an unranked list instead, because a call-out is someone missing a shift that starts soon and a spend cap must not become a staffing failure.
6. [ ] **Still open from the audit's AI section**: cut `polish-labor-warnings` (warnings are already generated deterministically in `labor_guardrails.dart`; the model only reproses them) and template `venue-briefing` (134 calls/30d, the largest AI line item). Both touch user-facing copy, so they want a fresh session, not the tail of a long one.

Audit's strategic call, for later: **payroll is the single gap that ends sales
calls.** Homebase, Push and Toast all have it. The fix is an export/integration
(Gusto/ADP mapped to `time_entries` + `server_tips`), not becoming a payroll
company. ~1 week.

**Resolved, do not action:** the audit flagged `supabase/keys.txt` as
verify-and-rotate, unable to decode a suspected second token. Checked — a JWT
contains two `eyJ` runs (header and payload), so one key reads as two. The file
is two lines: project URL and one token, `role: anon`. Public by design, ships
in the client bundles already. Nothing to rotate.

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
