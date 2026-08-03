---
type: meta
title: "Hot Cache"
tags: [meta, hot-cache, context]
updated: 2026-08-02
---

# Recent Context

> **DISPATCH FROM [[projects/APEX_MASTER_PLAN]].** It is the single ordered view of state, constraints and the work queue. If it disagrees with a tool's memory, the master plan wins.

## Last Updated

2026-08-02. Apex main at `b06469c`, 141 tests green, analyze clean. Build 8 on TestFlight (Waiting for Review). Build 9 staged (tip-pool routed, hours-from-punches, Admin gates, monitor fixes). Jigsy's is the only live customer.

## Security — DONE 2026-08-02

🟢 **anon EXECUTE revoked on 56 SECURITY DEFINER functions.** Only 8 guest-ordering functions remain anon-callable. Repo is public, so function names and signatures were published — this was the cheapest risk reduction available.

🟢 **apex_set_role privilege escalation closed.** `has_role(org,'owner')` returns true for any manager; was used as an owner gate. Fixed: `is_owner(org)` for owner-only, `can_administer(org)` for owner-or-admin.

🟢 **Trigger functions locked down.** `apex_protect_*`, `handle_new_user`, etc. revoked from public/anon/authenticated. Triggers unaffected (run as table owner).

🟢 **Identity moved to user_id.** `shifts.user_id` backfilled (27 live rows), sync triggers keep it in step. Sidework RLS policy still has a name fallback that fails open on ambiguity — see [[NOW]] known issues.

## Bugs fixed 2026-08-02

- **`.order()` was descending everywhere** — postgrest-dart defaults to `ascending: false`. All 26 unqualified calls sorted backwards. Next shift, menu order, every list.
- **Worked hours came from the schedule, not punches.** Emily's 13-hour scheduled shift clocked at 2.84 hours reported 13 hours worked. "Est. pay" overstated 4.6x. Fixed: `hoursWorkedFromEntries`, 6 tests pin the real row.
- **tip_allocations realtime subscription never existed.** Filtered on `organization_id` but that table has no such column. Realtime rejected the binding silently. Now scoped by `user_id`.
- **Admin excluded by hand-rolled role checks.** Six screens tested `role == 'owner' || role == 'manager'` — Admin is neither. Nav offered the tile, screen refused entry. All six now go through `StaffRole`.
- **Share sheet white-on-white.** Same class as the clock button. Fixed with explicit ink constant.
- **Guest site scrolled sideways.** `1fr` grid with `nowrap` labels pushed past viewport on mobile. Fixed: `minmax(0, 1fr)` with wrapping labels.
- **Monitor's Twilio alert was silent.** `notify()` checked secrets were set but not whether the send succeeded. Twilio pending approval = every send rejected, looked identical to delivered. Now fails loudly.

## Outside monitor — LIVE

GitHub Actions cron, headless browser (Playwright). Checks: venue page 200, menu opacity > 0, item count > 0, order page renders cart, next-shift date is nearest future shift, `apex_venue_health` not paused during opening hours. Five secrets set.

**Two corrections verified 2026-08-02:** GitHub throttles `*/15` schedules — it fired once in 2h15m, not every 15 min. And SMS cannot deliver: Twilio account pending approval. Until it clears, the real alarm is the GitHub Actions failure email. Detection latency is hours, not minutes.

## Tip-pool eligibility — staged, not applied

Client code routed at `/tip-pool` as manager-only (build 9). Migrations `20260831000000`/`20260831000001` are **not applied**. The hazard is asymmetric: client alone is safe (code can't run), migration alone breaks splitting with no in-app recovery (`apex_guard_tip_pool` blocks until owner confirms roster, and the only caller of `apex_confirm_tip_roster` was the orphaned screen). They ship in the same window as build 9 or not at all.

Live compliance issue: tip pools currently include managers and owners, which PA law bars when a tip credit is taken. Seeded eligible: Emily, Avi, Courtney, Dana, Kim, Marsha, Morgan. Not eligible: Robin (owner, never on floor).

## Payroll Lite — decided, not shipped

**Hours report, not a pay calculator.** The tipped overtime rate was wrong (1.5x cash wage instead of 1.5x min wage minus tip credit). A 45-hour tipped week returned $22.50 where ~$33.13 was owed. Decision: remove the money columns rather than fix the rate. See [[DECISIONS]] 2026-08-02 and `apex_v2/docs/PAYROLL_LITE_SCOPE_2026-08-02.md`.

`feat/payroll-export` is stale — predates the 08-02 work, would delete the monitor, five migrations and six test files. Needs rebase before merge.

## Business context

🎯 **Target: $300/mo from three paying OS venues.** Not thirty. Self-serve is the product, not an acquisition necessity. Jigsy's network is the year-one plan. Retention is the hard part — one churn is a third of the business.

💵 **Fee model: direct charges + 1.5% guest-paid service fee.** On $32.96: guest pays $35.46, venue nets $33.61, WiSense earns 52¢. Break-even on Stripe Connect is ~$950/mo online volume; Square has no per-account Connect fees (better for small venues). See [[projects/APEX_PLATFORM_FEE_ECONOMICS_2026-07-30]].

💰 **Jigsy's numbers:** net sales $291,668, orders -14%, customers -12%, average sale +11%. Traffic is the problem. Labor % of net sales = 0.0% — they track no labor in Square. Apex has scheduling + timeclock + labor cost. That gap is the recurring revenue pitch. See [[projects/JIGSYS_BUSINESS_NUMBERS_AND_REVENUE_MODEL_2026-08-01]].

## AI policy

**Haiku for everything, Opus only for image OCR.** Two calls removed that only reprosed existing data. `polish-labor-warnings` should be cut to templates (warnings are already deterministic in `labor_guardrails.dart`). `venue-briefing` at 134 calls/30d is the largest AI line item — template 90% of it. See `apex_v2/docs/AUDIT_2026-08-02_FABLE.md`.

## Standing warnings

🔴 **Migration history has DRIFTED — do not run `supabase db push`.** 117 local files, 96 ledger records. Twenty-four files have no `schema_migrations` row even though their objects are live. Apply individual migrations explicitly.

⚠️ After any deploy, **hard-refresh** — the Flutter service worker serves the old bundle.

⚠️ `apex/apex/supabase/config.toml` still points at Horizon's project.

🔴 **Square Sandbox cannot test the value proposition.** No POS, no Restaurants, no application-fee reporting. Sandbox de-risks the code, not the product.

❌ **Tap to Pay is struck** — not available on iPads. Jigsy's wired contactless reader is card-present hardware inside Square POS, a different thing.

## Active threads

- Build 9 ships tip-pool routing + hours fix + Admin gates + monitor fixes
- Then: Phase A testing (real clock-in, real pay-now order, real pay period)
- Then: tip-pool migrations applied in same window as build 9
- Parked: payroll export (stale branch, needs rebase), services vertical (gated on vertical one paying)

[[NOW]] · [[index]] · [[projects/Apex v2 — Restaurant OS Build]]