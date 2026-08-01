---
title: Reusable Platform Components — What Can Be Pulled for Any Business
tags: [business, product, platform, reusable, template, architecture, build]
aliases: [Reusable Components, Platform Architecture, What to Reuse]
date: 2026-07-26
status: active
---

# Reusable Platform Components — What Can Be Pulled for Any Business

> Analysis of the Jigsy's ordering platform repo (nicholaswittle/jigsysiteworking, branch agent/reusable-ordering-core).
> The repo is already structured as a reusable platform — Jigsy's is the first configuration, not a one-off.
> This note documents what can be reused for ANY business type, not just restaurants.

Related: [[business/Restaurant Ordering Template Product Strategy 2026-07-24]], [[business/Pricing Models Ownership and Exit 2026-07-26]], [[business/Jigsys Ordering Platform — Square Findings and Business Model 2026-07-26]], [[business/Jigsys Ordering Platform — Claude Handoff 2026-07-25]], [[prospects/Cold Outreach Target List]], [[NOW]]

---

## Source Repository

- GitHub: https://github.com/nicholaswittle/jigsysiteworking
- Branch: agent/reusable-ordering-core
- 59 files total
- Live demo: https://jigsys-ordering-demo.wisense.workers.dev

---

## FULLY REUSABLE (zero changes needed)

### 1. Backend Infrastructure
- `worker/index.ts` — Cloudflare Worker entry point, image optimization, API routing
- `db/schema.ts` — Order table, settings table, Square connection table (parameterized by restaurantId)
- All drizzle migrations (0000-0004)
- `docs/CLOUDFLARE-DEPLOY.md` — full deploy runbook
- Hosting: Cloudflare Workers + D1 database (free tier, $0 cost)

### 2. Order/Request Management System
- Order states: waiting, accepted, rejected, completed, cancelled
- Public order token (customer checks status without login)
- Staff login with PIN + time-limited session (12hr)
- Kitchen ticket printing (80mm thermal format, ESC/POS ready)
- Daily and monthly fee reporting (only completed+paid orders count)
- Nightly order pruning (cron trigger at 5am UTC, deletes orders older than 7 days)

### 3. Staff Console
- Accept/reject orders in one click
- Pause/reopen ordering (hides ordering buttons, keeps website visible)
- Sold-out item toggles (per-item availability control)
- Pickup time adjustment (10-90 min in 5-min steps)
- Repeating staff alerts (30s re-alert, escalates after 2 min)
- Screen Wake Lock (keeps kitchen tablet awake)
- "Didn't pay" no-show marking (reversible)
- Monthly totals rollup from daily_totals table

### 4. Customer Submission Flow
- Cart with modifiers and notes
- Pickup/appointment time scheduling
- Customer order status (waiting/accepted/rejected)
- Browser notifications + chime on accept/reject
- "Created by WiSense LLC" credit
- Customer private order-status token

### 5. Square Integration
- OAuth connection flow (owner approves on Square's page, you never see their password)
- Encrypted token storage (server-side, AES encrypted)
- Automatic token refresh
- Order push to Square (POST /v2/orders with line items + fee + pickup fulfillment)
- Manual pay-at-counter mode (no card processing needed)
- Sandbox test mode for demos

### 6. API Layer
- `api-client.js` — all fetch calls, reusable as-is (window.WiSenseOrdering object)
- `worker/api.ts` — all API routes, parameterized by restaurantId
- API endpoints: /api/public/settings, /api/orders, /api/staff/*, /api/staff/square/*

---

## RESTAURANT-SPECIFIC (must swap per client)

These are the ONLY things that change per business:

1. `config/menu-catalog.json` — the menu (products, prices, sizes, categories)
2. Restaurant name, logo, colors, copy, address, phone
3. Hours and pickup timing
4. Staff PIN
5. Online ordering fee amount ($0.99, $1.99, or $0)
6. Tax rate
7. Images (food photos, exterior, logo, icons)
8. Printer model and print method
9. `RESTAURANT_ID` constant in worker/api.ts (currently "jigsys")
10. `RESTAURANT_TIME_ZONE` in worker/api.ts (currently "America/New_York")

---

## How to Adapt for Non-Restaurant Businesses

The core is a "submit request → staff accepts → ticket prints → customer gets notified" system.
This works for any business where a customer submits something and staff needs to accept it.

### Restaurants (food orders)
- Menu catalog → food items with prices and sizes
- "Order" → "Order"
- "Kitchen ticket" → "Kitchen ticket"
- Already built and working

### Salons (appointment bookings)
- Menu catalog → services list (haircut, color, manicure, etc.)
- "Order" → "Appointment"
- "Kitchen ticket" → "Appointment ticket"
- "Pickup time" → "Appointment time"
- Customer selects service + stylist + time → staff accepts → confirmation sent

### Plumbers/HVAC (service calls)
- Menu catalog → service types (repair, install, estimate, emergency)
- "Order" → "Service request"
- "Kitchen ticket" → "Job ticket"
- "Pickup time" → "Preferred appointment window"
- Customer describes problem + selects service type → staff accepts → job ticket prints

### Auto Repair (job intake)
- Menu catalog → service types (oil change, brake, diagnostic, etc.)
- "Order" → "Repair request"
- "Kitchen ticket" → "Work order"
- Customer selects service + describes vehicle issue → staff accepts → work order prints

### Contractors (quote requests)
- Menu catalog → project types (kitchen, bath, addition, roof)
- "Order" → "Quote request"
- "Kitchen ticket" → "Lead ticket"
- Customer describes project + selects type → staff accepts → lead ticket prints

### Any Business (contact form + intake)
- Menu catalog → service types or inquiry categories
- "Order" → "Inquiry"
- Customer submits request → staff accepts → ticket prints → customer notified

---

## What Still Needs Building (Template-Level)

From docs/REUSABLE-RESTAURANT-PLATFORM.md:

1. Move menu from client file (config/menu-catalog.json) into database-backed catalog
2. Add owner setup screen for branding, hours, fees, and menu import
3. Add Square location chooser for multi-location accounts
4. Validate prices and taxes on the server from connected Square catalog
5. Add push notification service worker + optional SMS confirmation
6. Add multi-restaurant routing and separate staff authorization per restaurant

---

## Config Work Per New Client (2-4 hours)

For each new client:
1. Clone the repo
2. Swap config/menu-catalog.json with their products/services
3. Swap branding (name, logo, colors, copy, address, phone)
4. Swap images
5. Set up a new Cloudflare D1 database
6. Deploy to Cloudflare Workers
7. Connect their Square via OAuth (if they use Square)
8. Point their domain at the Worker URL
9. Test with a few demo orders

That's it. No new code. Just config and deploy.