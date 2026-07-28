# Apex v2 — Code Audit

**Date:** 2026-07-27
**Status:** Complete — findings awaiting triage
**Repo:** [nicholaswittle/apex_v2](https://github.com/nicholaswittle/apex_v2) @ commit `8f64bf8` (PA labor guardrails and calendar export)
**Related:** [[RESTAURANT_OS_BUILD_PLAN]]

---

## Verification Performed

Audit ran against a local clone, not just a read-through:

- `flutter analyze lib/ test/` — **clean, no issues** (confirms the working agreement is being honored)
- `flutter test` — **28/28 passing** (demo backend, clock QR, guardrails, calendar, text parser)
- Manual review of: all 3 Supabase migrations, `lib/core/*` (entitlements, shift_time, offline_sync, clock_qr, demo_backend, labor_guardrails, notification_service), app shell, and the major feature screens (dashboard, tips, schedule, photo import, QR wall)

---

## Overall Impression

Unusually disciplined for a fast-moving, multi-tool repo. The README working agreements, the module registry in `app.dart` (a genuine one-line plug-in contract), idempotent migrations with reasoning written into comments, the unique index on `tip_pools (org, date)` preventing double payouts, largest-remainder tip splitting so cents always reconcile, and demo mode at the HTTP layer — all endorseable decisions. The code reads like one coherent author despite many contributors; the process is working.

**The connecting thread across most findings:** the things that touch **money and identity** (name-keyed staff, client-only QR, non-transactional payouts, queue duplicates) are where the app is most trusting. Harden these before real payroll runs through it.

---

## High Priority

### 1. Staff identified by display name, not user id

`shifts.staff` is a name string. Name-matching runs through:

- `lib/core/labor_guardrails.dart` (`eq('name', staff)` for the DOB lookup)
- `lib/core/conflict_detector.dart`
- `lib/features/swaps/swaps_screen.dart`
- `lib/features/dashboard/employee_dashboard.dart`
- `lib/features/schedule/assign_days_screen.dart`

Consequences: two servers named "Sam" collide; a profile rename orphans all shift history; the guardrail minor-check can read the wrong person's DOB.

**Action:** migrate to `shifts.user_id` (keep the name as denormalized display text). Biggest structural debt in the repo.

### 2. Clock-QR is fraud-resistant in appearance only

- Payload is static per day (`APEX|<org>|<date>|<digest>`) — a photo of today's QR works from anywhere until midnight
- The 6-char short code can be texted to a friend
- Validation is client-side only; the server never sees QR proof
- The pepper has a compiled-in default (`apex-v2-clock`)

**Action:** `QrWallScreen` is already a live display — rotate the payload TOTP-style every 30–60s so photos die in a minute; validate punches server-side. At minimum, always set a real `CLOCK_PEPPER` in release builds.

### 3. Offline punch queue failure modes (payroll-relevant)

In `lib/core/offline_sync.dart` and `employee_dashboard.dart`:

- `flush()` stops on the first failure — one permanently-failing row (RLS denial, constraint) blocks every punch behind it forever; no dead-letter path
- `_clockIn` routes **any** exception to the offline queue, not just connectivity — a real server rejection masquerades as "Saved offline"
- No idempotency key — a flush that succeeds server-side but times out client-side double-inserts on retry

**Action:** distinguish network vs. permanent errors; add client-generated idempotency key; add max-attempts + surfaced failure state.

### 4. Two RLS gaps

- `messages` insert policy passes when `system_generated` is true **regardless of `user_id`** — any member can forge a system message attributed to someone else. Restrict (service-role only, or force `user_id` null when system-generated).
- `has_role(org, 'owner')` is satisfied by managers — the `role in (required, 'manager', 'owner')` pattern makes managers satisfy *any* required role. Latent today (nothing calls it with `'owner'`), but replace the string-list trick with an explicit rank map before it matters.

### 5. Tip publish is non-transactional and can wedge itself

`tip_management.dart` `_confirm()`: inserts the pool, **then** the allocations. If the allocations insert fails, the pool row remains — and both the client check and the unique index then block every retry with "Tips already split for this date." Manager is stuck with an empty payout for that date.

**Action:** wrap pool + allocations in an RPC/edge-function transaction. Same concern applies to multi-row schedule publish.

---

## Medium Priority

| # | Finding | Notes |
|---|---------|-------|
| 6 | Entitlements are client-side only | RLS checks membership/role, never tier — a free venue can reach pro-module data via the API. Fine as soft monetization gating, but say so in the README so nobody mistakes it for enforcement. |
| 7 | Venue timezone assumed = device timezone | `localDayBoundsUtc` uses phone-local midnight for tip/labor day boundaries — wrong when a manager travels. Store `timezone` on `organizations`. |
| 8 | Money-math duplication is worse than the README admits | The private `_hoursBetweenTimestamps` in `tip_management.dart` silently falls back to clock-face parsing on timestamp-parse failure — the exact mispricing the README warns against — while `shift_time.dart`'s version returns 0. Two copies, two behaviors. Consolidate into `shift_time.dart` with an explicit legacy flag. Credit: `labor_cost_dashboard` did wire to `shift_time` as agreed — but its doc comment still says "Currently unused." |
| 9 | Open punches price at zero, silently | A closer who forgets to clock out is excluded from the tip split with no warning in the draft review. Show "N open punches excluded" before confirm. |
| 10 | Server logic outside version control | Three edge functions are referenced (`route-notification`, `parse-schedule`, `send-push-notification`) but `supabase/functions/` is not in the repo — the code a concurrent writer can least afford to lose. |

---

## Low Priority / Hygiene

- **README is stale** — says "No Flutter scaffold yet — `lib/` only" and lists 3 features; repo has full scaffold, ~14 features, tests, demo mode. The doc meant to onboard AI tools now misleads them.
- **No CI** — with multiple tools committing concurrently, a GitHub Action running `flutter analyze` + `flutter test` per PR is the cheapest race-catcher. Also commit `pubspec.lock` (app, not package) so every tool resolves identical deps.
- **Untested money function** — `_splitByHours` (largest-remainder split) is private and untested. Move to `core/`, test reconciliation (sum == pool), remainder distribution, ties.
- `shifts.start_time` / `end_time` are `text` columns — make them `time` so invalid values can't be stored.
- Dashboard refetches everything on any realtime event across 4 tables — fine at restaurant scale; consider debouncing.

---

## What's Already Good (keep doing)

- README working agreements + commit-before-build discipline (born from the 2026-07-27 lost-file incident)
- Module registry in `app.dart` — one line per feature; shell handles entitlement gating and routing
- Idempotent, non-destructive migrations with design rationale in comments (the `organization_members` deviation note is exemplary)
- `tip_pools (organization_id, shift_date)` unique index — the double-payout guarantee is real at the DB level
- Largest-remainder tip split — allocated cents always equal the pool total
- `hoursBetween` vs `hoursBetweenTimestamps` distinction, documented and tested
- Demo mode at the HTTP layer — feature code untouched, demos run the real screens
- In-flight guards (`_clocking` / `_posting` / `_saving`), `mounted` checks, consistent SnackBar feedback
- Membership-model deviation documented in SQL rather than silently invented

---

## Suggested Triage Order

1. Tip publish transaction (#5) — money correctness, small fix
2. Offline queue error classification + idempotency (#3) — payroll correctness
3. `messages` RLS + `has_role` rank map (#4) — one migration
4. Staff `user_id` migration (#1) — largest, plan it before more features key off names
5. QR rotation + server validation (#2) — anti-fraud
6. Edge functions into the repo (#10) — then everything else
