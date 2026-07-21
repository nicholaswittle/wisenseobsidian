---
title: Apex — Feature Plan 2026-07-21
tags: [apex, plan, feature, ocr, ui, availability, ios, mac]
date: 2026-07-21
---

# Apex — Feature Plan (2026-07-21)

Three features for [[Apex Scheduler]], motivated by manually entering a month of Jigsys shifts — it was painful enough that doing it weekly is a non-starter. Plan is written to be executed on the **desktop** (the real work center). The **Mac** is only for building/signing the iOS app to QA on the iPhone (Windows can't build iOS — see [[Apex iOS build setup on Mac]] / [[xcode_notes]]).

Related: [[Apex Scheduler]], [[Apex Scheduler — Code Reference]], [[Supabase]], [[2026-07-21]] (the iOS build-setup daily log), [[Plan of Attack — Build While Mac-Blocked 2026-07-20]].

---

## 0. Prerequisite: make the iOS build work from canonical code

Before any feature branch can be QA'd on the phone, the Mac must be able to build **canonical** code with zero throwaway mods. Right now the Mac clone only builds because of local hacks that are NOT in the repo. Commit these as real decisions so the desktop→push→pull-on-Mac→phone loop is clean:

1. **Bundle ID** — decide the real product ID. `com.nicholaswittle.apex.local` was a free-Personal-Team workaround (the original `com.wisense.apex` was registered to another team). Pick one you actually own and commit it in `ios/Runner.xcodeproj/project.pbxproj` (6 `PRODUCT_BUNDLE_IDENTIFIER` spots) + `ios/Runner/Info.plist` line 67 (the Supabase auth-redirect URL scheme — keep aligned with whatever you choose, or leave `com.wisense.apex` if Supabase auth is already configured for it).
2. **Sentry** — either upgrade `sentry_flutter` to 9.x (supports Xcode 26 / current `sentry-cocoa`), or formally drop it: remove from `pubspec.yaml` and keep `lib/core/error_monitoring.dart` as the no-Sentry version. The local half-rewrite is not committed. (Sentry was unused anyway — no `SENTRY_DSN`.)
3. **iOS deployment target** — set `IPHONEOS_DEPLOYMENT_TARGET = 15.0` in `project.pbxproj` (3 spots). firebase-core/firebase-messaging require iOS 15.

Once those are real commits, the Mac builds any branch cleanly. Until then, every pull onto the Mac needs the hacks re-applied by hand.

---

## Feature A — Photo-to-schedule import (with user verification)

**Goal:** Photograph the paper schedule → app auto-fills shifts → user reviews/edits → publish. The verify step is the whole feature; OCR is just a component. The user's instinct to require verification is what makes an imperfect parser acceptable.

### Where it plugs in (already exists)
- Data target = the `shifts` table. A row is: `shift_date` (string date key), `day_num` (int), `title` (role), `staff` (name), `notes` (currently holds formatted hours, e.g. `"9:00 AM - 5:00 PM"`), `is_event` (bool), `zone` (nullable), `organization_id`.
- **The exact pattern to copy:** `ScheduleRepository.copyPreviousWeek()` (`lib/features/schedule/schedule_repository.dart:55-101`) already builds a `List<Map<String,dynamic>>` of shift rows and bulk-inserts with one `.insert(inserts)` + a notify. An import-from-photo flow is the same shape — you just produce the candidate rows from OCR instead of from last week's DB rows.
- `onPublish` lives in `lib/calendar_page.dart` + `lib/widgets/admin_publish_panel.dart` — that's the integration surface.

### Pipeline
1. **Capture** — `image_picker` (gallery or camera) or the `camera` plugin. New dep (none present today — pubspec only has supabase_flutter, firebase_core/messaging, shared_preferences, share_plus, wisense_ui).
2. **Recognize text** — see extraction-strategy decision below.
3. **Parse into candidate rows** — turn recognized words+positions into `{shift_date, title, staff, notes(hours), is_event, zone}`. This is the hard part; see "real difficulty."
4. **Match staff names** — fuzzy-match OCR'd names to the app's known `staffNames` (already flows into `AdminPublishPanel`). Unmatched names get flagged for the user to resolve.
5. **Verify screen** — editable list of candidate shifts: tap to fix a time/name/zone, toggle a row off, re-match a name to a real staffer, fix the target week. Then "Publish verified."
6. **Publish** — new repo method `publishVerifiedShifts({required List<Map<String,dynamic>> rows, required String organizationId, required String? excludeUserId})` ≈ the tail of `copyPreviousWeek` (build inserts → `.insert(inserts)` → `NotificationService.notifyOrganization`). Reuse, don't duplicate.

### Real difficulty (not the OCR)
Turning a pile of recognized words-with-bounding-boxes into structured rows, and matching OCR'd names to your staff list. Every paper template is different, so a pure grid parser is template-specific and brittle. The verify screen is the safety net that makes a brittle parser shippable.

### Open decision — extraction strategy
- **On-device ML Kit** (`google_mlkit_text_recognition`): free, offline, no upload, cross-platform. You then parse the grid yourself. More code, full privacy.
- **LLM vision** (send photo, or just the OCR text, to Claude/GPT-4o → structured JSON `{staff, day, start, end, zone}[]`): far more robust to layout variation, far less code. Cost negligible for ~1 photo/week (cents). Tradeoff: staff names + hours leave the device → privacy.

**Recommendation:** start on-device ML Kit; if grid parsing proves too fragile across templates, layer an LLM extraction on the OCR text (cheaper than sending the image; gate behind a setting). Pick based on your privacy stance for staff data.

### Risks / gotchas
- `notes` currently stores hours as a **formatted string**, not structured start/end. The import should format hours the same way the existing publish path builds `formattedHours`, so the card renders them correctly (see Feature B).
- Time parsing is ambiguous ("9-5", "9am-5pm", "9:00-17:00"). Normalize to the app's `timeSlots` vocabulary (`lib/core/schedule_constants.dart`) before storing.
- Target week/date mapping: paper schedules usually show a week range, not ISO dates. The user picks the target week in the verify screen; day headers map to dates off that week's Monday (reuse `copyPreviousWeek`'s Monday math).
- Staff not yet in the system: offer "create staff" inline from the verify screen.

---

## Feature B — Bigger names + bigger shift times on the schedule

**Goal:** "Who's assigned to what shift" and the shift time are too small to scan. Make the assigned name and the time the visual heroes of each card.

### Where (grounded)
`lib/widgets/shift_card.dart` — the per-shift card rendered in `lib/widgets/calendar_tab.dart:226` (the day's shift list). Current sizes:
- **Assigned name:** line 104-113 — `'Assigned to: $scheduled'` (or `'OPEN SLOT (UNASSIGNED)'`) at `fontSize: 12`, `Colors.black54`. Icon `account_circle` size 14 (line 100).
- **Shift time = `notes`:** lines 116-126 — italic brown `fontSize: 12`.
- Title (role): `fontSize: 15` bold (line 68). Zone chip: `fontSize: 9` (line 86).

### Change
- Assigned name: `fontSize: 12` → **~17, bold, `UniversalTheme.darkSlate`** (darker than the current black54 so it reads as the primary info). Icon 14 → ~18.
- Shift time (`notes`): `fontSize: 12` italic → **~15**, keep readable; consider dropping the italic (italic reads as "secondary" — but the time is now primary, so upright + semibold).
- Optionally bump the title 15 → 16 so the hierarchy still works, but the **name should be ≥ the title** since that's what the user scans for.
- Leave the zone chip and action buttons as-is (they're secondary).

### Also check
- `lib/widgets/event_shift_card.dart` (the private-event variant, `calendar_tab.dart:215`) — apply the same sizing so events aren't inconsistent.
- The day tile in `lib/widgets/shift_calendar_grid.dart` (`_buildDayTile`, fontSize 10/14) is the **date picker**, not assignments — leave it.

### Verification
On the phone, open a day with assigned shifts and confirm the name + time are readable at a glance without leaning in. (The 2.8px overflow fix from [[2026-07-21]] is in the *admin* publish panel, separate file — unaffected.)

---

## Feature C — Availability reflects existing shifts ("already booked that day")

**Goal:** If a staffer is already assigned to a shift on a given day, their availability should show them as **booked** (not "Available"), so the admin doesn't double-assign and the staffer sees they're committed. Today availability is purely the `availability` table + vacation — it ignores existing shifts.

### Where (grounded)
`lib/features/availability/availability_service.dart` — `loadForDate()` currently runs two queries in parallel: `availability` (user_name, available) and `time_off_requests` (Approved). It builds `availabilityForDay` as `{user_name, available, on_vacation}`. **It does not look at `shifts`.**

### Change
1. Add a third query to the `Future.wait` in `loadForDate` (line 19-27): `_client.from('shifts').select('staff').eq('shift_date', dateStr)`, collect non-`'Open'` staff names into `bookedSet`.
2. In the `availabilityForDay` map (line 43-49), add `'booked': bookedSet.contains(name)`. Derive a display status: `on_vacation` → "On vacation"; `booked` → "Booked" (and treat as not-available-for-another-assignment); else the existing `available` bool.
3. Return `bookedSet` (or a `booked` flag per staffer) up through `calendar_page.dart` (`_availabilityForDay`, lines 62/223) into the widgets that show availability (`StaffAvailabilityCard`) so the UI can render a "Booked" state distinct from "Available"/"Unavailable"/"On vacation".
4. **Admin assign dropdown** — `lib/widgets/admin_publish_panel.dart:197-211` (the "Assign Slot" `DropdownButton<String>`). Today it lists `['Open', ...staffNames]` with no per-staffer state. When the admin is building a shift for a target day, flag staffers already assigned that day — e.g. suffix "— already on a shift" or disable them. The day's existing shifts are already streamed in `calendar_tab.dart` (`shifts` eq `shift_date`, around line 152) — pass the booked names into the panel so it doesn't need a new query.

### Edge cases
- A staffer with two legit shifts in one day (split shift): "booked" should be a *warning*, not a hard block — the admin can still assign. Make it a visible flag, not a disable.
- `staff == 'Open'` rows are unassigned — exclude from `bookedSet`.
- Realtime: availability already streams; ensure the `booked` state recomputes when a shift is added/removed that day (the `shifts` stream in `calendar_tab` already fires — wire the booked names through the same rebuild path).

### Verification
On the phone, assign someone to a shift on a day, then look at that day's availability list — they should show "Booked," not "Available." Try to assign the same person again from the admin panel — they should be flagged/disabled in the dropdown.

---

## Sequencing (recommended)

1. **Feature B first** — small, isolated, purely `shift_card.dart` (+ `event_shift_card.dart`), no backend, no deps. Fastest morale win and de-risks nothing.
2. **Feature C second** — backend-ish but small (one extra query + UI flag), uses tables/streams that already exist. Improves the admin experience that Feature A relies on.
3. **Feature A last** — biggest scope, new deps, new screen, the parsing problem. Do B and C first so the app you're importing *into* is already pleasant and prevents double-assigns.

Do the **Section 0 prerequisite** (canonical iOS build) before the first time you want to QA any of these on the phone — otherwise the Mac pull won't build.

---

## Open decisions (before starting)
- Extraction strategy for Feature A: on-device ML Kit vs LLM vision (privacy vs robustness).
- Canonical bundle ID (Section 0).
- Whether to restore Sentry or formally drop it (Section 0).
- Feature C: "booked" as a hard block or a soft warning in the admin dropdown (recommend soft warning — split shifts exist).