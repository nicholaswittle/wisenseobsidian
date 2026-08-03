---
type: feature
title: "Apex v2 — Interactive Stripe Disconnect Button on Dashboard Card"
tags: [feature, stripe, disconnect, dashboard, apex-v2, UX]
date: 2026-07-28
status: completed
target_repo: "github.com/nicholaswittle/apex_v2"
target_commit: "6f4c00a"
---

# 🎉 Apex v2 — Interactive Stripe Disconnect Button on Dashboard Card

> **Rate-Limit Failover Completed:** Shipped and deployed live in commit `6f4c00a`.

---

## 1. What Was Fixed & Built

In `monetization_upsell_cards.dart` (the **"How online ordering works"** card on the main dashboard / menu checklist screen), the `[ Stripe connected ]` button previously had `onPressed: null` (disabled grey chip).

### The New Interactive UX:
1. **`[ Stripe connected ]`** is now a fully interactive, teal-themed button (`Icons.verified_rounded`).
2. Tapping **`[ Stripe connected ]`** opens a clean **Stripe Connected** dialog:
   * 🟢 **Title:** `Stripe Connected`
   * 📜 **Message:** *"Your Stripe Express account is active and accepting guest payments. Disconnecting will pause guest online card payments until reconnected."*
   * 🔘 **Actions:** `[ Close ]` and a prominent red **`[ Disconnect Stripe ]`** button.
3. Clicking **Disconnect Stripe** invokes the `stripe-connect-onboard` edge function with `disconnect: true`, wiping `stripe_account_id` and setting `stripe_charges_enabled: false`.
4. The UI immediately updates back to **`[ Connect Stripe ]`**.

---

## 2. Technical Implementation Details

- **File Modified:** `lib/features/dashboard/monetization_upsell_cards.dart`
- Added `_disconnectStripe()` method.
- Replaced `onPressed: null` with an async dialog handler triggering `_disconnectStripe()`.

---

## 3. Verification & Deployment

- **`dart analyze`:** **0 issues found.**
- **`flutter test`:** **29/29 unit tests passing.**
- **Vercel Build:** `npx vercel deploy --prod` — `READY` (`dpl_HnJtpXMw5KNB6ra9F6bCkdUErB1p`).
- **Live URL:** [https://apex-v2-ten.vercel.app](https://apex-v2-ten.vercel.app)

Related: [[APEX_V2_AUDIT_2026-07-27]]
