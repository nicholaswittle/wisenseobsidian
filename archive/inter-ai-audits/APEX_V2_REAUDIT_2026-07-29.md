---
type: audit
title: "Apex v2 — Re-Audit Report 2026-07-29"
tags: [audit, reaudit, security, correctness, rls, apex-v2, supabase]
date: 2026-07-29
status: completed
target_project: "apex_v2"
supabase_ref: "pqkremkwfkudrhtxasdj"
git_head: "e7a749a"
---

# 🛡️ Apex v2 — Security & Correctness Re-Audit Report

> **Target Environment:**
> - **Repository:** `C:\development\projects\apex_v2` (Branch `main`, HEAD `e7a749a`)
> - **Supabase Project Ref:** `pqkremkwfkudrhtxasdj` ("Apex")
> - **Shared DB Awareness:** Apex v1 (`C:\development\projects\apex\apex`) shares this database.
> - **Remediation Migrations:** `20260802120000` through `20260802150000`.

---

## Executive Summary & Verification Methodology

All 12 findings from the 2026-07-29 audit were re-evaluated alongside the additional `org_members_write` privilege escalation finding. Verification was performed by inspecting post-remediation migration files, database metadata catalogs (`pg_policies`, `pg_proc`, `information_schema.column_privileges`), triggers, Edge Functions, and Dart source code at HEAD `e7a749a`.

> **Verification Limit Notice:** The CLI database connection operates as `postgres` where `auth.uid()` is `NULL`. RLS policies and `SECURITY DEFINER` functions were verified **structurally** by inspecting SQL definitions and schema grants. Runtime session behavior for signed-in JWT users was evaluated analytically based on structural guarantees.

---

## Part 1 — Verification of Remediation Fixes (Items 1–12)

### 1. `org_members_write` Privilege Escalation
- **Status:** **CONFIRMED**
- **Evidence:** Migration `20260802120000_audit_authorization_fixes.sql` dropped `org_members_write` and recreated it requiring `has_role(organization_id, 'owner')` for both `USING` and `WITH CHECK`. Non-owner managers can no longer update `permission_role` or insert/delete memberships via direct RLS writes. A dedicated SECURITY DEFINER RPC `apex_set_member_job_role` was introduced for managers to update job titles (`job_role`) only.

### 2. `apex_remove_member` Role Rank Check
- **Status:** **CONFIRMED**
- **Evidence:** `apex_remove_member` in `20260802120000` checks `IF lower(v_target_role) = 'owner' AND lower(coalesce(apex_current_role(), '')) <> 'owner' THEN RAISE EXCEPTION 'owner_required'; END IF;`. A manager can no longer remove an owner. Self-removal (`cannot_remove_self`) and removing the last owner (`cannot_remove_last_owner`) remain blocked.

### 3. `apex_set_menu_item_available` Authorization Gate
- **Status:** **CONFIRMED**
- **Evidence:** `apex_set_menu_item_available` in `20260802120000` replaced `is_member(v_org)` with `IF NOT has_role(v_org, 'manager') THEN RAISE EXCEPTION 'manager_required'; END IF;`. Non-manager staff can no longer toggle menu item availability.

### 4. Multi-Venue Operational Read Scoping
- **Status:** **CONFIRMED**
- **Evidence:** Migration `20260802130000_scope_reads_to_active_venue.sql` explicitly dropped the old `<table_name>_member_select` policies on `online_orders`, `order_items`, `order_item_modifiers`, `tip_pools`, `tip_allocations`, `server_tips`, `shift_notes`, `messages`, `call_outs`, `call_out_notifications`, `daily_revenue`, and recreated them checking `organization_id = apex_current_org_id() OR is_super_admin()`. A query of `pg_policies` confirms **zero** `is_member` SELECT policies remain outside `organization_members`.

### 5. Membership-Based RPC Venue & Role Resolution (`apex_set_role`, `apex_set_org_module`, `apex_set_hourly_rate`)
- **Status:** **CONFIRMED**
- **Evidence:** Migration `20260802140000_rpcs_use_membership.sql` repointed all three RPCs to resolve the caller's venue via `v_org := apex_current_org_id();` and role via `has_role(v_org, 'owner'/'manager')`. Every existing exception name (`not_authenticated`, `target_required`, `invalid_role`, `owner_required`, `manager_required`, `different_organization`, `cannot_demote_last_owner`) was preserved.

### 6. `organizations` Table INSERT Policy Tightening
- **Status:** **CONFIRMED**
- **Evidence:** Policy `org insert authenticated` on `organizations` was updated to `WITH CHECK (is_super_admin())`. Verification of Apex v1 (`C:\development\projects\apex\apex\lib\core\profile_service.dart:43`) confirms v1 creates organizations via `ProfileService.createOrganization`, which invokes the `apex_create_organization` RPC (`SECURITY DEFINER`). Because `SECURITY DEFINER` functions bypass RLS, tightening table-level INSERT RLS did **not** break Apex v1.

### 7. Super-Admin Email Confirmation Trigger Requirement
- **Status:** **CONFIRMED**
- **Evidence:** Migration `20260802120000` added trigger `profiles_super_admin_requires_confirmation` executing `apex_require_confirmed_super_admin()`. It blocks setting `is_super_admin = true` on any profile whose matching `auth.users` row has `email_confirmed_at IS NULL`.

### 8. Positional `Future.wait` Destructuring Refactor
- **Status:** **CONFIRMED**
- **Evidence:** Commit `e7a749a` refactored `Future.wait` calls across all six screens (`capacity_screen.dart`, `labor_cost_dashboard.dart`, `labor_vs_revenue_dashboard.dart`, `staff_console_screen.dart`, `tip_management.dart`, `call_out_screen.dart`) to use Dart 3 tuple records: `final (a, b, c) = await (fetchA(), fetchB(), fetchC()).wait;`. Every variable maps directly to its intended query.

### 9. `capacity_events` Manager-Only Writes
- **Status:** **CONFIRMED**
- **Evidence:** Migration `20260802150000_tighten_member_write_policies.sql` dropped `capacity_events_member` and created `capacity_events_select` (`organization_id = apex_current_org_id() OR is_super_admin()`) and `capacity_events_write` (`has_role(organization_id, 'manager')`).

### 10. `online_orders` Money Guard Trigger
- **Status:** **CONFIRMED**
- **Evidence:** Function `apex_guard_order_money()` in `20260802150000` evaluates `IF current_user = 'service_role' OR is_super_admin() THEN RETURN NEW; END IF;`, allowing Edge Functions / Stripe Webhooks (`service_role`) to update payment status. The column list (`subtotal_cents`, `fee_cents`, `tax_cents`, `total_cents`, `platform_fee_cents`, `payment_status`, `stripe_payment_intent_id`) matches exact schema columns.

### 11. Pay Privacy (`hourly_rate`)
- **Status:** **CONFIRMED**
- **Evidence:** `SELECT has_column_privilege('authenticated', 'profiles', 'hourly_rate', 'SELECT');` returns `false`. Pay rates are retrieved only via `apex_my_hourly_rate()` and `apex_team_pay(p_org)`, which check `has_role(p_org, 'manager')`.

### 12. Tip Hours Fallback Cleanup
- **Status:** **CONFIRMED**
- **Evidence:** In valid ISO 8601 strings (e.g. `2026-07-28T17:00:00Z`), `DateTime.tryParse` succeeds 100% of the time. The orphaned fallback `_hoursBetween` was removed completely in `tip_management.dart`.

---

## Part 2 — Remediation Impact & Regression Analysis (Items 13–18)

### 13. Unauthenticated Guest Access & RLS Policies
- **Analysis:** Guest ordering uses `apex_guest_venue` RPC (reads restaurant metadata/settings) and `place_order` RPC (submits orders). Both functions are `SECURITY DEFINER` and execute with owner privileges.
- **Finding:** Direct PostgREST SELECT on `online_orders` by `anon` evaluates `organization_id = apex_current_org_id()`, returning 0 rows for unauthenticated users. This is intentional: guests interact with orders via RPCs and URL token parameters (`paymentReturnFromUri()`), not direct table reads.

### 14. Trigger Chains & Signup Transaction Interruption
- **Finding (Medium Severity):** Trigger `profiles_super_admin_requires_confirmation` fires BEFORE INSERT on `profiles`. During new user signup for hardcoded fleet admin emails (`nicholaswittle@wisensellc.com`), `apex_handle_new_user()` attempts to insert a profile with `is_super_admin = true`. Because `auth.users.email_confirmed_at` is `NULL` at the exact moment of initial signup, `apex_require_confirmed_super_admin()` raises exception `'super_admin_requires_confirmed_email'`, which **aborts the signup transaction**. Fleet admin emails must confirm their email before `is_super_admin` can be set.

### 15. Column-Level SELECT Privileges on `profiles`
- **Analysis:** `SELECT column_name, has_column_privilege('authenticated', 'profiles', column_name, 'SELECT')` was executed against all 14 columns of `profiles`.
- **Finding:** Exactly 13 columns return `true` for both `authenticated` and `anon`. `hourly_rate` returns `false` for both. All columns selected by the Flutter client app are granted.

### 16. `apex_current_org_id()` & Stale `active_org_id` Handling
- **Analysis:** `apex_current_org_id()` checks:
  ```sql
  SELECT p.active_org_id FROM profiles p
  JOIN organization_members m ON m.user_id = p.id
    AND m.organization_id = p.active_org_id AND m.left_at IS NULL
  WHERE p.id = auth.uid()
  ```
- **Finding:** If a user attempts to set an invalid `active_org_id` or if their membership in `active_org_id` is terminated (`left_at IS NOT NULL`), the JOIN fails and `coalesce` safely falls back to their most recent active membership (`m.left_at IS NULL ORDER BY m.joined_at DESC LIMIT 1`).

### 17. Fallback Logic & Departed Memberships
- **Analysis:** `is_member`, `has_role`, and `apex_current_org_id` check `EXISTS (SELECT 1 FROM organization_members m WHERE m.user_id = auth.uid())`.
- **Finding:** If a user has ANY row in `organization_members` (even with `left_at IS NOT NULL`), `EXISTS` returns `true`, forcing execution down the `organization_members` branch (which filters `m.left_at IS NULL` and returns `false`/`NULL`). It **never** falls back to `profiles.organization_id`.

### 18. Demo Build (`DEMO=true`) Isolation
- **Analysis:** `lib/app.dart:423` forces `_isSuperAdmin = false;` in `DemoMode.enabled`.
- **Finding:** Fleet console buttons and view-as capabilities are inaccessible in demo builds.

---

## Part 3 — Unasked & Additional Codebase Audits

### 19. Usability Constraint in `apex_set_member_job_role` (Job Role Whitelist)
- **Location:** PostgreSQL Function `apex_set_member_job_role` in `20260802120000` (lines 42–46)
- **Verification Method:** Structural SQL function inspection.
- **Finding (Low Severity):** `apex_set_member_job_role` hardcodes allowed job roles to: `'line cook'`, `'kitchen'`, `'server'`, `'bartender'`, `'host'`, `'dishwasher'`, `'manager'`. If a manager attempts to assign custom titles such as `'Busser'`, `'Shift Lead'`, `'Prep Cook'`, or `'Runner'`, the RPC throws exception `'unknown_job_role'`.
- **Proposed Fix:** Expand the allowed list or convert to case-insensitive trimming without hardcoded string restrictions.

---

## ⛔ Regressions Introduced by the Remediation

1. **Initial Fleet Admin Signup Exception (Trigger Order Issue):**
   - **Location:** Trigger `profiles_super_admin_requires_confirmation` on `profiles`.
   - **Regression Impact:** Initial signup attempts using hardcoded super-admin email addresses (`nicholaswittle@wisensellc.com` / `nicholaswittle@gmail.com`) fail with exception `'super_admin_requires_confirmed_email'` during the initial signup HTTP request because `auth.users.email_confirmed_at` is null prior to email verification link click.
   - **Fix:** In `apex_handle_new_user()`, default `is_super_admin = false` on initial signup, and upgrade to `is_super_admin = true` upon email confirmation or via fleet console.

---

## ❓ What Could Not Be Verified (& Why)

1. **Signed-in User JWT RLS Execution at Runtime:**
   - **Reason:** CLI database operations run as `postgres` where `auth.uid()` is `NULL`. RLS policies were verified **structurally** via PostgreSQL metadata catalogs (`pg_policies`, `information_schema`).
2. **Live Stripe Webhook Receipt:**
   - **Reason:** Webhook handler logic in `stripe-os-webhook/index.ts` was verified **structurally**, but live Stripe webhooks were not sent during this audit.
3. **Supabase Auth Project-Level Email Confirmation Setting:**
   - **Reason:** Supabase Auth service settings reside in the cloud dashboard and cannot be queried via standard PostgreSQL CLI.
