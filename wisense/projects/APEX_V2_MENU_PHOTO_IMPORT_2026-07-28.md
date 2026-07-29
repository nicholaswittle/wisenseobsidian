---
title: Apex v2 — Menu Extras + Photo Import 2026-07-28
tags: [apex, menu, vision, anthropic, decision]
date: 2026-07-28
status: decided
---

# Apex v2 — Menu Extras + Photo Import (2026-07-28)

## Shipped
- **Extras/toppings editor** in Edit Menu (modifier groups/options; edit name/price; Toppings/Sauce/Size packs; custom sets)
- **Speed:** duplicate item (+extras), quick paste `Name, 12.99`, photo → menu
- **Edge:** `parse-menu` + `menu_text_parser.dart`; schedule parser model updated too
- **Secret:** `ANTHROPIC_API_KEY` on project `pqkremkwfkudrhtxasdj`
- **Live:** https://apex-v2-ten.vercel.app

## Decision — vision model
**Keep `claude-sonnet-4-5`** for menu + schedule photo import.

| Option | Tradeoff |
|--------|----------|
| Sonnet 4.5 (current) | Better messy/dense menus; ~$0.02/photo in smoke test |
| Haiku 4.5 | Cheaper/faster; more miss risk on bad photos |

**Why Sonnet:** imports are rare (setup, not every order). Accuracy beats pennies. Switch to Haiku if venues start batch-importing many photos/day.

## Related
[[Apex v2 — Restaurant OS Build]] · [[hot]]
