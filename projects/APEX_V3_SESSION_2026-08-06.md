---
title: Apex v3 — 2026-08-06 build session
tags: [apex, apex_v3, build, session, onboarding, operations]
updated: 2026-08-06
---

# Apex v3 — 2026-08-06

38 commits. The app went from having no database client at all to a real
person creating an account, being found on Google, getting an AI-written
description, and completing setup — on production.

Related: [[APEX_V3_BUILD_AND_STATUS_2026-08-05]] · [[APEX_V3_OPEN_ITEMS_2026-08-06]] ·
[[APEX_COLD_USER_TEST_AND_TODDLER_BAR_2026-08-06]] · [[NOW]] · [[DECISIONS]]

## The two findings that mattered most

**The AI gate did not exist.** Six edge functions plus the shared helper called
`apex_charge_ai_call`; no migration defined it. PostgREST answered 404, the
helper translated that to `403 not_allowed`, so every AI feature in v3 had been
silently dead since the project began — and it read as a permissions bug. The
CI check meant to catch exactly this was blind to the call shape being used
(raw `fetch` rather than `.rpc()`). Both fixed; the gate now catches the whole
class.

**"This is your website. It's live right now" was false three ways.** Verified
against the live catalog, not inferred:
1. Every new signup is tier `free`; guest readability needs `onlineOrdering` at
   tier `os`.
2. Every anon-readable RLS policy is restaurant-pack. There is **no anon policy
   on `organizations` or `venues` at all** — a services business has zero
   publicly readable rows.
3. There is no served web route regardless; the reveal is a Flutter widget in
   the owner's own session.

`jr property maintenance` — the real signup from tonight — is published `true`,
ordering live `false`. Caught by an experienced-owner review the day before
three real people test it.

## Built

- **AI gateway** — `apex_charge_ai_call` defined and proven; `ai-suggest` and
  `business-search` written, deployed, and making real calls.
- **Auth, session routing, persistence** — email+password, forgot-password,
  routing by session and publish state.
- **Onboarding** — search-first (Google Places) plus the manual four questions,
  confirm-never-prefill throughout.
- **Restaurant menu flow** — photograph, check prices, confirm. Sizes and
  extras. Not yet persisted.
- **Staff order management** and **guest ordering**.
- **Schedule** — staff actions, authoring operations, and the builder screen.
- **The operation contract** (D28) with 15 registered operations and a CI gate.
- Test-restaurant seed, menu scoring harness.

## Decisions recorded (in `apex_v3/docs/decisions.md`)

- **D26** — the human holds the authority, the AI holds the capability. No
  capability lives inside a button; second grade is the AI threshold; friction
  goes where being wrong is expensive.
- **D27** — search is an accelerator, not the path. Jigsy's was found; JR
  Property Maintenance, with no online presence, was not — and that owner is
  the target user.
- **D28** — the operation contract. The confirmation tier is **derived from
  four facts, never declared**, so it cannot be forgotten.
- **D29** — split an RPC where the *facts* diverge, not where the verbs do.
  Corollary: when a derived tier feels wrong, **change reality or accept the
  ceremony** — never re-answer the fact.
- **D30** — schedule authoring as draft → publish.

## Measured, not assumed

- `parse-menu` on a real photo of Jigsy's menu: **zero wrong prices across five
  runs**, twice — before and after adding sizes and extras. Item names ~35/37;
  most misses were errors in the answer key, not the parser.
- `business-search` cost **60 Google calls per signup**, found in `ai_calls` on
  the live database. Now 2. The 40/hour ceiling would have locked an owner out
  mid-setup.

## Open

- No public site for services businesses — the largest gap.
- Menus and hours do not persist; restaurant path is deliberately **blocked**
  before the `vertical` write, because that column allows exactly one write and
  gates which modules an org can ever unlock.
- No authoring screen for time-off or availability; refunds never exercised;
  dashboard, admin, capacity, entitlements unported.
- `vertical` is permanent with no confirmation — breaks D26's own rule 3.

## Next

Three cold users tomorrow: brother-in-law (services, the target user), wife
(never onboarded, asked for the schedule builder), nine-year-old (tests the
reading level directly). Separately, same build, no helping.

Then: the adversarial review of what a confident-but-wrong assistant can do
with the owner's rights — a surface that did not exist before D26.
