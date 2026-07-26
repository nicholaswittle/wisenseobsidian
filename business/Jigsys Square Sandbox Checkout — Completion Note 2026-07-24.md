---
title: Jigsy's Square Sandbox Checkout — Completion Note
tags: [jigsys, square, sandbox, online-ordering, payments, demo]
date: 2026-07-24
status: sandbox-verified
---

# Jigsy's Square Sandbox Checkout — Completion Note

## Outcome

The reusable restaurant-ordering demo now has a working Square Sandbox
checkout. It uses fake cards and fake money only.

The full delayed-capture workflow was verified on the deployed site:

1. The customer enters a Square Sandbox test card.
2. Sending the order authorizes the complete test total.
3. Staff sees the order as **Sandbox authorized**.
4. **Accept & print** captures the authorization.
5. **Reject order** voids the authorization.
6. Only a captured order that staff marks completed counts toward the WiSense
   fee report.

## Live verification

| Test | Order | Result |
|---|---|---|
| Accept path | `J272265` | Authorized for $8.93, captured on staff acceptance, then marked completed |
| Reject path | `J758481` | Authorized for $8.93, voided on staff rejection, no WiSense fee added |

Square's standard Sandbox Visa was used. No real card or real money was
involved. The production worker showed no errors after the final tests.

## Controls and protections now working

- Square Sandbox is connected through OAuth to the **WiSense Test Restaurant**
  test location.
- The owner or manager can turn Sandbox card checkout on or return to manual
  pay-at-pickup from the staff Payments tab.
- Square access and refresh tokens stay server-side, encrypted in the hosted
  database.
- Access-token refresh is handled by the server.
- Customer card entry is hosted by Square's Web Payments SDK; the ordering
  application does not receive or store a card number.
- Customer order totals are recalculated on the server before authorization.
- The temporary payment-pending state is hidden from the kitchen queue.
- Staff cannot accept a Square order unless it has a valid authorization.
- A captured payment cannot be casually cancelled; a separate refund workflow
  must be built before production.
- No production Square mode or real-card charging is enabled.

## Current locations

| Resource | Location |
|---|---|
| Customer ordering | https://jigsys-ordering-demo.nicholaswittle.chatgpt.site/order-demo |
| Staff console | https://jigsys-ordering-demo.nicholaswittle.chatgpt.site/staff-demo |
| Demo repository | https://github.com/nicholaswittle/jigsysiteworking |
| Verified demo commit | `616a3faacdda187f5e4128ac34058ab8620b3132` |
| Deployed Sites version | 18 |

The deployed site is still a private WiSense practice site.

## What this proves

The proposed payment design is technically possible without Nicholas receiving
Jigsy's Square password, bank information, customer card numbers, deposits, or
payout access. In a real setup, an authorized Jigsy's owner would connect the
restaurant through Square's approval screen.

The $0.99 online-ordering fee can be included in the customer's Square total
while all customer funds go directly to Jigsy's. The software separately
records which completed orders count toward the later WiSense statement.

## Still required before a real Jigsy's pilot

- Written owner approval and connection of Jigsy's real Square location
- Confirmed menu prices, modifiers, tax treatment, and $0.99 fee wording
- Production Square application review and credentials
- Refund and cancellation rules plus a tested refund workflow
- Push notifications on the restaurant iPads
- Testing with the actual Star printer and Square station
- Customer confirmation method, support procedure, privacy terms, and outage
  fallback
- Small controlled live pilot before general release

Related notes:

- [[business/Jigsys Square Connection Requirements]]
- [[business/Jigsys Ordering Demo — Build Record 2026-07-23]]
- [[business/Jigsys Website [[Jigsys Website & Direct Ordering Master Plan Direct Ordering Master Plan]]
