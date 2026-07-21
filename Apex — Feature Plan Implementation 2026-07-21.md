---
type: session
title: "Apex — Feature Plan Implementation 2026-07-21"
created: 2026-07-21
updated: 2026-07-21
tags: [apex, implementation, ios, ui, availability, sentry, bundle-id]
status: developing
related:
  - "[[Apex — Feature Plan 2026-07-21]]"
  - "[[Apex Scheduler]]"
  - "[[Apex Scheduler — Code Reference]]"
  - "[[DECISIONS]]"
  - "[[NOW]]"
  - "[[apex update Xcode]]"
---

# Apex — Feature Plan Implementation (2026-07-21)

Execution record for [[Apex — Feature Plan 2026-07-21]]. Section 0 (canonical iOS build) plus Features B and C are implemented, committed, and green. Feature A (photo-to-schedule import) is untouched.

Repo: `C:\development\projects\apex\apex`. Work sits on branch **`feat/apex-plan-2026-07-21`**, branched from `main` at `a66b039`. **Pushed to `origin` 2026-07-21; not merged to `main`.**

| Commit | Scope |
|--------|-------|
| `0fabf68` | `build(ios)`: Section 0 — canonical Mac build |
| `b469b6c` | `feat(schedule)`: Features B + C |
| `594b4be` | `refactor(schedule)`: Directive §2 conformance remediation |

Verification: `flutter analyze` clean, 7/7 tests pass. **Nothing has been run on a device yet** — Features B and C are unverified on the iPhone.

> [!warning] Unaudited — do not merge to `main`
> This branch was built and pushed **outside the Tripartite Protocol**: no Judicial audit, no Completion Report, no Delivery Gate. See [[DECISIONS]] 2026-07-21. The Completion Report is issued **BLOCKED / MCA+MDT NOT RUN**. Self-review missed real Directive §2 defects, fixed in `594b4be` — treat the remaining diff as unverified for the same reason.

---

## Section 0 — canonical iOS build

The Mac clone previously only built because of local hacks that were never committed, so every pull needed them re-applied by hand. These are now real commits.

### Bundle ID → `com.nicholaswittle.apex` (iOS only)

Changed in all 6 `PRODUCT_BUNDLE_IDENTIFIER` spots in `ios/Runner.xcodeproj/project.pbxproj` and in the Supabase auth redirect URL scheme in `ios/Runner/Info.plist`.

**Android `applicationId` deliberately stays `com.wisense.apex`** — it is a separate namespace, already registered with Firebase (`google-services.json`) and referenced by the Kotlin package path. Changing it would have broken push for no gain. The platforms now legitimately differ; `docs/LAUNCH_CHECKLIST.md` records the split.

### Sentry dropped

Removed `sentry_flutter` from `pubspec.yaml`, `sentryDsn` from `lib/core/app_config.dart`, and the Sentry path from `lib/core/error_monitoring.dart` (now a plain `FlutterError.onError` handler). This also removed Sentry from the Linux/macOS/Windows generated plugin registrants.

Rationale: it had no `SENTRY_DSN`, so it reported nothing, and 8.x does not build under current Xcode. Dropping was strictly cheaper than upgrading to 9.x for a feature that was inert.

### Deployment target

`IPHONEOS_DEPLOYMENT_TARGET` 13.0 → 15.0 in 3 spots. Required by firebase-core and firebase-messaging.

---

## Feature B — scannable shift cards

`lib/widgets/shift_card.dart`: assigned name 12 → **17 bold `UniversalTheme.darkSlate`**, person icon 14 → 18, shift time (`notes`) 12 italic → **15 upright semibold**, title 15 → 16 so the name stays the visual hero. `lib/widgets/event_shift_card.dart` got the same treatment so private events stay consistent.

One fix beyond the plan: the name was an unconstrained `Text` inside a `Row`, which would have started overflowing at 17px with longer names. It is now wrapped in `Expanded`.

The day tile in `shift_calendar_grid.dart` was left alone — it is the date picker, not assignments.

---

## Feature C — availability reflects existing shifts

`lib/features/availability/availability_service.dart` `loadForDate()` now runs a **third parallel query** on `shifts` for the date, collects non-`'Open'` staff into a `bookedSet`, and exposes a per-staffer `booked` flag plus `isBooked` for the current user. The flag threads through `calendar_page_controller` → `calendar_page` → `calendar_tab` → `StaffAvailabilityCard`.

`StaffAvailabilityCard` renders a distinct blue "booked" state. The 3-deep nested ternaries were replaced with a `_StaffStatus` enum + style switch once a fourth state made them unreadable. **Precedence: vacation > booked > the self-declared availability flag** — an approved leave is authoritative, and an existing assignment is a fact.

### Two deliberate deviations from the plan

1. **The admin dropdown needs its own query.** The plan said to feed it from `calendar_tab`'s shifts stream. That is the wrong day: `AdminPublishPanel` publishes to `adminSelectedDays` within a *target week*, not the calendar's selected date. Added `AvailabilityService.loadBookedStaff({dateKeys})`, refreshed on day toggle, week change, and publish success.
2. **`booked` does not force `available` to false.** It surfaces as a warning icon in the dropdown plus an explanatory line beneath it, and booked staff stay selectable. Split shifts are legitimate — this matches the plan's own edge case, so the warning is soft by design.

Realtime recompute is wired into the existing `listenNewShifts` stream, so adding, removing, or reassigning a shift refreshes both the day's availability and the admin's target-day warning. The booked lookup swallows its errors deliberately: it is advisory, and a banner on every day toggle would be noise.

---

## Design-system conformance (`594b4be`)

Directive §2 requires `WiSenseSpacing` for spacing and app brand colors in the app's own theme file. The original Feature B/C diff violated both; self-review did not catch it.

- The booked-staff warning used raw `Color(0xFFD97706)` — **byte-identical to the existing `UniversalTheme.accent`** — and `0xFF92400E`, now `UniversalTheme.warningText`.
- The four availability state colors moved to `AvailabilityPalette` in `lib/theme.dart`; `vacationAccent`/`vacationText` reference the `UniversalTheme` constants instead of a third copy of the same values.
- Spacing I introduced now uses tokens. **Three gaps shift by 2px** (6→4, 6→8, 2→4) because the vendored scale has no 2px or 6px step. Colors are unchanged byte-for-byte.
- Pre-existing raw values elsewhere in those files were left alone — scope containment is itself an MCA check.

**Font sizes cannot conform.** `WiSenseTextStyles` does not exist in Apex's vendored `wisense_ui` (see [[DECISIONS]] 2026-07-21). Feature B's `fontSize: 17/16/15` stays hardcoded, consistent with every other Apex widget.

## Known gaps

- **Unaudited.** No Judicial (external) audit has run. This is the blocking gap.
- **Not merged.** Branch is pushed but `main` is untouched.
- **Not device-verified.** B and C have not been seen on the iPhone.
- **Feature C has no test coverage.** `AvailabilityService` takes a live `SupabaseClient` and the existing suite is pure-utility tests with no mocking infrastructure. Adding a test for the booked-set derivation means introducing that infrastructure first.
- **Feature A untouched** — new dependency, new screen, and the parsing problem. Its open decision (on-device ML Kit vs LLM vision, i.e. privacy vs robustness for staff names) is still unmade.

## First Mac build after pulling this branch

1. `flutter clean && flutter pub get`, then `cd ios && pod install`. The pod set changed (Sentry gone) and the 15.0 target affects every pod; `ios/Podfile` and `Podfile.lock` are not in the repo.
2. Re-select the signing team in Xcode → Runner → Signing & Capabilities. The bundle ID changed, so the provisioning profile will not match until reselected.
3. **If Supabase auth allows only `com.wisense.apex://` as a redirect, add `com.nicholaswittle.apex://` in the Supabase dashboard** or iOS login will not return to the app. Android is unaffected — its manifest has no deep-link scheme.

Related: [[Apex — Feature Plan 2026-07-21]], [[Apex Scheduler]], [[Supabase]], [[NOW]], [[DECISIONS]]
