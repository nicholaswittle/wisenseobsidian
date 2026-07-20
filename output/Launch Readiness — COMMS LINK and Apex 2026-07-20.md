---
title: Launch Readiness — COMMS LINK and Apex 2026-07-20
tags: [launch, comms-link, apex, android, ios, checklist, output]
aliases: [Launch Plan, COMMS LINK + Apex Launch, No-Mac Launch Plan]
date: 2026-07-20
status: active
---

# 🚀 Launch Readiness — COMMS LINK & Apex Scheduler

> Grounded against the live repos on 2026-07-20 (git state, merge status, security audit), **not** the vault's summary notes. Constraint: **no Mac access yet** → Android-first, iOS via cloud CI later.

> **⏱️ Progress (2026-07-20):** COMMS LINK gate verified (59/59) and **pushed** ✅. Apex **code-complete & pushed** ✅ — Gate A merge (`13971b0`), Gate B races/timezone (`9f723a5`), Gate B **RLS migration authored** (`a66b039`). All green (analyze clean, 7/7). **One human step remains for Apex:** apply `supabase/migrations/20260720000000_launch_blockers_rls.sql` to **Supabase staging → smoke-test → prod** (RLS is default-deny; only you have DB access). Then both apps → Android packaging (Gate C).

## Strategy — Android first, iOS via cloud

No Mac is required for **Google Play**: build the `.aab` on Windows or GitHub Actions. iOS/TestFlight needs macOS *somewhere* — a **GitHub Actions macOS runner** or **Codemagic**, not a local laptop. Apex already has `build-ios.yml` + `build-android.yml` scaffolded. Target for both: **Play Store internal testing**; defer iOS to cloud CI once the Apple Developer account ($99/yr) exists.

---

## COMMS LINK — closest to ready ✅

On-device Gemma 2B-IT, no backend, no auth → simplest launch. Working tree clean; **10 commits ahead of `origin/main`** (unpushed).

| # | Task | Type |
|---|------|------|
| 1 | ~~**Push `main`** (10 local commits)~~ **DONE 2026-07-20** — pushed to `origin/main` | ship |
| 2 | ~~Verify `flutter analyze` + `flutter test` green~~ **DONE** — analyze clean, 59/59 pass | gate |
| 3 | Android release signing: `upload-keystore.jks` + `key.properties` | launch |
| 4 | Finalize app icon, name, version in `pubspec.yaml` | launch |
| 5 | Build `.aab` → Play Console → internal testing | launch |
| 6 | Store assets: icon, feature graphic, 2–8 screenshots, descriptions | launch |
| 7 | Host privacy policy URL (on-device / zero-retention — strong, easy story) | launch |

**No code blockers.** ~1–2 days of packaging.

---

## Apex Scheduler — two gates before packaging ⚠️

### Gate A — finish the stuck merge ✅ DONE 2026-07-20
Mid-merge of `cursor/apex-store-launch-447c`. Conflicts in 6 files (`auth_page.dart`, `billing_page.dart`, `calendar_page.dart`, `main.dart`, `supabase/functions/create-payment-intent/index.ts`, `vercel.json`) were already hand-resolved (marker-free) but never `git add`-ed.
- **Done:** staged the 6 files; found + fixed a dropped import (`staff_repository.dart` now imports `core/profile_session.dart` for `defaultOrganizationId` — was 2 analyze errors); `flutter analyze` clean; `flutter test` 7/7; committed merge **`13971b0`** (local, **not pushed** — holds until Gate B security work is in).

### Gate B — security blockers (the real work) — **PARTIAL 2026-07-20**
From the GLM audit ([[output/Apex Security Audit 2026-07-19]]) — launch blockers, not polish:
1. **RLS not enforced** — ✅ **AUTHORED & pushed** (`a66b039`). Vendored `20260720000000_launch_blockers_rls.sql`: `apex_current_org()` helper + per-org policies on profiles/organizations/shifts/swaps/time_entries/time_off_requests/notifications, `organization_id` added to `swaps` (backfilled to the sole venue), partial unique index for atomic clock-in. Idempotent (`drop policy if exists`→`create`) so it reconciles whether or not equivalent policies are already live. `postSwap` now stamps `organization_id`. Doc contradiction reconciled (README vs MIGRATION_INVENTORY). **⚠️ Remaining human step (only you have DB access):** run this migration on **Supabase staging**, smoke-test sign-in/calendar/claim/clock/swap/notifications, then promote to prod. Per-query `.eq('organization_id')` scoping stays optional — RLS enforces it server-side regardless.
2. **Claim / clock races** — ✅ **FIXED** (commit `9f723a5`). `claimOpenShift` now has `.eq('staff','Open')` + `select()` result check (no more last-write-wins); `clockIn` returns any existing open entry instead of duplicating; *(swap-approval transactionality still an RPC-level TODO)*.
3. **Timezone** — ✅ **FIXED** (`9f723a5`). `clockOut` writes `toUtc()` ISO against `timestamptz`. *(The `loadActiveEntriesForToday` "today" boundary is venue-timezone-dependent — left for the RLS session since the right behavior is a policy call, not a guess.)*

**Remaining for Gate B:** resolve RLS (item 1, blocked on live-DB truth), swap-approval atomicity (RPC), partial unique index on `time_entries(user_id, shift_id) WHERE clock_out IS NULL`, and the active-entries timezone boundary.

### Gate C — Android packaging
Same as COMMS LINK (keystore, `.aab`, store listing). Infra scaffold already written by Cursor: `docs/LAUNCH_WITHOUT_MAC.md`, `docs/PRIVACY_POLICY.md`, `docs/LAUNCH_CHECKLIST.md`, CI workflows.

**Stripe stays deferred** (`billingEnabled = false`) — correct for the Jigsy's pilot; not a blocker. See [[DECISIONS]].

---

## Recommended sequence

1. **COMMS LINK → Play internal testing** — clean win; learn the Play flow with zero security risk.
2. **Apex Gate A** — finish the merge; unblocks the branch.
3. **Apex Gate B** — RLS + races + timezone; the real ship-blocker for multi-tenant staff data.
4. Both → iOS via **Codemagic** once the Apple account exists.

---

Related: [[COMMS LINK]], [[Apex Scheduler]], [[output/Apex Security Audit 2026-07-19]], [[output/Apex — Merge Conflict Resolution Plan]], [[DECISIONS]], [[index]], [[Home]], [[hot]]
