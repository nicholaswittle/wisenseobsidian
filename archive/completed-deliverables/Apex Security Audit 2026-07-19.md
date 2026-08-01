---
title: Apex Security Audit 2026-07-19
tags: [apex, audit, security, launch-blocker]
aliases: [Apex GLM audit, Apex tenancy audit]
---

# Apex Security Audit — 2026-07-19

**Source file:** `C:\development\projects\apex\apex\audit\AUDIT_2026-07-19.md`  
**Auditor:** GLM 5.2 (read-only). Spot-checked afterward — critical claims hold.

## Verdict

Treat the red section as **launch blockers**, not polish. Direction of the calendar refactor is sound; tenancy and races are not.

## Critical (fix before pilot)

1. **Multi-tenancy assumed, not enforced**
   - Hot reads/streams often filter by date only (e.g. `calendar_tab` shifts by `shift_date`; swaps stream global).
   - README names RLS migrations (`launch_blockers_rls_and_auth`, hotfix, complete features) that are **not** in `supabase/migrations/`.
   - Defense in depth: commit RLS SQL + add `.eq('organization_id', orgId)` on every query/stream.
2. **Claim / clock races**
   - `claimOpenShift` updates by shift id with **no** `.eq('staff','Open')` → last write wins.
   - Admin swap approval is multi-step non-transactional.
   - `clockIn` can insert duplicate open `time_entries`.
3. **Time clock timezone**
   - Filters/writes use naive local ISO strings against `timestamptz` → wrong day boundary and payroll math. Prefer UTC / server `now()`.

## High (soon after)

- Force-unwrapped `currentUser!.id` on submit paths
- Time-off overlap client-only / unscoped cache
- Duplicate notification paths
- `copyPreviousWeek` no de-dupe
- Conflict detector N+1 + name-as-key across orgs

## What it does well

Service/repository split from the old god-file, invite codes + `apex_create_organization`, dart-define config, realtime streams.

## Fix order

1. Pull remote RLS into repo; verify live on Supabase  
2. Org-scope every stream/query  
3. Atomic claim + clock guards + UTC timestamps  
4. Then high/medium cleanup  

Related: [[Apex Scheduler]], [[Apex Scheduler — Code Reference]], [[Audit Findings Loop]], [[Supabase]]
