---
title: FFL Dealer Website Research — Features and Best Practices
tags: [prospects, research, ffl, gun-store, website, features, Thompsontown]
date: 2026-07-26
status: active
---

# FFL Dealer Website Research — Features and Best Practices

> Research conducted 2026-07-26 by analyzing real FFL dealer websites (The Tactical Store, Shooter's Supply, Sportsman's Outdoor Superstore) and their FFL transfer pages. Findings applied to the Thompsontown FFL dealer pitch.

Related: [[prospects/Thompsontown FFL Dealer — Warm Lead Strategy]], [[business/Reusable Platform Components 2026-07-26]], [[business/Pricing Models Ownership and Exit 2026-07-26]], [[NOW]]

---

## Methodology

Analyzed 3 real FFL dealer websites for features, navigation structure, and FFL transfer process. Also examined their FFL transfer forms and information pages to understand what data they collect from customers.

## Common Features Found Across All FFL Dealer Sites

### 100% of sites had:
- FFL transfer information or form
- Phone number prominently displayed
- Store hours
- Map/directions
- Facebook page link
- E-commerce or product browsing (firearms, ammo, accessories)

### 67% of sites had:
- NFA/Class 3/suppressor information
- Shooting range information
- Reviews/testimonials
- Newsletter signup
- Blog
- Instagram
- Email contact

### 33% of sites had:
- Gunsmithing/repair services
- FAQ page
- Payment info
- Shipping info
- Policies page
- Layaway
- Consignment
- Special orders

---

## FFL Transfer Process — What Real Dealers Do

### How FFL transfers work (from Shooter's Supply):

**Inbound transfer (customer buys from out-of-state dealer):**
1. Customer buys a firearm online (GunBroker, PSA, Bud's, etc.)
2. The selling dealer needs a copy of the receiving FFL's license
3. Customer contacts the local FFL, provides their info and the firearm details
4. Selling dealer ships the firearm to the local FFL
5. Customer comes in, fills out ATF Form 4473, passes NICS background check
6. Customer pays transfer fee ($25-35 typical) and picks up the gun

**What the FFL needs from the customer:**
- Customer name and phone number
- Firearm info: make, model, serial number, caliber, type
- Selling dealer info (name, contact, FFL number)
- Order number or reference

**What the FFL needs from the selling dealer:**
- A copy of their FFL license (emailed or faxed)
- The firearm shipped to their address

### Transfer fees found:
- The Tactical Store: $30.00 flat
- Typical range: $25-35 per firearm

---

## The FFL Transfer Form — Most Important Feature

This is the #1 feature for a small FFL dealer website. It captures warm leads (people actively buying firearms who need a transfer) and saves phone time.

### What the form should collect:

**Customer info:**
- Full name
- Phone number
- Email address

**Firearm info:**
- Make (manufacturer)
- Model
- Caliber
- Type (handgun, rifle, shotgun)
- Serial number (if known)

**Transfer details:**
- Selling dealer name
- Selling dealer phone/email
- Order/reference number
- Has the selling dealer been contacted about shipping? (yes/no)

**Additional:**
- Notes/special instructions
- Preferred pickup time

### What happens when the form is submitted:
- Email notification to the shop owner
- Customer gets an auto-reply: "We received your transfer request. We'll contact you when the firearm arrives."
- Shop owner follows up with the selling dealer if needed
- Customer comes in for pickup, pays fee, fills out 4473

---

## Recommended Website Structure for the Thompsontown FFL Dealer

Based on the research, here's what to build:

### Must-have pages:
1. **Home** — Store name, FFL badge, phone, hours, hero image
2. **FFL Transfer Request** — Online form (the money-maker)
3. **Services** — FFL transfers, manufacturing/gunsmithing, NFA/Class 3, special orders
4. **Inventory Inquiry** — "Looking for something specific?" form
5. **Hours & Directions** — Google Maps embed, store hours, phone
6. **Contact** — Phone, email, form, Facebook

### Nice-to-have pages (add later):
7. **FAQ** — Transfer process, NICS info, wait times, what to bring
8. **About** — Store history, FFL credentials, staff
9. **NFA/Class 3 Info** — Suppressor process, Form 4 help, timeline

### Key design elements:
- Tactical dark aesthetic (charcoal, steel grey, muted brass/bronze accents)
- FFL number displayed as trust badge (with owner's permission)
- Phone number in the header (click-to-call on mobile)
- Hours in the footer (every page)
- Facebook link
- Mobile-first (most gun buyers search on their phone)

---

## What NOT to Include (for a small Type 07 FFL)

- Full e-commerce/shopping cart — too complex for a first version, and firearms sales online have heavy legal requirements. Start with inquiry forms, not online sales.
- Background check portal — NICS is done by the FFL, not the website
- Online payment for firearms — legal complications, use "pay at pickup" model
- Live inventory — unless the owner wants to manage it. Start with "inquiry" forms.

---

## The Pitch for the Thompsontown Dealer

Based on research, the strongest pitch is:

1. **FFL Transfer Request Form** — "Guys buying on GunBroker can't find you. This form lets them request a transfer from their phone. One transfer per week pays your entire monthly fee."

2. **Professional web presence** — "Right now you're only on GunMade. Buyers can't find your hours, your transfer fee, or how to contact you. I'll fix that."

3. **Zero technical work for the owner** — "You never touch code, domains, or servers. I handle everything. $79/mo, cancel anytime."

4. **Censorship-proof hosting** — "Shopify and Squarespace restrict gun stores. My hosting doesn't. Your site stays up."

---

## Revenue Math for This Client

- Transfer fee: $25-35 per firearm
- 1 transfer/week = $100-140/mo in transfer fees alone
- Your monthly fee: $79/mo (warm lead, inbound pricing)
- The site pays for itself with ONE transfer per month
- At 4 transfers/month, the owner makes $100+ profit after paying you

---

## Comparison: What Real FFL Sites Have vs What We'll Build

| Feature | Real FFL Sites | Thompsontown Demo | Notes |
|---|---|---|---|
| FFL transfer form | Yes (all) | Yes | #1 priority |
| Phone number | Yes (all) | Yes | Click-to-call |
| Store hours | Yes (all) | Yes | In footer |
| Map/directions | Yes (all) | Yes | Google Maps embed |
| Facebook | Yes (all) | Yes | If they have one |
| NFA/Class 3 info | 67% | Yes | Type 07 eligible |
| Gunsmithing services | 33% | Yes | Type 07 = manufacturer |
| E-commerce | 67% | No (phase 1) | Too complex for v1 |
| Shooting range | 67% | No | Only if they have one |
| FAQ | 33% | No (phase 2) | Add after launch |
| Blog | 67% | No | Not needed for small dealer |
| Newsletter | 67% | No | Not needed for small dealer |

---

## Build Plan (for this weekend)

1. Ask Claude to generate a tactical-themed one-page site with:
   - Hero with store name + FFL trust badge
   - FFL transfer request form (name, phone, email, firearm details, selling dealer info, notes)
   - Services section (transfers, manufacturing, NFA, special orders)
   - Hours + phone + address + Google Maps embed
   - Dark tactical aesthetic (charcoal, steel, brass)
2. Deploy to Vercel (free)
3. Text the owner the live link
4. If he likes it: $0 setup + $79/mo (warm lead pricing, 3-month minimum)