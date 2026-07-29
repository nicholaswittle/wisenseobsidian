---
title: Apex v2 — Restaurant OS Build
tags: [apex, restaurant-os, flutter, supabase, build, active]
aliases: [Apex v2, Restaurant OS Build, apex_v2]
date: 2026-07-27
updated: 2026-07-28
---

# Apex v2 — Restaurant OS Build

> Reimagined Apex building toward full Restaurant OS: scheduling + ordering + labor cost + tips + chat + call-outs + capacity. Employee-first, 3-tap max, dark Material 3. Built on existing Apex Supabase with org_id scoping.

**HEAD:** `nicholaswittle/apex_v2` `main` @ `e99b8ff` (2026-07-28 evening — menu extras + photo import).  
**Live:** Real https://apex-v2-ten.vercel.app · Demo https://apex-v2-demo.vercel.app.  
**Jigsy Order online (2026-07-28):** https://jigsyssite.vercel.app · staff https://jigsyssite.vercel.app/staff.html · `jigsysite` @ `4505cd4`. Accept & print; menu stock sync; no online alcohol — [[Jigsy Online Ordering — Live Status 2026-07-28]].  
**Archive:** [[restOS]] @ `a6cb554`.  
**Theme:** New Horizon dark palette (purple/teal on blue-black `0xFF0A0C10`) — replaced original brown-on-black.  
**Audits:** [[wisense/projects/APEX_V2_AUDIT_2026-07-27]] (code) · [[wisense/projects/APEX_V2_OS_JIGSYS_INTEGRATION_AUDIT_2026-07-28]] (Jigsy OS integration) · [[wisense/projects/APEX_V2_FULL_SYSTEM_SECURITY_AUDIT_2026-07-28]].

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
- Vision: Anthropic via edge (`ANTHROPIC_API_KEY` secret) — model `claude-sonnet-4-5` (menu + schedule photo). Paste/text parsers stay free on-device.

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
| 36 | Menu extras / toppings editor | Soft-launch | DONE (2026-07-28) | `menu_editor_screen.dart` — per-item modifier groups/options; edit prices; presets; duplicate item; quick paste add |
| 37 | Photo → menu import | Soft-launch | DONE (2026-07-28) | `menu_photo_import_screen.dart` + `parse-menu` edge + `menu_text_parser.dart` — Anthropic vision; ~$0.02/photo on Sonnet in smoke test |

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

### Soft-launch money path (2026-07-28)

- Stripe Connect Express + 1.5% platform fee on Pay Now (`create-guest-payment`, `stripe-connect-onboard`, `stripe-os-webhook`)
- Security hardening: no anon forged order inserts; guest venue RPC; money-field locks; webhook amount match
- Decision: keep vision on **Sonnet 4.5** for menu/schedule photos until volume makes Haiku worth the accuracy tradeoff (~$0.02/menu photo observed)

### Verdict

Weekend 1–3 scope **plus** Phase 2 ordering and Phase 3 OS bridges (labor vs revenue, call-outs, smart capacity) are built and pushed to GitHub. Soft-launch path includes Connect fees + menu editor extras + photo menu import. Remaining items (29–35) are polish/integrations and later phases. Product audit by Nicholas still pending — plan to show Emily (Jigsy's) the v2 demo for feedback. "Apex v2 lite" for v1 launch = set org tier to `free` in entitlements, same app shows only 4 modules.

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
- `parse-schedule/index.ts` — cloud vision for photo-to-schedule (`claude-sonnet-4-5`, needs `ANTHROPIC_API_KEY`)
- `parse-menu/index.ts` — cloud vision for photo-to-menu (same model/key; on-device text fallback via `menu_text_parser.dart`)
- `route-notification/index.ts` — SMS fallback via Twilio
- `route-callout/index.ts` — ranks available staff and routes call-out (SMS optional / Twilio)
- `stripe-connect-onboard`, `create-guest-payment`, `stripe-os-webhook` — Connect Express + OS Payment Links

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
---

## Security Audit — 2026-07-29 (CRITICAL found + fixed)

**Privilege escalation via `profiles` self-update.** Fixed in `a6d6d42`,
migration `20260729000000_profiles_privilege_escalation.sql`, **applied to the
live database and verified**.

The old policy let a user update their own row with `WITH CHECK` constraining
only `organization_id`. `profiles` also holds `role`, `is_super_admin` and
`hourly_rate`, so any authenticated user could run:

```sql
update profiles set is_super_admin = true, role = 'Owner' where id = auth.uid();
```

`is_super_admin()` reads that column, so this granted platform-wide read on
**every** organization and profile, plus UPDATE on every organization including
`tier`. One statement from any invited staff member reached **every other
venue's data**. Inherited from v1 — live the whole time.

**The lesson worth keeping:** `apex_create_invite` is genuinely well-built
(manager required, role whitelist, owner-only for Owner invites, expiry,
collision retry). It gates on `apex_current_role()` which reads `profiles.role`.
*Every authorization decision in this schema resolves through columns the user
could rewrite*, so one weak `WITH CHECK` voided all of it.
**Audit the columns that policies read, not just the policies themselves.**

Fix: self-updates freeze the three privileged columns via SECURITY DEFINER
readers (avoids RLS recursion). Owners manage staff including pay, but can never
mint a super admin. Uses `IS NOT DISTINCT FROM` because fleet admins have
`organization_id = NULL`, and `null = null` is NULL rather than true — plain
equality would have locked those accounts out of their own profile.

**Also fixed:** password reset had no `redirectTo`, so the emailed link went to
the project default Site URL instead of the deployment (that flow was broken in
production); invite-join now verifies the profile actually linked instead of
assuming, so a bad code no longer strands an account that can neither join nor
sign up again; two empty `catch {}` blocks now log.

### Still open — deliberately not fixed

- `org insert authenticated` has `WITH CHECK true` — unbounded org creation.
  The obvious guard is `apex_current_org_id() is null`, but **v1's
  `ProfileService.createOrganization` still uses this path** and the signup
  trigger now auto-assigns an org, so the guard would likely break v1 signup.
  Needs ten minutes inside v1 first.
- **Verify email confirmation is ON.** `apex_handle_new_user` hardcodes
  `nicholaswittle@gmail.com` and `@wisensellc.com` to `is_super_admin = true`.
  Safe only if Supabase actually confirms the address; if confirmation is off,
  anyone signing up with that email gets platform admin.

**Corrected an earlier claim:** `_createRestaurant` *does* work. The trigger
`apex_handle_new_user` reads `org_name` from signup metadata and provisions the
org, assigning Owner to the first profile in it. The earlier note saying it
never creates the restaurant came from reading only the client.

## Micro AI Assistants — preserve as-is (Nicholas, 2026-07-29)

Two tiers, and the split is the good part:

1. **On-device deterministic parsers** — `core/menu_text_parser.dart`,
   `core/schedule_text_parser.dart`. No API key, no network, no per-parse cost,
   works offline. Used for paste/text input.
2. **Vision via edge function** — photo import calls `parse-menu` /
   `parse-schedule`, which hold `ANTHROPIC_API_KEY` as a **Supabase secret,
   server-side**. Model `claude-sonnet-4-5`, ~$0.02/menu photo observed.

Both feed a human review step before anything is inserted. The key is never in
the client bundle, which is the thing that matters — a Flutter web build is
readable by anyone. Nicholas asked explicitly that these be kept.

*(Correction: an earlier version of this section said the photo import screens
made no network calls. Only tier 1 is offline; tier 2 goes through the edge
function. The security property — no key in the client — holds for both.)*

## God Mode — Fleet Admin Console (2026-07-29)

**Verified working by Nicholas.** Commits `f806a38` (audit trail), `642e99a`
(view-as + health + grants). Live at https://apex-v2-ten.vercel.app

The console itself already existed (`features/admin/admin_console_screen.dart`,
883 lines — venues, tiers, module toggles, users, invites, billing). Audit found
all six `admin_*` RPCs correctly `SECURITY DEFINER` + gated on
`is_super_admin()`. The gaps were logging and support access.

### Added

| Piece | What it does |
|---|---|
| `admin_audit` + triggers | Every tier/module/invite/privilege change, however it arrived |
| `admin_view_sessions` | Time-boxed read-only "view as venue" |
| `admin_fleet_health()` | 5 integrity checks — all currently **zero** |
| `admin_set_super_admin()` | Audited grant path, refuses self-revocation |

### Two design decisions worth keeping

**Audit via triggers, not inside the RPCs.** Triggers on `organizations`,
`organization_invites`, `profiles` capture a change however it arrived — RPC,
the `orgs super admin update` policy, or by hand in the SQL editor. Auditing
inside the six functions would have left those paths silent, and the SQL-editor
path is the one that otherwise leaves no trace at all. Rows with no `auth.uid()`
are labelled **"direct SQL"** rather than blank.

**View-as is a session, not a flag.** The obvious implementation — a blanket
`using (is_super_admin())` SELECT policy on all 30 org-scoped tables — was
written, then **blocked by a safety classifier, correctly**. That access is
permanent, always on, and covers orders/tips/revenue for every venue, resting
entirely on a boolean any user could write until `20260729`. Nicholas chose the
harder option. Result: ~60 more lines of SQL, and a bug in `is_super_admin`
exposes **one venue for 30 minutes** instead of every venue forever.

Guards that matter: `admin_active_view_org()` re-checks `is_super_admin()` on
every call, so revoking the flag kills open sessions immediately rather than at
expiry; only one venue open at a time (two would make the function ambiguous and
the banner a lie); **SELECT only** — no write policy accompanies a session, so
view-as can never become edit-as. Red banner + Exit, and the session survives a
browser refresh.

### Still not built (from the original design)

Suspend/restore venue · invite revocation UI · broadcast to all venues · export
a venue's data · per-venue AI parse spend attribution (photo import runs ~$0.02
each through the edge function and is currently unattributed — it will grow with
your best customers).

### Open from the security audit

- `org insert authenticated` still `WITH CHECK true` — unbounded org creation.
  Guard would likely break v1's `ProfileService.createOrganization`.
- **Verify email confirmation is ON** — `apex_handle_new_user` hardcodes two
  emails to `is_super_admin = true`. If confirmation is off, anyone signing up
  with those addresses gets platform admin.

## Tiers + Pricing Alignment (2026-07-29)

Commit `2622129`. Both deployments verified clean.

### Free/Pro line moved to match the site

| Tier | Modules |
|---|---|
| **Free** | scheduling · swaps · time clock · push · **offline** · **team chat** |
| **Pro $25** | tips · labor cost · log book |
| OS $99 | ordering · capacity · no-show engine · labor vs revenue |
| Multi $199 | multi-location |

`offlineMode` and `teamChat` moved Pro → Free, with the reasoning written into
`entitlements.dart` so they do not drift back:

- **Offline is reliability, not a feature.** Charging for "works when the wifi is
  bad" makes the free tier feel broken in exactly the venues this sells into.
- **Chat kills the group text.** If messages stay in the group text the switch is
  never finished, and the venue is one bad week from leaving.

Pro keeps the money tools. People pay to fix money problems and resent paying
for convenience.

### Branded-site price: $1,499 → $299 setup + $79/mo

The in-app upsell still quoted **$1,499 one-time**, a price removed from the
website on 2026-07-23. The app was selling something the business no longer
offers.

**Nicholas's call, and it is the right one:** Cursor and Antigravity both landed
on $1,499, but with no case studies and no demand yet, a new provider cannot ask
for that upfront. Start low, raise once there is demand.

Worth keeping in mind:
- **This is deferral, not a discount.** $79/mo crosses $1,499 at ~19 months and
  keeps going. The recurring model is worth *more*, just later.
- **$299 setup probably does not cover build time.** The monthly is what makes it
  work, which means **churn is the real risk in this model, not price.**
- **Raising prices is easy for new customers, hard for existing ones.**
  Grandfather the early ones deliberately and say so — "locked in" is a selling
  point, and a surprise increase is how a small client base sours.
- Three AI tools agreeing on $1,499 is not market evidence — they reason from
  similar priors about national rates. Nicholas talking to businesses in Enola is
  better data than model consensus.

### Client name removed from four more places

Website upsell card, billing sheet, support email, demo fixtures. **Worst was
the printed kitchen receipt** (`staff_console_screen.dart`) — a client name was
hardcoded into both the plain-text and HTML ticket, so every other venue's
printer would have produced receipts under someone else's banner. Now uses the
loaded venue name, falling back to `ONLINE ORDER`.

**Pattern worth naming:** three of these were decisions made once, applied to one
surface, and left wrong on another — same shape as the demo `is_super_admin`
flag. **When a commercial term changes, grep both repos rather than fixing only
where it was noticed.**

---

# Session — 2026-07-29 (afternoon/evening): membership, multi-venue, two audits

HEAD `542fbb1`. All work applied to the live database and deployed.
Audits: [[wisense/projects/APEX_V2_FULL_SECURITY_AND_CORRECTNESS_AUDIT_2026-07-29]]
· [[wisense/projects/APEX_V2_REAUDIT_2026-07-29]]

## 1. Membership became a relationship

**Problem Nicholas identified:** an account belongs to the *person*, not the
venue. Hospitality staff work two jobs and change them constantly, so leaving a
venue must not disable a login they need tomorrow. `profiles.organization_id`
could not express that — one venue per person.

Nulling that column looked like the cheap fix, but it trades an access problem
for a **records** problem: the profiles read policy is
`organization_id = apex_current_org_id()`, so an unlinked profile becomes
unreadable and last month's labor report loses its names. **History has to
outlive the employment.**

New `organization_members` (org, user, permission_role, job_role, joined_at,
left_at). Done in **two migrations on purpose**: step 1 backfilled and changed
nothing, so old and new answers could be compared per user before any
authorization moved. That comparison caught the one real risk — a fleet admin
has `profiles.role = 'Owner'` but no membership row, so a naive repoint would
have silently demoted Nicholas to Staff.

Step 2 (the helper repoint) was **hand-applied by Nicholas** after a safety
classifier blocked the write; filed afterwards as
`20260802050000_membership_helpers_repoint.sql`.

**Two fallback rules that matter:**

- Users with *no* membership rows fall back to `profiles`, so fleet admins keep Owner.
- That fallback can **never** revive a `left_at` membership — otherwise
  offboarding would appear to work while doing nothing.

## 2. What the refactor broke, three times

The read side was verified carefully. The write side was not, and **every one of
these was found by Nicholas using the app, not by me checking.**

1. `apex_redeem_invite` never created a membership row — new joiners worked only
   through the legacy fallback.
2. `apex_set_role` wrote `profiles.role`, which nothing reads for authorization
   any more. **Promoting someone to Manager silently did nothing**, including the
   wage visibility that promotion is supposed to carry.
3. `apex_set_org_module` and `apex_set_hourly_rate` resolved the caller's venue
   from `profiles` — fine today, broken for a multi-venue user.

Fixed by widening a `profiles` trigger to mirror into memberships, so every path
converges rather than each having to remember.

> **Lesson:** when a read path is repointed, enumerate the **writers** in the
> same pass. Verifying reads and calling it done is how all three of these
> shipped.

## 3. Multi-venue

Nicholas's design: after login, pick which venue you are looking at.

- `profiles.active_org_id` plus `apex_set_active_venue`, which refuses venues you
  do not work at. Stored server-side because RLS resolves through
  `apex_current_org_id()` — the database has to agree which venue is in view.
  Not a session variable: PostgREST pools connections.
- `apex_my_venues()` exists because the organizations read policy is
  `id = apex_current_org_id()` — one venue by definition — so a two-job employee
  could not read the *name* of the job they were not currently in.
- The picker only appears with a genuine choice; single-venue staff sign straight in.
- Existing accounts can now **join** a second venue. The join flow always called
  `signUp` and died on `user_already_exists`.

## 4. Pay privacy — a fix that had to be done twice

`profiles read same org` let **any member read every profile including
`hourly_rate`**. Hiding it in the UI would have been decorative.

RLS could not help: staff legitimately need teammate **rows**, because chat, the
log book and tips all resolve author names through profiles. So it had to be
column-level.

**The first attempt silently failed** — `REVOKE SELECT (column)` does nothing
when the role holds a *table-wide* grant. Fixed by revoking the table grant and
re-granting per column, generated rather than hand-listed.
⚠️ **Any column added to `profiles` later is unreadable until granted.**

Rates now come from `apex_my_hourly_rate()` and `apex_team_pay()`
(manager-gated). Team is open to staff — names and job roles, no pay, no contact
details, no invite or remove.

## 5. Capacity: suggest, do not act

Nicholas asked how it could know a venue's normal minimum. It could not:
`auto_pause_threshold` defaulted to **1** with auto-pause **on**, so a brand-new
venue acted on a number nobody chose.

**The baseline was already in the data:** the schedule is the manager's own
statement of what normal looks like. `core/staffing_signal.dart` (pure, 8 tests)
compares who is scheduled *now* against who is clocked in.

Auto-pause now defaults **off**. The errors are asymmetric and both expensive —
pausing wrongly kills revenue on the busiest night, staying open wrongly blows
ticket times — so the app states what is true and offers the button. It does not
know that tonight is slow, that the missing cook is ten minutes out, or that the
owner is on the line. `capacity_events` records decision context for later
learning: **it is not learning yet, and should not be described as such.**

## 6. Two audits (Antigravity) — findings and corrections

All 12 findings plus 1 the audit missed are closed. Both times, **the most severe
item was something the auditor missed or mis-rated**, found by checking claims
against the live database rather than reading the report.

| Finding | Reality |
|---|---|
| `org_members_write` **(missed)** | Any manager could set their own `permission_role = 'Owner'`, bypassing the owner-only `apex_set_role`. Same shape as July's escalation: a `WITH CHECK` constraining *who*, not *what*. |
| Manager could remove an owner | Real. Rank now checked against rank. |
| Any staff could 86 a menu item | Real. Now manager-gated. |
| Multi-venue cross-reads | Real, and introduced by §3. |
| Tip hours, "CRITICAL, zeroed payouts" | **Overstated.** The fallback only fires when `DateTime.tryParse` fails, which valid ISO timestamps never do. Removed anyway; it was not on fire. |
| Email-confirmation guard | **My fix only looked like a fix** — see below. |

### The fix that wasn't

Finding #7 (hardcoded emails granting `is_super_admin`) was "fixed" by requiring
`email_confirmed_at`. But **Supabase sets that at signup when email confirmation
is disabled — which it is on this project**: every `auth.users` row was confirmed
within two seconds of creation. The guard passed instantly and the hole stayed
open. The re-audit caught the mirror problem: with confirmation *enabled*, the
same guard aborts signup entirely.

Ineffective in one configuration, breaking in the other. **Fixed the class
instead:** signup never grants fleet admin, and the flag is granted only through
the audited `admin_set_super_admin`.

## 7. Camera was never going to work on iOS

Menu photo import was tested by **uploading a file from a desktop**. The camera
path was never exercised — and `Info.plist` declared no
`NSCameraUsageDescription`, so on iOS the app **terminates** the moment someone
taps the camera, and App Store review rejects the binary. `pickImage` was also
unguarded, so a denied permission threw uncaught.

Both fixed. **Desktop upload and camera are different code paths on different
platforms, and only the untested one is the one the pitch describes.**

## Hard-won operational notes

- **Service worker caching.** After every deploy, hard-refresh before concluding
  something is broken. Cost an hour chasing an already-fixed bug.
- **Two Vercel projects:** `apex-v2` (real) and `apex-v2-demo`. `flutter build
  web` wipes `build/web` including `.vercel`, so every deploy must
  `vercel link --project <name>` explicitly. **I once deployed the demo build
  over the real app** by relying on inference.
- **`service_role` bypasses RLS but NOT triggers.** A money-guard trigger nearly
  broke the Stripe webhook.
- **Permissive policies are OR'd.** Dropping a policy by the wrong name is a
  silent no-op that leaves the old permissive one in force — the multi-venue
  scoping fix looked applied and was not.
- **plpgsql resolves record fields at execution**, so a trigger naming a column
  that does not exist creates cleanly and fails on every write.
- **Positional `Future.wait` results** are now Dart 3 records across six screens.
  Inserting a query mid-list silently shifted every access after it, and the
  analyzer could not see it.

## What is left

**Blocking the pilot**

1. Test **camera capture on a phone** — the AI parse is proven on a real menu via
   desktop upload; capture on device and handwriting-at-an-angle are not.
2. **Clean the test data** — 3 venues and 6 people, most fake (`moe`, `moe2`,
   `test rest`, `test@wisense.com`, `nikwit13@aol.com`).
3. **Job roles set for 1 of 6 people.** Until they are set the capacity signal
   stays silent, correctly but invisibly.

**Nicholas's to do**

- **Turn on email confirmation** (Dashboard → Authentication → Providers →
  Email). Nothing depends on it for privilege now, but with it off anyone can
  sign up as an address they do not own.
- **`apex/apex/supabase/config.toml` points at Horizon's project.** A
  `supabase db reset --linked` from that folder hits the wrong database.
  Unfixed pending confirmation.

**Mine when asked**

- **Usage instrumentation for the pilot** — which screens get opened, by whom,
  how often. Worth having *before* the cohort starts, not after.
- **Migration history reconciliation** — the database matches neither repo, so
  `supabase db push` cannot work from anywhere. Fine solo, a problem with a
  second developer.
- **Per-venue AI spend attribution** — photo imports cost ~$0.02 each, untracked,
  and grow with the best customers.
