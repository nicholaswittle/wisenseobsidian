---
title: API Contract Registry
tags: [api, contract, json, payload, schemas]
aliases: [API Contracts, Payload Schemas]
---

# 📄 API CONTRACT REGISTRY

Canonical JSON payload specifications and data model contracts across all WiSense applications to ensure 100% frontend-backend alignment.

---

## 1. WiSense OS Engine API (`http://127.0.0.1:5050`)

### `GET /api/v1/health`
```json
{
  "status": "ok",
  "version": "0.1.0",
  "engine": "wisense-os-native"
}
```

### `POST /api/v1/tasks` (Submit Task)
```json
{
  "request": "Fix billing calculation",
  "project_root": "C:\\development\\projects\\apex\\apex",
  "mode": "ask_before_changes",
  "chat_model": "gemma4:31b-cloud",
  "builder_model": "gemma4:31b-cloud",
  "offline": false
}
```

### `POST /api/v1/tasks/<task_id>/propose` (Prepare Proposal Response)
```json
{
  "task_id": "uuid",
  "proposal": {
    "digest": "a1b2c3d4e5f6...",
    "summary": "Updated billing logic in lib/billing.dart",
    "files": {
      "lib/billing.dart": "diff content..."
    }
  }
}
```

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
