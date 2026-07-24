# Jigsy's Square Connection Requirements

## Purpose

This note records what WiSense needs to connect Jigsy's online ordering system
to Square, what the owners need to provide, and how the connection should be
made without WiSense handling the restaurant's passwords or banking details.

## What Jigsy's needs to provide

- [ ] Name of the owner or Square administrator who is authorized to approve
  the connection.
- [ ] Confirmation of which Jigsy's Square location should receive online
  orders.
- [ ] Confirmation that the Square account is active and approved for online
  card payments.
- [ ] Confirmation that the existing Square menu and prices are accurate enough
  to use as the online menu source.
- [ ] Permission for the WiSense application to read the menu, create and
  update orders, authorize or cancel payments, and verify payment status.
- [ ] Decision from Jigsy's or its accountant about whether the $0.99 online
  ordering fee is taxable.
- [ ] Approved cancellation and refund rules.
- [ ] Name and contact information for the owner who should receive a warning
  if the Square connection stops working.

## Information WiSense does not need

WiSense should never ask for or receive:

- A Square username or password
- Bank account or routing information
- Personal Square access tokens
- Card-processing secrets copied from Jigsy's account
- Access to deposits, payouts, payroll, or employee information
- Customer card numbers

## How the connection is obtained

1. WiSense creates and owns the ordering application in the Square Developer
   Console.
2. WiSense adds a **Connect Square** button to the restaurant setup screen.
3. An authorized Jigsy's owner presses the button and signs in on Square's own
   secure page.
4. Square shows the exact permissions requested.
5. The owner presses **Allow** and selects the correct restaurant location.
6. Square sends the WiSense server a limited authorization. The owner's
   password is never shown to WiSense.
7. The connection can be viewed, disconnected, or reauthorized by the owner.

## Minimum Square permissions

| Square permission | Plain-language purpose |
| --- | --- |
| `MERCHANT_PROFILE_READ` | Identify the restaurant and correct location |
| `ITEMS_READ` | Read menu items, modifiers, prices, and taxes |
| `ORDERS_READ` | Check order status |
| `ORDERS_WRITE` | Create and update online orders |
| `PAYMENTS_READ` | Confirm whether a payment succeeded |
| `PAYMENTS_WRITE` | Authorize, complete, cancel, or refund a payment |

WiSense should not request bank-account, payout, payroll, employee, or
stored-card permissions.

## Recommended payment and acceptance flow

1. The customer builds an order.
2. Checkout shows the food, tax, and the $0.99 online ordering fee.
3. Square securely collects the customer's payment information and authorizes
   the full amount without completing the charge.
4. The order appears on the Jigsy's staff console.
5. If staff accepts the order, the payment is completed, the ticket prints, and
   the customer receives confirmation.
6. If staff rejects the order, the authorization is cancelled, the customer is
   not charged, and no WiSense fee is counted.

Square calls this process **delayed capture**.

## How the $0.99 WiSense fee is handled

The full customer payment, including the $0.99 online ordering fee, goes
directly into Jigsy's Square account. WiSense does not split, receive, or hold
the customer's payment.

The ordering system records one $0.99 WiSense fee only when an order is
accepted and successfully paid. A daily and monthly report shows:

- Completed online orders
- Rejected, cancelled, and failed orders
- Total online ordering fees collected
- Amount Jigsy's owes WiSense

Jigsy's pays WiSense separately at the agreed billing time.

## Backup if Square is not connected

The ordering system can operate in manual pay-at-pickup mode:

1. Staff accepts the web order and prints the ticket.
2. Staff enters the order into Square manually.
3. Staff adds a fixed **Online Order Fee - $0.99** line.
4. The customer pays Jigsy's at pickup.
5. Staff marks the online order **Paid / Completed**.

Because the website cannot verify a manual Square payment automatically, only
orders marked completed should count toward the WiSense monthly fee.

## Reusable product direction

The ordering platform should be built as a reusable restaurant system rather
than a Jigsy's-only codebase. Restaurant-specific information should be
configuration:

- Restaurant name, logo, colors, and contact information
- Menu, modifiers, prices, taxes, and availability
- Hours and pickup-time options
- Online ordering fee and tax treatment
- Manual payment or Square-connected payment mode
- Staff access, notification, printer, and reporting settings

Jigsy's can be the first restaurant configuration. If Jigsy's waits or declines,
the same platform can be demonstrated and configured for another restaurant
without rebuilding the ordering system.

