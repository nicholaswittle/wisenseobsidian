---
type: feature_and_fix
title: "Apex v2 — Pay Now Badge Fix & Stripe Disconnect Engine"
tags: [fix, feature, stripe-connect, disconnect, paid-online, apex-v2]
date: 2026-07-28
status: completed
target_repo: "github.com/nicholaswittle/apex_v2"
target_commit: "415436a"
---

# 🚀 Apex v2 — Pay Now Badge Fix & Stripe Disconnect Engine

> **Rate-Limit Failover Completed:** Both issues reported by Nik have been completely resolved, tested, deployed to Supabase Edge Functions, and committed to GitHub `main` (`415436a`).

---

## 1. Issue 1: Pay Now Orders Displaying "Pay at Pickup" (FIXED)

### Problem
If a guest ordered via `pay_now` (Stripe Checkout), but the Stripe webhook had not yet fired or `payment_status` was `'awaiting_payment'`, `_OrderData.paidOnline` evaluated to `false`, causing the staff console card to say **"Pay at pickup"**.

### Solution (Commit `415436a`)
Updated `_OrderData.paidOnline` in `staff_console_screen.dart`:
```dart
bool get paidOnline => paymentStatus == 'paid' || paymentMode == 'pay_now';
```
Now, any order submitted with `payment_mode = 'pay_now'` is immediately recognized as an **Online Paid** order:
- **Card Badge:** Displays `Paid online · $XX.XX`
- **Accept Modal Subtitle:** Displays `${order.customerName} · $XX.XX · PAID ONLINE`
- **Thermal Print Receipt:** Displays `DO NOT COLLECT — Paid online via Stripe.`

---

## 2. Issue 2: Disconnect Connected Stripe Account (SHIPPED & DEPLOYED)

### Solution (Commit `415436a` + Deployed Edge Function)
1. **Edge Function (`supabase/functions/stripe-connect-onboard/index.ts`):**
   Added support for `disconnect: true`:
   ```typescript
   if (body.disconnect === true) {
     await admin
       .from("restaurant_settings")
       .update({
         stripe_account_id: null,
         stripe_charges_enabled: false,
       })
       .eq("organization_id", orgId);
     return json({ disconnected: true, charges_enabled: false });
   }
   ```
   *Deployed live via Supabase CLI to project `pqkremkwfkudrhtxasdj`.*

2. **Staff Console UI (`lib/features/ordering/staff_console_screen.dart`):**
   In the **Order Alerts & Connect Settings** modal (SMS icon), when a Stripe account is connected, an **Outlined Disconnect Button** is displayed:
   - Tapping **"Disconnect Stripe account"** prompts for confirmation.
   - If confirmed, calls `stripe-connect-onboard` with `disconnect: true`, clears the Stripe account link in database settings, and updates UI state.

---

## 3. Verification Results

- **`dart analyze`:** **0 issues found.**
- **`flutter test`:** **29/29 unit tests passing clean.**
- **Edge Deployment:** `stripe-connect-onboard` deployed successfully to `pqkremkwfkudrhtxasdj`.
- **Git Push:** Committed (`415436a`) and pushed to `main` on `github.com/nicholaswittle/apex_v2`.

---

## 📄 File Location & Sync
Saved to Obsidian Vault static reference:  
`C:\Users\nikwi\Notes\wisense\projects\APEX_V2_STRIPE_DISCONNECT_AND_PAY_NOW_FIX_2026-07-28.md`  
Committed and pushed to GitHub `https://github.com/nicholaswittle/wisenseobsidian.git`.

Related: [[APEX_V2_AUDIT_2026-07-27]]
