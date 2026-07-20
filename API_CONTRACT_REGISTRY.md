---
title: API Contract Registry
tags: [api, contract, json, payload, schemas]
aliases: [API Contracts, Payload Schemas]
---

# 📄 API CONTRACT REGISTRY

Canonical JSON payload specifications and data model contracts across all WiSense applications to ensure 100% frontend-backend alignment.

---

## 1. ~~WiSense OS Engine API~~ (DELETED 2026-07-19)

> WiSense OS was deleted. This section retained for historical reference only. See [[Abandoned Projects — Lessons]].

---

## 2. Apex Scheduler Schemas (Supabase)

### `organizations`
- `id`: `UUID` (Primary Key)
- `name`: `TEXT`
- `invite_code`: `TEXT` (Unique)

### `staff_profiles`
- `id`: `UUID` (References `auth.users`)
- `organization_id`: `UUID` (CRITICAL: Never use `defaultOrganizationId`)
- `full_name`: `TEXT`
- `role`: `TEXT` (`owner` | `manager` | `staff`)

---

## 3. COMMS LINK Schemas (On-Device FSM)

### Debrief Turn Payload
```json
{
  "turn_id": "uuid",
  "stage": "intake | validate | commit | close",
  "user_text": "Debrief input text...",
  "crisis_detected": false,
  "directive_summary": "Action plan summary..."
}
```

---

## 4. New Horizon Alignment Engine Schemas (Duffel Proxy)

### Flight Offer JSON
```json
{
  "offer_id": "off_12345",
  "carrier": "AA",
  "origin": "JFK",
  "destination": "LAX",
  "price_usd": 249.00,
  "affiliate_source": "duffel | viator"
}
```

Related: [[00_AI_AGENT_MANIFEST]], [[ENVIRONMENT_MAP]]
