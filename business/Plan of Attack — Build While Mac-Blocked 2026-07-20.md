---
title: Plan of Attack — Build While Mac-Blocked 2026-07-20
tags: [business, strategy, plan, revenue, web, apex, roadmap]
aliases: [Plan of Attack, Mac-Blocked Plan, Web-First Plan]
date: 2026-07-20
status: active
---

# 🎯 Plan of Attack — Ship Revenue While iOS Is Blocked

> Constraint: no Mac/Xcode for ~2 weeks. **Finding: this blocks iOS only — not revenue.** Verified against the repos 2026-07-20.

## The reframe: you are not blocked

| Product | Web? | Android? | iOS | Verdict |
|---|---|---|---|---|
| **Apex** | ✅ **Vercel pipeline already built** (`vercel.json`, `scripts/build_web.sh`, `web/manifest.json` PWA) | ✅ no Mac needed | ⛔ needs Mac | **Ship web NOW** |
| **COMMS LINK** | ❌ native-only (`flutter_gemma`, `local_auth`, `flutter_secure_storage`) | ✅ no Mac needed | ⛔ needs Mac | **Ship Google Play NOW** |

**Strategic consequence:** for the next two weeks, build **web-delivered software sold by Stripe link** — not mobile apps. Web sidesteps the blocker entirely, avoids the 15–30% app-store cut, needs no review approval, and ships updates instantly. Mobile becomes a *distribution channel added later*, not a prerequisite to revenue.

---

## Scoring: ROI vs. speed (only things sellable without an app store)

| # | Play | Build effort | Time to cash | ROI | Needs store? |
|---|------|---|---|---|---|
| **A** | Apex web pilot → Jigsy's live + venue #2 | **None (deploy)** | 1–2 wks | ⭐⭐⭐⭐⭐ | No |
| **B** | Shift Compliance Checker (Fair Workweek) — free scan → paid | **Low-Med** | 2–4 wks | ⭐⭐⭐⭐⭐ | No |
| **C** | Tip Pooling Calculator (web micro-tool) | **Low** | 2–4 wks | ⭐⭐⭐⭐ | No |
| **D** | Tier 0 services (AI automation retainers) | **None** | 2–4 wks | ⭐⭐⭐⭐ | No |
| **E** | COMMS LINK → Google Play | Low (packaging) | 3–6 wks | ⭐⭐⭐ | Play only |
| **F** | COMMS LINK B2B org licensing | Med | 8–16 wks | ⭐⭐⭐⭐ | No (direct sale) |

**Everything in Tier 3 of [[business/Revenue Ideas — 12 Buildable B2B Plays 2026-07-20]] stays parked.** No design partner = no.

---

## THE PLAN

### Week 1 — Turn Apex into a live, referenceable product
Goal: **one real venue running daily on Apex.** This is the asset everything else is sold on.

1. **Apply the RLS migration** (`20260720000000_launch_blockers_rls.sql`) → Supabase staging → smoke-test → prod. *(Only human-gated step; do it first.)*
2. **Deploy Apex web to Vercel** — pipeline already exists. Custom domain e.g. `apex.wisense.*`.
3. **Verify PWA install** on an Android and an iPhone (Safari → Share → Add to Home Screen). Staff get an app icon with **no store involved** — this is the key unlock; iPhone staff are covered *today*.
4. **Onboard Jigsy's for real**: owner + full staff roster, publish a real week, run clock-in/out for actual shifts.
5. **Instrument the number that sells venue #2**: hours/week saved for the owner, swap resolution time. Log in [[customers/Jigsys Brewpub]].

### Week 2 — Make it purchasable + build the wedge
6. **Stand up billing.** Stripe is already scaffolded (`create-payment-intent`, `stripe-webhook`, `billing_page.dart`, `AppConfig.billingEnabled`). Flip it on with **flat per-location pricing** — the direct attack on competitors' per-seat tier creep. Start: **$99/location/mo**, first venue free/discounted as the design partner.
7. **One landing page** — problem, 3 screenshots, price, "Start free trial" → Stripe Checkout. Nothing else. Host on Vercel.
8. **Start building B: Shift Compliance Checker** (see below).
9. **In parallel (async):** COMMS LINK keystore + `.aab` → Play Console internal testing. Costs $25 and a few hours; runs while other work proceeds.

### Weeks 3–4 — Ship the wedge that gets you venue #3–20
**B. Shift Compliance Checker** — the highest-ROI *new* build.
- **Why:** Fair-workweek laws are live in **15 cities + 4 states** (advance posting, predictability pay, right-to-rest) with per-employee penalties. Most independents don't know they're exposed. Fear + a budget line = fast close, low churn.
- **Build (reuses Apex's schema — this is why it's fast):**
  - Jurisdiction rules table (start with 2–3 cities you can actually sell into).
  - Validator over the existing `shifts` data: posting lead-time, rest gaps, change-premium triggers.
  - **Preventive, not retrospective** — flag at *publish time* + keep an audit log. (Competitors ship a report after the fact.)
- **Go-to-market — the clever part:** ship the **free single-schedule scan** as a public web tool. Owner uploads/pastes a schedule → sees violations → "fix this automatically inside Apex." It's a lead magnet *and* the paid feature's demo.
- **Money:** $49–79/location/mo add-on, or bundled in an Apex "Pro" tier.

---

## How people actually buy (no app store)

1. **Stripe Payment Link / Checkout** on the landing page — live in an hour, no store review.
2. **Self-serve trial** → org signup already exists (`apex_create_organization` RPC).
3. **Direct/warm sales for the first 10** — this is the real channel. Ask the Jigsy's owner for intros to 3 other operators; local hospitality is a tight network and a peer reference outsells any ad.
4. **The free compliance scan** as top-of-funnel once B ships.

**Do not** build a self-serve growth engine before ~10 paying venues. First 10 are sold by hand.

---

## When the Mac lands
Nothing above gets thrown away. Add iOS as a *channel*: Apex iOS build + COMMS LINK App Store via Xcode or Codemagic. See [[output/Gate C — Android Packaging & Store Listings 2026-07-20]].

## Kill criteria
- No second paying venue by **~week 12** → problem is distribution, not product. Stop building; sell full-time.
- Compliance Checker gets <5 free scans in its first 2 weeks live → the fear thesis is wrong; drop it and put the time into services (D).

## Why this ordering
Every week Apex isn't live at Jigsy's is a week with **no reference customer**, and the reference customer is what makes plays B, C, and F sellable. Sequence is: *make one venue succeed → prove a number → sell that number.*

---

Related: [[business/Revenue Ideas — 12 Buildable B2B Plays 2026-07-20]], [[business/WiSense Service as a Software Execution Strategy]], [[business/Go-to-Market]], [[business/Pricing Strategy]], [[Apex Scheduler]], [[COMMS LINK]], [[customers/Jigsys Brewpub]], [[output/Gate C — Android Packaging & Store Listings 2026-07-20]], [[NOW]], [[business/Ideas Log]]
