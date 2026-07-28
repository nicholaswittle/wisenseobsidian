---
title: Jigsy Online Ordering — Live Status 2026-07-28
tags: [jigsy, ordering, apex, restaurant-os, pilot]
aliases: [Jigsy order online, print and pay]
date: 2026-07-28
updated: 2026-07-28
status: active
---

# Jigsy Online Ordering — Live Status 2026-07-28

Guest ordering is live on the Jigsy site, wired to Apex Supabase. Staff run tickets in **Apex Orders** and/or **staff.html**. Pay is at pickup; Accept prints the kitchen ticket.

## Live links

| Surface | URL |
|---------|-----|
| Jigsy site (Order online) | https://jigsyssite.vercel.app |
| Staff console (HTML) | https://jigsyssite.vercel.app/staff.html |
| Apex real app | https://apex-v2-ten.vercel.app |
| Apex demo | https://apex-v2-demo.vercel.app |
| Supabase | `pqkremkwfkudrhtxasdj` |
| Restaurant | `public_token = jigsys` · id `a1000000-0000-4000-8000-000000000001` · org `c82c025d-7300-4d41-a84d-bc17d3c3104f` |

## What works today

1. **Order online** on the site → live menu/capacity → cart → `place_order` → pickup code.
2. **Full Nov 2025 board** in Supabase (trays, specialty, gourmet, wings, stromboli/flatbreads, starters, salads, subs, dessert). **No alcohol online** (House Brews To-Go removed).
3. **Extras / modifiers:** tray toppings by cut, wing sauce/sides, salad dressings, sub fries, fries cheese/bacon.
4. **Staff (Apex or staff.html):** Waiting → **Accept & print** (one tap; ticket prints; kitchen done). Reject if needed. Re-print from Done. Pay at counter is separate — no “Paid & done” click.
5. **Menu availability:** category tabs / inventory icon — tap item to sold-out or restock. Same `menu_items.available` for app + HTML + guest site (realtime).
6. **Pause online orders:** hides all `[data-apex-open]` CTAs (website-only sell mode).
7. **Capacity:** auto-pause off for testing (`auto_pause_enabled=false`).

## Pay (current)

- `restaurant_settings.payment_mode = manual`
- Guest does **not** pay online. Pay at counter on pickup.
- Accept does **not** mark `payment_status` paid — money is a counter concern.

## Print (current)

- Browser print ticket fires on Accept (staff.html + Apex dialog).
- Thermal Star/Epson auto-print still a later upgrade.

## Migrations (menu / stock)

- `20260801400000` — full board
- `20260801410000` — extras
- `20260801500000` — `apex_set_menu_item_available` RPC (any org member can 86)
- `20260801510000` — remove online alcohol SKUs

## Source paths / GitHub HEADs (2026-07-28 afternoon)

- Apex: `nicholaswittle/apex_v2` @ `dc40da8`
- Site: `nicholaswittle/jigsysite` @ `4505cd4`
- Archive: [[restOS]] `nicholaswittle/restOS` @ `a6cb554`

## Next product steps

1. Optional thermal printer on Accept.
2. Re-enable auto-pause after staffing test.
3. Confirm topping $ tiers with Emily / kitchen.
4. Optional later: prepaid card at checkout.

Related: [[Apex v2 — Restaurant OS Build]], [[customers/Jigsys Brewpub]], [[Jigsys Website Concept]], [[restOS]], [[wisense/projects/APEX_V2_OS_JIGSYS_INTEGRATION_AUDIT_2026-07-28]]
