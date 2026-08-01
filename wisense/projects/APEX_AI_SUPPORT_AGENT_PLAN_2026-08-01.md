---
title: Apex AI Support Agent — Plan
tags: [apex, ai, support, operations, build-doc, strategy]
date: 2026-08-01
---

# Apex AI Support Agent — Plan

The build that makes a side business viable. Not a feature — the thing that
protects the only resource that cannot be bought more of.

Related: [[wisense/projects/APEX_SELF_SERVE_PLAN_REVISIONS_2026-08-01]],
[[wisense/projects/APEX_V2_TEMPLATE_TO_PRODUCT_GAP_MAP_2026-07-31]]

---

## Why this outranks growth loops

Apex is a side business targeting **$300/month — three paying venues.** At that
scale acquisition is not the constraint; **founder attention is.** Restaurants
operate Friday at 7pm, and a solo operator is the entire on-call rotation. One
bad service night destroys trust built over months.

Referral machinery brings customers there is no time to support. **The support
agent protects the scarce thing.** Build it before Build 5 growth loops.

Running cost is negligible — Haiku at roughly $0.001/call, so a venue could
trigger it a hundred times a month for pennies.

---

## The core insight

**Most Apex failures are diagnosable from data already in the database.**

Of everything that broke on 2026-07-31 — the stuck deploy, the `check-capacity`
401 loop, the unsubstituted placeholder key, the unpaid Square test order — all
but the deploy were visible in `net._http_response`, `cron.job`,
`online_orders`, and function config. A reader with the right scope would have
named them in seconds.

The agent is not a chatbot bolted on. It is a diagnostician with the venue's
live state as context.

---

## Read scope

Everything needed to answer "what is wrong right now":

- `online_orders` — stuck `awaiting_payment`, unaccepted, unusually old
- `net._http_response` — real HTTP status of scheduled jobs (**not**
  `cron.job_run_details`, which reports whether the SQL ran, not what the call
  returned — this exact trap cost hours on 07-31)
- `cron.job` — schedules present, active, last run
- `restaurant_settings` — `paused`, `square_charges_enabled`,
  `stripe_charges_enabled`, `payment_provider`, **`square_environment`**
- Square/Stripe connection state and credential expiry
- `capacity_events` — auto-pause history
- Recent edge-function error logs

**Never in scope:** decrypted secrets, card data, `vault.decrypted_secrets`,
raw tokens. The agent diagnoses configuration, never reads the credential
itself. Digest comparison is sufficient and was proven so on 07-31.

---

## Safe-action allowlist

The agent may act **only** where the action is reversible, verifiable, and
cannot move money.

**Allowed**
- Re-run `reconcile-pending-payments` for one order
- Retry a failed webhook delivery
- Un-pause ordering (`paused = false`) after confirming capacity recovered
- Re-send an order to Square (`CreateOrder` idempotent retry)
- Re-print a ticket
- Run the Square printer test order

**Forbidden — always human**
- Refunds, re-charges, any movement of money
- Disconnecting or reconnecting a payment provider
- Permanent configuration changes (tax, fees, hours, menu prices)
- Anything it cannot verify succeeded afterwards
- Anything touching another venue

**Rule:** every allowed action must be idempotent and must re-check state after
executing. If it cannot confirm the fix worked, it escalates rather than
claiming success. *Reporting a fix that did not happen is worse than reporting
no fix* — the 07-31 lesson where `cron.job_run_details` said `succeeded` while
every request 401'd.

---

## Honest limits — set these expectations in the UI

**Most restaurant outages are physical.** Unplugged printer, dead wifi, sleeping
iPad, Square itself down. The agent cannot fix any of them.

What it *can* do is say **which** one it is, which is most of the value at 7pm.
Staff do not need a repair — they need to stop guessing.

Design the copy accordingly: the agent is a diagnostician that sometimes
repairs, never a technician that always fixes.

---

## Escalation

Three tiers, and the middle one is the point of the whole build:

1. **Self-serve** — agent explains, staff act ("your printer profile lost its
   online-tickets toggle; here is where to turn it back on")
2. **Agent-fixed** — allowlist action, verified, logged, owner notified
3. **Escalate to Nick** — with the diagnosis already written: what broke, what
   was ruled out, what it tried

Tier 3 must arrive as a **conclusion, not an alert.** The difference between
"ordering is down" and "orders are reaching Square but the printer profile's
online-tickets toggle is off; I cannot change that setting remotely" is an hour
of a Friday night.

---

## Build order

1. **Read layer + diagnosis** — no actions at all. A `SECURITY DEFINER` RPC
   returning one venue's health snapshot, and a prompt that reads it. Ships
   value immediately and cannot break anything.
2. **Escalation with pre-written diagnosis** — push to Nick with findings
   attached. This is where the on-call relief actually starts.
3. **Safe-action allowlist** — one action at a time, each with verification,
   starting with re-run reconciliation (lowest risk, already idempotent).
4. **Proactive watch** — the agent notices before staff do: order stuck 10+
   minutes, no successful cron response in 30, `square_environment` not
   `production` on a live venue.

Step 4 is where it stops being support and becomes the reason they stay.

---

## Seam discipline

Same rules that held all week:

- Venue-scoped by `organization_id` at the database layer, via definer RPC —
  never app-side filtering. One venue's agent can never read another's state.
- Every agent action written to an audit log with the reasoning that triggered
  it.
- The agent never marks a problem resolved on the owner's behalf; resolution is
  confirmed by re-reading state, not by the agent asserting it.
- No secret material in scope, ever — configuration and digests only.

---

## Why this is the moat

A POS vendor can ship online ordering tomorrow. What they will not ship is a
guide that knows *this* venue's numbers and tells the owner what to do next —
at onboarding, and at 7pm when something breaks.

That is the difference between "I want this" and "I need this," and it is the
same assistant in both places. The onboarding wizard is chapter one; this is
chapter two.
