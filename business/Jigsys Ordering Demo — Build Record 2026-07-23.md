---
title: Jigsy's Ordering Demo — Build Record
tags: [jigsys, business, online-ordering, demo, printer, vercel, github]
date: 2026-07-23
updated: 2026-07-24
status: demo-deployed
---

# Jigsy's Ordering Demo — Build Record

## Outcome

A separate, non-transactional clone of the Jigsy's website was built so new
ordering features can be demonstrated without changing the original concept
site or repository.

| Resource | Location |
|---|---|
| **Customer demo** | https://jigsys-ordering-demo.nicholaswittle.chatgpt.site/order-demo |
| **Staff screen** | https://jigsys-ordering-demo.nicholaswittle.chatgpt.site/staff-demo |
| **Demo home** | https://jigsys-ordering-demo.nicholaswittle.chatgpt.site |
| **GitHub** | https://github.com/nicholaswittle/jigsysiteworking |
| **Original concept** | https://github.com/nicholaswittle/jigsysite — unchanged |

The new repository is public. The local demo checkout tracks
`jigsysiteworking/main`; the original repository remains fetch-only with its
push URL disabled in the demo workspace.

## What was built

### 2026-07-24 demo expansion

- Expanded the customer ordering data to **52 priced items** across eight
  categories: house trays, specialty trays, gourmet trays, wings, stromboli
  and flatbreads, starters, salads, and subs and platters.
- The staff **Menu availability** screen now mirrors that full priced menu with
  category tabs. Switching an item off disables its customer order button.
- Checkout is blocked if an item becomes sold out after entering the cart.
- Staff can **Accept & Print** or **Reject** an order. Rejected orders remain in
  the daily record and earn no 99-cent fee.
- The active queue rolls over at midnight without deleting history. The Daily
  report can select an archived date and print all received, accepted,
  rejected, and still-waiting requests.
- Replaced the confusing combined “Due at pickup” dashboard number with
  **WiSense fees today**.
- The staff pickup estimate is adjustable from 10–90 minutes in five-minute
  steps. It is an estimate, not a countdown or expiration.
- The customer page prominently mirrors the current staff-set estimate.
- The latest customer request has a persistent **Waiting → Accepted / Not
  accepted** status card. Production still requires shared storage and an SMS
  notification so customers do not need to keep the page open.
- Added a linked **Created by WiSense LLC** credit.
- Restored the original Peanut Butter Pie feature and made all menu jump
  buttons equal width.

### Customer ordering

- Menu uses Jigsy's real categories, tray sizes, prices, wing sauces, and
  dressings from the existing concept.
- Customers can customize items, add notes, choose pickup timing, and submit a
  pickup request.
- Tray toppings are limited to four.
- Wings require a sauce; salads require a dressing.
- The total shows food, estimated tax, and a **$0.99 Online ordering fee**.
- No card fields exist. The customer pays Jigsy's in person at pickup.

### Staff screen

- Focused workflow: **Accept & Print Ticket**.
- Accepted orders can be reprinted.
- Staff can pause or reopen online ordering at any time.
- Staff can change estimated prep time.
- Staff can mark menu items sold out.
- The printable 80 mm kitchen ticket includes customer details, modifiers,
  notes, subtotal, estimated tax, the $0.99 online-ordering fee, and total due.
- Ticket explicitly says **Collect payment at counter** and **no online
  payment**.

### Demo infrastructure

- Deployed independently to Vercel under the WiSense account.
- Source committed to the separate `jigsysiteworking` repository.
- Printer and production-flow documentation lives in
  `docs/PRINTER-AND-ORDER-FLOW.md` in that repository.
- The original Jigsy concept website and `nicholaswittle/jigsysite` repository
  were not modified.

## Settled Jigsy-specific commercial model

- Core website and demo: **free to Jigsy's**.
- Online-ordering revenue: **$0.99 per accepted online order**.
- The 99 cents is added to the restaurant ticket and collected by Jigsy's with
  the food payment.
- WiSense does **not** process customer payments, hold restaurant funds, issue
  customer refunds, or manage chargebacks.
- Only accepted orders count.
- The intended settlement is a monthly statement:
  `accepted online orders × $0.99`.
- Jigsy's can pay WiSense separately by check, cash, or bank transfer.
- The broader `$299 setup + $79/month` website offer may remain valid for other
  clients, but it is **not the current Jigsy-specific offer**.

## Important limitation: current demo is not a live ordering system

The demo uses browser-local storage. Customer and staff screens only share
orders inside the same browser profile. A customer phone and a restaurant
tablet do not yet communicate.

Before a real pilot, build:

- Hosted order database shared by customer and staff devices
- Password-protected staff access
- Live new-order notifications
- Customer SMS for accepted/rejected confirmation
- Durable accepted-order and monthly-fee reporting
- Backup procedure for internet or printer failure

## Printer plan

### Cheapest pilot

Connect an 80 mm receipt printer to the computer running the staff screen.
**Accept & Print** opens the normal system print dialog. Staff selects the
receipt printer.

### Production option

After the printer model and connection type are known, add a print bridge:

- PrintNode for API-driven PDF or RAW remote printing
- QZ Tray for direct browser-to-printer ESC/POS printing

Do not promise silent or automatic printing until the actual Jigsy printer has
been identified and tested.

## Owner relationship status

- Emily showed the owner the concept website; the owner liked it.
- Nicholas later discussed it with the owner while helping at Jigsy's.
- The owner said she liked the website.
- The site visually suggested online ordering, which opened the conversation.
- Nicholas explained that ordering was not live but could be added and planted
  the idea.
- Current state: interested, but no formal ordering-pilot approval yet.

## Next conversation

Show the owner the isolated ordering demo and focus on:

1. No upfront website cost to Jigsy's.
2. Customers still pay at pickup.
3. Jigsy's can pause ordering whenever the kitchen is busy.
4. Staff only needs **Accept & Print**.
5. The 99-cent fee applies only to accepted online orders.
6. Ask what printer they currently use and where the staff screen would live.

Do not frame the demo as production-ready. Ask for a small, reversible pilot
only after confirming the staff device, printer, menu rules, and owner approval.

## Supporting materials

- [[Jigsys Website Concept]]
- [[customers/Jigsys Brewpub]]
- [[business/Jigsys Website & Direct Ordering Master Plan]]
- [[business/WiSense Operational Partner Plan — Jigsy's]]

## Material revision required

The previously generated planning-pack and owner leave-behind PDFs were written
before the final **pay at pickup / no Stripe / monthly accepted-order
statement** decision. Do not hand them to the owner until they are regenerated
with the current model.
