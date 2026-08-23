---
type: project
title: "Apex Core + Apex Comms — the two apps and the seam"
tags: [apex, apex-core, apex-comms, architecture, launch]
date: 2026-08-23
status: current
---

# Apex Core + Apex Comms

> **This supersedes the vault's picture of "Apex v3".** Same product, new name.
> Everything filed under `APEX_V3_*` before 2026-08-10 is history, not status.
> Live status lives in the repo at `docs/STATE.md`, which is machine-refreshed
> and goes red in CI when it is more than seven days stale.

## The naming, settled 2026-08-23

| | Icon says | Repo / identifiers |
|---|---|---|
| **Apex Core** | **Apex** | `apex_v3`, `com.wisense.apexV3` — unchanged on purpose |
| **Apex Comms** | **Apex Comms** | separate repo, separate Supabase project |

"Core" is how we tell the two apart in writing. **No customer ever sees the
word Core** — the flagship app is just *Apex*. Identifiers keep the `v3`
spelling deliberately: the bundle id is bound to the App Store record, the
TestFlight builds and a Tap to Pay entitlement request sitting with Apple.
Changing it would make a different app to fix something nobody can see.

## What each one is

**Apex Core** — the iPhone app a business runs itself from. Quotes, jobs,
scheduling, crew, a **free public website Apex hosts** for each business at
`theirname.madewithapex.com`, and three ways to take money: online orders,
payment links a customer opens alone, and contactless card on the owner's own
iPhone. Flutter + Supabase + Next.js. Apex takes **1.5%** on top of Stripe.

**Apex Comms** — a separate app and separate Supabase project that **answers a
customer's text at 3am** on the business's behalf. Works out what they want,
offers times, agrees a price, books the job. **No human is in the request.**

## Apart

Each stands alone, and they are not equally strong alone.

- **Core without Comms** is a complete product: a business can quote, book,
  invoice, publish a website and take money without Comms existing. This is
  what the two pilots will use.
- **Comms without Core** is an AI receptionist. It could in principle serve a
  business that uses Jobber or nothing at all — but nothing has been built to
  let it, and its booking flow currently assumes Core's state machine.

## Together

A customer texts at 3am. By morning there is a booked job with an agreed
price in the owner's diary and a payment link ready. Neither app does that
alone, and the owner slept through it.

## The seam — how Comms acts inside Core

Full detail: `apex_v3/docs/COMMS_CORE_SEAM.md`. The shape is the interesting
part, and it was chosen against the obvious alternative.

- **A service user per organization.** Linking creates a real, visible,
  revocable Core member named "Apex Comms" at **Manager** — never owner, so a
  partner can never change the plan or remove people. Deliberately **not**
  "act as the owner": the audit trail names a machine a person can point at
  and revoke, not a human's identity borrowed at 3am.
- **Delegation, not a header.** Writes are stamped `assistant` through the
  trusted branch, not by a header anyone could type.
- **Hashed key, scoped operations.** Plaintext shown once at minting. The org
  is resolved *from the key*, never the request body. A key that can book
  cannot refund. Revoking is one update and does not touch the venue.
- **Only already-registered Core operations** — `create_request`,
  `send_quote`, `accept_quote`, `propose_day`, `book_request`. **No new Core
  operations were needed for an external caller**, which is the strongest
  evidence the operations layer was built right.
- **Booking is four calls, not one**, because Core refuses to book without a
  price **and** a day agreed. That is discipline, not overhead.

**Who owns what:** availability lives in **Comms** (Core has no bookable-slot
concept for services); what is actually booked lives in **Core**. Both
directions are read, because reading alone leaves Comms bookings invisible to
Core — half the double-booking problem, and the more embarrassing half.

**Built and deployed on the Core side (2026-08-23).** **Not built:** the Comms
side that consumes the key, calling the link against a live venue, and the
venue-facing screen with a Disconnect button.

## Where Core actually stands — do not mistake polish for traction

- **Five real-money transactions in the product's entire history, totalling
  under $17.** Existence proof, not reliability proof.
- **Zero paying customers. Not launched.** Two pilots committed, not started.
- ~293 migrations, ~1165 tests, 12 CI workflows, RLS pinned in CI **and**
  diffed against production nightly, restore drilled at ~20 seconds.
- An independent audit on 2026-08-21 graded it **B overall**, A− engineering
  and security, **C+ on UX for a non-technical owner**.
- **Nobody who is not Nicholas has ever used it.** The seven-day venue week
  (12–18 Aug, verified) proved the machine runs; Nicholas operated it. The
  [[projects/APEX_COLD_USER_TEST_AND_TODDLER_BAR_2026-08-06]] fixes have never
  been put back in front of a cold user.
- **No customer has ever paid a balance through the "pay the rest" link** —
  the one path a customer walks alone. Both balance payments in history were
  taps, with Nicholas standing there.

## What gates launch

**Legal, and nothing else technical.** Terms and privacy are self-drafted; the
SMS consent language and the Stripe Connect platform posture have not been
looked at. The deposit-cap question **is settled** — see [[DECISIONS]].

Tap to Pay does **not** gate launch. The build split means the App Store
build ships without the entitlement and the card button refuses with a plain
sentence; orders and payment links carry money meanwhile.

## The defect class this product keeps producing

Worth knowing before reading any Apex code or audit: **a failure returning the
reassuring answer.** Seven instances found in one week — a discarded database
error becoming a 200 that stops Stripe retrying; a missing settings row
reading as "ordering is open"; a sweep voiding a refund on an unread row; a
capacity check pausing a live restaurant because a failed read counted zero
staff. Each one wrote a *false reason* into a ledger somebody would later read
to understand what happened.

A source-level CI gate now fails any webhook or destructive sweep that
discards a read error. Related and same family: **eight test files that no
workflow ran**, a smoke check dormant since it was written, and three red
workflows nobody was reading. The gates here are good; nobody was reading
their output.

## Pointers

- Live status: `apex_v3/docs/STATE.md` — challenge it, do not quote it
- The seam: `apex_v3/docs/COMMS_CORE_SEAM.md`
- Cutting builds: `apex_v3/docs/BUILDS.md`
- Filming Apple's videos: `apex_v3/docs/TAP_TO_PAY_FILMING.md`
- Review prompts: `docs/EXTERNAL_REVIEW_PROMPT.md` (Core),
  `docs/EXTERNAL_REVIEW_PROMPT_TWO_APPS.md` (both + the seam)
