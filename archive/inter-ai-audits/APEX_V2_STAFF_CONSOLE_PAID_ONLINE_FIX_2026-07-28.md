---
type: fix
title: "Apex v2 — Staff Console Paid Online vs. Pay at Pickup Ticket Fix"
tags: [fix, staff-console, stripe, payment-status, apex-v2, thermal-print]
date: 2026-07-28
status: completed
target_repo: "github.com/nicholaswittle/apex_v2"
target_commit: "6f80167"
target_file: "lib/features/ordering/staff_console_screen.dart"
---

# 🛠️ Apex v2 — Staff Console Paid Online vs. Pay at Pickup Ticket Fix

> **Issue Discovered:** When a guest paid for food online via Stripe (`pay_now`), the backend correctly set `payment_status = 'paid'`, BUT the Staff Console modal and printed thermal receipt displayed **"pay at pickup"** and **"Collect on Square at counter"**, misleading staff into asking for payment again at pickup!

---

## 1. Root Cause Identification & Fix (Commit `6f80167`)

In `lib/features/ordering/staff_console_screen.dart`:

1. **Accept Dialog (Line 351):**  
   *Old:* Hardcoded `'pay at pickup'`.  
   *Fixed:* Dynamic text `${order.paymentStatus == 'paid' ? 'PAID ONLINE' : 'pay at pickup'}`.
2. **Printed Thermal Receipt Footer (Lines 597 & 635):**  
   *Old:* Hardcoded `Collect on Square at counter.`.  
   *Fixed:* Dynamic footer:
   * **Paid Orders:** `DO NOT COLLECT — Paid online via Stripe.`
   * **Unpaid Orders:** `Collect on Square at counter.`

---

## 2. Verification Results

- **`dart analyze`:** **0 issues found.**
- **`flutter test`:** **28/28 tests passing clean.**
- **Deployment:** Pushed to `apex_v2` `main` branch in commit `6f80167` (Live on [https://apex-v2-ten.vercel.app](https://apex-v2-ten.vercel.app)).

---

## 📄 File Location & Sync
Saved to Obsidian Vault static reference:  
`C:\Users\nikwi\Notes\wisense\projects\APEX_V2_STAFF_CONSOLE_PAID_ONLINE_FIX_2026-07-28.md`  
Committed and pushed to GitHub `https://github.com/nicholaswittle/wisenseobsidian.git`.

Related: [[APEX_V2_AUDIT_2026-07-27]]
