---
title: Apex Public API — Sketch from the v3 Operations Layer
tags: [apex, api, ecosystem, strategy, v3]
aliases: [Apex API, Public API, Ecosystem API]
date: 2026-08-07
---

# Apex Public API — Sketch from the v3 Operations Layer

**Context:** Antigravity's [[projects/apex_family_ecosystem]] vision (10 apps on a
shared core). The smartest move in it is the API platform play — but it's
sequenced wrong. It's not "after v3 + 100 customers"; it's the *mechanism* that
makes the ecosystem possible, and v3 already built 80% of it.

**Thesis:** the v3 operations layer (17 registered operations, domain events,
actor_kind, no service-role lane) IS the Apex Public API. Exposing it is mostly
a packaging problem, not a build problem. This is the highest-leverage thing to
do after v3, and it's mostly already done.

---

## What the operations layer already gives you

- **17 registered operations** — every mutation is a named, described, JSON-schema'd
  capability callable by a screen OR an assistant identically. That's already an
  API contract.
- **`ci/operations.json`** — the machine-readable mirror. Each entry has name,
  description, rpcName, roleFloor, risks, confirmation tier, idempotency, input
  schema. This is literally the shape an API endpoint takes.
- **`actor_kind`** (human / assistant / system) — provenance on every audited
  write. Third parties get attribution for free.
- **`domain_events` outbox** — every state change is an event with a drain. That's
  the webhook/subscription channel satellites consume.
- **No service-role lane** — every operation runs under the caller's own session,
  RLS, and guards. A third-party app gets exactly the rights its user has, nothing
  more. This is the security model that makes a public API safe.

## The 17 operations (the API surface today)

Order lifecycle: `progress_order`, `complete_order`, `stop_order`,
`mark_order_paid`, `set_order_pickup`, `correct_order_contact`, `refund_order`
(the money lane, via `refund-order` edge function).

Schedule: `stage_shift`, `edit_draft_shift`, `discard_draft_shift`,
`publish_schedule`, `claim_open_shift`, `claim_swap`, `approve_swap`,
`cancel_shift`.

Site/org: `publish_site`, `connect_payouts`.

Guest (third actor class, token-authed, NOT in the registry): `place_order`,
`create-guest-payment`.

## What a satellite would call — Apex Supply (the recommended first satellite)

Apex Supply = automated inventory + vendor management. It's the natural first
satellite: closest to the transaction without being regulated (unlike Capital/
lending and Guard/insurance).

**What Supply needs from Core:**
- **Sales data** → to deduct inventory in real time. Reads: order history,
  `daily_revenue`, menu/line items. (Reads are not operations — they're the
  `ui_read_rpcs` / read surface.)
- **The schedule** → to predict demand ("3 crews out Friday = order more mulch").
  Reads the published schedule.
- **The AI gateway** → to draft a vendor PO for one-click approval. This is the
  peer-actor AI, exposed as an endpoint.

**What Supply writes back (as operations):**
- A new `apex_place_vendor_order` operation (registered, manager-gated, audited).
- A new `apex_receive_inventory` operation (deducts/credits stock, emits a
  `domain_event`).
- A new `apex_set_inventory_level` operation (correction path, like
  `correct_order_contact`).

**The pattern:** Supply calls Core's existing operations for what Core already
does, and registers NEW operations for what Supply introduces. It never writes a
table directly — it goes through the operations layer, exactly like the app and
the assistant do. That's the whole point: **a satellite is just another
operation-caller.**

## The three layers of the API

1. **Operations (mutations)** — the 17 today, growing. Each is a named capability
   with a JSON schema. Third parties call these to change state.
2. **Reads** — the read surface (RPCs + PostgREST). Satellites read what they need
   under the user's session.
3. **Events (domain_events)** — the subscription/webhook channel. Satellites
   subscribe to "order completed", "shift published", "inventory low" and react
   without polling.

## What's actually left to build (it's small)

- **Auth for third parties** — OAuth so a satellite can act as a user. The
  operations layer already assumes a session; the API just needs a way to mint one
  for a third-party app.
- **API keys / rate limiting** — per-app keys, quotas. The operations layer has
  the per-venue attempt ceiling pattern (BL-1) to reuse.
- **Webhook delivery** — the `domain_events` drain exists; wire it to deliver to
  registered satellite endpoints (exactly-once, the pattern is proven).
- **Documentation / SDK** — the operations.json is already the spec; generate docs
  from it.

## The honest caveats

- **No developer base yet.** Third parties won't build on a platform with a
  handful of venues. The API is the *mechanism*, but it only becomes valuable once
  v3 has real customers. Don't build the full public API before v3 earns its
  first venues.
- **The 30% App Store cut is optimistic.** Real app stores have millions of
  users. Apex would have a handful. The API is the right long-term shape, not the
  near-term revenue.
- **Sequencing:** v3 earns customers → expose the operations layer as the API →
  build ONE satellite (Supply) in-house to prove the model → then open it to
  third parties. Each step proves the next.

## Bottom line

The v3 operations layer is the Apex Public API in embryo. The move isn't to
"build an API later" — it's to **recognize the operations layer as the API and
package it** once v3 has customers. A satellite is just another operation-caller.
That's the architecture that makes the 10-app ecosystem real without building 10
apps.

---

Related: [[projects/apex_family_ecosystem]] · [[projects/APEX_V3_BUILD_AND_STATUS_2026-08-05]] · [[projects/APEX_V3_VS_V2_ASSESSMENT_2026-08-05]] · [[Inbox]] · [[hot]] · [[NOW]] · [[index]]
