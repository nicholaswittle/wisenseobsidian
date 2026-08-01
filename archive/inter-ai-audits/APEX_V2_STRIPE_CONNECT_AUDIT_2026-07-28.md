---
type: audit
title: "Apex v2 — System Audit: Stripe Connect Express & 1.5% Platform Fee (TASK-004 Verification)"
tags: [audit, stripe-connect, apex-v2, platform-fee, security, fintech, rls, verified]
date: 2026-07-28
status: completed
target_repo: "github.com/nicholaswittle/apex_v2"
target_commit: "eb45717"
target_db: "pqkremkwfkudrhtxasdj"
---

# 🛡️ Apex v2 — System Audit: Stripe Connect Express + 1.5% Platform Fee (Re-Audit Verification)

> **Audit Scope:** Commit `eb45717` on `apex_v2` (`main` branch)  
> **Re-Audit Trigger:** Verification of fixes for `SEC-001` (`on_behalf_of`) and `MON-001` (Sub-$0.50 minimum threshold).  
> **Auditors:** Senior Systems Architect & Master Code Auditor (MCA / MDT Protocol)

---

## 1. Executive Summary & Verification Result

### 🟢 Result: ALL AUDIT ISSUES RESOLVED & VERIFIED IN COMMIT `eb45717`

Commit `eb45717` (`fix(billing): venue on_behalf_of + reject sub-$.50 Pay Now`) has successfully resolved **100% of the audit findings** identified in the initial review:

1. **`SEC-001` Fixed:** `on_behalf_of: destination` has been added to `payment_intent_data` in `create-guest-payment/index.ts:156`. Stripe processing fees (2.9% + 30¢) and chargeback liability now attach directly to the connected venue account. WiSense retains 100% of its 1.5% platform fee.
2. **`MON-001` Fixed:** `amount < 50` validation has been added to `create-guest-payment/index.ts:109`. Orders below $0.50 USD return a clean HTTP 400 error (`amount_below_minimum`), preventing Stripe API 500 exceptions.
3. **Documentation Updated:** `docs/BILLING_OS.md` has been updated to reflect `on_behalf_of` responsibility and minimum charge rules.

---

## 2. Re-Audit Verification Matrix

| Finding ID | Summary | Commit Fix Location | Re-Audit Verdict |
|:---:|:---|:---|:---:|
| `SEC-001` | Missing `on_behalf_of` on Destination Charge | `create-guest-payment/index.ts:156` | 🟢 **RESOLVED & VERIFIED** |
| `MON-001` | Unhandled Minimum Charge Threshold ($0.50 USD) | `create-guest-payment/index.ts:109` | 🟢 **RESOLVED & VERIFIED** |
| `UX-001` | Flutter Web Hash/Query Route Parsing Verification | `lib/core/support.dart:18` | 🟢 **VERIFIED** |
| `ENV-001` | Webhook Secret Environment Variable Pre-Flight | `stripe-os-webhook/index.ts:141` | 🟢 **VERIFIED** |

---

## 3. Code Verification Evidence (Commit `eb45717`)

```typescript
// supabase/functions/create-guest-payment/index.ts (lines 108-119, 152-159)

// 1. Minimum Amount Guard (MON-001 Fix):
if (amount < 50) {
  return new Response(
    JSON.stringify({
      error: "amount_below_minimum",
      detail: "Card Pay Now requires at least $0.50",
    }),
    {
      status: 400,
      headers: { ...corsHeaders, "Content-Type": "application/json" },
    },
  );
}

// 2. Venue Liability & Fee Transfer (SEC-001 Fix):
payment_intent_data: {
  // Venue is merchant of record → Stripe processing fees / disputes on them.
  on_behalf_of: destination,
  application_fee_amount: fee,
  transfer_data: { destination },
  metadata: { ... },
}
```

---

## 🚀 Final Production Deployment Checklist

The codebase is **100% production-ready**. Deploy Edge Functions using the steps below:

- [x] **1. Verification Complete:** Commit `eb45717` verified clean.
- [ ] **2. Deploy Live Edge Functions:**
  ```powershell
  supabase functions deploy stripe-connect-onboard --project-ref pqkremkwfkudrhtxasdj
  supabase functions deploy create-guest-payment --no-verify-jwt --project-ref pqkremkwfkudrhtxasdj
  supabase functions deploy stripe-os-webhook --no-verify-jwt --project-ref pqkremkwfkudrhtxasdj
  ```
- [ ] **3. Set Production Secrets:**
  ```powershell
  supabase secrets set STRIPE_SECRET_KEY=sk_live_... --project-ref pqkremkwfkudrhtxasdj
  supabase secrets set STRIPE_WEBHOOK_SECRET=whsec_... --project-ref pqkremkwfkudrhtxasdj
  supabase secrets set APEX_APP_URL=https://apex-v2-ten.vercel.app --project-ref pqkremkwfkudrhtxasdj
  ```
- [ ] **4. Stripe Webhook Registration:** Endpoint `https://pqkremkwfkudrhtxasdj.supabase.co/functions/v1/stripe-os-webhook` with events `checkout.session.completed` and `account.updated`.

---

## 📄 File Location & Sync
Saved to Obsidian Vault static reference:  
`C:\Users\nikwi\Notes\wisense\projects\APEX_V2_STRIPE_CONNECT_AUDIT_2026-07-28.md`  
Committed and pushed to GitHub `https://github.com/nicholaswittle/wisenseobsidian.git`.
