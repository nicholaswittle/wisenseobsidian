---
title: Apex v2 — Restaurant OS Build
tags: [apex, restaurant-os, flutter, supabase, build, active]
aliases: [Apex v2, Restaurant OS Build, apex_v2]
date: 2026-07-27
status: active
---

# Apex v2 — Restaurant OS Build

> A **new build**, borrowing patterns from Apex v1 but aimed at the OS end-state
> — not a port and not required to resemble v1. Employee-first, 3-tap max, dark
> Material 3, built on the same Supabase with org_id scoping. The long game is a
> **multi-venue OS** (restaurant is vertical one); modules plug in individually
> or as the whole system.

**Repo:** https://github.com/nicholaswittle/apex_v2 · **Demo:** https://apex-v2-demo.vercel.app

Related: [[Apex Scheduler]], [[business/Restaurant OS Unified Build Plan 2026-07-27]], [[business/Apex Scheduler Reimagined 2026-07-27]], [[business/Apex Reimagined Build Order 2026-07-27]], [[business/WiSense Restaurant OS Master Plan 2026-07-27]], [[NOW]]

---

## Session Rollup — 2026-07-27

### Shipped

| Piece | File | State |
|---|---|---|
| Employee dashboard | `features/dashboard/employee_dashboard.dart` | Built |
| Manager log book | `features/log_book/manager_log_book.dart` | Built (Cursor) |
| Tip management | `features/tips/tip_management.dart` | Built (Cursor) |
| Labor cost dashboard | `features/labor_cost/labor_cost_dashboard.dart` | Built (Cursor) |
| **Schema migration** | `supabase/migrations/0001_apex_v2_foundation.sql` | Written, **never run** |
| **Entitlements** | `core/entitlements.dart` | Built — the plug-in mechanism |
| **App shell** | `app.dart` + `main.dart` | Built — module registry |
| Shared helpers | `core/shift_time.dart` | Built, used by labor cost |
| Demo backend | `core/demo_backend.dart` | Built, deployed |
| Platform targets | `android/` `ios/` `web/` | Added — it is a **phone app**; web exists only for the browser demo |

`flutter analyze lib/` clean. **Weekend 1 (log book · tips · labor cost) complete.**

### Key decisions

- **Entitlements are the OS plug-in contract.** `OsTier` (free/pro/os/multi)
  grants a baseline module set; per-org `enabled_modules` / `disabled_modules`
  add or remove individual pieces. A venue can buy one module or the whole OS.
  Adding a feature = one entry in `_moduleRoutes` in `app.dart`.
- **Membership stays on `profiles`(organization_id, role).** The build plan's
  RLS helpers referenced an `organization_members` table defined nowhere,
  absent from v1, and contradicted by all shipped screens. Documented in the
  migration; add it later only if a user must belong to several orgs.
- **Money math is not duplicated** — `hoursBetweenTimestamps` lives once.
  Clock punches are timestamps, never `HH:MM` strings.
- **Multi-venue, not restaurant-only.** Free+Pro tiers are already
  vertical-agnostic; only the vertical pack (ordering, capacity rule, revenue
  adapter, vocabulary) changes. See the Master Plan's *Multi-Venue by Design*.
- **All named-client references removed** from docs, incl. two pitch lines
  claiming an unmeasured "32% → 24%" result.

### What's LEFT — honest gaps

**Blocking anything real:**
1. **Migration has never been applied.** `shift_notes`, `tip_pools`,
   `tip_allocations`, `messages` do not exist in the database. Until it runs
   (staging → prod), no screen can load real data.
2. **No login/auth screen exists.** The shell shows "Sign in to continue" and
   there is no way past it. Required before any real-data test.

**Not built yet (v2 is 4 screens — these live only in v1):**
schedule / calendar UI · shift swaps UI · availability & time-off ·
staff management · team chat · onboarding. *A v2 demo cannot show a schedule
because v2 has no schedule screen.* This surprised Nicholas on 2026-07-27 —
worth stating up front in future demos.

**Open issues:**
- ~~Demo renders black~~ **RESOLVED 2026-07-27** (`0986d53`). Not a rendering
  or headers problem at all — a layout bug in `app.dart`. Each module button's
  `Column` defaulted to `MainAxisSize.max`, so the bottom bar expanded to the
  full viewport and the dashboard was laid out at **zero height**. The module
  screens were never affected. Diagnosed from Nicholas's screenshots, which
  showed the three nav cards stretched floor-to-ceiling and all three module
  screens rendering perfectly.
  **Lesson:** two rounds were spent guessing at COEP/CanvasKit because Claude's
  preview pane cannot composite frames. One user screenshot settled it. When
  rendering cannot be verified from this side, **ask for a screenshot first**
  instead of deploying speculative fixes.
- Demo writes are echoed but do not persist (deliberate — better than fake
  persistence).
- Display code (`_LoadingView`, `_ErrorView`, `_SkeletonBox`, `_EmptyCard`,
  `_formatDayLabel`) duplicated 3–4×. Deliberate: measured first, it is
  cosmetic only, and refactoring verified files during parallel tooling was
  not worth the regression risk.
- Terminology layer (vertical vocabulary from config) deferred until a second
  vertical is real — avoid hardcoding words that would have to change.

### Next up

1. Login screen (required for the product, not throwaway) + apply the migration
   to staging → real-data deploy.
2. Then the schedule/calendar as the first big v2-native screen.

---

## Project Location

`C:\development\projects\apex_v2`

## Tech Stack

- Flutter + Supabase (same backend as Apex v1)
- `supabase_flutter: ^2.5.0`, `intl: ^0.19.0`
- Dark Material 3 theme (seed color `0xFFFF8C42`)
- **Phone app first** (iOS + Android). A web target exists only so the demo can be clicked in a browser; it is not the delivery platform.
- Bundle id `com.wisense.apex_v2`

## What's Built

### Employee Dashboard (`lib/features/dashboard/employee_dashboard.dart`)
- 786 lines, dart analyze clean, MCA/MDT PASS (Ollama gpt-oss:20b)
- Merged file: Hermes base (realtime, parallel queries, dark M3, standalone) + Kimi features (typed models, clock-out, guard, SnackBar, status enum)
- Features: greeting + smart status line, today's shift card with Clock In/Out, week summary (hours/pay/tips), next shift, swap/request-off buttons, shift notes, team chat preview
- Realtime Supabase streams on shifts, time_entries, messages, shift_notes
- Typed data models: _ProfileData, _ShiftData, _WeekSummary, _ShiftNoteData, _ChatPreview
- _WorkStatus enum (onTheClock / upcoming / shiftStarted / offToday)
- Clock-out + _clocking guard (prevents double-tap) + duplicate clock-in prevention
- SnackBar feedback on clock in/out
- Callback-based navigation with Navigator.pushNamed fallbacks
- Run: `flutter run --dart-define=SUPABASE_URL=<url> --dart-define=SUPABASE_ANON_KEY=<key> --dart-define=ORG_ID=<uuid>`

### Planning Docs (`docs/`)
- `Apex Reimagined Build Order 2026-07-27.md` — build priority, what's already built, what to pull from the existing ordering platform
- `Apex Scheduler Reimagined 2026-07-27.md` — full product vision (10 sections: onboarding, employee experience, notifications, scheduling, time clock, owner dashboard, chat, tips, offline, integrations)
- `Restaurant OS Unified Build Plan 2026-07-27.md` — 16-table phased SQL schema, RLS helpers, role hierarchy, 6-week build order
- `WiSense Restaurant OS Master Plan 2026-07-27.md` — 5-year vision, 7 OS phases, revenue projections, moat analysis

## Build Order (from planning docs)

| Step | Feature | Status | Weekend |
|------|---------|--------|---------|
| 0 | Ship Apex v1 to stores | Blocked (keystore + accounts) | Friday |
| 1a | Manager log book | **Built** (2026-07-27, analyze clean) | Weekend 1 |
| 1b | Tip management | **Built** (2026-07-27, analyze clean) | Weekend 1 |
| 1c | Labor cost dashboard | **Built** (2026-07-27, analyze clean) | Weekend 1 |
| 2 | Unified Supabase backend (ordering) | Not started | Weekend 3 |
| 3a | Labor vs revenue dashboard | Not started | Weekend 4 |
| 3b | No-show call-out engine | Not started | Weekend 5 |
| 3c | Smart ordering capacity | Not started | Weekend 6 |

## Reuse Strategy

- **Apex v1** at `C:\development\projects\apex\lib\` — 16 working features. Pull query patterns, RLS shapes, repository structure.
- **employee_dashboard.dart** is the architectural pattern for ALL new screens: typed models, parallel Future.wait, realtime streams, SnackBar, withValues, callback nav.
- **Planning docs** contain SQL schema for all new tables — don't design schema, use what's documented.
- Pure helpers (_hoursBetween, _formatTime, _firstName, _relativeTime) are copied between standalone files. **`lib/core/shift_time.dart` now holds shared versions** — currently unused; wire the labor cost dashboard to it instead of making a fourth private copy.
- **The money math is deliberately NOT duplicated:** `_hoursBetweenTimestamps` lives only in `tip_management.dart`. Clock punches are full timestamps and must never be parsed as `HH:MM` clock-face strings — that silently discards the date and mis-prices a punch left open across days.

## ⚠️ Concurrent-tooling hazard (2026-07-27)

`apex_v2` had **no git repo** while Claude and Cursor wrote to it simultaneously.
A Claude-written `manager_log_book.dart` was **overwritten mid-session** by the
parallel Cursor build (both features landed at 19:10). Nothing was recoverable —
there was no history.

**Fixed:** `git init` + `.gitignore` + baseline commit `3dc582a`, and a
`README.md` (`a858a7c`) with working agreements for multi-tool sessions:
commit before building, read before overwriting, `flutter analyze` clean before
"done". **Start every future session with a clean `git status`.**

## AI Agent Prompts

Claude and Cursor prompts for building log book + tip management are in `docs/` (or ask Hermes to regenerate). Key instruction: READ employee_dashboard.dart FIRST, match its pattern exactly, REUSE query shapes from Apex v1.

## Schema (from Unified Build Plan)

### Already referenced by employee dashboard
- `profiles` (name, hourly_rate, organization_id)
- `shifts` (shift_date, staff, start_time, end_time, role, zone, organization_id)
- `time_entries` (organization_id, user_id, shift_id, clock_in, clock_out)
- `tip_allocations` (amount_cents, tip_pool_id, user_id) + `tip_pools` (shift_date, organization_id)
- `shift_notes` (note, shift_date, created_at, profiles(name), organization_id)
- `messages` (text, created_at, profiles(name), organization_id)

### Needed next
- `tip_pools` + `tip_allocations` tables (write side — employee dashboard already reads)
- `shift_notes` write side (employee dashboard already reads)
- Full SQL in [[business/Restaurant OS Unified Build Plan 2026-07-27]]

## Pricing

| Tier | Price | Features |
|------|-------|----------|
| Free | $0 | 1 location, 20 employees, basic scheduling + swaps + time clock |
| Pro | $25/mo | Unlimited employees + log book + tips + labor cost + offline |
| OS | $99/mo | Everything in Pro + ordering + smart capacity + no-show engine |
| Multi-Location | $199/mo | Up to 3 locations, full OS |

## Verification History

- 2026-07-27: dart analyze clean, 19-case logic script ALL PASSED (ad-hoc, not suite green)
- 2026-07-27: MCA PASS, MDT PASS (Ollama gpt-oss:20b external audit)
- Untested: widget rendering (no emulator), live Supabase (no backend wired to apex_v2)