---
title: Restaurant OS Build Strategy — Bridge Pieces
tags: [business, product, restaurant-os, apex, supabase, build-plan, strategy]
date: 2026-07-27
status: active
---

# Restaurant OS — Build Strategy (Bridge Pieces)

> How to build the OS without falling into the "build a platform with no users" trap. Each piece is useful standalone. Build the bridge, not the monument.

Related: [[business/WiSense Restaurant OS Master Plan 2026-07-27]], [[Apex Scheduler]], [[business/Jigsys Ordering Platform — Square Findings and Business Model 2026-07-26]], [[business/Reusable Platform Components 2026-07-26]], [[NOW]]

---

## The Lesson from Dead Projects

wisense-os, my_ai, command_center, local-agent-work-center — all failed for the same reason: built the big thing before proving the small thing. OS before a single feature. Neural command deck before a single conversation.

This is different. Apex works. Ordering works. We're not building in the dark — we're connecting two working systems.

But the same trap exists: don't build "the OS" as one big project. Build the bridge — one piece at a time, each piece useful on its own.

---

## The Two Tech Stacks

| System | Frontend | Backend | Database |
|---|---|---|---|
| Apex Scheduler | Flutter (mobile + web) | Supabase (PostgreSQL, auth, realtime, RLS) | Supabase Postgres |
| Jigsy's Ordering | HTML/JS (web) | Cloudflare Workers | Cloudflare D1 (SQLite) |

They can't share a database directly. Three options:

- **Option A: Shared Supabase** — move ordering data into Supabase. One database, both apps. Most migration work but cleanest result.
- **Option B: API bridge** — keep them separate, add API calls between them. Less migration but two systems to maintain.
- **Option C: Rebuild ordering as Flutter web** — one app, one codebase, one database. Biggest build but simplest long-term.

**Recommendation: Option A.** Move ordering into Supabase. Apex already has the auth, RLS, realtime infrastructure. The ordering frontend stays as HTML/JS, just points at Supabase instead of D1.

---

## Build Order — Bridge Pieces

### Piece 1: Unified Supabase Backend (1 weekend)

Move ordering data into Apex's Supabase:
- Add tables: `orders`, `menu_items`, `restaurant_settings`, `order_status`
- Port the Jigsy's ordering frontend to read/write from Supabase instead of D1
- Now both apps share one database

**Useful standalone:** One backend instead of two. Easier to maintain. Realtime updates work across both apps.

**OS value:** The foundation. Everything else depends on this.

### Piece 2: Labor vs Revenue Dashboard (1 weekend)

- Pull time clock data (Apex) + order totals (ordering) from the same Supabase
- Show the owner: "Tonight: $2,400 in orders, $480 in labor = 20%"
- Daily/weekly/monthly view
- Alert when labor exceeds 30% of projected sales

**Useful standalone:** Labor cost reporting is a feature competitors charge extra for. 7shifts requires Pro ($69.99/mo) for labor cost tracking. You include it.

**OS value:** First time scheduling and ordering "talk" — the owner sees the connection between who's working and what's coming in.

### Piece 3: No-Show Call-Out Engine (1 weekend)

- Staff member marks "can't make it" in Apex
- System auto-texts all available employees with matching role: "Mike called out for tonight 5-11. Can you cover? Tap yes."
- First responder gets the shift
- Ordering system adjusts capacity if no one fills it

**Useful standalone:** No-show management is a daily restaurant pain. Only Sling Business and Deputy track no-shows. Nobody auto-fills them.

**OS value:** Where scheduling actively affects ordering. If no cook shows up, ordering auto-pauses. No competitor does this.

### Piece 4: Manager Log Book (1 weekend)

- Add `shift_notes` table to Supabase
- Manager writes handoff notes at end of shift
- Next shift sees them when they clock in
- Attach photos of issues (broken equipment, low stock)

**Useful standalone:** 7shifts charges $14.99/mo for this. You include it free.

**OS value:** Shift-to-shift communication becomes part of the operating system, not a separate notebook.

### Piece 5: Tip Management (1 weekend)

- Add `tip_pool` table to Supabase
- Owner enters total tips at end of shift
- System splits by hours worked (pulls from time_entries)
- Employees see their tip breakdown in the app

**Useful standalone:** 7shifts charges $49.99/mo for this. Homebase charges $25/mo. You include it free.

**OS value:** Tips connect to time clock data connect to scheduling. The full labor picture.

### Piece 6: Smart Ordering Capacity (1 weekend, after Pieces 1-3)

- Ordering system checks Apex schedule before accepting orders
- "You have 1 cook on shift. Max 15 orders/hour. Order #16 gets 'longer wait time' warning."
- Auto-pause ordering when critically understaffed
- Auto-resume when the next cook clocks in

**Useful standalone:** Nobody has this. It's the killer feature.

**OS value:** The system actively manages the restaurant based on who's working and what's coming in.

---

## What to Build This Weekend

Start with Piece 1 — unified Supabase backend. That's the foundation.

Then Piece 4 (log book) and Piece 5 (tip management) — they're the easiest builds (CRUD + math) and the highest-value differentiators (competitors charge $65/mo extra for both combined, you include them free).

Then Piece 2 (labor dashboard) — first "OS" feature.

Then Piece 3 (no-show engine) and Piece 6 (smart capacity) — the features nobody else has.

---

## Why This Is Different from Dead Projects

| Dead Projects | This |
|---|---|
| Built OS before any feature | Building features that work standalone |
| No user in mind | Jigsy's + FFL dealer + 67 cold prospects |
| Built for self | Built for real restaurants with real pain |
| Hoped users would come | Users exist, product is shipping |
| Over-engineered infrastructure | Using existing Supabase + existing code |
| Speculative features | Features competitors already charge for (validated demand) |

---

## Database Schema (Piece 1 — What to Add to Supabase)

```
-- Ordering tables (new, for the OS bridge)
CREATE TABLE orders (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id UUID REFERENCES organizations(id),
  public_token TEXT UNIQUE NOT NULL,
  status TEXT NOT NULL DEFAULT 'waiting', -- waiting/accepted/rejected/completed/cancelled
  submitted_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  accepted_at TIMESTAMPTZ,
  rejected_at TIMESTAMPTZ,
  completed_at TIMESTAMPTZ,
  pickup_minutes INT NOT NULL DEFAULT 30,
  customer_json JSONB NOT NULL,
  notes TEXT NOT NULL DEFAULT '',
  items_json JSONB NOT NULL,
  subtotal_cents INT NOT NULL,
  fee_cents INT NOT NULL DEFAULT 0,
  tax_cents INT NOT NULL DEFAULT 0,
  total_cents INT NOT NULL,
  payment_mode TEXT NOT NULL DEFAULT 'manual',
  payment_status TEXT NOT NULL DEFAULT 'pending'
);

CREATE TABLE menu_items (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id UUID REFERENCES organizations(id),
  name TEXT NOT NULL,
  category TEXT NOT NULL,
  description TEXT,
  price_cents INT NOT NULL,
  available BOOLEAN NOT NULL DEFAULT true,
  sort_order INT NOT NULL DEFAULT 0
);

CREATE TABLE restaurant_settings (
  organization_id UUID PRIMARY KEY REFERENCES organizations(id),
  name TEXT NOT NULL,
  paused BOOLEAN NOT NULL DEFAULT false,
  prep_minutes INT NOT NULL DEFAULT 30,
  fee_cents INT NOT NULL DEFAULT 0,
  tax_rate REAL NOT NULL DEFAULT 0.06,
  payment_mode TEXT NOT NULL DEFAULT 'manual'
);

-- OS bridge tables (new)
CREATE TABLE shift_notes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id UUID REFERENCES organizations(id),
  author_id UUID REFERENCES auth.users(id),
  shift_date DATE NOT NULL,
  note TEXT NOT NULL,
  photo_url TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE tip_pools (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id UUID REFERENCES organizations(id),
  shift_date DATE NOT NULL,
  total_cents INT NOT NULL,
  split_method TEXT NOT NULL DEFAULT 'hours',
  created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE tip_allocations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tip_pool_id UUID REFERENCES tip_pools(id),
  user_id UUID REFERENCES auth.users(id),
  hours_worked REAL NOT NULL,
  amount_cents INT NOT NULL
);

-- RLS policies on all new tables (organization_id scoping)
-- Same pattern as existing Apex tables
```

---

## Reuse Beyond Restaurants

The core pattern — "submit request → staff accepts → track → report → optimize" — works for:
- Salons (appointments + stylist schedules + revenue)
- Plumbers (service calls + dispatch + labor cost)
- Auto repair (job intake + mechanic schedules + revenue)
- Contractors (project requests + crew schedules + labor cost)

Build it for restaurants first (you have the code and the prospects). Prove it there. Then expand to other verticals by swapping config — same as the website template strategy.