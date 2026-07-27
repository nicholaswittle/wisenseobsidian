---
title: Apex v2 — Restaurant OS Build
tags: [apex, restaurant-os, flutter, supabase, build, active]
aliases: [Apex v2, Restaurant OS Build, apex_v2]
date: 2026-07-27
status: active
---

# Apex v2 — Restaurant OS Build

> Reimagined Apex building toward full Restaurant OS: scheduling + ordering + labor cost + tips + chat. Employee-first, 3-tap max, dark Material 3. Built on existing Apex Supabase with org_id scoping.

Related: [[Apex Scheduler]], [[business/Restaurant OS Unified Build Plan 2026-07-27]], [[business/Apex Scheduler Reimagined 2026-07-27]], [[business/Apex Reimagined Build Order 2026-07-27]], [[business/WiSense Restaurant OS Master Plan 2026-07-27]], [[NOW]]

---

## Project Location

`C:\development\projects\apex_v2`

## Tech Stack

- Flutter + Supabase (same backend as Apex v1)
- `supabase_flutter: ^2.5.0`, `intl: ^0.19.0`
- Dark Material 3 theme (seed color `0xFFFF8C42`)
- No Flutter scaffold — lib files + pubspec only, drop into existing app

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
- `Apex Reimagined Build Order 2026-07-27.md` — build priority, what's already built, what to pull from Jigsy's
- `Apex Scheduler Reimagined 2026-07-27.md` — full product vision (10 sections: onboarding, employee experience, notifications, scheduling, time clock, owner dashboard, chat, tips, offline, integrations)
- `Restaurant OS Unified Build Plan 2026-07-27.md` — 16-table phased SQL schema, RLS helpers, role hierarchy, 6-week build order
- `WiSense Restaurant OS Master Plan 2026-07-27.md` — 5-year vision, 7 OS phases, revenue projections, moat analysis

## Build Order (from planning docs)

| Step | Feature | Status | Weekend |
|------|---------|--------|---------|
| 0 | Ship Apex v1 to stores | Blocked (keystore + accounts) | Friday |
| 1a | Manager log book | Not started | Weekend 1 |
| 1b | Tip management | Not started | Weekend 1 |
| 1c | Labor cost dashboard | Not started | Weekend 1 |
| 2 | Unified Supabase backend (ordering) | Not started | Weekend 3 |
| 3a | Labor vs revenue dashboard | Not started | Weekend 4 |
| 3b | No-show call-out engine | Not started | Weekend 5 |
| 3c | Smart ordering capacity | Not started | Weekend 6 |

## Reuse Strategy

- **Apex v1** at `C:\development\projects\apex\lib\` — 16 working features. Pull query patterns, RLS shapes, repository structure.
- **employee_dashboard.dart** is the architectural pattern for ALL new screens: typed models, parallel Future.wait, realtime streams, SnackBar, withValues, callback nav.
- **Planning docs** contain SQL schema for all new tables — don't design schema, use what's documented.
- Pure helpers (_hoursBetween, _formatTime, _firstName, _relativeTime) are copied between standalone files, not shared.

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