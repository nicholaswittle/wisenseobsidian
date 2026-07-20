---
title: Launch Readiness — COMMS LINK and Apex 2026-07-20
tags: [launch, comms-link, apex, android, ios, checklist, output]
aliases: [Launch Plan, COMMS LINK + Apex Launch, No-Mac Launch Plan]
date: 2026-07-20
status: active
---

# 🚀 Launch Readiness — COMMS LINK & Apex Scheduler

> Grounded against the live repos on 2026-07-20 (git state, merge status, security audit), **not** the vault's summary notes. Constraint: **no Mac access yet** → Android-first, iOS via cloud CI later.

## Strategy — Android first, iOS via cloud

No Mac is required for **Google Play**: build the `.aab` on Windows or GitHub Actions. iOS/TestFlight needs macOS *somewhere* — a **GitHub Actions macOS runner** or **Codemagic**, not a local laptop. Apex already has `build-ios.yml` + `build-android.yml` scaffolded. Target for both: **Play Store internal testing**; defer iOS to cloud CI once the Apple Developer account ($99/yr) exists.

---

## COMMS LINK — closest to ready ✅

On-device Gemma 2B-IT, no backend, no auth → simplest launch. Working tree clean; **10 commits ahead of `origin/main`** (unpushed).

| # | Task | Type |
|---|------|------|
| 1 | **Push `main`** (10 local commits) | ship |
| 2 | Verify `flutter analyze` + `flutter test` green | gate |
| 3 | Android release signing: `upload-keystore.jks` + `key.properties` | launch |
| 4 | Finalize app icon, name, version in `pubspec.yaml` | launch |
| 5 | Build `.aab` → Play Console → internal testing | launch |
| 6 | Store assets: icon, feature graphic, 2–8 screenshots, descriptions | launch |
| 7 | Host privacy policy URL (on-device / zero-retention — strong, easy story) | launch |

**No code blockers.** ~1–2 days of packaging.

---

## Apex Scheduler — two gates before packaging ⚠️

### Gate A — finish the stuck merge (~30 min, do first)
Mid-merge of `cursor/apex-store-launch-447c`. Conflicts in 6 files (`auth_page.dart`, `billing_page.dart`, `calendar_page.dart`, `main.dart`, `supabase/functions/create-payment-intent/index.ts`, `vercel.json`) were **already hand-resolved — zero conflict markers remain** — but never `git add`-ed, so the merge is frozen (`.git/MERGE_HEAD` present).
- **Do:** review resolutions → `git add` the 6 files → `flutter analyze` → commit the merge.

### Gate B — security blockers (the real work)
From the GLM audit ([[output/Apex Security Audit 2026-07-19]]) — launch blockers, not polish:
1. **RLS not enforced** — queries filter by date, not `organization_id`; the RLS migrations the README names are **absent from `supabase/migrations/`**. Cross-org data leak. → commit RLS SQL, verify live on Supabase, add `.eq('organization_id', orgId)` to every stream/query.
2. **Claim / clock races** — `claimOpenShift` has no `.eq('staff','Open')` guard (double-booking); swap approval non-transactional; `clockIn` can double-insert open `time_entries`.
3. **Timezone** — naive local ISO strings vs `timestamptz` → wrong payroll day boundaries. Use UTC / server `now()`.

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
