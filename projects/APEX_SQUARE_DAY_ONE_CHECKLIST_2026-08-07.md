---
title: Apex Square Day-One Checklist + Vercel Domain
tags: [apex, square, payments, launch, vercel, domain, checklist]
aliases: [Square Day One, Square Go Live, Vercel Domain]
date: 2026-08-07
---

# Apex Square Day-One Checklist + Pointing the Vercel Site at Jigsy's Domain

**Goal:** get the Square rail live so Jigsy's orders push into their real Square
account, tickets print on their existing printer, and the 1.5% lands in your
account — without Jigsy's switching to Stripe.

**Source of truth:** `apex_v2/docs/SQUARE_DAY_ONE_LIVE_VALIDATION.md` and
`SQUARE_ONLINE_TIP_ONBOARDING.md`. This is the runnable version.

---

## Part 0 — The money-safety check (do this FIRST)

The code default is `square_environment = 'sandbox'`. The fee only lands in
production. **Before anything else, confirm the live value:**

- [ ] In Supabase dashboard → Table Editor → `restaurant_settings` → Jigsy's row
- [ ] Check `square_environment`:
  - `production` → good, fee will land
  - `sandbox` → **orders go to a sandbox account and NO fee is collected** — the
    silent-failure case. Set it to `production` before going live.

> ⚠️ The vault's standing warning: *"a production venue whose row was never
> updated takes real money while Apex collects nothing, no error anywhere."*

---

## Part 1 — Square OAuth (owner must do this)

- [ ] **Owner login required:** `jigsy895@yahoo.com` (account "Jigsy's Inc").
  Emily is a trusted manager with **no** Square authority — she cannot authorize.
- [ ] In the Apex staff console, tap **Connect Square** (or the equivalent
  button). This starts the OAuth flow.
- [ ] Owner approves on Square's page. This is the **production** OAuth (real
  account, not sandbox).
- [ ] Confirm the venue row now has `square_merchant_id` and `square_location_id`
  populated, and `square_charges_enabled = true`.

---

## Part 2 — Square POS & printer setup (owner/manager, in Square)

- [ ] **Order creation enabled** in Square (per the day-one doc).
- [ ] **Printer profile:** confirm one profile has **"Online & kiosk order
  tickets"** and **"Automatically print new orders"** enabled.
- [ ] **PICKUP** fulfillment is attached to that printer profile. (The code
  routes on `type: PICKUP` — get this wrong and the ticket does not print.)
- [ ] **`Checkout → Order tickets`** — currently `Manual`. Confirm whether it
  needs to change to auto-print. This affects their existing in-person workflow,
  so it needs the owner's agreement, not just ours.
- [ ] **Tipping:** confirm Square's **Pool tips per transaction** is appropriate
  for Jigsy's (see Part 4). Square hosted tips are credited to the merchant and
  carry no `team_member_id`, so Apex cannot assign them to an individual.

---

## Part 3 — The live test (with Jigsy's present)

Square Sandbox cannot test POS delivery, printing, or the app fee. All of this
must be verified in production with the merchant present.

- [ ] Confirm `square_environment = 'production'` (Part 0).
- [ ] **Send test order** from the staff console. Confirm the labelled test
  order reaches Square POS / Order Manager and its ticket prints automatically.
  Remove the recorded one-cent external test payment afterward.
- [ ] Place **one real, low-value production Pay Now order.** Confirm it's
  visible in Square POS Order Manager and the ticket prints.
- [ ] In Square's **Banking** tab, confirm the payment shows Apex's
  `app_fee_money`. **This is the only proof the 1.5% landed** — not available in
  Sandbox.
- [ ] On the hosted Square checkout, select a tip. Confirm the completed payment
  has the tip in Square AND the matching Apex `online_orders` row has the same
  `tip_cents`. The order's base total must not change from the post-checkout tip.

---

## Part 4 — Tip policy (owner confirmation, before enabling Pay Now)

- [ ] Obtain the owner's **written confirmation** that Square's **Pool tips per
  transaction** is appropriate for Jigsy's.
- [ ] Confirm the venue's payroll policy for pooled card tips (card tips paid
  through payroll every two weeks; cash tips outside the system).
- [ ] In Square, configure eligible clocked-in team members for **Pool tips per
  transaction**.
- [ ] Place the first live order only after confirming the hosted tip screen and
  Apex's `tip_cents` record agree.
- [ ] Reconcile the day's Apex **Tips today** metric to Square before payroll.

---

## Part 5 — Point the Vercel site at Jigsy's actual domain

The multi-tenant guest site is `wisense-apex.vercel.app` (Vercel project
**apex-site**). To serve it on Jigsy's own domain:

### Option A — Custom domain on the existing Vercel project (simplest)

- [ ] In Vercel → project **apex-site** → **Settings → Domains**
- [ ] Add Jigsy's domain (e.g. `jigsyspizza.com` or whatever they own)
- [ ] Vercel shows the DNS records to add. At the domain registrar (wherever
  Jigsy's bought the domain), add the CNAME / A record Vercel gives you.
- [ ] Wait for DNS propagation (minutes to hours). Vercel auto-provisions the
  SSL cert.
- [ ] **Update `site_url`** in the Jigsy's `restaurant_settings` row to the new
  domain, so `create-guest-payment` builds the return URL from the right origin
  (the code reads `site_url` and only honors https + allowlisted origins).

### Option B — Point the domain at the site via the registrar

- [ ] If Jigsy's domain is hosted elsewhere, add a CNAME from `www` (and/or the
  apex) to `wisense-apex.vercel.app`, then add the domain in Vercel as above.

### What NOT to do

- [ ] Do NOT change `APEX_VENUE_SITE_URL` env var to the domain unless you want
  it to apply to ALL venues. It's the global fallback. Per-venue domains go in
  `site_url` on the row.
- [ ] Do NOT point the domain at `apex-v2-ten.vercel.app` — that's the staff
  app, not the guest site. A guest returned there hits a login wall.

---

## Definition of done

- [ ] `square_environment = 'production'`
- [ ] OAuth complete (owner), merchant + location id populated
- [ ] Test order prints on Jigsy's printer
- [ ] One real order reaches Square POS Order Manager
- [ ] `app_fee_money` (1.5%) visible in Square Banking tab
- [ ] Tip reconciles between Square and Apex
- [ ] Guest site serves on Jigsy's domain, `site_url` updated

When all of these pass, Jigsy's is a **paying venue on the Square rail** — not a
free pilot.

---

Related: [[projects/Apex v2 — Restaurant OS Build]] · [[projects/APEX_PAYMENTS_AND_POS_STRATEGY_2026-07-31]] · [[projects/APEX_SQUARE_PROVIDER_DECISION_2026-07-30]] · [[projects/JIGSYS_PILOT_LAUNCH_STRATEGY_2026-07-31]] · [[hot]] · [[NOW]] · [[index]]
