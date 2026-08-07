---
type: assessment
title: "Apex v3 vs v2 — Honest Assessment"
tags: [apex, apex-v3, assessment, strategy, review]
date: 2026-08-05
status: active
---

# Apex v3 vs v2 — Honest Assessment

> **STATUS UPDATE 2026-08-06.** The strategic verdict below still holds, but the
> factual state has moved. v3's Phase 0 is now **closed** (not "not closed" as
> this note's 08-05 snapshot says), and v2 has hardened its money path. Updated
> comparison with grades, grounded in both repos at HEAD 08-06:
>
> **v2 — B+ (launch product).** Complete product (96 lib files, 182 tests, 168
> migrations, 28 edge functions, live customer Jigsy's + services vertical).
> Cold-test rework in. **Deposit cap fixed** (`100/3`), **refund-binding security
> fix in** (`acac08c` — closes the exact shared-Stripe hole v3 flagged). Two-branch
> situation resolved (`services-merged` + `template-to-product` merged into
> release). Repo private. **Drag: process/trust C+** — 46 branches, `main` 127
> commits behind the release branch, ledger-drift history. Launchable, but the
> process debt is why v3 exists.
>
> **v3 — B+ (foundation).** 82 commits, 69 migrations, 248 RLS assertions, 12 CI
> gates green, 173 tests. Phase 0 closed (all 3 blockers proved). Money path
> deployed (15/28 functions). Phase 1 started: onboarding conveyor, auth, Toddler
> Bar as compile error, operations layer (D26). **Process/trust A-** — gates test
> reachability, red-then-green discipline, repo↔ledger md5 parity. **Completeness
> C+** — foundation, not product; no feature screens, no real payment taken.
>
> **The two B+ grades mean opposite things:** v2 is a complete product with a
> process problem; v3 is a clean process with an incomplete product. Exactly the
> trade the plan makes — v2 carries revenue and the customer, v3 carries trust.
>
> **Three things that matter most now:**
> 1. v2's refund-binding fix (`acac08c`) de-risks the shared-Stripe-platform debt
>    — confirm it's deployed to v2 prod, not just committed.
> 2. The v2 nightly drift alarm is still deferred (needs a read-only credential).
>    Pull it forward — v2 is the launch product being actively changed.
> 3. v3's Phase 1 gate ("real order paid") will hit the shared Stripe platform.
>    Decide the own-platform question before that gate, not after.

---

A candid read of the rebuild versus the incumbent, and of v2's current state.
Written 2026-08-05 alongside [[projects/APEX_V3_BUILD_AND_STATUS_2026-08-05]] and
[[projects/APEX_AUDIT_FABLE5_2026-08-03]]. Intended as the reasoning behind the
choice, not as a brag sheet.

---

## Apex v2 — current state

**Grade: C+** (Fable's verdict, agreed). A-range code craft, B+ restaurant
half, but a product fragile-by-process and unproven where it counts.

### What is genuinely good
- Restaurant ordering + payments (7,277 lines) survived two payment audits. Hardened.
- AI briefing is the most-used feature (112 calls) — the real differentiator.
- Late security work (anon EXECUTE revoke, identity to user_id, tip pool live)
  proves the codebase CAN be fixed.
- Services vertical is ~80% built and the site is live — a real second product.

### What is genuinely broken
- **Process never recovered**: 25 branches, double-applied migrations,
  repo/prod drift, secrets in .rtf. The audit could not reliably say "what is
  running" — the fatal indictment for a money product.
- Employee core is unproven (~1 row per table). Staff do not use it. The
  "daily loop" pitch rests on something never validated.
- Services invisible to customers: quote link lands on a sign-in screen
  (quote→accept loop unreachable), $n-spots-open bug on the first screen a
  crew sees, one payment test file.
- Two diverged branches need careful merge; the GPT branch reverts five fixes
  including the deposit cap — a live money bug in a branch.
- Services/restaurant fee asymmetry (contractor-paid vs guest-paid 1.5%) is an
  accident, not a decision.

**The core problem is not the code — it is trust in the codebase.** You cannot
take real money with a system you do not trust. That is the specific thing
blocking the revenue gate.

---

## Apex v3 vs v2

V3 as designed is correct, for exactly the right reason: **it fixes the floor.**
One repo, one main, deploy from CI only, nightly drift alarm, money-guard
completeness test, RLS suite. These mechanically answer "we don't know what's
running." V3 is the precondition for ever taking a card safely. It beats v2
decisively here — v2 could never patch its way there, because the rot was
process-level.

**But v3 is already repeating v2's disease.** Five adversarial review rounds;
four of them each introduced the next round's defect. The money-guard gate
stayed green with the trigger dropped. The RLS suite counted NULL as a pass.
That is the same failure mode as v2, in miniature: fixes reasoned-about instead
of proven, tests that check text instead of reachability.

The plan diagnosed it correctly — "every fix ships with an assertion watched
FAIL before and PASS after" is the right countermeasure. And the three blockers
(QR flood, unbounded open punches minting hours/tips, self-promotion to super
admin) are serious, real defects; the review process IS finding them. But it
took five rounds to learn a lesson that should have been the first rule, and
Phase 0 still is not closed. That is expensive.

**The honest verdict:** v3 is the right foundation and better than v2's. But
"we'll get it right this time by being careful" is not a strategy — it is the
same bet v2 lost. What saves v3 is the mechanization, and it only helps if the
gates test reachability, not text. That is the one thing v3 must not fumble —
and it already did, for four rounds.

---

## The strategic question that matters more than the code

Fable's structural claim inverts the master plan: **services is the better
market than restaurants, and it is already built.** Restaurants are capped at
$0-40 by free incumbents (7shifts, Homebase, Square). Services tolerates $99
flat, has a villain in per-seat pricing, and a statutory hook (HICPA) no
national competitor bothers with. The audit recommends customers #2 and #3 be a
landscaper and an HVAC/plumbing shop — not a second restaurant.

The tension: v3 restarts the clock on a foundation rebuild while the services
vertical that could make money sits in v2 at C+, one week of evenings from B+.
Pouring all energy into v3's foundation starves the thing most likely to
produce the $300/mo target.

**Framed as a sequence, not a code choice:**
- V2's services vertical, fixed and deployed (quote link, branch merge,
  deposit cap), is the fastest path to a paying customer — B+ within a week.
- V3 is where that customer should eventually live, but only after v3's floor
  (gates that actually test reachability) is proven. Porting a paying services
  customer onto a rebuilt foundation you do not yet trust would recreate v2's
  problem.

**Recommendation: do not choose v3-or-v2 as the product.** Treat v3 as the
future home, but monetize the already-built v2 services vertical NOW to fund
and validate it. V3's job is to become the house that revenue moves into — not
to delay the revenue while the house is being built.

---

## What to do next

1. **Fix the two-branch situation first** — a live money bug (deposit cap
   reverted) regardless of anything else. Merge in the documented order,
   keeping both the manager-check and the `100/3` cap.
2. **Deploy the services site + fix the quote link** — the C+ → B+ move in a
   week, and the actual path to a paying customer.
3. **Keep v3 going, but treat Phase 0's "gates must test reachability, not
   text" as the make-or-break deliverable** — not the schema, not the packs.
   If v3 closes Phase 0 with gates that cannot lie, it has earned the right to
   be the home. If it closes with "carefully reviewed" gates, it is v2 in a new
   repo.

**The uncomfortable summary:** v3 is the right answer to a real problem, but
its execution already inherited v2's core flaw. The code will get fixed. The
thing to watch is whether the process gets fixed — and the proof of that is a
gate that goes red when it should, not a plan that says it will.

---

Related: [[projects/APEX_V3_BUILD_AND_STATUS_2026-08-05]] ·
[[projects/APEX_AUDIT_FABLE5_2026-08-03]] ·
[[projects/APEX_SERVICES_COMPETITIVE_BUILD_MAP]] ·
[[projects/APEX_MASTER_PLAN]] · [[DECISIONS]] · [[hot]] · [[NOW]]
