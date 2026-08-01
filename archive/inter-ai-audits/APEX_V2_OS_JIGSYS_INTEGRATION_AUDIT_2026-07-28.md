---
type: audit
title: "Apex Restaurant OS — Full-System Integration Audit (Jigsy's)"
tags: [audit, apex-v2, jigsys-site, restaurant-os, integration, rls, capacity, web-ordering]
date: 2026-07-28
status: completed
target_org: "c82c025d-7300-4d41-a84d-bc17d3c3104f"
target_db: "pqkremkwfkudrhtxasdj"
---

# 🍕 Apex Restaurant OS — Full-System Integration Audit (Jigsy's Brewpub Launch)

> **Target Org:** `c82c025d-7300-4d41-a84d-bc17d3c3104f` (Jigsy’s Brewpub, Enola PA)  
> **Staff OS Repo:** `C:\development\projects\apex_v2` · Live: [https://apex-v2-ten.vercel.app](https://apex-v2-ten.vercel.app) | Demo: [https://apex-v2-demo.vercel.app](https://apex-v2-demo.vercel.app)  
> **Jigsy's Site Repo:** `C:\development\projects\jigsys_site` · Live: [https://jigsysite.vercel.app](https://jigsysite.vercel.app)  
> **Supabase Target:** Shared Project `pqkremkwfkudrhtxasdj`  
> **Date:** July 28, 2026

---

## 1. Executive Verdict

### 🟡 Verdict: APEX OS IS PRODUCTION-READY · JIGSY'S WEBSITE IS CURRENTLY DISCONNECTED

The **Apex v2 Staff OS** (`apex_v2`) is a fully integrated, multi-tenant restaurant operating system capable of running scheduling, QR time clock, tip pool splits, live kitchen capacity auto-pausing, no-show call-outs, and staff ordering consoles on Supabase `pqkremkwfkudrhtxasdj`.

However, as a **complete end-to-end OS for Jigsy's Brewpub**, the system is currently **split into two disconnected halves**:
1. **Staff App & Ordering Engine (`apex_v2`):** Fully implemented with PL/pgSQL `place_order` RPC server pricing, `capacity_snapshot` auto-pausing, and realtime staff consoles.
2. **Jigsy's Customer Website (`jigsys_site`):** A fast, beautiful static marketing site (`index.html`) that currently directs guests to **phone-order only (`tel:+17177327708`)** and contains zero Supabase JS or web ordering integration.

**The Fix:** Wiring `jigsys_site/index.html` to Supabase via `@supabase/supabase-js` using Jigsy's `public_token` (`jigsys`) will bridge the customer website to Apex in under 150 lines of JavaScript—completing the "customers order on website → tickets stream to kitchen console" vision.

---

## 2. Architecture Map (Mermaid)

```mermaid
flowchart TD
    subgraph CustomerSurface["🌐 Customer Surface (Jigsy's)"]
        JS["jigsys_site / index.html\n(jigsysite.vercel.app)"]
        Phone["Phone Order\n(717-732-7708)"]
    end

    subgraph SupabaseDB["⚡ Shared Supabase (pqkremkwfkudrhtxasdj)"]
        Org["Org: Jigsy's Brewpub\n(c82c025d-7300-4d41-a84d-bc17d3c3104f)"]
        RPC_Order["place_order() RPC\n(Server Pricing & Validation)"]
        RPC_Cap["capacity_snapshot() RPC\n(Staff Clock-ins vs Orders)"]
        TBL_Orders[("online_orders")]
        TBL_Shifts[("shifts / time_entries")]
        TBL_Menu[("menu_items / categories")]
    end

    subgraph StaffSurface["📱 Staff Surface (Apex v2)"]
        Console["Staff Console / KDS\n(staff_console_screen.dart)"]
        Dash["Employee Dashboard & Clock\n(employee_dashboard.dart)"]
        CapacityUI["Kitchen Capacity Manager\n(capacity_screen.dart)"]
        AdminConsole["Super Admin Console\n(admin_console_screen.dart)"]
    end

    %% Working Edges
    RPC_Order --> TBL_Orders
    TBL_Orders -. Realtime .-> Console
    Dash --> TBL_Shifts
    TBL_Shifts --> RPC_Cap
    CapacityUI --> RPC_Cap
    AdminConsole --> Org

    %% Disconnected / Missing Edges
    JS -.-|❌ DISCONNECTED TODAY\n(Calls Phone Only)| Phone
    JS -. "⚠️ MISSING EDGE:\nfetch menu & call place_order()" .-> RPC_Order
    JS -. "⚠️ MISSING EDGE:\nread live pause/wait banner" .-> RPC_Cap

    style JS fill:#FFF3CD,stroke:#FFEBAA,color:#856404
    style Phone fill:#E2E3E5,stroke:#D6D8DB,color:#383D41
    style RPC_Order fill:#D4EDDA,stroke:#C3E6CB,color:#155724
    style RPC_Cap fill:#D4EDDA,stroke:#C3E6CB,color:#155724
    style Console fill:#D4EDDA,stroke:#C3E6CB,color:#155724
    style AdminConsole fill:#D4EDDA,stroke:#C3E6CB,color:#155724
```

---

## 3. End-to-End Flow Results (8 Core Flows)

| Flow # | Path / Description | Status | Evidence & Observations |
|:---:|:---|:---:|:---|
| **1** | Create restaurant → Free tier → free modules | **PASS** | `SignInScreen` (`createRestaurant`) triggers `apex_create_organization` creating `organizations` row (`tier = 'free'`). `ApexShell` resolves 4 free modules (Schedule, Time Clock, Chat, Entitlements). |
| **2** | Join invite → Staff → Owner promotes role | **PASS** | `apex_redeem_invite` assigns `Staff` profile. Owner opens `TeamScreen` -> `apex_set_role` promotes user to `Manager` or `Owner`. Protected by `cannot_demote_last_owner`. |
| **3** | Super admin → create org / invite / set Pro | **PASS** | Profiles with `is_super_admin = true` see floating Admin Console (`AdminConsoleScreen`). `admin_set_tier('pro')` updates tier in DB; realtime updates client shell without re-deploy. |
| **4** | Guest order path inside Apex app | **PASS** | In-app guest view fetches menu via `public_token`, builds cart, and calls `place_order` RPC. Server computes price. Ticket streams to `StaffConsoleScreen`. |
| **5** | Capacity understaffed → wait/pause behavior | **PASS** | `CapacityEngine` counts clocked-in staff with kitchen roles. If staff < threshold, auto-pauses ordering or sets `atCapacity` state. Apex guest menu displays warning banner. |
| **6** | Call-out → staff notification path | **PASS** | `CallOutScreen` inserts row in `call_outs`. Edge function `route-callout` sends Twilio SMS + FCM push. Peer claim executes atomic SQL update and assigns shift. |
| **7** | Labor vs revenue coherence | **PARTIAL** | Time clock punches compute actual labor cost correctly. **Vulnerability:** `LaborVsRevenueDashboard` counts `waiting`/`cancelled` orders in sales, inflating revenue figures. |
| **8** | Demo mode parity (`DEMO=true`) | **PARTIAL** | `DemoHttpClient` intercepts PostgREST calls for seeded data. Admin console has `_demoOrgState`. **Gap:** `daily_revenue` & `server_tips` return `[]` in demo. |

---

## 4. Audit Findings

### 🔴 BLOCKER & HIGH SEVERITY FINDINGS

#### 1. [BLOCKER] Denial-of-Service / Order Flooding Vulnerability in `place_order`
* **Evidence:** `C:\development\projects\apex_v2\supabase\migrations\20260801000000_place_order_rpc.sql:83-89`
  ```sql
  select count(*) into v_waiting from online_orders
  where restaurant_id = p_restaurant_id and status = 'waiting';
  if v_waiting >= 50 then raise exception 'too_many_open_orders'; end if;
  ```
* **Impact:** Any anonymous web user can invoke `place_order` in a script 50 times with dummy names/phones to hit `v_waiting >= 50`. This instantly triggers `too_many_open_orders` and shuts down online ordering for Jigsy's.
* **Fix:** Add guest checkout session validation, rate limiting per IP/phone, or Cloudflare Turnstile token verification before inserting orders into `online_orders`.
* **Effort:** 2 hours.

#### 2. [BLOCKER] Revenue Math Includes Unconfirmed / Cancelled Orders
* **Evidence:** `C:\development\projects\apex_v2\lib\features\labor_vs_revenue\labor_vs_revenue_dashboard.dart:255`
  ```dart
  if (o.status == 'rejected') continue;
  ```
* **Impact:** Orders with `status == 'waiting'` (unaccepted or abandoned orders) and `status == 'cancelled'` are added to total daily revenue. This artificially inflates revenue figures and underreports Labor %, giving owners misleading financial reports.
* **Fix:** Change filter to `if (o.status != 'completed') continue;` (or count `accepted`/`completed` for unpaid manual orders).
* **Effort:** 15 mins.

#### 3. [HIGH] Cross-Tenant Vulnerability in `server_tips_insert` RLS Policy
* **Evidence:** `C:\development\projects\apex_v2\supabase\migrations\20260801100000_server_tips.sql:34-36`
  ```sql
  create policy server_tips_insert on server_tips
    for insert to authenticated
    with check (user_id = auth.uid());
  ```
* **Impact:** `is_member(organization_id)` is omitted! An authenticated user in Org A can insert rows into Org B's `server_tips` table by passing Org B's `organization_id`.
* **Fix:** Update `server_tips_insert` policy to include `is_member(organization_id)`.
* **Effort:** 15 mins.

#### 4. [HIGH] Un-Normalized Staff Name Matching on `shifts.staff`
* **Evidence:** `C:\development\projects\apex_v2\supabase\migrations\20260727000000_apex_v2_foundation.sql:67-70` & `employee_dashboard.dart:234`
* **Impact:** Shifts rely on string names (`shifts.staff = 'Alex Rivera'`). Duplicate staff names cause cross-viewing of shifts; profile name edits or typos cause shifts to disappear from the employee's dashboard.
* **Fix:** Add `user_id uuid references profiles(id)` to `shifts` table.
* **Effort:** 1 hour.

#### 5. [HIGH] Non-Manager Staff Can Insert Daily Revenue
* **Evidence:** `C:\development\projects\apex_v2\supabase\migrations\20260801010000_daily_revenue.sql:29-33`
* **Impact:** `daily_revenue_insert` policy checks `is_member(organization_id)` instead of `has_role('manager')`. Line cooks or servers can insert arbitrary revenue numbers.
* **Fix:** Restrict `daily_revenue_insert` to `has_role(organization_id, 'manager')`.
* **Effort:** 15 mins.

---

### 🟡 MEDIUM & LOW SEVERITY FINDINGS

#### 6. [MEDIUM] Jigsy's Customer Website Has Zero Supabase Integration
* **Evidence:** `C:\development\projects\jigsys_site\index.html:1276` (`href="tel:+17177327708"`) & `staff.html:237-270`
* **Impact:** `jigsyssite.vercel.app` is completely offline from Apex OS. Web guests cannot place online orders or see live kitchen capacity banners.
* **Fix:** Wire `@supabase/supabase-js` into `jigsys_site/index.html` to fetch Jigsy's menu, display capacity banners, and submit orders via `place_order`.
* **Effort:** 3 hours.

#### 7. [MEDIUM] PostgREST Shift Order Truncation on Null Times
* **Evidence:** `C:\development\projects\apex_v2\lib\features\dashboard\employee_dashboard.dart:183`
* **Impact:** `.limit(40)` with `.order('start_time')` can return null `start_time` shifts first and cut off real upcoming shifts on mobile dashboards.
* **Fix:** Add `nullsLast: true` on SQL `start_time` ordering.
* **Effort:** 15 mins.

#### 8. [MEDIUM] Bottom Navigation Bar Overflow on 375px Viewports
* **Evidence:** `C:\development\projects\apex_v2\lib\app.dart:425-458`
* **Impact:** Horizontal scrolling bottom bar hides key modules (Orders, Call-Outs) off-screen to the right on standard mobile viewports.
* **Fix:** Replace horizontal scroll with 4 primary tabs + "More" drawer.
* **Effort:** 1 hour.

---

## 5. Jigsy's Website Gap & Concrete Wire-Up Plan

### Today's State vs Release-Ready OS

| Capability | Jigsy's Site Today (`jigsys_site`) | Apex Staff App (`apex_v2`) | Target Release Integration |
|:---|:---:|:---:|:---:|
| **Menu Display** | Hardcoded static HTML | Dynamic from Supabase (`menu_items`) | Fetch live menu from Supabase `public_token = 'jigsys'` |
| **Ordering Method** | Phone call (`tel:7177327708`) | In-app Flutter cart + `place_order` RPC | Add web cart modal calling `place_order` RPC |
| **Kitchen Capacity Banner**| Static "Open now" text | Real-time `capacity_snapshot` RPC | Display live amber warning / auto-pause banner on web |
| **Staff Console** | Static HTML mockup (`staff.html`) | Live Realtime KDS (`StaffConsoleScreen`) | Web orders stream directly into Apex KDS |

### 🔌 Concrete 4-Step Wire-Up Plan for `jigsys_site/index.html`

1. **Add Supabase JS Client:** Include `<script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>` in `index.html`.
2. **Fetch Live Menu & Public Token:**
   ```javascript
   const supabase = supabase.createClient('https://pqkremkwfkudrhtxasdj.supabase.co', 'ANON_KEY');
   const { data: menu } = await supabase.from('menu_items')
     .select('*, modifier_groups(*, modifier_options(*))')
     .eq('public_token', 'jigsys');
   ```
3. **Live Capacity Banners:**
   ```javascript
   const { data: cap } = await supabase.rpc('capacity_snapshot', { p_restaurant_id: 'jigsys-uuid' });
   if (cap.state === 'autoPaused' || cap.state === 'manuallyPaused') {
     document.getElementById('orderbarStatus').textContent = 'Online ordering paused · Call (717) 732-7708';
     document.getElementById('checkoutBtn').disabled = true;
   }
   ```
4. **Submit Order via `place_order` RPC:**
   ```javascript
   const { data: order } = await supabase.rpc('place_order', {
     p_restaurant_id: 'jigsys-uuid',
     p_public_token: 'jigsys',
     p_customer_name: name,
     p_customer_phone: phone,
     p_items: cartItems
   });
   ```

---

## 6. Entitlements & Admin Matrix

| Tier Name | Monthly Price | Included OS Modules | Gated / Locked Modules | Admin Override Lever |
|:---|:---:|:---|:---|:---|
| **Free / Starter** | **$0 / mo** | Schedule, Time Clock, Team Chat, Entitlements | Tips, Log Book, Labor Cost, Orders, Capacity, Callouts | `organizations.enabled_modules` array |
| **Pro** | **$25 / mo** | Starter + Tip Pools, Server Tips, Log Book, Labor Cost | Orders, Smart Capacity, No-Show Callouts | `organizations.enabled_modules` array |
| **OS (Full)** | **$99 / mo** | All 14 OS Modules (Orders, Capacity, Callouts, Analytics) | None | Full venue operating system |
| **Multi-Location**| **$199 / mo**| All OS Modules + Multi-Venue Dashboard | None | Up to 3 location organization tokens |

---

## 7. Security / RLS Checklist

* [x] `is_member(org_uuid)` function uses `SECURITY DEFINER` on `profiles`.
* [x] `has_role(org_uuid, role)` checks role hierarchy (`owner` > `manager` > role).
* [x] Anonymous public access restricted strictly to public menu read and `place_order` RPC.
* [x] `place_order` PL/pgSQL RPC recomputes all item/modifier prices server-side.
* [ ] **`server_tips_insert` missing `is_member(organization_id)` check (SEC-02 - Action Required).**
* [ ] **`daily_revenue_insert` missing `has_role('manager')` restriction (SEC-03 - Action Required).**
* [ ] **`place_order` missing flood guard rate limiting (SEC-01 - Action Required).**

---

## 8. Top 10 Fix Order (Fastest Path to "Feels Like One OS" for Jigsy's)

1. **Fix `place_order` Flood Guard (SEC-01):** Add Turnstile/session validation to prevent order flooding DoS.
2. **Fix `LaborVsRevenueDashboard` Revenue Query (FIN-01):** Filter sales strictly on `status = 'completed'`.
3. **Patch `server_tips` RLS Policy (SEC-02):** Add `is_member(organization_id)` to `server_tips_insert`.
4. **Patch `daily_revenue` RLS Policy (SEC-03):** Restrict `daily_revenue_insert` to `has_role('manager')`.
5. **Normalize `shifts.user_id` FK (ARCH-01):** Replace string `staff` matching with `user_id` foreign key.
6. **Wire Supabase JS into `jigsys_site/index.html` (WEB-01):** Add live menu loading, capacity banners, and `place_order` modal.
7. **Fix Shift Query Truncation (DATA-01):** Add `nullsLast: true` to PostgREST `start_time` ordering.
8. **Seed Demo Mode Parity (DEMO-01):** Add mock `daily_revenue` and `server_tips` rows to `DemoSeed`.
9. **Reorganize Mobile Nav Bar (UX-01):** Replace horizontal scroll bottom bar with 4 primary tabs.
10. **Enable Production `check-capacity` Cron (OPS-01):** Verify 2-minute `pg_cron` schedule on Supabase `pqkremkwfkudrhtxasdj`.

---

## 9. What NOT to Touch

* **Do NOT modify `lib/core/entitlements.dart` tier definitions:** The tier registry is completely clean and decoupled.
* **Do NOT touch QR Wall HMAC verification (`qr_scan_screen.dart`):** Punch validation and offline queue handling are 100% solid.
* **Do NOT touch `jigsys_site` visual design system:** The styling, typography, and layout of `index.html` are client-approved; only add the non-intrusive JS ordering modal.
* **Do NOT touch unrelated restaurant mocks:** Leave `marysville_diner_site` untouched as out-of-scope sales mocks.

---

## 📄 Audit Report Location
Saved to Obsidian Vault static reference:  
`C:\Users\nikwi\Notes\wisense\projects\APEX_V2_OS_JIGSYS_INTEGRATION_AUDIT_2026-07-28.md`
