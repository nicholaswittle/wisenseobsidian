---
title: Apex Services Vertical — Full Build Plan
tags: [apex, verticals, services, build-doc, architecture, pricing]
date: 2026-08-02
---

# Apex Services Vertical — Full Build Plan

**Rewritten 2026-08-02.** Two earlier drafts scoped this as *landscaping
software* and sized it against Jobber/Yardbook parity. That was the wrong
question. This is not a landscaping product — it is **the same core serving any
small service business**, with the restaurant remaining the focus.

Related: [[projects/APEX_MULTI_VERTICAL_PLAN_2026-08-02]],
[[projects/APEX_DECOUPLE_PILOT_2026-08-02]], [[DECISIONS]]

---

## 1. The thesis, stated correctly

Every small service business has the same skeleton:

- People show up somewhere at a time
- Somebody pays for something
- The owner needs to know whether that made money
- Everyone needs to be told what changed

Salon, detailer, landscaper, cleaner, dog groomer, mobile mechanic,
photographer, tutor, contractor. **Different end goals, identical skeleton.**

The vertical toggle is **the role split on a different axis** — the same
`disabled_modules` mechanism that already decides what a server sees versus a
manager. It is not new architecture. It already exists, already wins over
everything (`20260730210000:41–44`), and already fails closed.

**Restaurant stays the focus.** Services is opportunistic capture: when a
non-restaurant walks in through a warm introduction, Apex can say yes instead
of no.

---

## 2. ⭐ The edge piece — **get the money, fastest path from yes to collected**

> **Revised 2026-08-02 after research. The "did that job make money?" hypothesis
> below failed three independent tests and is demoted to a month-three
> retention view. The mechanism was right; the urgency was wrong.**

### Why job margin is not the wedge

1. **The structure doesn't generalise.** "Flat quote against hourly labour"
   needs a *quoted* price and *employed* labour. Menu-priced owner-operated
   trades (salon, barber, groomer, massage, tattoo, tutoring, training,
   photography) don't have it at all. Realistically **9–11 of 25 trades, and
   only past ~3 employees** — maybe 20–30% of the base.
2. **Owners don't name it.** NFIB (n=2,873): top problems are health insurance,
   supply costs, uncertainty, hiring. Job costing is **not on the list.** Fed
   SBCS 2024: paying operating expenses 56%, uneven cash flow 51%.
3. **The entire evidence base is vendors selling job-costing software.** No
   independent survey ranks it. And the self-reported adoption blocker in that
   same literature is *"it takes too much time"* — bad odds for a feature whose
   value is retrospective and whose cost is daily data entry.

### What the evidence says instead

| Source | Finding |
|---|---|
| JPMorgan Chase Institute (597k businesses) | **Median small business holds 27 cash buffer days.** 25% hold ≤13. Labour-intensive trades hold fewer. |
| QuickBooks Small Business Insights | **59% have invoices 30+ days overdue** (up from 47%); avg **$17.7K** owed; 59% paid extra fees to access money already earned |
| Fed SBCS 2024 | Uneven cash flow 51% |

**At 27 buffer days a two-week payment delay is existential; a 4-point margin
misestimate is not.**

### The build: money attached to the job record

Deposit at booking · balance on completion · card on file · one-tap "send
invoice + payment link" from the phone in the driveway · automatic nudges on
anything unpaid.

Universal 25/25, but **two modes** — build both, gate by template:

- **Mode A — point of service (~14/25):** salon, barber, groomer, massage,
  tattoo, tutoring, training, detailing, pet sitting, cleaning, photography,
  pressure/window washing. They get paid at the chair. **Their loss is the
  empty slot, not the unpaid invoice** — so the fix is deposits + reminders.
- **Mode B — quote to invoice (~11/25):** plumbing, HVAC, landscaping, moving,
  painting, handyman, appliance repair, junk removal, rental, catering, pool.
  The fix is the deposit they didn't take and the invoice nobody chased.

It is felt, immediately measurable (*"you got paid 9 days faster"*), and
**requires no daily data entry.**

### Where the old hypothesis survives — keep this

Once money is attached to the job record, **job profitability is nearly free.**
The system already holds the quote and the invoice; the only missing input is
time, and the cheapest capture is not a timesheet — it is **one tap on arrival,
one tap on departure**, on a job record the tech is already looking at.

So: **money layer first, profitability falls out as a reporting view.** It is
the month-three feature that makes them never cancel, not the thing that gets
them in the door.

---

## 2b. Superseded — the original job-margin argument

One feature, needed by all of them, that Apex can do and the competitors
structurally cannot at this price point.

### The universal problem

Every one of these businesses **quotes a flat price and pays hourly labor.**
The spread between the two is the whole margin — and at the small end nobody
can see it, because the time clock is in one app (or a paper sheet) and the
invoice is in another (or QuickBooks, or a Square terminal).

So the owner learns whether the month was good, from their accountant, six
weeks late, in aggregate. They never learn **which jobs** were good.

The consequence is always the same and every one of them has it: **two or
three customers who are quietly net-negative, and a service line priced from a
guess made three years ago.**

### Why Apex specifically

| Product | Labor | Revenue | Job costing |
|---|---|---|---|
| Homebase / 7shifts / When I Work | ✅ | ❌ | ❌ |
| Square / Wave / HoneyBook | ❌ | ✅ | ❌ |
| Jobber / Housecall Pro | weak | ✅ | shallow |
| ServiceTitan / Aspire | ✅ | ✅ | ✅ — **$250–350/user/mo** |
| **Apex** | ✅ | ✅ | **the opening** |

Apex already holds both sides in one tenant. `time_entries` is the clock,
`jobs`/orders is the revenue, and `shifts.job_id` — already in the plan for
crew dispatch — **is the join key.** The feature is a query over data the
system will already have.

Verified market evidence that this carries price: **LMN charges $297/mo aimed
at sub-10-employee companies, and its pitch is estimating and budgeting
rigor.** The research found the upper market buys specifically on job-costing
rigor at $300–650/mo. **Nobody offers it at the bottom.**

### What it looks like

**On completion, one line:**

> Maple Street cleanup — quoted **$450** · labor **6.5 hrs / $162** · materials
> $80 · **margin $208 (46%)**

**After 20 jobs, the part that actually changes behaviour:**

> Lawn maintenance jobs are averaging **3.4 hrs**. You quote them as 2.
> At your labor rate that is **$38 per job** you are not charging.
>
> **The Hendersons** are your least profitable customer — 6 jobs, average
> margin **4%**.

That second screen is the reason someone switches. It is not a report; it is
someone telling them something true about their business that nobody has ever
told them.

### Why it is hard to copy

Not the maths — the maths is trivial. **The data position.** A scheduling app
cannot compute it because it never sees the invoice. An invoicing app cannot
compute it because it never sees the clock. Adding the missing half means
building a whole other product and convincing the customer to move it over.
Apex has both because it started as a restaurant OS, where labor-versus-revenue
was always the point.

### The honest catch

It requires **both** sides populated — staff must clock in *and* money must run
through Apex. That is a real adoption dependency, and it is also the lock-in:
once both are in, the number only exists here.

Which sets the onboarding order: **free website → take a deposit → then "add
your crew so I can tell you if that job made money."** The labor side sells
itself once the first job shows a margin.

### It ships to restaurants too

`labor_vs_revenue_dashboard.dart` already exists at the *day* level. This is
the same idea at the *job* level — and a restaurant version ("this catering
order took 9 labor hours against $600") is the same code. **The edge piece is
not a services feature. It is an Apex feature that services makes obvious.**

---

## 3. The minimal build

Not a Jobber clone. Two earlier drafts made that mistake.

### 3.1 One new table

```
requests(id, organization_id,
         client_name, client_phone, client_email, service_address,
         services jsonb, notes,
         status,            -- requested|quoted|approved|scheduled|
                            -- complete|cancelled
         quoted_cents, deposit_cents, deposit_paid_at,
         final_cents, scheduled_date, created_at)
```

**Why not `online_orders`:** it is sub-hour by construction —
`pickup_minutes`, escalation on "still WAITING" (`notify-order-event:157`),
support-agent timers at 10/30 min (`venue-support-agent:102–105`), capacity
throttling per hour. A deposit sits for two weeks. Verified, not assumed.

### 3.2 The flow

Site quote request → owner quotes → Stripe deposit link → paid → scheduled →
`shifts.job_id` dispatch → crew clocks in → **margin line**.

Every step but the table and the margin query already exists.

### 3.3 Adapt

- `menu_items` → services. **Blocker:** `price_cents int not null check (>= 0)`
  (`20260729000000:71`) — "call for estimate" is unrepresentable. Nullable
  price + `price_type` ∈ `fixed|quote|hourly`.
- `modifier_groups` → service options. **Zero schema change.**
- `shifts` + `job_id` + `property_address` → dispatch. Scheduling, swaps, time
  clock, call-outs and payroll all flow through unchanged.
- Onboarding + `venue_auto_bootstrap` branch on `organizations.vertical`
  (added `20260901000000`).
- `check-capacity` and `venue-briefing` must **skip services orgs** — both run
  service-role and ungated today, so "module off" does not stop them.
- Label layer: ~100–150 strings in the shared core.
- Support-agent SYSTEM_PROMPT (`:77–129`) needs a non-restaurant variant.

### 3.4 Keep untouched

Auth · RLS · orgs/members · entitlements · `shifts` · `swaps` · `time_entries` ·
`apex_resolve_member` · log book · call-outs · **messages/group chat** ·
Stripe Connect · Twilio · AI metering · `enrich-business` · `parse-schedule` ·
monitoring · `venue_hours` · renderer core · payroll register.

### 3.5 Not now

Recurring schedules · invoices spanning jobs · route optimisation · materials
markup · change orders · chemical tracking · **GPS geofence**.

**Geofence stops being a blocker under this framing.** The wall QR
(`lib/core/clock_qr.dart`) works fine for a salon, shop, detailer or office —
anyone with a wall. GPS matters only for crews at client properties, which is
one vertical, not the general case. Earlier drafts called it structural because
they were reasoning about landscaping.

---

## 4. Pricing

| | |
|---|---|
| Website | **Free forever**, own domain, quote requests |
| Custom build | **$499–799** one-time — one-time revenue is immune to seasonality and is literally what a warm intro asks for |
| Ops | **~$99/mo flat, unlimited users** |
| Transaction | **1.5%** on deposits and payments collected |

**Flat per-company, never per-user.** The market's loudest complaint and Apex's
one clean structural advantage. Jobber is +$29/user, Housecall +$35. Say it
out loud: *"add your whole crew, the price doesn't move."*

**Kill the $25 tier.** You cannot beat free, and $25 signals hobby software to
someone who pays $297 for LMN.

**The transaction economics are better here than in restaurants.** Corrected
arithmetic: a $1,000 deposit at 1.5% is **$15** (not $150 — that would be 15%).
But per customer:

| | Ticket | Fee | Monthly volume | Monthly |
|---|---|---|---|---|
| Restaurant | $33 | $0.50 | $2.4–6k | **$36–91** |
| Service business | $800 | $12 | $16k | **~$240** |

**3–6x better per customer**, because the tickets are large. This reverses the
earlier "drop the fee to zero" recommendation, which was reasoning about
competing with Yardbook on price rather than about the transaction itself.

**Seasonality still bites** for outdoor trades — annual prepay sold in
Feb–March, or an off-season rate. Indoor businesses (salon, detailer, cleaner)
do not have this problem at all, which is another argument for not scoping to
landscaping.

---

## 5. What the earlier research does and does not say

A market study was run 2026-08-02 against *landscaping software specifically*.
Its central finding — **Yardbook is a YC-backed, ad-monetized incumbent running
free-software-plus-1%-fee since 2018, with 20,000 companies** — kills the
strategy of *entering landscaping as a free landscaping OS*.

It says much less about this plan, because this is not that. Yardbook, Jobber
and LMN are irrelevant to a salon, a detailer, a photographer or a dog groomer.
**The competitive set for "any small service business" was not researched.**

What still holds from it, regardless of vertical:
- Free auto-generated websites are **table stakes** — Jobber bundles one on
  every plan. The surviving pitch is *your own domain, no subscription*, not
  *free*.
- **Per-user pricing is the market's loudest complaint.** Flat wins.
- Job costing carries real price at the top and **is absent at the bottom** —
  which is the §2 opening.
- Nobody switches for scheduling and a group chat. **They switch for the edge.**

---

## 6. Sequence

| Phase | Work | Gate |
|---|---|---|
| **0** ✅ | `organizations.vertical` · de-Jigsy | done |
| **1** | Brother-in-law's site — theme, service list, quote-request form | **safe now** — `site/`, not the Flutter build |
| **2** | `requests` table · deposit link · vertical onboarding · gate the crons · label layer | Jigsy's live |
| **3** | ⭐ **Job margin** — `shifts.job_id`, the completion line, the 20-job insight | Phase 2 |
| **4** | Per-vertical polish as real customers arrive | demand |

**Phase 1 requires no architecture decision at all** — the renderer is already
a separate, multi-tenant Next.js app. The one-app-versus-two question does not
arrive until Phase 2, and deferring it costs nothing.

> `docs/WiSense Restaurant OS Master Plan 2026-07-27.md:156` — *"Do not build
> vertical two until vertical one is paying."* Phase 1 does not violate it: it
> is a website for one person, not a vertical.
