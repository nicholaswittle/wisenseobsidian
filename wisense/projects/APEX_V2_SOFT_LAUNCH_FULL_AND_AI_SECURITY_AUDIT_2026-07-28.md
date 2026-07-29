---
type: audit
title: "Apex v2 — Soft-Launch Full Audit + AI Security Addendum"
tags: [audit, security, soft-launch, ai, apex-v2]
date: 2026-07-28
status: completed
target_repo: "github.com/nicholaswittle/apex_v2"
---

# Apex v2 — Soft-Launch Full Audit + AI Security Addendum (2026-07-28)

> Complements [[wisense/projects/APEX_V2_FULL_SYSTEM_SECURITY_AUDIT_2026-07-28]] (money/PII/RLS) with **post-AI** surfaces and a soft-launch product checklist.  
> Related: [[wisense/projects/APEX_V2_FUTURES_AI_PLAN_2026-07-28]] · [[wisense/projects/APEX_V2_AI_ASSISTED_PRODUCT_VERDICT_2026-07-28]]

## 1. Soft-launch full audit (product / ops)

| Area | Status | Notes |
|------|:------:|-------|
| Auth + org scoping | PASS | Staff JWT; guest token path |
| Schedule + Assign Days | PASS | Guardrails advisory |
| Photo → schedule | PASS | Sonnet; review → publish |
| Menu editor + extras | PASS | Prices editable; packs |
| Photo/text → menu | PASS | Sonnet photo / Haiku text |
| Online ordering + KDS | PASS | OS-gated live checkout |
| Stripe Connect 1.5% | PASS | Destination + app fee (test mode OK for soft launch) |
| AI assisted ops | PASS | Log / call-out / briefing / labor polish |
| Entitlements funnel | PASS | Free/Pro/OS Payment Links (test) |
| Store apps / live Stripe | DEFER | Not required for soft web launch |
| Square / payroll / geofence | DEFER | Plan backlog |

**Product verdict:** Soft-launch **ready** on web for demos + pilot venues in **test Stripe**. Live cards = later cutover checklist (live keys, webhooks, Connect reconnect).

---

## 2. Security audit addendum (AI layer)

Prior money/PII audit remains the baseline. New findings from AI edges:

| Finding | Severity | Fix / status |
|---------|:--------:|--------------|
| `parse-log-summary` / `polish-labor-warnings` lacked app-level JWT user check (anon could burn Anthropic if JWT verify misconfigured) | HIGH | **Fixed 2026-07-28** — require `Authorization` + `getUser()`; note length / warning count caps |
| `venue-briefing` / `route-callout` already auth’d | OK | Keep |
| `parse-menu` / `parse-schedule` gateway `verify_jwt` + large body hang risk | MED | Client compress + timeout; photo size cap |
| AI prompt injection via log/menu text | LOW | Outputs review-gated; never write money |
| Anthropic key in chat history (ops) | MED | Rotate when convenient; secret lives in Supabase only |
| Broad menu public SELECT (pre-existing) | MED | Known; guest needs menu; tighten later with RPC |
| No per-org AI rate limit | MED | Futures F1 usage meter |

**AI security verdict:** **PASS for soft launch** after JWT harden on log/labor polish edges. Re-audit before live Stripe + multi-tenant AI scale.

---

## 3. What’s left **for now** (do / don’t)

### Do before first real customer money (live cards)
1. Stripe **live** keys + webhook + Payment Links recreate  
2. Venues re-onboard Connect in live mode  
3. Rotate Anthropic key (was pasted in chat)  
4. Smoke: Pay Now, call-out, log AI, briefing, menu photo on a clean org  

### Nice soon (not blocking soft demo)
- OT-dollar call-out ranking (Futures F1)  
- Admin AI usage meter  
- Commit/push vault notes to wisenseobsidian if you want remote mirror  

### Explicitly **not** required now
- Futures F2/F3 AI  
- Google Calendar OAuth / Square / geofence  
- App Store / Play packages  
- New Copilot SKU  

---

## 4. One-line launch stance

**Soft-launch Apex web as AI-assisted Restaurant OS in test mode now; flip live Stripe when a paying venue needs real cards.**
