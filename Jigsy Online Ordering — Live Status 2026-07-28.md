---
title: Jigsy Online Ordering — Live Status 2026-07-28
tags: [jigsy, ordering, apex, restaurant-os, pilot]
aliases: [Jigsy order online, print and pay]
date: 2026-07-28
updated: 2026-07-28
status: active
---

# Jigsy Online Ordering — Live Status 2026-07-28

Guest ordering is live on the Jigsy site, wired to Apex Supabase. Staff run tickets in Apex **Orders**. Pay is at pickup; kitchen print is not built yet.

## Live links

| Surface | URL |
|---------|-----|
| Jigsy site (Order online) | https://jigsyssite.vercel.app |
| Apex real app | https://apex-v2-ten.vercel.app |
| Apex demo | https://apex-v2-demo.vercel.app |
| Supabase | `pqkremkwfkudrhtxasdj` |
| Restaurant | `public_token = jigsys` · id `a1000000-0000-4000-8000-000000000001` · org `c82c025d-7300-4d41-a84d-bc17d3c3104f` |

## What works today

1. **Order online** on the site opens a modal → loads live menu/capacity from Supabase → cart → `place_order` RPC → pickup code.
2. **Full Nov 2025 board** is in `menu_categories` / `menu_items` (trays by cut, specialty, gourmet, wings, stromboli/flatbreads, starters, salads, subs, dessert, house brews). Stub seed items are unavailable and hidden from guests.
3. **Extras / modifiers:** tray toppings (priced by cut: +$1.50 / $2.00 / $2.50), wing sauce + sides (+$1), salad dressings, sub add fries (+$2), fries cheese/bacon.
4. **Staff console (Apex → Orders):** Waiting → Accept / Reject → Complete. Screen ticket is the kitchen ticket.
5. **Capacity:** `capacity_snapshot` + auto-pause exist; auto-pause was turned **off** for testing (`auto_pause_enabled=false`). Re-enable when ready for real volume.
6. Confirmed test order example: pickup code **5AD2BB** (Mozzarella Sticks) landed in Orders as `waiting`.

## Pay (current)

- `restaurant_settings.payment_mode = manual`
- Guest does **not** pay online. Pay at counter on pickup (cash / card terminal).
- Order rows stay `payment_status: pending`.
- Matches the settled Jigsy model: website free, ordering optional, pay at pickup, no Stripe in Phase 1 ([[business/Jigsys Website & Direct Ordering Master Plan]]).

## Print (current)

- **No** 80mm thermal / auto-print on accept yet (still on the master-plan wishlist).
- Ops today: staff watch Apex Orders on phone/tablet; Accept = kitchen starts; Complete when ready.
- Sensible next build: print (or browser print) on Accept, then optional **Mark paid** on Complete. Online card (Stripe) is a later slice.

## Migrations applied (menu)

- `20260801400000_jigsys_full_board_menu.sql` — full board SKUs + wing sauces
- `20260801410000_jigsys_menu_extras.sql` — toppings, sides, dressings, sub fries

## Source paths

- Apex: `C:\development\projects\apex_v2` · GitHub `nicholaswittle/apex_v2`
- Site: `C:\development\projects\jigsys_site` · GitHub `nicholaswittle/jigsysite` · `ordering.js`
- Archive (one-time dumps): [[restOS]] → `nicholaswittle/restOS`

## Next product steps

1. Kitchen ticket print on Accept (browser or Star/Epson).
2. Mark paid on Complete (still manual money).
3. Re-enable auto-pause after staffing test.
4. Confirm topping $ tiers with Emily / kitchen (board listed toppings without prices).
5. Optional later: prepaid card at checkout.

Related: [[Apex v2 — Restaurant OS Build]], [[customers/Jigsys Brewpub]], [[Jigsys Website Concept]], [[restOS]], [[wisense/projects/APEX_V2_OS_JIGSYS_INTEGRATION_AUDIT_2026-07-28]]
