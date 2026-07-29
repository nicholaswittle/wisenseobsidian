---
type: audit
title: "Apex v2 — System Audit: Stripe Connect Express & 1.5% Platform Fee (TASK-004)"
tags: [audit, stripe-connect, apex-v2, platform-fee, security, fintech, rls]
date: 2026-07-28
status: completed
target_repo: "github.com/nicholaswittle/apex_v2"
target_commit: "fc5b6dc"
target_db: "pqkremkwfkudrhtxasdj"
---

# 🛡️ Apex v2 — System Audit: Stripe Connect Express + 1.5% Platform Fee

> **Audit Scope:** Commit `fc5b6dc` on `apex_v2` (`main` branch)  
> **Target Subsystems:** Stripe Connect Express Onboarding, Guest Checkout (`create-guest-payment`), Edge Webhooks (`stripe-os-webhook`), Database RLS & RPC (`20260728231446_stripe_connect_platform_fee.sql`), Cart & KDS UI.  
> **Auditors:** Senior Systems Architect & Master Code Auditor (MCA / MDT Protocol)

---

## 1. Executive Summary & Verdict

### 🟢 Overall Status: READY FOR TEST-MODE VENUES (With 1 Recommended Edge Patch)
The Stripe Connect Express integration and 1.5% platform application fee architecture is **exceptionally well-designed, secure, and robust**. Multi-tenant isolation is airtight, database integrity is guarded via PL/pgSQL `place_order`, and guest orders are strictly held in `pending_payment` until webhooks confirm funds capture.

Before flipping to **Live Mode**, 1 critical configuration setting (`on_behalf_of`) must be added to `create-guest-payment/index.ts` so Stripe processing fees are properly absorbed by the venue rather than WiSense.

---

## 2. Structured Audit Findings Matrix

| Risk Level | Finding ID | Summary | Target Location |
|:---:|:---:|:---|:---|
| 🔴 **HIGH** | `SEC-001` | Missing `on_behalf_of` on Destination Charges (Platform Fee Liability) | `create-guest-payment/index.ts:142` |
| 🟡 **MEDIUM** | `MON-001` | Unhandled Minimum Charge Threshold ($0.50 USD) | `create-guest-payment/index.ts:101` |
| 🟢 **LOW** | `UX-001` | Flutter Web Hash/Query Route Parsing Verification | `lib/core/support.dart:18` |
| 🟢 **LOW** | `ENV-001` | Webhook Secret Environment Variable Pre-Flight | `stripe-os-webhook/index.ts:141` |

---

## 3. Detailed Audit Findings

### 🔴 HIGH: `SEC-001` — Missing `on_behalf_of` on Destination Charges

* **File:** [create-guest-payment/index.ts:142-151](file:///C:/development/projects/apex_v2/supabase/functions/create-guest-payment/index.ts#L142-L151)
* **Failure Scenario:** `payment_intent_data` specifies `transfer_data: { destination }` and `application_fee_amount: fee`, but **omits `on_behalf_of: destination`**.
* **Financial Impact:** In Stripe Connect, creating destination charges on the platform without `on_behalf_of` causes Stripe's 2.9% + 30¢ processing fees and chargeback liability to be billed to the **Platform Account (WiSense LLC)** instead of the connected venue.
  * *Example:* On a $10.00 order, WiSense collects a 15¢ platform fee, but Stripe deducts 59¢ in processing fees from WiSense's balance. WiSense loses **44¢ net per order**!
* **Recommended Fix:** Add `on_behalf_of: destination` to `payment_intent_data` in `create-guest-payment/index.ts`:

```typescript
payment_intent_data: {
  application_fee_amount: fee,
  transfer_data: { destination },
  on_behalf_of: destination, // <--- Add this line: transfers Stripe fees & liability to venue
  metadata: { ... }
}
```

---

### 🟡 MEDIUM: `MON-001` — Unhandled Minimum Charge Threshold ($0.50 USD)

* **File:** [create-guest-payment/index.ts:101-107](file:///C:/development/projects/apex_v2/supabase/functions/create-guest-payment/index.ts#L101-L107)
* **Failure Scenario:** Checks `if (amount <= 0)`, but Stripe Checkout requires a minimum total of **$0.50 (50 cents USD)** per session.
* **Failure Impact:** If a guest orders a single $0.35 item (e.g. extra dipping sauce), Stripe API throws an `InvalidRequestError` ("Amount must be at least 50 cents"), returning an unhandled 500 error response to the client.
* **Recommended Fix:** Add minimum amount validation:

```typescript
if (amount < 50) {
  return new Response(
    JSON.stringify({ error: "minimum_amount_50_cents", detail: "Stripe requires orders to be at least $0.50." }),
    { status: 400, headers: { ...corsHeaders, "Content-Type": "application/json" } }
  );
}
```

---

## 4. Subsystem Verification Evaluation

### 🔒 1. Security & Authorization (Pass)
* **`create-guest-payment` Gating:** No-JWT by design. Protected by `order_id` (UUID) + `public_token` (6-char secret token). An attacker cannot query or trigger payments without both parameters.
* **Client-Side Tampering:** **Impossible.** `total_cents` and `platform_fee_cents` are calculated server-side inside the `place_order` PL/pgSQL function and stored in Postgres. `create-guest-payment` reads `total_cents` directly from `online_orders` in DB.

### 💵 2. Money Correctness & Fee Math (Pass)
* **Platform Fee Math:** `v_platform_fee := round(v_total * 0.015)::int;` correctly calculates 1.5% in integer cents.
* **Edge Case Bounds:** Includes bounds checking `if (v_platform_fee >= v_total) v_platform_fee := greatest(v_total - 1, 0);` ensuring fee never exceeds total order value.

### ⚡ 3. Webhook Reliability & Idempotency (Pass)
* **`account.updated` Webhook:** Handles race conditions where `account.updated` arrives before `stripe_account_id` is persisted by checking `metadata.restaurant_id` and `metadata.organization_id`.
* **Live Status Sync:** `stripe-connect-onboard` includes a `sync_only` mode that queries Stripe API live on page load, guaranteeing UI sync even if webhooks lag.
* **Idempotency:** `checkout.session.completed` updates `online_orders` conditionally (`in('payment_status', ['awaiting_payment', 'pending'])`). Duplicate webhooks are completely safe.
* **KDS Isolation:** Orders in `pending_payment` are hidden from the Kitchen KDS screen and do not play alert sounds until money is confirmed by Stripe webhook (`status = 'waiting'`).

### 🏢 4. Multi-Tenant Isolation (Pass)
* Org A cannot onboard or trigger payments against Org B's Connect account. `place_order` reads `stripe_account_id` directly from `restaurant_settings` linked to the verified `organization_id`.

---

## 🚀 Live Cutover Checklist (Test → Production)

Before switching from Stripe Test Mode to Live Production:

- [ ] **1. Apply `SEC-001` Patch:** Add `on_behalf_of: destination` to `create-guest-payment/index.ts`.
- [ ] **2. Deploy Live Edge Functions:**
  ```powershell
  supabase functions deploy stripe-connect-onboard --project-ref pqkremkwfkudrhtxasdj
  supabase functions deploy create-guest-payment --no-verify-jwt --project-ref pqkremkwfkudrhtxasdj
  supabase functions deploy stripe-os-webhook --no-verify-jwt --project-ref pqkremkwfkudrhtxasdj
  ```
- [ ] **3. Set Live Secrets:**
  ```powershell
  supabase secrets set STRIPE_SECRET_KEY=sk_live_... --project-ref pqkremkwfkudrhtxasdj
  supabase secrets set STRIPE_WEBHOOK_SECRET=whsec_... --project-ref pqkremkwfkudrhtxasdj
  supabase secrets set APEX_APP_URL=https://apex-v2-ten.vercel.app --project-ref pqkremkwfkudrhtxasdj
  ```
- [ ] **4. Register Live Webhook Endpoint in Stripe Dashboard:**
  * Endpoint URL: `https://pqkremkwfkudrhtxasdj.supabase.co/functions/v1/stripe-os-webhook`
  * Subscribed Events: `checkout.session.completed`, `account.updated`
- [ ] **5. Re-Connect Venue Accounts:** In Stripe Live mode, connected venues must complete Stripe Express onboarding once to link live bank payout details.

---

## 📄 File Location & Sync
Saved to Obsidian Vault static reference:  
`C:\Users\nikwi\Notes\wisense\projects\APEX_V2_STRIPE_CONNECT_AUDIT_2026-07-28.md`  
Committed and pushed to GitHub `https://github.com/nicholaswittle/wisenseobsidian.git`.
