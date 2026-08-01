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

> ⚠️ **CORRECTION (2026-07-20, later):** COMMS LINK is **NOT ready**. Its on-device model never ran — the hardcoded HuggingFace URL is licence-gated and returns **401 for every user**. The 59/59 suite never touched Gemma. Fixed in `94cecea`; **on-device verification still pending** (iPhone 15 + Mac + free Xcode). Gate C is blocked for this app until `integration_test/model_smoke_test.dart` passes on a physical device. The table below described packaging readiness, not functional readiness.

## COMMS LINK — closest to ready ✅

On-device Gemma 2B-IT, no backend, no auth → simplest launch. **`main` in sync with `origin/main`** (pushed 2026-07-20). Next = Gate C packaging.

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

## Apex Scheduler — packaging blocked on human RLS apply ⚠️

### Gate A — finish the stuck merge ✅ DONE & PUSHED 2026-07-20
Merge **`13971b0`** completed and pushed with Gate B. Historical detail: mid-merge of `cursor/apex-store-launch-447c` (6 conflict files) was staged, analyze-fixed, and committed.

### Gate B — security blockers ✅ CODE DONE & PUSHED — human DB step open
From the GLM audit ([[output/Apex Security Audit 2026-07-19]]):
1. **RLS** — ✅ Authored & pushed (`a66b039`, `20260720000000_launch_blockers_rls.sql`). **⚠️ Remaining human step:** apply on **Supabase staging → smoke-test → prod**.
2. **Claim / clock races** — ✅ Fixed (`9f723a5`).
3. **Timezone** — ✅ Fixed (`9f723a5`).

**Post-pilot polish (not blocking Gate C once RLS is live):** swap-approval RPC atomicity; venue-tz “today” boundary for active entries.

### Gate C — Android packaging
Same as COMMS LINK (keystore, `.aab`, store listing). Infra scaffold already written by Cursor: `docs/LAUNCH_WITHOUT_MAC.md`, `docs/PRIVACY_POLICY.md`, `docs/LAUNCH_CHECKLIST.md`, CI workflows.

**Stripe stays deferred** (`billingEnabled = false`) — correct for the Jigsy's pilot; not a blocker. See [[DECISIONS]].

---

## Recommended sequence

1. **Nicholas:** Apply Apex RLS on staging→prod (only remaining code-adjacent blocker).
2. **Both apps → Gate C** — [[output/Gate C — Android Packaging & Store Listings 2026-07-20]] (keystore, AAB, graphics, privacy, Play listing).
3. **COMMS LINK → Play internal testing** — learn the Play flow with zero multi-tenant risk.
4. **Apex → Jigsy pilot** — baseline metrics in [[customers/Jigsys Brewpub]].
5. Both → iOS via **Codemagic** once the Apple account exists.

Weekly board: [[NOW]].

---

Related: [[COMMS LINK]], [[Apex Scheduler]], [[NOW]], [[output/Apex Security Audit 2026-07-19]], [[output/Apex — Merge Conflict Resolution Plan]], [[DECISIONS]], [[index]], [[Home]], [[hot]]
