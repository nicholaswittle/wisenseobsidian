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
- Unified the public experience around **one website**. When staff pauses
  ordering, every public pickup-order entry point disappears; the menu, hours,
  phone, directions, and restaurant content remain available as a normal
  website. Staff can reopen ordering during a slower period without WiSense.
- Corrected the final paused-state styling bug that left the large hero
  **Preview online pickup** button visible after the other ordering links were
  hidden.

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

- Emily first showed the concept website to an owner, who liked it.
- On 2026-07-24 Nicholas presented the owner deck and live customer/staff demo
  in person to **two of the three ownership stakeholders**.
- Both owners appeared positive. They are waiting for their son, the remaining
  stakeholder, to review it before making a group decision.
- The owners described online ordering as a competitive gap: in their view,
  Jigsy's is the only restaurant in the immediate area without it.
- The **pause online ordering** control was a particularly strong feature for
  them because it lets staff turn ordering off during a rush and keep using the
  site as a normal restaurant website.
- They were comfortable with the staff console being a Home Screen web app on
  their existing iPad and with accepted orders printing as ordinary kitchen
  tickets.
- Nicholas explained the proposed **$0.99 per accepted online order** model.
  They asked whether online-order fees are normal but did not reject the fee.
  Exact approval and customer-facing wording are still pending.
- Current state: **two-owner positive signal; final stakeholder review and
  formal pilot approval still pending**.

## 2026-07-24 hardware evidence

Two on-site photographs confirmed:

- The checkout device is an **iPad running Square Point of Sale in a Square
  Stand**.
- Square's device bar showed both **Stand** and **Printer** connected.
- The existing printer is a **Star Micronics TSP100 futurePRNT-series direct
  thermal printer**.
- The front branding identifies the printer family but not the exact model or
  connection type. TSP100 variants can use USB, Ethernet, Wi-Fi, or Bluetooth.

The original photographs remain local and were not added to the public vault
repository.

### Evidence still needed

1. Photograph the printer's model label underneath or behind it without
   unplugging it.
2. Photograph Square's **More → Settings → Hardware → Printers** screen,
   including the connection and printer profile.
3. Check **Settings → General → About** for the iPad model and iPadOS version;
   do not record or publish the serial number.

## Feasibility after owner meeting

**Overall assessment: feasible, with one unresolved integration risk.**

Confirmed:

- The owners accept the iPad web-app operating model.
- A Home Screen web app can provide staff push notifications on iPadOS 16.4 or
  later after staff explicitly enables notifications.
- The existing Square station already has a working printer connection.
- Pay at pickup, owner-controlled pause/reopen, accept/reject, and the
  99-cent accepted-order accounting model remain compatible with the proposed
  workflow.

Still to validate:

- A browser web app does not automatically inherit the printer configured
  inside Square.
- Square can create and manage fulfillment orders through its Orders API, but
  the documented automatic Point of Sale visibility path emphasizes fully paid
  fulfillment orders. Jigsy's proposed orders stay unpaid until pickup.
- Therefore, **Accept & Print** must be tested against Jigsy's actual unpaid
  pickup workflow before silent automatic printing is promised.

### Printing paths, in preferred order

1. **Square-managed path:** after acceptance, create or transfer the pickup
   order into Square and let the existing Square printer profile produce the
   ticket. This is the cleanest experience if an unpaid-order test succeeds.
2. **Dedicated print bridge:** send the accepted ticket through a compatible
   local/network print service after the exact TSP100 variant is known.
3. **Pilot fallback:** staff accepts the request, then manually enters or
   confirms it in Square while the automatic path is being validated.

No new printer should be purchased until the existing model label, connection
type, Square printer profile, and unpaid pickup behavior are tested.

References:

- [Apple: Sending web push notifications in web apps and browsers](https://developer.apple.com/documentation/UserNotifications/sending-web-push-notifications-in-web-apps-and-browsers)
- [WebKit: Web Push for Web Apps on iOS and iPadOS](https://webkit.org/blog/13878/web-push-for-web-apps-on-ios-and-ipados/)
- [Square: Orders API](https://developer.squareup.com/docs/orders-api/what-it-does)
- [Star Micronics: TSP100III specifications](https://starmicronics.com/Resources/UploadedDataSheet/TSP100IIIWLAN_LAN_BT_USB.pdf)

## Notification plan

- Install the staff console on each approved iPad as a Home Screen web app.
- Add an explicit **Enable order alerts** setup step.
- Send a push for every new request with the order number, total, and a link to
  accept or reject.
- Show an app-icon badge for waiting orders.
- Add an in-console chime and prominent visual alert while the console is open.
- Notify both approved staff iPads for redundancy.
- During the pilot, escalate an unacknowledged order after 60–90 seconds; SMS
  can be added later if the owners want a paid fallback channel.

## Next conversation

The remaining ownership stakeholder—the owners' son—needs to review the demo.
Keep that walkthrough focused on:

1. The website remains useful when ordering is paused.
2. Staff receives a notification and accepts or rejects every request.
3. Customers pay at pickup through the existing Square workflow.
4. The proposed 99-cent fee applies only to accepted online orders.
5. The first test is small and reversible.

Do not frame the demo as production-ready. After stakeholder approval, run a
hardware/workflow validation before committing to a live pilot.

## Supporting materials

- [[output/Jigsys_Owner_Leave_Behind.pdf]] - five-page owner handout.
- [[output/Jigsys_Complete_Planning_Pack.pdf]] - private proposal, meeting
  guide, and material audit.
- [[Jigsys Website Concept]]
- [[customers/Jigsys Brewpub]]
- [[business/Jigsys Website & Direct Ordering Master Plan]]
- [[business/WiSense Operational Partner Plan — Jigsy's]]

## Material revision completed

The planning pack and five-page owner leave-behind were regenerated on
2026-07-24. They now use the final **pay at pickup / no Stripe / monthly
accepted-order statement** model and lead with the one-website pause strategy.
