---
title: Apex — Merge Conflict Resolution Plan
tags: [plan, apex, launch-priority, high-risk]
status: pending-ratification
---

# Apex — Store-Launch Branch Merge Plan

## Context

Apex has its own git repo (`nicholaswittle/apex.git`). The `cursor/apex-store-launch-447c` branch is 10 commits ahead of `main`, but main also has divergent work (security-hardening, multi-tenancy, ProfileSession refactor). A merge produces 13 conflicts across 6 files. This is High risk per the Tripartite Protocol — touches auth, payment, main entry point.

## Conflict analysis (6 files, 13 conflict markers)

### 1. lib/auth_page.dart (2 conflicts)
- **Import:** main uses `ProfileSession`, store-launch uses `profile_service` + `app_config`
- **Post-auth flow:** main loads ProfileSession after auth; store-launch has business-name signup + try/catch error handling
- **Resolution needed:** reconcile ProfileSession vs profile_service — are these the same thing renamed, or different services?

### 2. lib/billing_page.dart (2 conflicts)
- **Imports:** main has `wisense_ui` + `ProfileSession`; store-launch removed both
- **Class body:** main has full `_BillingPageState` class; store-launch has just `build()` method (billing deferred)
- **Resolution needed:** keep the deferred billing version (store-launch) since billing is disabled for pilot

### 3. lib/calendar_page.dart (2 conflicts)
- **Imports:** main has `wisense_ui` + `ProfileSession` + theme; store-launch has `dart:async`
- **Staff loading:** main has inline Supabase query scoped to venue; store-launch delegates to `_ctrl.loadStaffNames()` with Result pattern
- **Resolution needed:** store-launch's controller delegation is the newer pattern; keep that

### 4. lib/main.dart (4 conflicts)
- **Imports:** main has `flutter_stripe` + `ProfileSession`; store-launch doesn't
- **Bootstrap:** store-launch has `_bootstrapApp()` function; main doesn't
- **Home widget:** main uses `_AuthenticatedHome`; store-launch uses `_AuthGate`
- **Classes:** main has `_AuthenticatedHome`; store-launch has `_StartupErrorApp`
- **Resolution needed:** store-launch's `_AuthGate` + `_StartupErrorApp` + `_bootstrapApp` is the newer, more robust pattern. Keep store-launch version. Integrate ProfileSession if still needed.

### 5. supabase/functions/create-payment-intent/index.ts (1 conflict)
- **Main:** uses `esm.sh/stripe@14.21.0` import
- **Store-launch:** uses `jsr:@supabase/functions-js` + `jsr:@supabase/supabase-js@2`
- **Resolution needed:** store-launch's jsr imports are the modern Deno pattern. Keep that.

### 6. vercel.json (2 conflicts)
- **Main:** simpler config, `outputDirectory: build/web`, has rewrites
- **Store-launch:** has `$schema`, `buildCommand`, different header sources, no rewrites
- **Resolution needed:** store-launch version is more complete with buildCommand. Need to merge: keep store-launch's buildCommand + $schema, keep main's rewrites if they're needed for SPA routing.

## Proposed resolution strategy

1. Prefer the store-launch branch version in all conflicts (it's the newer, more robust code with controller delegation, auth gate, bootstrap, and modern imports)
2. Verify `ProfileSession` (main's pattern) vs `profile_service` (store-launch) — if ProfileSession was renamed to profile_service, use store-launch. If they're different, need to understand what ProfileSession does.
3. For vercel.json: merge both (store-launch buildCommand + main rewrites)
4. After resolving: `flutter analyze` + `flutter test` must pass
5. Gemini review required before final merge commit (High risk — auth + payment)

## Risk assessment

- **Risk level:** HIGH (auth logic, payment edge function, main entry point)
- **Gemini review:** REQUIRED per protocol
- **Destructive?** No — merge preserves both histories; conflicts resolved by choosing + verifying

## Prerequisite

Before resolving: need to understand whether `ProfileSession` (main branch) was renamed to `profile_service` (store-launch) or if they're different services. This determines the auth_page.dart resolution.

Related: [[Apex Scheduler]], [[Apex Scheduler — Code Reference]]