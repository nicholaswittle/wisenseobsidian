---
title: Apex Scheduler — Code Reference
tags: [app, reference, apex, launch-priority]
aliases: [jigsy, apex-code, jigsy-schedule]
---

# Apex Scheduler — Code Reference

**Repo:** `C:\development\projects\apex\apex` — own git repo (`main` / `origin/main`)
**Bundle ID:** `com.wisense.apex`
**Stack:** Flutter · Supabase · Firebase Cloud Messaging · Sentry
**Platforms:** iOS · Android · Web
**Phase:** Stabilize Jigsy's pilot before new features. Billing deferred.
**Security audit:** [[Apex Security Audit 2026-07-19]] (`apex/audit/AUDIT_2026-07-19.md`)

## File inventory

### lib/ (39 Dart files)

#### Top-level screens (5)
- `auth_page.dart` — Supabase auth (login/signup)
- `billing_page.dart` — Stripe billing (deferred, `billingEnabled = false`)
- `calendar_page.dart` — main calendar tab host
- `setup_page.dart` — org setup / invite codes
- `main.dart` — app entry + Firebase bootstrap

#### core/ (11)
- `analytics_service.dart` — usage analytics
- `app_config.dart` — build config (Supabase URL, anon key, billing flag)
- `date_utils.dart` — date helpers
- `error_monitoring.dart` — Sentry integration
- `firebase_bootstrap.dart` — Firebase init + push token sync
- `notification_service.dart` — in-app notifications
- `profile_service.dart` — user profile (Supabase)
- `push_notification_service.dart` — FCM push
- `schedule_constants.dart` — shift/schedule constants
- `shift_hours_util.dart` — shift hour calculations
- `zone_colors.dart` — color zones for calendar

#### features/ (9 feature dirs)
- `availability/availability_service.dart` — staff availability
- `calendar/calendar_page_controller.dart` — calendar state controller (~690+ lines)
- `schedule/conflict_detector.dart` — shift conflict detection
- `schedule/schedule_repository.dart` — Supabase schedule CRUD
- `sidework/sidework_service.dart` — sidework checklists
- `smart_suggestions/suggestion_engine.dart` — smart shift suggestions
- `staff/staff_repository.dart` — staff CRUD
- `swaps/swap_service.dart` — shift swap board + approval
- `time_clock/time_clock_service.dart` — time clock + CSV export
- `time_off/time_off_service.dart` — time-off requests + approval

#### widgets/ (16)
- `admin_publish_panel.dart` — owner publish schedule
- `calendar_tab.dart` — calendar tab view
- `config_missing_screen.dart` — missing Supabase config fallback
- `csv_time_card_exporter.dart` — CSV export (web + mobile)
- `event_shift_card.dart` — shift card in calendar
- `labor_cost_panel.dart` — labor cost display
- `notification_bell.dart` — notification badge
- `org_invite_panel.dart` — org invite codes
- `shift_calendar_grid.dart` — **swipeable calendar grid** (horizontal swipe navigates weeks/months, velocity threshold 200)
- `shift_card.dart` — shift detail card
- `sidework_section.dart` — sidework checklist UI
- `smart_suggestions_panel.dart` — suggestion UI
- `staff_availability_card.dart` — staff availability display
- `swaps_tab.dart` — swap board tab
- `time_off_tab.dart` — time-off requests tab
- `tutorial_overlay.dart` — onboarding overlay

### supabase/
- `functions/` — edge functions
- `migrations/` — DB migrations

### scripts/
- `run_dev.sh` — local dev
- `build_web.sh` — web build for Vercel
- `build_release.sh` — iOS/Android release builds
- `gate0_verify.sh` — Gate 0 verification

### docs/
- `GATE0.md` — active checklist (merge, 7-day clean run, exit criteria)
- `JIGSYS_BASELINE.md` — baseline metrics (fill by Day 3)
- `ROADMAP.md` — full plan (Pillars 0–E, timeline, kill list)
- `VERCEL.md` — web deploy guide
- `LAUNCH_CHECKLIST.md` — TestFlight + Play Store steps
- `LAUNCH_WITHOUT_MAC.md` — Android-first launch path

## Dependencies (pubspec.yaml)

| Package | Purpose |
|---------|---------|
| `supabase_flutter` | Auth + database |
| `firebase_core` / `firebase_messaging` | Push notifications |
| `sentry_flutter` | Error monitoring |
| `shared_preferences` | Local prefs |
| `share_plus` | CSV share sheet |
| `wisense_ui` (vendored) | Shared UI package |

## Architecture

- **State management:** Mixed — controllers per feature (CalendarPageController), services per domain
- **Auth:** Supabase (auth_page.dart)
- **Database:** Supabase (schedule_repository, staff_repository, etc.)
- **Push:** Firebase Cloud Messaging → push_notification_service → notification_bell
- **Calendar swipe:** shift_calendar_grid.dart uses `GestureDetector` + `onHorizontalDragEnd` with velocity threshold (200) to navigate weeks/months. This is the swipe pattern that could be reused.
- **CSV export:** Platform-aware (csv_downloader_stub.dart + csv_downloader_web.dart)
- **Billing:** Stripe deferred (`AppConfig.billingEnabled = false`)

## Git state (updated 2026-07-20)

- **Repo:** Own `.git` + GitHub remote (`nicholaswittle/apex.git`). Not nested under other app repos.
- **Branch:** `main` tracking `origin/main` (in sync after launch-blocker pushes)
- **Key commits:** merge `13971b0`, races+timezone `9f723a5`, RLS migration `a66b039`
- Live status: [[hot]] / [[NOW]] / [[Apex Scheduler]]

## Launch blockers

- [x] Merge + security code in-repo (claim/clock races, timezone, RLS SQL authored)
- [ ] **Human:** Apply RLS migration on Supabase staging→prod
- [ ] Fill `JIGSYS_BASELINE.md` + log pilot truth in [[customers/Jigsys Brewpub]]
- [ ] Gate C Android packaging — [[Gate C — Android Packaging & Store Listings 2026-07-20]]
- [ ] 7-day clean run per `GATE0.md` once pilot live
- [ ] iOS / TestFlight later (Codemagic)

Related: [[projects/dormant/Apex Scheduler]], [[NOW]], [[Code Reuse Analysis]], [[Parent Repo Cleanup]]