---
type: project
title: "Apex v3 — Build Plan and Status"
tags: [apex, apex-v3, build-plan, security, phase-0]
updated: 2026-08-05
status: active
---

# Apex v3 — Build Plan and Status

> **STATUS UPDATE 2026-08-07 — Phase 1 WELL UNDERWAY.** The vault snapshot below
> (written 08-05) is 60+ commits out of date. The authoritative plan is now
> `apex_v3/docs/MASTER_PLAN.md` (canonical 08-06; its Status section is itself
> stale — the repo is ahead); the running open-items list lives in the private
> vault at `Notes/projects/APEX_V3_OPEN_ITEMS_2026-08-06.md`. What changed since
> the 08-06 update:
>
> - **Phase 1 WELL UNDERWAY** (08-07, 116 commits, +34 since 08-06). Tests
>   doubled 173→339, lib files doubled, **16 registered operations**. Built:
>   onboarding conveyor (iterated vs "what two real people could not do"),
>   restaurant menu flow (photograph/check/confirm), **guest ordering** live
>   (venue link → guest flow → cart → order status), schedule (author-as-draft
>   /publish-as-one-decision, photo import, staff self-serve), staff console
>   (vocabulary by vertical, D32/D33), parse-menu scoring harness ported from
>   v2, synthetic restaurant seed.
> - **Operations layer is now real, not just a contract** — 16 registered ops:
>   order lifecycle split by facts (D29), refund as ONE atom with reserve/
>   settle/void protocol (D34), idempotency via client_token. Order, schedule,
>   money paths all built as operation callers.
> - **Payment gate amended honestly (08-07):** "real order placed → paid →
>   refunded" now explicitly **in Stripe test mode** — the plan states what test
>   mode proves and what it cannot (real bank arrival, KYC, chargebacks).
> - **Grade 08-07: A- as a Phase-1 foundation.** Architecture A, process A-,
>   velocity A- (doubled test/lib in ~24h, gates green). Not yet a product —
>   no real payment, no real customer, **the "same order placed by the
>   assistant" gate unproven** (the make-or-break).
> - **Still open / largest structural weakness:** the `service_role` exemption
>   is load-bearing across five guards. Shared Stripe platform with v2 still
>   behavioral isolation. Blocked on Nick: Sentry connector, function secrets
>   re-issue, `APEX_V3_SERVICE_ROLE_KEY`, v2 nightly drift alarm
>   (`V2_READONLY_DB_URL`).
>
> The historical record below (five review rounds, the three root causes, the
> gates that lied) is retained as-is — it is the reason the standing rules exist.

---

> **Status on 2026-08-05: Phase 0 is NOT closed.** Five review rounds have run.
> Round five (four parallel reviewers) returned **3 BLOCKERs and ~12 HIGHs**, and
> the database-tier remediation is in flight. Do not treat v3 as ready to build
> an app on. Do not port UI yet.

Repo: `C:\development\projects\apex_v3` — GitHub `nicholaswittle/apex_v3` (private).
Supabase: **apex-v3-prod** `fnsonnhumcvxdnyarguv`. v2 project `pqkremkwfkudrhtxasdj`
is a different live product and is **off-limits** to v3 work.

---

## Why v3 exists

v2 was audited in full on 2026-08-05 (scorecard + plan in `apex_v2/docs/`). The
finding that drove the rebuild: ordering and AI are proven and carry real
traffic, while the employee core and notifications are largely unused. v3 keeps
the core principle and rebuilds the foundation rather than patching around it.

The mandate Nicholas set, and which governs every decision below:

> "We aren't rushing this project like we did with Apex v2. Let's get it all
> right the first time. As close to perfect as we can prior to moving on."

That is why Phase 0 has consumed five review rounds instead of shipping.

---

## Architecture

**Vertical packs.** A neutral core spine (organizations, profiles, membership,
venues, push tokens) with a restaurant pack layered on top (orders, menu,
capacity, revenue). Modules gated by tier entitlements. This is what lets a
second vertical land without forking the schema — the v2 lesson.

**Domain event outbox.** `apex_emit_event` writes every state change to
`domain_events`. ⚠️ **The producer is wired and there is no consumer** — see
Open Items.

**Idempotency everywhere.** `client_token` on orders, `operation_id` on punches,
both unique-indexed, both replay-safe.

**CI as the contract.** Seven gates run on every push: migration replay against
a clean Postgres 17 container, migration lint, ledger-vs-repo drift, a grant
allowlist, money-guard completeness, the RLS suite, and a Flutter detector.

---

## What has actually been built

| Piece | State |
|---|---|
| Core spine schema | Applied |
| Restaurant pack (orders, menu, capacity, revenue) | Applied |
| Auth/signup, invite-only membership | Applied |
| Scheduling + time clock | Applied, **holes found round 5** |
| Tips (pools, allocation, eligibility) | Applied, **correction path missing** |
| 33 migrations, repo↔ledger checksum-identical | Verified by md5 |
| RLS/authorization suite | ~112 assertions |
| 7 CI gates | Green, **but see "gates that lied"** |
| 27 edge functions in repo | **0 deployed, 0 tested, 0 gated** |
| Flutter app | Not started |

---

## The five review rounds

Each round was an adversarial review by a cold-context agent instructed to
assume the previous author was confidently wrong.

| Round | Result |
|---|---|
| 1–2 | Baseline defects; Supabase default-privilege re-grant discovered |
| 3 | **BLOCKER: the money guard never fired on any RPC path.** Demonstrated live — a $10.00 order rewritten to $0.01 |
| 4 | 0 blockers, 3 HIGHs — incl. staff able to backdate a time punch |
| 5 | **3 BLOCKERs, ~12 HIGHs** — four parallel scoped reviewers |

**Round 5 ran as four parallel reviewers** (money/payroll, guest/tenancy,
CI honesty, seams) rather than one generalist. That format is now standard —
see [[DECISIONS]].

### The signature failure of this project

**Four consecutive rounds each introduced the next round's defect.** Every fix
was reasoned about rather than proven. Every gate tested a rule's *text* and
never its *reachability*.

The countermeasure, now mandatory: **every fix ships with an assertion watched
FAIL before the change and PASS after.** No exceptions, no "obviously correct."

---

## Round 5 — the three root causes

Twelve findings, three causes. This is the important section.

### 1. The security model lives in the wrong place

Every clamp, window and guard built across five rounds lives **inside RPCs**,
while `authenticated` holds direct INSERT/UPDATE/DELETE through PostgREST on
`time_entries`, `online_orders`, `organization_members`, `daily_revenue`,
`server_tips` and `swaps`. None of the RPC logic is in that path.

Proven consequences: a manager can rewrite or delete any employee's hours with
no trace; an owner can graft a stranger into their org and read that person's
name, email, phone and date of birth; declared tips (a tax record) are
self-mutable with a client-supplied timestamp; a manager can mark a shift swap
approved without the shift ever moving.

Policies say **who** may write. Nothing says **what values**, or **that it be
recorded**.

### 2. The privilege alphabet was incomplete

The audit-table lockdown used `revoke insert, update, delete`. That leaves
`TRUNCATE`, `TRIGGER` and `REFERENCES` — and **TRUNCATE ignores RLS entirely**,
cross-tenant, in one statement, on `domain_events` and `tip_eligibility_audit`.
The CI gate could not catch it because it enumerated four of the seven
privileges that "ALL" means.

Not reachable via PostgREST today (no TRUNCATE verb), so defence-in-depth
rather than live exploit — but the append-only guarantee rested on missing
policies alone.

### 3. Half the system has never been reviewed

27 edge functions in the repo, **zero deployed**. No deploy job, no type check,
no `config.toml` — so `verify_jwt` for the Stripe and Square webhooks is
unversioned and would be set by hand in a dashboard. No `pg_cron`, no `pg_net`,
so the three sweep functions have no trigger mechanism and `domain_events` has
no drain.

And every edge function runs as `service_role`, which the money guard exempts
**unconditionally**. The entire tier that will actually move money bypasses five
rounds of hardening by construction.

---

## The three BLOCKERs

**BL-1 — a QR-code holder can take a venue's ordering offline.** The per-venue
attempt ceiling was evaluated unconditionally at the top of `place_order`, so
once flooded, real customers were refused too. Self-inflicted in round 4; the
code comment claimed the anti-DoS property survived because the error string
differed, which is a rename, not a control.

**BL-2 — unbounded open punches mint hours and tips.** Round 4 capped
*backdating*. Nothing capped *duration*. Clock in Friday 17:00 legitimately,
clock out Saturday 16:00 with no arguments — every clamp passes, 23-hour entry.
Tip pools split strictly pro-rata by hours, so this captures roughly half a
shift's pool. No duration constraint, no `pg_cron` to sweep stale punches.

**BL-3 — self-promotion to super admin.** `authenticated` holds UPDATE on
`profiles`; one untested trigger branch is the only thing stopping it. And
`apex_guard_order_money`'s first statement exempts `is_super_admin()`
unconditionally. The suite tested the INSERT branch only.

---

## Gates that lied (and why it matters)

- The gate named **money-guard-completeness** parsed the guard function's text.
  Dropping the trigger entirely left it at **exit 0, green** — it would certify
  a table with no guard on it.
- The RLS suite's failure detector used `where not pass`, which **drops NULL**.
  Any assertion evaluating to NULL scored as a pass. (Latent, not live — zero
  assertions were actually NULL when checked.)
- One assertion proven vacuous: `tips: staff cannot commit a split` raised for
  an unrelated reason, so deleting the manager-only gate on the tip-commit money
  path left it green.
- The **grant-allowlist** gate records which *roles* a policy names, never what
  it restricts. `USING (is_member(organization_id))` rewritten to `USING (true)`
  produces a byte-identical allowlist line.
- **17 policy-bearing tables have zero behavioural assertions**, including
  `order_items` (the priced line items of every order) and `server_tips`.

> The reviewer's verdict, worth keeping: *the coverage is shaped like the review
> history, not like the attack surface.*

---

## Standing rules carried into every later phase

These are the generalised causes. They apply to Phase 1 and beyond.

1. **Every new table gets its write path decided explicitly** — RPC-only, or
   direct DML with pinned immutable columns plus an audit trigger. Never "add
   table, add policy, move on."
2. **Every revoke uses `revoke all` then re-grants.** Never enumerate privileges.
3. **Every control gets a test of the victim, not the attacker.** A rate-limiter
   test that asserts the attacker was refused proves nothing; assert the real
   customer still gets through.
4. **For every operation locked down, prove a legitimate actor can still
   complete it.** This caught the tip-split becoming permanently unfixable.
5. **Ask what a user with entirely legitimate credentials can abuse** — dishonest
   employee, dishonest owner, dishonest customer — and separately, **would
   anyone be able to tell afterward?** Missing provenance is itself a finding.
6. **Audit the catalog, not the repo.** Supabase re-grants EXECUTE on new
   functions, so an early revoke can be silently undone by a later
   `create or replace`.

---

## Open items

**Blocked on Nicholas**
- Secrets into Supabase function secrets: Twilio, FCM, Anthropic, Gemini,
  Stripe, Square.
- Sentry has no Apex v3 project — org `wisense-llc` holds only `flutter`, so v3
  errors have nowhere to land.
- **Product decision:** menus are intentionally public, but the same policies
  expose every ordering-enabled venue's address, phone, coordinates, tenant id
  and full pricing **with no token at all** — effectively the platform's
  customer list. Deliberate choice needed, not an inherited default.

**Done by Nicholas 2026-08-05:** leaked-password protection enabled on the
project.

**Next stage (not started)**
- The edge-function tier: `config.toml` with explicit `verify_jwt` for all 27,
  a `deno check` CI job, a deployment drift gate mirroring the ledger gate,
  webhook idempotency, and a decision on `pg_cron` for the outbox drain.
- Restore drill and v2 drift alarm (Phase 3).

**UI — gated on Nicholas's review.** Standing instruction: he is to be notified
before any screen is ported, and wants a design-system + flow proposal reviewed
first. v2's layout was not user-friendly and cohesion is the explicit goal.
Competitive research on the top four competitors is done.

---

## Sequencing decision (2026-08-05)

Nicholas chose: **fix the database tier now, edge-function tier as its own gated
stage, UI after.** Rationale — each stage stays provable, and the database is
what the functions write through. He also set the general rule: **fix each phase
as it is built, to prevent drift.**

Related: [[projects/APEX_MASTER_PLAN]] · [[DECISIONS]] · [[hot]] · [[NOW]] · [[projects/APEX_V3_VS_V2_ASSESSMENT_2026-08-05]]
