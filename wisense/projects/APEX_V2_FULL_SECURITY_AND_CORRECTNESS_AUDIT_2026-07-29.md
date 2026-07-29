---
type: audit
title: "Apex v2 — Full Security and Correctness Audit Report 2026-07-29"
tags: [audit, security, correctness, rls, apex-v2, supabase]
date: 2026-07-29
status: completed
target_project: "apex_v2"
supabase_ref: "pqkremkwfkudrhtxasdj"
---

# 🛡️ Apex v2 — Full Security & Correctness Audit Report

> **Target Environment:**
> - **Repository:** `C:\development\projects\apex_v2` (Branch `main`)
> - **Marketing Site:** `C:\development\projects\wisense_horizon_v2\marketing`
> - **Supabase Project Ref:** `pqkremkwfkudrhtxasdj` ("Apex")
> - **Shared DB Awareness:** Apex v1 (`C:\development\projects\apex\apex`) shares this database.

---

## Executive Summary & Verification Methodology

This audit was conducted structurally by inspecting SQL schemas, RLS policies (`pg_policies`), column privileges (`information_schema.column_privileges`), trigger functions, `SECURITY DEFINER` procedures (`pg_proc`), Edge Functions, and Dart source code.

> **Verification Limit Notice:** The CLI database connection operates as `postgres` where `auth.uid()` is `NULL`. RLS policies and `SECURITY DEFINER` functions were verified **structurally** by evaluating AST/source definitions and column grants. Runtime session behavior for signed-in JWT users was evaluated analytically based on structural guarantees.

---

## 🚨 CRITICAL SEVERITY FINDINGS

### 1. `tip_management.dart`: Hours-Weighted Tip Allocation Miscalculated to Zero (`_hoursBetweenTimestamps` Calling `_hoursBetween`)
- **Location:** `lib/features/tips/tip_management.dart:486-495`
- **Verification Method:** Structural code analysis of `_hoursBetweenTimestamps` in `tip_management.dart`.
- **Concrete Impact:** In `tip_management.dart`, `_hoursBetweenTimestamps` is called to compute worked hours from `time_entries` ISO 8601 timestamps (e.g., `2026-07-28T17:00:00Z`). On line 491, if `DateTime.tryParse` fails or falls through, it calls `_hoursBetween(clockIn, clockOut)`. `_hoursBetween` splits by `:` expecting `HH:MM` time strings. Passing an ISO string to `_hoursBetween` causes `int.tryParse("2026-07-28T17")` to return `null`, evaluating worked hours to **0.0**. Any tip pool split weighted by hours receives 0.0 worked hours for employees, leading to divide-by-zero errors or zeroed tip payouts.
- **Proposed Fix:** Replace local `_hoursBetween` fallback in `tip_management.dart` with standard `hoursBetweenTimestamps` from `lib/core/shift_time.dart`.
- **v1 Risk:** None (Apex v1 does not contain `tip_management.dart`).

---

### 2. `apex_remove_member`: Managers Can Demote/Remove Owners (Missing Target Role Check)
- **Location:** PostgreSQL Function `public.apex_remove_member(p_user uuid, p_org uuid)`
- **Verification Method:** Structural SQL definition analysis via `pg_proc`.
- **Concrete Impact:** `apex_remove_member` checks `has_role(p_org, 'manager')` for the caller. It checks `p_user = auth.uid()` (cannot remove self) and `v_other_owners = 0` (cannot remove the last owner). However, if a venue has 2 Owners (Owner A and Owner B), any non-owner Manager can call `apex_remove_member(Owner B)` to remove Owner B from the venue! A Manager should never be able to remove an Owner.
- **Proposed Fix:** Add a role hierarchy guard inside `apex_remove_member`:
  ```sql
  IF lower(v_target_role) = 'owner' AND lower(v_caller_role) <> 'owner' THEN
    RAISE EXCEPTION 'owner_required';
  END IF;
  ```
- **v1 Risk:** Low. Ensure any v1 staff deletion calls use `apex_remove_member` safely.

---

### 3. `apex_set_menu_item_available`: Any Staff Member Can Toggle Menu Availability (Missing Role Gate)
- **Location:** PostgreSQL Function `public.apex_set_menu_item_available(p_item_id uuid, p_available boolean)`
- **Verification Method:** Structural SQL definition analysis via `pg_proc`.
- **Concrete Impact:** The RPC checks `is_member(v_org)`, but does **not** check `has_role(v_org, 'manager')`. Any authenticated employee (e.g. Line Cook, Busser, Server) can invoke `apex_set_menu_item_available` via Supabase client and disable or enable any menu item, bypassing manager inventory control.
- **Proposed Fix:** Change authorization check from `IF NOT is_member(v_org)` to `IF NOT has_role(v_org, 'manager')`.
- **v1 Risk:** None (Apex v1 does not use `apex_set_menu_item_available`).

---

## ⚠️ HIGH SEVERITY FINDINGS

### 4. Multi-Venue Cross-Read Leakage via `is_member` RLS Policies
- **Location:** RLS Policies on `restaurants`, `restaurant_settings`, `restaurant_locations`, `tip_pools`, `tip_allocations`, `server_tips`, `shift_notes`, `online_orders`
- **Verification Method:** Structural query of `pg_policies` `qual` expressions.
- **Concrete Impact:** Tables `shifts`, `swaps`, `time_entries`, and `time_off_requests` use `organization_id = apex_current_org_id()`, restricting reads strictly to the active venue. However, `online_orders`, `tip_pools`, `tip_allocations`, `server_tips`, `shift_notes`, `restaurants`, and `restaurant_settings` use `is_member(organization_id)`. If a user is an active member of Venue A AND Venue B, they can query Venue B's live `online_orders`, `tip_pools`, `shift_notes`, and revenue settings while their `active_org_id` is set to Venue A.
- **Proposed Fix:** Standardize operational table RLS policies to check `(organization_id = apex_current_org_id() OR is_super_admin())`.
- **v1 Risk:** Medium — Single-venue v1 users are unaffected; multi-venue users in v1 will be scoped to their active venue.

---

### 5. Legacy Profile Field Dependency in RPC Authorization Checks (`apex_set_role`, `apex_set_org_module`, `apex_set_hourly_rate`)
- **Location:** PostgreSQL Functions `apex_set_role`, `apex_set_org_module`, `apex_set_hourly_rate`
- **Verification Method:** Structural SQL definition analysis via `pg_proc`.
- **Concrete Impact:** `apex_set_role` and `apex_set_org_module` inspect `profiles.organization_id` and `profiles.role` for the caller instead of `apex_current_org_id()` and `has_role(org, 'owner')`. If an owner's active venue is managed via `organization_members` (or if `profiles.organization_id` is null for a multi-venue user), these RPCs fail with `owner_required` or target the wrong organization.
- **Proposed Fix:** Update RPC caller authority checks to use `apex_current_org_id()` and `has_role(apex_current_org_id(), 'owner')`.
- **v1 Risk:** Low — Maintains v1 backward compatibility via `has_role` fallback logic.

---

### 6. Unbounded Organization Creation via `organizations` RLS Policy (`INSERT WITH CHECK true`)
- **Location:** RLS Policy `org insert authenticated` on `public.organizations`
- **Verification Method:** Structural query of `pg_policies` on `organizations`.
- **Concrete Impact:** Any authenticated user can perform direct `INSERT` queries on `organizations` table without restriction. While `apex_create_organization` RPC exists for creation, table-level RLS `WITH CHECK (true)` allows malicious clients to flood the database with empty organization rows.
- **Proposed Fix:** Update `organizations` INSERT policy to `WITH CHECK (is_super_admin())` or require `auth.uid() IS NOT NULL AND NOT EXISTS (SELECT 1 FROM organization_members WHERE user_id = auth.uid() AND left_at IS NULL)`. Because Apex v1 calls `apex_create_organization` (SECURITY DEFINER), tightening table-level INSERT RLS will **not** break Apex v1.
- **v1 Risk:** Zero risk to v1 (v1 uses `apex_create_organization` RPC which bypasses RLS).

---

### 7. Super-Admin Email Trigger Assignment Without Email Confirmation Check
- **Location:** Trigger function `apex_handle_new_user()` on `auth.users`
- **Verification Method:** Structural analysis of `apex_handle_new_user()` source code.
- **Concrete Impact:** `apex_handle_new_user()` checks `v_email IN ('nicholaswittle@wisensellc.com', 'nicholaswittle@gmail.com')` and sets `is_super_admin = true`. If email confirmation is disabled in Supabase Auth, any user signing up with those unverified email addresses immediately gains `is_super_admin` fleet access.
- **Proposed Fix:** Require `new.email_confirmed_at IS NOT NULL` before assigning `is_super_admin = true` in `apex_handle_new_user()`.
- **v1 Risk:** None.

---

## 🟡 MEDIUM SEVERITY FINDINGS

### 8. `Future.wait` Index Fragility Across Key Application Screens
- **Location:** Dart Files: `capacity_screen.dart:162`, `labor_cost_dashboard.dart:113`, `labor_vs_revenue_dashboard.dart:121`, `staff_console_screen.dart:118`, `tip_management.dart:98`, `call_out_screen.dart:94`
- **Verification Method:** Structural search and indexing review in Dart codebase.
- **Concrete Impact:** `Future.wait` queries load datasets into a positional `List<dynamic>` (`results[0]`, `results[1]`, `results[2]`). If a query is inserted or reordered mid-list, positional accesses shift silently at runtime without compiler or `flutter analyze` errors.
- **Proposed Fix:** Use Dart 3 record destructuring: `final (shifts, entries, members) = await (fetchShifts(), fetchEntries(), fetchMembers()).wait;`.
- **v1 Risk:** None.

---

### 9. Silent Error Swallowing in Critical Flow `catch {}` Blocks
- **Location:** `menu_screen.dart:98, 107`, `staff_console_screen.dart:185, 221`, `cart_screen.dart:237`, `notification_bell.dart:43`
- **Verification Method:** Code search for `catch (_) {}` in `lib/`.
- **Concrete Impact:** Runtime exceptions (such as Stripe return parsing failures, KDS ticket updates, or thermal print errors) are swallowed silently, making debugging in production difficult.
- **Proposed Fix:** Add structured logging (`debugPrint('...')`) or user-facing notification snacks.
- **v1 Risk:** None.

---

### 10. Fallback Membership Resolution Edge-Case (`left_at` Handling)
- **Location:** PostgreSQL Functions `apex_current_org_id()`, `is_member()`, `has_role()`
- **Verification Method:** Structural SQL definition analysis.
- **Concrete Impact:** When a user is removed from all venues (all `organization_members` rows have `left_at IS NOT NULL`), `exists (select 1 from organization_members where user_id = auth.uid())` evaluates to `true`. This forces evaluation down the `organization_members` branch (returning `NULL` / `false`), correctly preventing fallback to `profiles.organization_id`. However, if `profiles.organization_id` was not cleared during manual database edits, legacy code querying `profiles` directly could read stale venue IDs.
- **Proposed Fix:** Ensure all venue resolution relies strictly on `organization_members` and `apex_current_org_id()`.
- **v1 Risk:** Low.

---

## 🔵 LOW SEVERITY FINDINGS

### 11. Column Grants Maintenance Risk on `profiles` Table
- **Location:** Table `public.profiles` Column Privileges
- **Verification Method:** Database query of `information_schema.column_privileges`.
- **Concrete Impact:** Table `profiles` uses column-level SELECT grants (`hourly_rate` returns `false` for `has_column_privilege`). Any future column added to `profiles` without an explicit `GRANT SELECT (new_col) TO authenticated;` will be unreadable by the client app, causing silent nulls or PostgREST errors.
- **Proposed Fix:** Maintain a migration checklist rule requiring explicit column grants on `profiles`.
- **v1 Risk:** Low.

---

### 12. Client-Side Entitlement & Role Gating Without Server RPC Equivalent
- **Location:** `lib/app.dart`, `employee_dashboard.dart`, `menu_editor_screen.dart`
- **Verification Method:** Code comparison between client navigation guards and database RLS.
- **Concrete Impact:** Navigation buttons (e.g. Menu Editor, Labor Cost) are hidden client-side based on `_role.canManage` or `Entitlements`. While table RLS protects sensitive writes, UI-only visibility guards allow curious users with direct DB access to query public tables if RLS is permissive.
- **Proposed Fix:** Ensure every manager-only action is backed by an RPC with `has_role(org, 'manager')`.
- **v1 Risk:** Low.

---

## 🔍 Detailed Component Audit Answers

### A. Authorization & Security Definer Functions
- **RLS Policies & `WITH CHECK`:**
  - `profiles.update_self` uses `WITH CHECK ((id = auth.uid()) AND NOT (organization_id IS DISTINCT FROM apex_current_org_id()) AND NOT (role IS DISTINCT FROM apex_current_role()) AND NOT (is_super_admin IS DISTINCT FROM is_super_admin()) AND NOT (hourly_rate IS DISTINCT FROM apex_current_hourly_rate()))`. This correctly prevents self-privilege escalation.
  - `organizations.org insert authenticated` uses `WITH CHECK (true)` — flagged in **Finding #6**.
- **Admin Functions:**
  - All 13 `admin_*` functions (`admin_list_users`, `admin_set_tier`, `admin_toggle_module`, `admin_begin_view_as`, `admin_fleet_health`, etc.) explicitly check `is_super_admin()`.

### B. Membership Refactor Stragglers
- **`profiles.role` and `profiles.organization_id` Writers:**
  - `apex_grant_membership` handles `organization_members` synchronization.
  - `apex_profiles_membership_sync` trigger on `profiles` automatically invokes `apex_grant_membership` whenever `profiles.organization_id` or `role` is updated.
- **Multi-Venue `active_org_id`:**
  - `apex_set_active_venue(p_org)` verifies `organization_members.left_at IS NULL` for `p_org` before updating `profiles.active_org_id`.

### C. Column Grants & Pay Privacy
- **`hourly_rate` Privacy Verification:**
  - `SELECT has_column_privilege('authenticated', 'profiles', 'hourly_rate', 'SELECT');` returns `false`.
  - Pay rates are fetched strictly via `apex_my_hourly_rate()` and `apex_team_pay(p_org)` (gated on `has_role(p_org, 'manager')`).

### D. Money Paths & Stripe Connect
- **Stripe Edge Functions (`create-guest-payment`, `stripe-os-webhook`):**
  - **Amount Forgery:** `create-guest-payment` fetches `total_cents` directly from the `online_orders` DB row (computed server-side in `place_order` RPC). Amounts cannot be forged client-side.
  - **Webhook Signatures:** `stripe-os-webhook` validates `stripe-signature` using `stripe.webhooks.constructEventAsync(body, signature, webhookSecret)`.
  - **Amount Match Verification:** `stripe-os-webhook` verifies `session.amount_total === order.total_cents` before marking an order as `paid`.

### E. Secrets & Exposure
- **Service Role Key & Credentials:**
  - `SUPABASE_SERVICE_ROLE_KEY` is referenced only in Supabase Edge Functions (`Deno.env.get`). It is absent from client Dart code, `--dart-define`, and `build/web/main.dart.js`.
- **`ANTHROPIC_API_KEY`:**
  - Referenced only in Edge Functions (`parse-menu`, `parse-schedule`, `venue-briefing`). Not present in client bundles.
- **Public Demo (`apex-v2-demo.vercel.app`):**
  - In `lib/app.dart`, `DemoMode.enabled` forces `_isSuperAdmin = false`, preventing fleet admin console access.

### F. Known-Open Issues
- **`organizations` INSERT Policy:**
  - Apex v1 creates organizations via `ProfileService.createOrganization`, which calls the `apex_create_organization` RPC (`SECURITY DEFINER`). Tightening table-level `INSERT` RLS on `organizations` will **not** break Apex v1.

---

## ❓ What Could Not Be Verified (& Why)

1. **Signed-in User JWT RLS Execution at Runtime:**
   - **Reason:** Database CLI commands connect as `postgres` superuser where `auth.uid()` is `NULL`. Direct runtime execution of RLS queries as an authenticated JWT user could not be executed via CLI. All policies were verified **structurally** via PostgreSQL metadata catalogs (`pg_policies`, `information_schema.column_privileges`, `pg_proc`).
2. **Live Stripe Webhook Receipt:**
   - **Reason:** Webhook signature verification and handler logic were verified **structurally** in `stripe-os-webhook/index.ts`, but live webhooks were not triggered during audit.
3. **Supabase Auth Email Confirmation Project Setting:**
   - **Reason:** Project-level Auth settings (whether email confirmation is enabled or disabled) reside in the Supabase Cloud Management API / Dashboard, which is not queryable via raw SQL CLI.
