---
title: "Apex — Cold-User Test, the Toddler Bar, and the 2026-08-06 Rework"
tags: [apex, services, ux, cold-user-test, design-bar, v2]
updated: 2026-08-06
---

# Apex — Cold-User Test, the Toddler Bar, and the 2026-08-06 Rework

> The first genuine outside user touched Apex on 2026-08-06. He nearly
> uninstalled it. This note records what he hit, the design bar that came out
> of it, everything shipped in response that day, and — deliberately — what is
> still not built.

## 1. The test

A landscaper (Nicholas's sister-in-law's boyfriend) built his own account and
site on the TestFlight app while Nicholas watched, with minimal instructions.
Unpaid cold user. The point was to see where a stranger gets stuck in
self-serve onboarding, which had never been observed.

**Nicholas's verdict: "if I wasn't there to get him through, he would have
just uninstalled the app."**

### What he actually hit

1. **Blank-page paralysis.** Tagline, About, and the one-line service blurbs
   stopped him dead. He knows his trade; he does not know what to type about
   it. Nicholas's own suggestion: AI suggestions inline.
2. **Publish read as "back to step one."** Second independent report of this.
   (Cause found — see §3 — it was literally true.)
3. **"Stripe" meant nothing to him.** Nicholas had to explain what a Stripe is.
4. **Every step forced a full back-out.** Finish a step, land back on a list,
   no idea what the next move was.
5. **General wayfinding failure** — trouble locating things and knowing what
   anything meant.
6. **The quote board looked unchanged** — correct, and initially misdiagnosed
   as a stale build. See §4.

### What landed — do not lose these

The test was not all failure, and the two things that *worked* are the more
commercially useful half of it:

1. **Seeing his own website was the moment.** He was "super excited" to see it.
   That is the emotional payoff of the entire funnel, and right now it sits at
   the *end* of a seven-step checklist he had to be walked through. **The
   product implication: get to the site preview far earlier.** A rough,
   half-filled site he can see and react to beats a finished one he never
   reaches — and it converts the setup work from chores into edits to something
   that already exists and is *his*.
2. **He liked online pay on sight, unprompted.** The thing Nicholas assumed
   would need selling sold itself; the thing that needed explaining was the
   *word* Stripe, not the idea of getting paid by card. This validates the
   $99/mo direction, and says the upsell copy should lead with the outcome
   ("get paid by card, money in your account") and never with the provider.

Read together: **the demo is the product.** The site preview is the hook and
online pay is the want; the checklist standing between them is the problem.

## 2. The Toddler Bar — the governing rule

Nicholas, after the test:

> "Can a first time user that has no computer or technology abilities work this
> app? I should be able to give it to my 2 year old and be able to do it."

This is now the standing design bar for Apex. Operationally it means five
tests every screen must pass before shipping:

1. **One obvious next action, named as a verb, always visible.** Never a bare
   status to interpret — a button that says what happens.
2. **No jargon.** No "Stripe", "quoted", "publish", "sync" without a plain
   translation in the same breath.
3. **No blank boxes.** Every empty field gets a suggestion, an example, or a
   prefill.
4. **Backing out lands on a signpost.** Nothing dead-ends, nothing silently
   resets, "where was I?" always has an on-screen answer.
5. **If the app can do it for them, it does** — then lets them correct it.
   *Suggest, never commit.*

This supersedes ad-hoc UX judgement in v2 and is inherited by v3.

## 3. What shipped on 2026-08-06

Branch `release/services-plus-counter` (apex_v2). Analyze clean, 177/177 tests
green at every commit below.

### Against the cold-user findings

| Finding | Fix | Commit |
|---|---|---|
| Blank-page fields | ✨ **Suggest** button on Tagline, About, and each service blurb — three AI-drafted options in a sheet, tap to fill, everything stays editable. New `draft-site-copy` edge function (Sonnet). | `8e90f01` |
| Publish reads as restart | **It was literally true**: "Review & publish" routed through the *find-your-business search* — the first screen of setup. Now opens the review form directly, prefilled from what is saved. | `8e90f01` |
| Stranded after each step | Checklist leads with one button: **"Next: <first unfinished step>"**. | `8e90f01` |
| Stripe jargon | "Connect Stripe" → *"Get paid by card online — Stripe handles the cards for you, free to set up, optional."* Restaurant wording pin updated deliberately; the cold-user test is the evidence it was waiting for. | `8e90f01` |
| Quote board unchanged | Board rebuilt: **"Your move"** leads with a named button per card (Name a price / See their reply / Pick a day / Collect $450); money cut from four tiles to two; status chips speak English (New / Waiting on them / Said yes / Booked / Done). | `4acc1ba` |

### Also shipped that day

- **Services owner home** (`abb43ab`) — replaces the punch-clock restaurant
  dashboard for services admins. One scroll: Needs you now → Today → Money →
  Your page. Staff and restaurants keep the classic dashboard.
- **One-round-trip quoting** (`dc50d61`) — owner proposes a day *with* the
  price; the customer's single Accept books both. Status-gated request sheet
  so each stage shows only its own actions.
- **Customer counter-offers** (`4bee3c0` + site `73dae15`) — "Suggest a change
  instead" on the quote page: a note, an optional counter price, or both. Lands
  the request back in the owner's needs-action pile; the owner's next saved
  price clears it. Accept stays available throughout.
- **Solo-owner scheduling fix** (`1dda485`) — `CrewPicker` now pre-selects a
  one-person roster. The server accepted the call all along; the unchecked
  crew sheet was the refusal. Read as *"it won't let me post to the schedule."*
- **Services setup progress** (`efbd55e`) — checklist counts services (not
  restaurant menu categories), go-live no longer gated on the ordering module
  (the public site is the free product), "Online ordering is paused" no longer
  shown to services venues, manual entry prefills from saved data.
- **Refund reasons** (`1ca958f`) — chips (wrong order / unavailable /
  cancelled / quality / error) or free text; written to `online_orders.refund_reason`
  *and* the provider's records.
- **Counter orders stop reading as to-go** (`1ca958f`) — "Ready in 12m" /
  "Running late" / "Pay at counter" for in-house orders.
- **Navigation that remembers** (`1ca958f`) — Manage no longer pops itself
  before opening a module, so back returns where you came from.
- **Pinned back bar everywhere** (`45a49e1`) — thirteen screens drew their own
  scrolling headers; some (both Labor tabs, Team, Chat, Time Off, Admin) only
  rendered a back button if a caller passed `onBack`, which the module routes
  never did. One frame at push time instead of thirteen header surgeries.

### Production changes (live regardless of app build)

- `20260806021805_fix_request_photo_intents_grants` — **hero photo upload 403**.
  Root cause was not venue-assets: the `request_photos_intent_insert` policy
  ran an `EXISTS` against `request_photo_upload_intents`, which had RLS on,
  zero policies and zero grants — so *every* storage insert 42501'd. One
  bucket's policy poisoned all buckets.
- `quote_with_proposed_day` — `requests.proposed_at`; `apex_save_request_quote`
  gains `p_proposed_at`; `accept_public_quote` books the day.
- `quote_customer_reply` + `quote_customer_reply_visible` — `requests.customer_reply`,
  `respond_public_quote()`.
- `order_refund_reason` — `online_orders.refund_reason`.
- `counter_orders_marked_in_house` — `customer_json.source = 'counter'`.
- Edge functions: `refund-order` v22, **`draft-site-copy` v1** (new).
- Site (Vercel project **apex-site** → `wisense-apex.vercel.app`): hours-block
  fix, hero photos rendered, honest quote-guard messages, proposed-day UI,
  suggest-a-change.

## 4. Two traps this day exposed

**The site deploy lineage.** `wisense-apex.vercel.app` is Vercel project
**apex-site** (`prj_txxA554McjYIWkIBIsjY5mEqFvMQ`), not `apex-venue-site` — the
folder's `.vercel/project.json` was stale and pointed at the wrong project. The
production site's code lineage is **`feat/services-merged`**, not
`feat/template-to-product`; deploying from the latter silently regressed eight
site commits (vertical detection, photo intake, quote-version binding,
dual-vertical landing copy). Corrected branch: `site/services-merged-hours`.

**"Looks the same" was two different things.** The quote *flow* rework was real
but invisible on the board, because the board itself was never touched — the
work had gone into the detail sheet and the customer's page. Diagnosing it as a
stale TestFlight build was wrong. Worth remembering: *when the user says
"nothing changed", check what they are actually looking at before blaming the
delivery chain.*

## 5. Migration files are NOT in the repo — open risk

Every production migration applied on 2026-08-06 was applied **via MCP** and is
recorded in `supabase_migrations.schema_migrations`, but the matching `.sql`
files were **not written into `apex_v2/supabase/migrations/`** (the permission
classifier blocked writes to that directory). They exist only in the session
scratchpad.

Combined with the pre-existing ledger drift (ledger says latest applied is
`20260901000000` while `20260915*` objects plainly exist), this means:

🔴 **`supabase db push` remains forbidden**, and the repo no longer describes
production. Recovering the files from the ledger and reconciling is an open
task. Note also that `20260806021805` sorts *before* the drifted `20260901*`
series.

## 6. What is NOT built — the honest gap list

Held against the toddler bar, these still fail:

1. **The launch wizard is still a checklist of seven nouns.** The toddler
   version is a conveyor: one question per screen, one big Next, no list to
   decode. **This is the biggest remaining swing** and deserves its own
   session with the bar pinned at the top.
2. **Payments onboarding** hands off to Stripe's own hosted forms, which cannot
   be simplified — only warned about ("the next screens ask about your bank,
   about 5 minutes").
3. **Restaurant-side names** — "86 Board", "QR Wall", "Sidework" — mean nothing
   to a first-timer.
4. **Get Found / share card** have never been held to the bar at all.
5. **Wayfinding generally.** The owner home and pinned nav attack it; whether
   they are *enough* is unjudged until Nicholas holds build 20.

## 7. Build state at time of writing

- Builds 17, 18, 19 were cut during the day; each raced ahead of in-flight
  commits, so no single build below 20 contains the full set.
- **Build 20 (from `4acc1ba` or newer) is the first build containing
  everything above.** The Mac was pulling for it when this note was written.
- Acceptance check on the iPad: version line reads `0.1.0 (20)`; services owner
  lands on the new home (NEEDS YOU NOW / TODAY / MONEY / YOUR PAGE); Requests
  opens with "Your move" and named buttons; Launch shows the "Next: …" button
  and "Review & publish" opens a filled form; the Tagline ✨ returns three
  options; back chevron pinned on Schedule/Team/Tips/Labor.

---

Related: [[projects/Apex v2 — Restaurant OS Build]] · [[projects/APEX_MASTER_PLAN]], [[projects/APEX_SERVICES_BUILD_PLAN_CANONICAL]],
[[projects/APEX_V3_BUILD_AND_STATUS_2026-08-05]], [[projects/APEX_V3_VS_V2_ASSESSMENT_2026-08-05]],
[[DECISIONS]], [[hot]], [[NOW]], [[index]]
