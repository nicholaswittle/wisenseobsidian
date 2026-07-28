---
title: Apex v2 — Restaurant OS Build
tags: [apex, restaurant-os, flutter, supabase, build, active]
aliases: [Apex v2, Restaurant OS Build, apex_v2]
date: 2026-07-27
updated: 2026-07-28
status: active
---

# Apex v2 — Restaurant OS Build

> Reimagined Apex building toward full Restaurant OS: scheduling + ordering + labor cost + tips + chat + call-outs + capacity. Employee-first, 3-tap max, dark Material 3. Built on existing Apex Supabase with org_id scoping.

**HEAD:** `c9e1eae` on `main` (synced to GitHub). All 3 OS bridge features + ordering + color scheme swap pushed.  
**Live:** Real https://apex-v2-ten.vercel.app · Demo https://apex-v2-demo.vercel.app (auto-deploys from main).  
**Theme:** New Horizon dark palette (purple/teal on blue-black `0xFF0A0C10`) — replaced original brown-on-black.  
**Audits:** [[wisense/projects/APEX_V2_AUDIT_2026-07-27]] (code) · [[wisense/projects/RESTOS_FULL_SYSTEM_AUDIT_2026-07-27]] (full system + PDF) — both predate ordering/call-outs/capacity commits.

Related: [[Apex Scheduler]], [[business/Restaurant OS Unified Build Plan 2026-07-27]], [[business/Apex Scheduler Reimagined 2026-07-27]], [[business/Apex Reimagined Build Order 2026-07-27]], [[business/WiSense Restaurant OS Master Plan 2026-07-27]], [[NOW]]

---

## Project Location

`C:\development\projects\apex_v2`

## Tech Stack

- Flutter + Supabase (same backend as Apex v1)
- `supabase_flutter: ^2.5.0`, `intl: ^0.19.0`, `shared_preferences`, `image_picker`, `crypto`
- Dark Material 3 theme (New Horizon palette: purple `0xFF8B5CF6` + teal `0xFF14B8A6` on blue-black `0xFF0A0C10`)
- Full Flutter scaffold (Android, iOS, Web) + Supabase migrations + edge functions
- Demo mode: `DemoHttpClient` seeds fake data so screens run without a backend

## What's Built — Plan vs Actual

### Plan highlights (from 4 planning docs)

The Reimagined vision defined 10 sections. The Unified Build Plan defined a 6-week build order. Here's where each landed:

| # | Plan Feature | Priority | Status | File(s) |
|---|-------------|----------|--------|---------|
| 1 | Employee dashboard redesign | Weekend 1 | DONE | `lib/features/dashboard/employee_dashboard.dart` (1288 lines) |
| 2 | Manager log book | Weekend 1 | DONE | `lib/features/log_book/manager_log_book.dart` (704 lines) |
| 3 | Tip management | Weekend 1 | DONE | `lib/features/tips/tip_management.dart` (1163 lines) |
| 4 | Labor cost dashboard (one number) | Weekend 1 | DONE | `lib/features/labor_cost/labor_cost_dashboard.dart` (1162 lines) |
| 5 | Team chat | Weekend 2 | DONE | `lib/features/chat/team_chat_screen.dart` (393 lines) |
| 6 | Smart notification routing (push + SMS) | Weekend 2 | DONE | `lib/features/notifications/notification_prefs_screen.dart` (260 lines) + `supabase/functions/route-notification/index.ts` + migration |
| 7 | Photo-to-schedule import | Weekend 2 | DONE | `lib/features/schedule/photo_import_screen.dart` (400 lines) + `supabase/functions/parse-schedule/index.ts` |
| 8 | QR clock-in + geofencing | Weekend 2 | DONE | `lib/features/time_clock/qr_scan_screen.dart` + `qr_wall_screen.dart` + `lib/core/clock_qr.dart` (daily rotating QR with crypto digest) |
| 9 | Schedule guardrails (labor laws) | Weekend 3 | DONE | `lib/core/labor_guardrails.dart` (185 lines, PA rules: minor hours, break notice, consecutive days, overtime) + migration for DOB |
| 10 | Offline mode | Weekend 2 | DONE | `lib/core/offline_sync.dart` (128 lines, SharedPreferences punch queue) |
| 11 | Entitlements / pricing tiers | — | DONE | `lib/core/entitlements.dart` (192 lines, OsModule enum + OsTier + per-org overrides) |
| 12 | Schedule screen | — | DONE | `lib/features/schedule/schedule_screen.dart` (895 lines) |
| 13 | Swaps screen | — | DONE | `lib/features/swaps/swaps_screen.dart` |
| 14 | Time-off screen | — | DONE | `lib/features/time_off/time_off_screen.dart` |
| 15 | Sidework screen | — | DONE | `lib/features/sidework/sidework_screen.dart` |
| 16 | Team screen | — | DONE | `lib/features/team/team_screen.dart` |
| 17 | Auth gate + sign-in | — | DONE | `lib/features/auth/auth_gate.dart` + `sign_in_screen.dart` |
| 18 | Notification bell | — | DONE | `lib/features/notifications/notification_bell.dart` |
| 19 | App shell with module routing | — | DONE | `lib/app.dart` (439 lines, plug-in route system gated by entitlements) |
| 20 | Demo backend | — | DONE | `lib/core/demo_backend.dart` (772 lines, fake HTTP client with seeded data) |
| 21 | Calendar export | — | DONE | `lib/core/calendar_export.dart` |
| 22 | Schedule text parser | — | DONE | `lib/core/schedule_text_parser.dart` (local OCR fallback) |
| 23 | Shift time helpers | — | DONE | `lib/core/shift_time.dart` |
| 24 | Conflict detector | — | DONE | `lib/core/conflict_detector.dart` |
| 25 | Online ordering platform | Phase 2 | DONE | `lib/features/ordering/` (menu · cart · staff console) + migration `20260729000000_ordering_platform.sql` — seeded Jigsy's Brewpub, public token `jigsys`, 19 items |
| 26 | Smart ordering capacity | Phase 3 | DONE | `lib/features/capacity/capacity_screen.dart` + migration `20260731000000_smart_capacity.sql` |
| 27 | No-show call-out engine | Phase 3 | DONE | `lib/features/callout/` + `supabase/functions/route-callout` + migration `20260730000000_no_show_callout.sql` (in-app primary; SMS optional) |
| 28 | Labor vs revenue dashboard | Phase 3 | DONE | `lib/features/labor_vs_revenue/labor_vs_revenue_dashboard.dart` |

### NOT yet built (from the plan)

| # | Plan Feature | Status | Notes |
|---|-------------|--------|-------|
| 29 | Google Calendar sync | PARTIAL | ICS + Google Calendar TEMPLATE export shipped; two-way OAuth not started |
| 30 | Payroll export (CSV for QuickBooks/Gusto/ADP) | NOT STARTED | Labor CSV exists; full payroll adapters deferred |
| 31 | Geofencing for clock-in | NOT STARTED | QR is built, geofence not yet (plan mentions `geolocator` plugin) |
| 32 | Photo verification (selfie on clock-in) | NOT STARTED | Plan #5 in Reimagined vision |
| 33 | Square integration | NOT STARTED | Phase 4 — deferred until Jigsy's commits |
| 34 | Prep-time snapshots (ML data capture) | NOT STARTED | Phase 5 — capture only, no model yet |
| 35 | Floor plan / table management | NOT STARTED | Phase 6 — future |

### Verdict

Weekend 1–3 scope **plus** Phase 2 ordering and Phase 3 OS bridges (labor vs revenue, call-outs, smart capacity) are built and pushed to GitHub (`c9e1eae`). Color scheme swapped from brown to New Horizon dark palette. Remaining items (29–35) are polish/integrations and later phases. Product audit by Nicholas still pending — plan to show Emily (Jigsy's) the v2 demo for feedback. "Apex v2 lite" for v1 launch = set org tier to `free` in entitlements, same app shows only 4 modules.

## Architecture

### App shell (`lib/app.dart`)
- Module-based route system: each feature registers with an `OsModule` tag
- Entitlements gate which modules appear based on tier (Free/Pro/OS/Multi) + per-org overrides
- Adding a new OS piece = one entry in `_moduleRoutes`

### Entitlements (`lib/core/entitlements.dart`)
- `OsModule` enum: scheduling, shiftSwaps, timeClock, pushNotifications (Free), managerLogBook, tipManagement, laborCost, offlineMode, teamChat (Pro), onlineOrdering, smartCapacity, noShowEngine, laborVsRevenue (OS), multiLocation (Multi)
- `OsTier` enum with inheritance: each tier grants everything below it
- Per-org `enabled_modules` / `disabled_modules` arrays on `organizations` table

### Demo mode (`lib/core/demo_backend.dart`)
- `DemoHttpClient` intercepts Supabase REST calls, returns seeded data
- All screens run without a real backend — enables testing and demos

### Supabase migrations
- `20260727000000_apex_v2_foundation.sql` — new tables (shift_notes, tip_pools, tip_allocations, messages), new columns (shifts.start_time/end_time/role, time_entries.organization_id), entitlements columns on organizations, RLS helpers
- `20260728000000_notification_routing.sql` — notification_preferences table + RLS
- `20260728010000_labor_guardrails_dob.sql` — date_of_birth column on profiles
- `20260729000000_ordering_platform.sql` — restaurants / menu / cart / orders (applied live on Apex DB `pqkremkwfkudrhtxasdj`)
- `20260730000000_no_show_callout.sql` — call-out requests + responses (applied)
- `20260731000000_smart_capacity.sql` — capacity settings / windows (applied with capacity feature)

### Edge functions
- `parse-schedule/index.ts` — cloud vision for photo-to-schedule (optional, uses Anthropic API key)
- `route-notification/index.ts` — SMS fallback via Twilio
- `route-callout/index.ts` — ranks available staff and routes call-out (SMS optional / Twilio)

## Reuse Inventory — What Can Be Pulled Into Future Builds

### From Apex v1 (`C:\development\projects\apex\lib\`)
Already reused in v2: query patterns, org_id scoping, shift/time_entry patterns. Still available:

| Asset | Location | What It Does |
|-------|----------|-------------|
| ScheduleRepository | `features/schedule/schedule_repository.dart` | Shifts CRUD, publish, copy-week, realtime streams — v2 has its own but v1's is battle-tested |
| TimeClockService | `features/time_clock/time_clock_service.dart` | Clock in/out with duplicate guard — v2 reimplemented this inline |
| SwapService | `features/swaps/swap_service.dart` | Post/claim/approve swaps with org scoping |
| AvailabilityService | `features/availability/availability_service.dart` | Availability + time-off + booked status in parallel Future.wait |
| SuggestionEngine | `features/smart_suggestions/suggestion_engine.dart` | Rules-based staffing from 4 weeks of history |
| StaffRanker | `features/smart_suggestions/staff_ranker.dart` | Scoring: title match, weekday match, booked penalty, overtime penalty |
| NotificationService | `core/notification_service.dart` | In-app + FCM push via `apex_notify_user` RPC + edge function |
| ShiftHoursUtil | `core/shift_hours_util.dart` | AM/PM time parsing + shift duration — v2 has its own version |
| DateUtils | `core/date_utils.dart` | `dateKey()` / `parseDateKey()` — tiny but universal |
| ZoneColors | `core/zone_colors.dart` | Color per zone (Bar/Kitchen/Floor) |
| LaborCostPanel | `widgets/labor_cost_panel.dart` | Basic labor cost display — v2 built a full dashboard instead |
| ConflictDetector | `features/schedule/conflict_detector.dart` | Double-booking detection |
| SideworkService | `features/sidework/sidework_service.dart` | Sidework assignment CRUD |
| CSV exporter | `widgets/csv_time_card_exporter.dart` | Time card CSV export (web + stub) |
| RLS migration | `supabase/migrations/20260720000000_launch_blockers_rls.sql` | `apex_current_org()` helper, per-table RLS policies, atomic clock-in guard |

### From Jigsy's ordering (`C:\development\projects\jigsy\src\App.jsx`)
**Ported 2026-07-27** into Flutter + Supabase under `lib/features/ordering/`. Original React demo still lives at Cloudflare for the isolated website pitch. Archive copy also in [[restOS]] (`github.com/nicholaswittle/restOS`).

### From wisense_ui (`C:\development\projects\apex\packages\wisense_ui\`)
- `loading_indicator.dart`, `spacing.dart` — shared UI primitives (not yet imported by v2)

## Verification Status

- **dart analyze**: No issues found (full project)
- **flutter test**: 28 tests passed across 4 test files (clock_qr, demo_backend, guardrails_calendar, schedule_text_parser)
- **MCA/MDT**: PASS on employee dashboard (Ollama gpt-oss:20b, 2026-07-27)
- **Untested**: widget rendering (no emulator), live Supabase (demo mode covers functional paths)

## Build Order — What's Next

| Step | Feature | Status | Blocked On |
|------|---------|--------|------------|
| 0 | Ship Apex v1 to stores | Blocked | Keystore + Apple/Google accounts (Friday) |
| 1 | Standalone features (log book, tips, labor cost, chat, QR, offline, guardrails, notifications) | DONE | — |
| 2 | Unified Supabase backend (port ordering) | DONE (`d3a218e`) | Pushed to GitHub |
| 3 | OS bridge (labor vs revenue, no-show engine, smart capacity) | DONE (`1b74a4e`) | Pushed to GitHub |
| 4 | Color scheme swap (New Horizon palette) | DONE (`d3a218e`) | Pushed |
| 5 | Apex v1 schedule builder port (month grid) | DONE (Cursor) | Uncommitted in v1 — needs commit |
| 6 | Show Emily the v2 demo | Pending | Nicholas's schedule |
| 7 | Production integration (Square, etc.) | Deferred | Jigsy's written approval |
| 8 | ML data capture (prep-time snapshots) | Deferred | Step 2 |

## Pricing (Implemented in entitlements.dart)

| Tier | Price | Modules |
|------|-------|---------|
| Free | $0 | scheduling, shiftSwaps, timeClock, pushNotifications |
| Pro | $25/mo | + managerLogBook, tipManagement, laborCost, offlineMode, teamChat |
| OS | $99/mo | + onlineOrdering, smartCapacity, noShowEngine, laborVsRevenue |
| Multi | $199/mo | + multiLocation |