# Restaurant OS — Build Plan

**Date:** 2026-07-27  
**Status:** Proposed — awaiting Nicholas Wittle ratification  
**Blocking threads:** Apple Dev ($99 paid, DBA pending from Emily), Google Play ($25 paid, DBA pending), Jigsy's third owner (son)

---

## Strategic Rationale

Downtime across 3 blocked threads. Restaurant OS plan is fully designed (16 phases, 20 tables, RLS, security model). When Jigsy's commits, deploy immediately — no build scramble.

Risk: scope creep. Mitigation: defer multi-location to Phase 2, start single-location.

---

## Phase 0: Executive Review (NOW)

- [ ] Ratify plan amendments: single-location first, multi-location deferred
- [ ] Decision: backend-only Phase 1–3, or include frontend shell?
- [ ] Confirm: no POS integration until Jigsy's verbal commit

---

## Phase 1: Core Schema (7 tables)

Defer 14-table Phase 1 from original plan → 7 tables first.

| Table | Purpose |
|-------|---------|
| restaurants | Core tenant record |
| restaurant_settings | Config + feature flags + delivery zone JSONB |
| restaurant_locations | Single location first, multi-ready |
| location_tables | Floor plan tables with status |
| menu_categories | Display grouping |
| menu_items | Core sellable items |
| modifiers | Add-on options |

**Security:** RLS enabled + forced on all tables. Helper functions: `is_member()`, `has_role()`, `is_staff()`. Roles: owner → manager → server → kitchen → readonly.

---

## Phase 2: Menu Engine

| Table | Purpose |
|-------|---------|
| modifier_groups | Grouped rules (size, crust, toppings) |
| modifier_options | Selectable options with pricing |
| menu_item_modifier_groups | Junction with overrides |

Features: availability windows, prep times, pricing inheritance, 86'd items.

---

## Phase 3: Order Lifecycle

| Table | Purpose |
|-------|---------|
| online_orders | Full order record with state machine |
| order_items | Line items snapshot |
| order_item_modifiers | Selected options |

State machine: pending → confirmed → preparing → ready → completed → cancelled.  
Data capture: `prep_time_snapshots` for ML training later.

---

## Phase 4: Integration Layer (DEFER to Jigsy's commit)

| Table | Purpose |
|-------|---------|
| integration_connections | Square etc. status |
| integration_credentials | Encrypted tokens (Supabase Vault) |
| pos_menu_map | Internal ↔ external ID mapping |
| sync_jobs | Sync audit trail |
| pos_order_map | Idempotency keys |
| dead_letter_queue | Failed operations |

Webhook endpoint: HMAC validation → payload check → persist raw → enqueue → idempotent processor.

---

## Phase 5: Frontend (New Horizon app)

- Customer-facing ordering UI
- Staff console (kitchen display, order state)
- Admin (menu, floor plan, settings)

---

## ML Service (Phase 6+)

Prep-time estimation, order throttling, staffing recommendations. Capture from day one via `prep_time_snapshots`, train later.

---

## Deployment Tiers

**Tier 2A:** Single DB, shared schema + RLS (start here)  
**Tier 2B:** Per-client DB on paid tier (scale)  
**Tier 2C:** Dedicated projects (enterprise/franchise)

---

## Audit Trail (Tripartite Protocol)

| Branch | Action |
|--------|--------|
| Executive | Ratify this plan |
| Legislative | Write schema + RLS SQL files |
| Judicial | Codex audit (Gemini unavailable until July 2026) |
| Executive | Completion Report → Nicholas Wittle |

---

## Dependencies

| Dependency | Status |
|------------|--------|
| Supabase project | Exists (Apex Scheduler) — new project for Restaurant OS? |
| Square developer account | Not created — defer to Phase 4 |
| Jigsy's commit | Pending third owner |

---

*Plan committed by Hermes Agent. Awaiting ratification.*

Related: [[projects/APEX_MASTER_PLAN]] · [[projects/Apex v2 — Restaurant OS Build]]
