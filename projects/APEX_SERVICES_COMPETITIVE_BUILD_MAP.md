---
title: "Apex Services — Competitive Build Map"
tags: [apex, services, plan, competitive, build-map]
date: 2026-08-03
source: "[[projects/APEX_AUDIT_FABLE5_2026-08-03]]"
---

# Apex Services — Competitive Build Map

Built from the Fable 5 audit, ordered by the question Nick asked:
**"if we can't compete out the gate, what's the point?"**

---

## The thesis: you cannot out-feature Jobber, and you do not have to

Jobber has ~10 years, a team, invoicing, a client CRM, automations, QuickBooks
sync and an install base. Feature parity is not reachable by one person with a
full-time job, and chasing it loses slowly.

What is reachable is a **different shape**:

| Jobber / Housecall Pro | Apex |
|---|---|
| $29–39 **per user**, tiers at 5/10/15 seats | **Flat. Unlimited crew.** |
| Add-ons: $99 AI receptionist, $79 marketing | **No add-ons, ever** |
| Dispatch is top-down — office assigns | **Crew claims open slots themselves** |
| Quote is a PDF or an email | **A page with photos, licence numbers and an Accept button** |
| Generic across 50 states | **HICPA-correct for PA out of the box** |

"Your whole crew, one price" is a sentence Jobber **structurally cannot say**
without repricing their company. That is the wedge. Everything below either
sharpens that wedge or clears a floor beneath which "cheaper and simpler" stops
reading as *simpler* and starts reading as *can't do the job*.

**The floor is the risk, not the ceiling.** A landscaper does not reject Apex
because it lacks QuickBooks sync. They reject it the moment they discover they
must re-enter every mowing visit by hand.

---

## Phase 0 — Unblock. Nothing counts until this is done.

**Effort: one evening. Mostly waiting on a deploy.**

| # | Work | Why |
|---|---|---|
| ✅ 0.1 | **LIVE at https://wisense-apex.vercel.app** (`apex-site.vercel.app` taken globally). Deployment protection set previews-only — SSO was serving a login page as the venue site |
| ✅ 0.2 | `apexSiteBase` default → wisense-apex.vercel.app (`4ba0851`) |
| ⏳ 0.3 | pubspec at `0.1.0+11`, pushed — Mac builds it. First build whose links resolve |
| ⏳ 0.4 | Nick sends https://wisense-apex.vercel.app/bradley-sons-landscaping |

Already done: link repointing (`31a8594`), photo upload (`0e0f41f`),
`$n spots open` (`9bb9cab`).

---

## Phase 1 — The floor. Can a real services business actually run on this?

**Effort: ~6–8 evenings. Do not skip any of it.**

### 1.1 Recurring jobs ⭐ the single biggest gap — ✅ DONE 2026-08-03 (`f5ea759`)

**Nothing in the codebase supports recurrence.** Verified: no `recurring`,
`rrule`, `repeat_every` — anywhere.

This is not a nice-to-have in services, it is the revenue base:

- **Lawn care** — weekly or biweekly mowing, Apr–Oct. That *is* the business.
- **Cleaning** — weekly or fortnightly.
- **Pest control** — quarterly.
- **Snow** — per-event, but the same contract shape.

The vault filed "recurring schedules" under *deliberately not built*. That was
correct when services was a side bet. It is now the primary market, and this
decision has to reverse. A landscaper trialling Apex in April discovers in week
two that they must hand-enter 40 mowing visits, and they leave — having already
concluded the product is a toy.

**Shape it as a series, not as 40 rows.** A `job_series` row (client, service,
address, crew, interval, season start/end) that materialises the next N shifts
forward and rolls on. Editing the series edits the future, not the past.
Cancelling one visit does not cancel the series. This is where most cheap
implementations get it wrong, and getting it right is a genuine advantage.

### 1.2 Balance collection that finishes the job — ✅ DONE 2026-08-03 (`edff6ad`)

The `invoice` payment kind already exists in `create-request-payment`, and
deposit/balance links work. What is missing is the *closing* motion: mark the
job complete → balance link goes out → paid → done. Today the owner has to
remember. Every competitor automates this because it is how contractors get
paid, and unpaid balances are the #1 thing that makes them chase software.

### 1.3 Customer notification — ✅ email DONE 2026-08-03 (`edff6ad`); SMS waits on A2P

Requests notify the *owner*. Nothing ever notifies the *customer* after they
accept. Housecall Pro's consumer-facing polish is largely this one message, and
it is the difference between a homeowner who trusts the process and one who
rings to ask if anyone is coming. Post-A2P, template only, no model.

### 1.4 Client history — ✅ DONE 2026-08-03 (`edff6ad`)

Second job for the same person creates a second unrelated request. No "this is
the Hoffmans, third time, previous quotes $680 and $1,450." Not a full CRM —
just group requests by phone number and show the history in the detail sheet.
Repeat work is the majority of services revenue.

---

## Phase 2 — The wedge. Things Jobber structurally cannot copy.

**Effort: ~4–5 evenings. This is what makes the pitch, and most of it is small.**

| # | Work | Effort | Why it wins |
|---|---|---|---|
| ✅ 2.1 (`6b8e78b`) | Quote-page footer: *"Powered by Apex — quote pages for your business"* + referral code into the existing `referrals` table | 1 evening | Every accepted quote is read closely by a homeowner who hires other trades. Calendly/Stripe-invoice mechanics: the customer's paperwork recruits the next customer |
| ✅ 2.2 complete (`6b8e78b` copy, `pricing page live at /pricing`) | Pricing page and in-app copy built on **flat / unlimited crew**, explicitly against per-seat | 1 evening | The one sentence Jobber cannot say. Their own #1 complaint |
| ✅ 2.3 (`6b8e78b`) | Photo-to-quote drafting (Opus vision → line items, owner edits, nothing auto-sends) | 2 evenings | Photo upload already shipped. Turns a site visit into a phone quote — the contractor's biggest hidden cost |
| ✅ 2.4 (`6b8e78b`) | HICPA landing page — "your quote page is compliant out of the box" | 1 evening | No national competitor bothers with PA statute. Your geography, your warm network |

---

## Phase 3 — Apex Front Desk (AI), once A2P clears

**Effort: ~3 evenings. Gated on Twilio approval, which is outside your control.**

Missed call → instant SMS with the quote link → Haiku SMS agent fills the
request form conversationally → writes a `requests` row. **Creates a request,
never a booking** — the existing doctrine holds.

- ~74% of home-services calls go unanswered; Jobber charges **$99/mo** for this
- Cost: number $1.15/mo + SMS ~0.8¢ + fractions of a cent per turn ≈ **under
  $5/mo per busy tenant**
- Chargeable at **$29–49/mo**

Build it **last**, despite the economics. It is the only feature that puts an AI
between your customer and *their* customer, and the failure mode lands on the
contractor's reputation, not yours. Earn the right to it.

---

## Pricing, restated for the shape above

| Tier | Price | Contents |
|---|---|---|
| Free | $0 | Scheduling, swaps, clock, chat, offline |
| **Services Pro** | **$49/mo** | Inbox, quote pages, deposit + balance links, jobs board, **unlimited crew** |
| **Services OS** | **$99/mo** | Adds recurring series, Get Found automation, the site |
| Front Desk | +$29–49/mo | Phase 3 add-on — the only add-on, and it replaces a $99 Jobber one |
| Site build | $499 launch / $799 list | One-time. HICPA-compliant out of the box |
| Multi | *unlisted* | Quote privately. Do not market what you cannot support |

Fee framing: **"$99/mo, and 1.5% only on money we collect for you."** Never
"platform fee" — on services the contractor pays it out of their deposit, unlike
restaurants where the guest pays on top.

Grandfather **Jigsy's only**, forever, in writing. Customer #2 pays list. Use
annual prepay (2 months free) instead of discounts — it converts churn risk into
cash now.

---

## Deliberately not building

- **QuickBooks / POS integrations** — unwinnable, and every one is a support
  surface you cannot staff
- **Route optimisation, GPS geofencing** — enterprise theatre for ≤3 crews
- **Materials markup, change orders, full CRM** — Phase 4 at the earliest, and
  only if a paying customer asks twice
- **Voice AI** — see Phase 3
- **Cold acquisition anything** — the audit is explicit that Apex cannot support
  it. Warm intros only, and the quote-page loop

---

## Grade trajectory

| Milestone | Grade |
|---|---|
| Today | **C+** — services invisible to customers, one payment test |
| Phase 0 done | **B−** — a customer can complete a quote request end to end |
| Phase 1 done | **B+** — a landscaper can actually run their business on it |
| Phase 2 done | **A−** — and it has a reason to be chosen over Jobber |

**The honest answer to "can we compete out the gate":** not on features, and
not today. After Phase 0 you can be *used*. After Phase 1 you can be *chosen by
someone who tried Jobber and found it expensive*. Phase 2 is what makes that
choice defensible rather than accidental.

Phase 1 is the one that cannot be skipped, and **recurring jobs is the item most
likely to be underestimated** — it is a week, not an evening, and skipping it
means the first real landscaper churns in their second week.

---

Related: [[projects/Apex v2 — Restaurant OS Build]] · [[projects/APEX_SERVICES_BUILD_PLAN_CANONICAL]] · [[projects/APEX_AUDIT_FABLE5_2026-08-03]] · [[hot]] · [[NOW]] · [[index]]
