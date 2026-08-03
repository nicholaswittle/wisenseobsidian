---
type: feature
title: "Apex v2 — Guest Stripe Payment Confirmation Dialog"
tags: [feature, stripe, checkout, payment-confirmation, apex-v2, UX]
date: 2026-07-28
status: completed
target_repo: "github.com/nicholaswittle/apex_v2"
target_commit: "6de8ac3"
---

# 🎉 Apex v2 — Guest Stripe Payment Confirmation Dialog

> **Rate-Limit Failover Completed:** Shipped and deployed live in commit `6de8ac3`.

---

## 1. What Was Built

When a guest finishes paying for their order on Stripe Checkout (`pay_now`), Stripe redirects their browser back to the web application with return URL parameters:
`https://apex-v2-ten.vercel.app/?token=PUBLIC_TOKEN&paid=1&code=PICKUP_CODE`

### The New UX Flow:
1. `MenuScreen` detects `paid=1` on page load.
2. An elevated, beautiful **Payment Confirmed!** modal pops up in front of the guest:
   * 🟢 **Icon:** Large green check circle (`Icons.check_circle_rounded`).
   * 💳 **Header:** **Payment Confirmed!**
   * 📜 **Message:** *"Your payment was received on Stripe. Show this code at pickup:"*
   * 🏷️ **Pickup Code Badge:** **`#8F2A`** (styled in a bold, primary container pill).
   * ⏱️ **Status Subtitle:** *"Status: Paid Online · Kitchen preparing now"*
   * 🔘 **Action Button:** `[ Back to Menu ]`

---

## 2. Technical Implementation Details

In `lib/features/ordering/menu_screen.dart`:
- `_checkedPaymentReturn` flag prevents duplicate dialog popups during realtime stream updates.
- `_checkPaymentReturn()` parses `Uri.base.queryParameters['paid'] == '1'` and `Uri.base.queryParameters['code']`.
- Invokes `_showPaymentSuccessDialog(code)` via `WidgetsBinding.instance.addPostFrameCallback`.

---

## 3. Verification & Deployment

- **`dart analyze`:** **0 issues found.**
- **`flutter test`:** **29/29 unit tests passing clean.**
- **Git Push:** Committed (`6de8ac3`) and pushed to `main` on `github.com/nicholaswittle/apex_v2`.
- **Live Web App:** Auto-deploying on Vercel: [https://apex-v2-ten.vercel.app](https://apex-v2-ten.vercel.app).

---

## 📄 File Location & Sync
Saved to Obsidian Vault static reference:  
`C:\Users\nikwi\Notes\wisense\projects\APEX_V2_STRIPE_PAYMENT_CONFIRMATION_MODAL_2026-07-28.md`  
Committed and pushed to GitHub `https://github.com/nicholaswittle/wisenseobsidian.git`.

Related: [[APEX_V2_AUDIT_2026-07-27]]
