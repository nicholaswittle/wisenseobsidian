---
title: Apex v2 — Futures AI Plan (Customer-Funded Roadmap)
tags: [apex, ai, roadmap, haiku, sonnet, product]
date: 2026-07-28
status: planned
aliases: [Apex AI futures, Copilot roadmap]
---

# Apex v2 — Futures AI Plan (when customers offset cost)

> Ship more AI **only after** venues pay enough to cover Anthropic + support.  
> Related: [[wisense/projects/APEX_V2_SMALL_MODEL_AI_AUDIT_2026-07-28]] · [[wisense/projects/APEX_V2_AI_ASSISTED_PRODUCT_VERDICT_2026-07-28]] · [[Apex v2 — Restaurant OS Build]]

## Gate to unlock this plan
Any of:
- ≥3 paying OS venues, **or**
- ≥$200/mo net OS MRR after Stripe fees, **or**
- One flagship client under contract that includes Copilot

Until then: **maintain** current AI assisted surfaces; don’t expand.

---

## Already shipped (baseline Copilot)
| Surface | Model | Gate |
|---------|--------|------|
| Menu/schedule **photo** OCR | Sonnet 4.5 | Review → publish |
| Menu/schedule **text** cloud parse | Haiku 4.5 | Review → publish |
| Log Book tags / actions | Haiku | AI assist → Save |
| Call-out top covers | Haiku | Dialog after notify |
| Labor warning polish | Haiku | Continue dialog |
| Morning briefing | Haiku | Async Home card |

---

## Phase F1 — Make the pitch true (2–3 weeks)
**Goal:** Marketing claims match product.

1. **OT-dollar call-out ranking** — score eligible by `hourly_rate`, projected week hours, role match; Haiku only writes the one-line “why”.
2. **Optional “notify top 3 first”** — staged SMS fan-out (still human confirm).
3. **Briefing depth** — include yesterday’s sales (if Connect/orders live) + 86 list + open call-outs.
4. **Usage meter** — per-org AI call counts in admin (kill switch if abuse).

**SKU:** bundled in OS ($99); meter for abuse, not a new SKU yet.

---

## Phase F2 — Copilot Add-on $19 (if F1 sticky)
**Goal:** Monetize heavy AI without raising OS for light users.

1. Voice → schedule line (“Sam Tuesday four to ten”).
2. Specials / 86 natural language → stock toggles (confirm chips).
3. Shift handoff digest from chat + log → one SMS to next manager.
4. Guest menu Q&A (allergens) — **strict allowlist** answers only; no free chat.

**Rule:** still no AI money math, tip splits, or auto-publish schedules.

---

## Phase F3 — Scale / privacy (multi / enterprise)
1. On-device OCR (ML Kit) on mobile before cloud.
2. Per-venue Anthropic budget caps + soft degrade to Dart-only.
3. Optional customer-owned key (BYOK) for larger groups.
4. Eval harness: golden menus/whiteboards; regression before model bumps.

---

## Explicit non-goals (still)
- Autonomous schedule publish  
- AI tip pool / fee calculation  
- Open guest chatbot  
- Silent KDS state changes  

---

## Model routing (locked until F3)
```
Photo / messy OCR     → Sonnet 4.5
High-frequency text   → Haiku 4.5
Deterministic formats → Dart / SQL ($0)
```
