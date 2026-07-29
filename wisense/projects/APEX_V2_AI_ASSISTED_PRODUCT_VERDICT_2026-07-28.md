---
title: Apex v2 — AI Assisted Product Verdict 2026-07-28
tags: [apex, ai, product, sales, haiku, sonnet]
date: 2026-07-28
status: decided
aliases: [Apex AI verdict, Copilot verdict]
---

# Apex v2 — AI Assisted Product Verdict (2026-07-28)

> Does the Haiku/Sonnet review-gated AI layer help or hurt Apex? **It helps.**  
> Related: [[wisense/projects/APEX_V2_SMALL_MODEL_AI_AUDIT_2026-07-28]] · [[Apex v2 — Restaurant OS Build]] · [[hot]]

## Verdict

This AI architecture **strengthens Apex v2** without introducing product fragility or financial risk — when sold honestly as **AI assisted** with human confirm, not as autonomous ops.

---

## Why it helps

### Manager fatigue (real jobs, not gimmicks)
- **Call-out assist** — after a call-out, AI surfaces top cover suggestions while the team is still notified.
- **Morning briefing** — 3-bullet card on manager Home (async; dashboard never blocked).
- **Log Book AI assist** — tags / action items / summary chips; manager reviews then Save.

### Cost control
- **Text → Claude Haiku 4.5** (logs, briefing, labor polish, text menu/schedule when cloud is used).
- **Photos → Claude Sonnet 4.5** (whiteboard + paper menu OCR).
- On-device regex parsers stay free when paste is clean.
- Directionally: AI spend should stay **well under $1/venue/month** at soft-launch volumes (exact ¢ figures are estimates — watch Anthropic usage).

### Zero silent hallucination into money/schedule/orders
- AI **never** writes silently to payments, tip math, shift times, or kitchen order state.
- Outputs are staged (chips, dialogs, briefing card) → human **Confirm / Save / Continue**.
- Missing `ANTHROPIC_API_KEY` → graceful fallback (501 / on-device parsers / raw warnings), not a crash.

---

## Risks guarded

| Risk | Guard | Status |
|------|--------|--------|
| AI guessing money / tip splits / fees | Postgres + Dart integer math only | SAFE |
| Dashboard blocked on AI latency | Briefing loads async after Home | SAFE |
| App breaks without API key | Soft fail + local parsers | SAFE |
| Unreviewed schedule publish | Draft / review gates only | SAFE |

---

## Honesty for sales (do not oversell)

| Pitch language | Live today |
|----------------|------------|
| “AI assisted” log / briefing / labor tips / menu-schedule photo | Yes |
| Ranks best covers after call-out | Yes (top 3 + reasons) |
| Ranks by overtime $ risk ($16 vs $24 OT) | **Not yet** — Phase 2 if we feed hourly rates into the ranker |
| Notifies only the top ranked person | **No** — still fans out to all eligible; AI ranks for manager UX |

---

## Bottom line

Gives Apex a modern **Copilot** feel vs. clunky legacy schedulers, at low inference cost, with review gates that keep Soft-launch money and labor compliance defensible.

**Ship stance:** Advertise **AI assisted**. Upgrade OT-dollar ranking before claiming it in decks.

## Shipped surfaces (2026-07-28)
- `parse-log-summary` · `polish-labor-warnings` · `venue-briefing`
- `route-callout` ranked suggestions
- `parse-menu` / `parse-schedule` — Haiku text / Sonnet image
- Live: https://apex-v2-ten.vercel.app · GitHub `apex_v2` (~`c5aebff`)
