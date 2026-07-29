---
type: fix
title: "Apex v2 — Staff Console Paid Online vs. Pay at Pickup Ticket Fix"
tags: [fix, staff-console, stripe, payment-status, apex-v2, thermal-print]
date: 2026-07-28
status: active
target_repo: "github.com/nicholaswittle/apex_v2"
target_file: "lib/features/ordering/staff_console_screen.dart"
---

# 🛠️ Apex v2 — Staff Console Paid Online vs. Pay at Pickup Ticket Fix

> **Issue Discovered:** When a guest paid for food online via Stripe (`pay_now`), the backend correctly set `payment_status = 'paid'`, BUT the Staff Console modal and printed thermal receipt displayed **"pay at pickup"** and **"Collect on Square at counter"**, misleading staff into asking for payment again at pickup!

---

## 1. Root Cause Identification

In `lib/features/ordering/staff_console_screen.dart`:

1. **Accept Dialog (Line 351):**  
   The confirmation text was hardcoded:  
   `Text('${order.customerName} · ${formatCents(order.totalCents)} · pay at pickup')`  
   This printed `"pay at pickup"` even when `order.paymentStatus == 'paid'`.
2. **Printed Thermal Ticket Footer (Lines 597 & 635):**  
   The footer of every printed receipt was hardcoded:  
   `<div>Collect on Square at counter.</div>`  
   Even when the header badge said `PAID ONLINE $18.50`, the ticket footer instructed kitchen/counter staff to collect payment on Square.

---

## 2. Technical Fix Specifications (Dispatched via TASK-005)

### A. Dynamic Accept Modal Subtitle (Line 351)
```dart
Text(
  '${order.customerName} · ${formatCents(order.totalCents)} · '
  '${order.paymentStatus == "paid" ? "PAID ONLINE" : "pay at pickup"}',
  style: Theme.of(ctx).textTheme.bodyMedium,
)
```

### B. Dynamic Thermal Ticket Footer (Lines 587 & 635)
```html
<div><b>${order.paymentStatus == 'paid' ? 'DO NOT COLLECT — Paid online via Stripe.' : 'Collect on Square at counter.'}</b></div>
```

---

## 📄 File Location & Sync
Saved to Obsidian Vault static reference:  
`C:\Users\nikwi\Notes\wisense\projects\APEX_V2_STAFF_CONSOLE_PAID_ONLINE_FIX_2026-07-28.md`  
Committed and pushed to GitHub `https://github.com/nicholaswittle/wisenseobsidian.git`.
