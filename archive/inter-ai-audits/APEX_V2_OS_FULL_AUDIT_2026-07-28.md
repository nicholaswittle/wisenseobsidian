---
type: audit
title: "Apex v2 — Full System Audit: WiSense Restaurant OS"
tags: [audit, apex-v2, restaurant-os, security, rls, money-integrity, pre-pilot]
date: 2026-07-28
status: completed
target_db: "pqkremkwfkudrhtxasdj"
---

# 🛡️ Apex v2 — Full System Audit: WiSense Restaurant OS (Pre-Pilot Gate)

> **Audit Target:** `C:\development\projects\apex_v2` · GitHub `github.com/nicholaswittle/apex_v2`  
> **Supabase Target:** Project `pqkremkwfkudrhtxasdj`  
> **Live Deployments:** Real: [https://apex-v2-ten.vercel.app](https://apex-v2-ten.vercel.app) | Demo: [https://apex-v2-demo.vercel.app](https://apex-v2-demo.vercel.app)  
> **Date:** July 28, 2026

---

## 1. Executive Verdict

### ⚠️ Verdict: CONDITIONAL PILOT READY (2 Blockers, 4 High Items Must Be Patched)

The WiSense Restaurant OS (Apex v2) is an exceptionally well-engineered, employee-first unified platform. The consolidation of **scheduling, time clock, tip pool auditing, online ordering (Jigsy menu port), labor-vs-revenue analytics, no-show call-outs, and smart kitchen capacity** onto a single Supabase backend is clean, performant, and architecturally sound.

However, **two blocking defects** and **four high-severity vulnerabilities** must be resolved before onboarding Jigsy's Brewpub or any commercial venue:

1. **[BLOCKER] Denial-of-Service / Order Flooding Vulnerability:** Anonymous users can flood `place_order` with 50 fake orders to hit the waiting cap (`v_waiting >= 50`), instantly locking out legitimate online guests.
2. **[BLOCKER] Labor-vs-Revenue Inflation Bug:** Unconfirmed/waiting/cancelled online orders are included in revenue calculations, artificially inflating sales and falsifying labor % metrics.
3. **[HIGH] Multi-Tenant Leak in `server_tips` RLS:** `server_tips_insert` policy is missing `is_member(organization_id)` validation, allowing cross-tenant row insertion.
4. **[HIGH] Shift Assignment via Un-Normalized Strings (`shifts.staff`):** Shifts store employee string names instead of `user_id`, causing shift invisibility on duplicate names or profile name edits.

Once these 6 items are patched, Apex v2 is **100% pilot-ready** for Jigsy's Brewpub.

---

## 2. Findings Table

| ID | Severity | Area | Evidence (File:Line) | Impact | Recommended Fix |
|:---|:---|:---|:---|:---|:---|
| **SEC-01** | **BLOCKER** | Security & RPC | `20260801000000_place_order_rpc.sql:83-89` | Any anonymous web user can submit 50 fake orders in seconds to hit `v_waiting >= 50`, causing complete Denial-of-Service for online ordering. | Implement rate-limiting, CAPTCHA/turnstile, or temporary guest checkout session validation before setting status to `waiting`. |
| **FIN-01** | **BLOCKER** | Money Integrity | `labor_vs_revenue_dashboard.dart:255` | `if (o.status == 'rejected') continue;` includes `waiting` and `cancelled` orders in total revenue, inflating sales and underreporting labor %. | Change filter to count only `status == 'completed'` (or `completed`/`accepted` in manual mode). |
| **SEC-02** | **HIGH** | Security & RLS | `20260801100000_server_tips.sql:34-36` | `server_tips_insert` policy checks `user_id = auth.uid()` but omits `is_member(organization_id)`, allowing cross-tenant tip data injection. | Update `server_tips_insert` policy to include `is_member(organization_id)`. |
| **ARCH-01** | **HIGH** | Architecture & Data | `20260727000000_apex_v2_foundation.sql:56-58` & `employee_dashboard.dart:234` | `shifts` table uses string `staff` name instead of `user_id` FK. Typos or duplicate names break schedule queries. | Add `user_id uuid references profiles(id)` to `shifts` table. |
| **SEC-03** | **HIGH** | Security & RLS | `20260801010000_daily_revenue.sql:29-33` | `daily_revenue_insert` allows any authenticated member (`is_member`) to insert daily sales, while `update` requires manager role. | Restrict `daily_revenue_insert` to `has_role(organization_id, 'manager')`. |
| **OPS-01** | **HIGH** | Launch & Cron | `20260801020000_check_capacity_cron.sql:17-31` | `check-capacity` edge function requires an active 2-minute `pg_cron` schedule on production Supabase (`pqkremkwfkudrhtxasdj`). | Enable and verify `apex-check-capacity` cron on Supabase production. |
| **FIN-02** | **MEDIUM** | Money Integrity | `labor_vs_revenue_dashboard.dart:268-275` | `manualByDay` supersedes online orders entirely. Mid-day POS entry ignores evening online orders. | Combine POS + online sales or add explicit UI toggle for POS-only vs Hybrid sales. |
| **FIN-03** | **MEDIUM** | Money Integrity | `tip_management.dart:311` | Integer division of tip pools can leave 1–2 unallocated cents stranded. | Allocate leftover rounding cents to the employee with the highest hours worked. |
| **DATA-01** | **MEDIUM** | Data & Queries | `employee_dashboard.dart:183` & `schedule_screen.dart:114` | PostgREST `.limit(40)` with NULL `start_time` rows can truncate upcoming shifts on phone dashboards. | Explicitly specify `nullsLast: true` on SQL `start_time` ordering. |
| **UX-01** | **MEDIUM** | Mobile UX | `lib/app.dart:425-458` | Horizontal scrolling module bar on 375px phone screens hides critical modules (Orders/Call-Outs) off-screen. | Reorganize module navigation into a structured 4-tab bottom bar + "More" drawer. |
| **DEMO-01** | **MEDIUM** | Demo Parity | `demo_backend.dart:1093-1096` | `daily_revenue` & `server_tips` return `[]` in Demo Mode (`DEMO=true`), causing blank audit charts in demos. | Seed mock sample rows for `daily_revenue` and `server_tips` in `DemoSeed`. |
| **SEC-04** | **LOW** | Security & RPC | `20260801000000_place_order_rpc.sql:95` | `v_qty` has no upper bound check, allowing integer multiplication overflow if huge quantities are sent. | Add max quantity validation (`v_qty <= 100`). |
| **ARCH-02** | **LOW** | Maintainability | `shift_time.dart` & `employee_dashboard.dart` | Date formatting and hour calculation logic duplicated across multiple screens. | Centralize date formatting into `lib/core/shift_time.dart`. |
| **UX-02** | **LOW** | Mobile UX | `assign_days_screen.dart` | Sticky bottom action bar overlaps last item when keyboard is active on small screens. | Add bottom list padding (`SizedBox(height: 80)`). |

---

## 3. Module Scorecard

| Module | Status | Core Capability | Assessment & Status Explanation |
|:---|:---:|:---|:---|
| **1. Employee Dashboard** | 🟡 Partial | Today/Next shift, clock-in, week earnings | **Works well**, but uses string name matching on `shifts.staff` and has heavy unthrottled realtime queries. |
| **2. Scheduling & Assign Days** | 🟢 Works | Month-grid builder, calendar export, guardrails | **Excellent**. Ported calendar grid works smoothly; guardrails and conflict detection hold. |
| **3. Time Clock & QR Wall** | 🟢 Works | Offline punch queue, QR verification, auto-sync | **Solid**. Offline queue flushes cleanly on re-connection; HMAC digest prevents QR tampering. |
| **4. Tip Management (Pools)** | 🟢 Works | Hours-weighted tip pool split & manager confirmation | **Works**. Prevents duplicate pools via unique constraint `tip_pools_org_date_key`. |
| **5. Server Tips (My Tips)** | 🟡 Partial | Self-reported daily tip log & audit compare | **Partial**. Functionality works, but RLS insert policy missing `is_member` check. |
| **6. Labor Cost Dashboard** | 🟢 Works | Real-time labor cost tracking by role & shift | **Works**. Calculates actual vs scheduled hours accurately against profile hourly rates. |
| **7. Labor vs Revenue** | 🔴 Broken | Labor % vs online orders + daily POS sales | **Broken revenue math**. Counts `waiting`/`cancelled` orders as sales, inflating revenue figures. |
| **8. Online Ordering (Console)**| 🟢 Works | Staff console, status updates (accept/reject/complete) | **Works**. Staff console updates status cleanly and triggers realtime UI refresh. |
| **9. Guest Menu & Cart** | 🟡 Partial | Public token menu, server-priced `place_order` | **Partial**. `place_order` works, but lacks DoS rate limiting against order flooding. |
| **10. Smart Capacity Engine** | 🟢 Works | Auto-pause ordering on low staff, wait banners | **Works**. `capacity_snapshot` RPC handles guest wait banners; cron handles auto-pause. |
| **11. No-Show Call-Out Engine**| 🟢 Works | One-tap shift callout, peer claiming, Twilio SMS | **Works**. Single open callout constraint holds; claiming updates shift assignment. |
| **12. Team Chat** | 🟢 Works | In-app messaging, pinned announcements | **Works**. Scoped by org; author/manager deletion rules enforced. |
| **13. Manager Log Book** | 🟢 Works | Shift notes, photo attachments, author editing | **Works**. Clean timeline view for shift handover notes. |
| **14. Entitlements & Tiers** | 🟢 Works | Tier gating (Free / Pro / OS / Multi) & overrides | **Excellent**. Clean module registry in `app.dart` with zero hardcoded tier checks. |

---

## 4. Security & RLS Matrix

| Table Name | Anon Access | Authenticated (Staff) | Manager / Owner | Assessment & Notes |
|:---|:---:|:---:|:---:|:---|
| `organizations` | None | SELECT (via `is_member`) | UPDATE (Tier / modules) | 🟢 Secure |
| `profiles` | None | SELECT (own org) | SELECT / UPDATE | 🟢 Secure (uses SECURITY DEFINER helpers) |
| `shifts` | None | SELECT (own org) | INSERT / UPDATE / DELETE | ⚠️ Functional, but relies on string `staff` |
| `time_entries` | None | SELECT (own) / INSERT | SELECT / UPDATE (all staff) | 🟢 Secure |
| `shift_notes` | None | SELECT / INSERT (own) | DELETE (any note) | 🟢 Secure |
| `tip_pools` | None | SELECT (own org) | INSERT / UPDATE / DELETE | 🟢 Secure |
| `tip_allocations` | None | SELECT (own lines) | SELECT / INSERT / DELETE (all) | 🟢 Secure |
| `server_tips` | None | SELECT / INSERT / UPDATE | SELECT (audit compare) | 🔴 **VULNERABLE:** Insert missing `is_member` |
| `daily_revenue` | None | SELECT | INSERT / UPDATE / DELETE | 🔴 **VULNERABLE:** Insert allows non-managers |
| `restaurants` | SELECT (public_token) | SELECT / UPDATE | FULL ACCESS | 🟢 Secure |
| `restaurant_settings` | SELECT (public) | SELECT | UPDATE (Pause / Capacity) | 🟢 Secure |
| `menu_items` | SELECT (public) | SELECT | INSERT / UPDATE / DELETE | 🟢 Secure |
| `online_orders` | INSERT (`place_order`) | SELECT / UPDATE status | FULL ACCESS | 🔴 **VULNERABLE:** DoS via 50 fake orders |
| `call_outs` | None | SELECT / INSERT / UPDATE | FULL ACCESS | 🟢 Secure |
| `capacity_events` | None | SELECT / INSERT | FULL ACCESS | 🟢 Secure |

---

## 5. Top 10 Fix Order (Action Plan for Next Engineering Session)

1. **Fix `place_order` DoS / Order Flooding (SEC-01):** Add checkout rate-limiting / session verification or Turnstile challenge before setting status to `waiting`.
2. **Fix `LaborVsRevenueDashboard` Revenue Math (FIN-01):** Update order revenue query to filter strictly on `status = 'completed'` (or `status in ('completed', 'accepted')`).
3. **Patch `server_tips` RLS Policy (SEC-02):** Add `and is_member(organization_id)` to `server_tips_insert` policy in SQL.
4. **Patch `daily_revenue` RLS Policy (SEC-03):** Change `daily_revenue_insert` policy to require `has_role(organization_id, 'manager')`.
5. **Normalize Shift User Assignment (ARCH-01):** Add `user_id uuid references profiles(id)` to `shifts` table and update `AssignDaysScreen` + `EmployeeDashboard`.
6. **Fix Revenue Supersede Logic (FIN-02):** Update `LaborVsRevenueDashboard` to merge POS daily revenue with evening online orders or provide explicit toggle.
7. **Fix PostgREST Shift Order Truncation (DATA-01):** Update shift queries to specify `nullsLast: true` on `start_time` ordering.
8. **Add Tip Pool Rounding Residual Logic (FIN-03):** Distribute leftover rounding cents to the employee with the highest hours worked.
9. **Seed Demo Backend Parity (DEMO-01):** Add mock `daily_revenue` and `server_tips` rows to `DemoSeed` in `demo_backend.dart`.
10. **Reorganize Mobile Navigation Bar (UX-01):** Replace horizontal scrolling bottom bar on 375px screens with a structured tabbed layout.

---

## 6. Out of Scope / Deferred Items

The following items are intentionally deferred for post-pilot iterations:
* **Square / Clover POS Live API Sync:** Currently supported via manual `daily_revenue` entry.
* **Geofenced Clock-In Enforcement:** Currently supported via QR Wall + HMAC digest.
* **Google / Apple App Store Packaging:** Web builds live on Vercel (`apex-v2-ten.vercel.app`).
* **Multi-Location Hierarchy (Tier 4):** Reserved for enterprise multi-venue expansion.

---

## 📄 File Location
Saved to Obsidian Vault static reference:  
`C:\Users\nikwi\Notes\wisense\projects\APEX_V2_OS_FULL_AUDIT_2026-07-28.md`
