---
title: Apex Self-Serve — Plan Revisions and the $300/Month Target
tags: [apex, strategy, self-serve, plg, revenue, build-doc]
date: 2026-08-01
---

# Apex Self-Serve — Plan Revisions and the $300/Month Target

Revisions to [[projects/APEX_V2_TEMPLATE_TO_PRODUCT_GAP_MAP_2026-07-31]]
and [[projects/APEX_V2_SELF_SERVE_OS_GAMEPLAN_2026-07-28]], written
after reviewing the `feat/template-to-product` branch.

---

## The goal, stated plainly

**$300/month by the end of year one from launch.**

At the OS tier that is **three paying venues**. Not thirty. Three.

That number should govern every prioritisation call, because it changes what
"done" means. The gap map's definition of done — *"business #2 goes live with
Nick watching, not touching"* — is the right long-term bar, but it is not what
the first year requires. Three venues can be onboarded by hand.

**The self-serve build is therefore not an acquisition necessity. It is the
product.** It is what makes the demo land, what makes the thing feel inevitable
rather than bespoke, and what makes venue #4 cost nothing to add. Build it
because it is the differentiator, not because three customers demand it.

### What that reframe changes

**Acquisition is not the hard part at this scale.** Three venues is a
warm-introduction problem, and Jigsy's is well established with the local
mom-and-pop owners. That network, not a funnel, is the year-one plan.

**Retention is the hard part.** The thesis — *"once one person is invested in
all this they'll stay if it works right"* — is correct, and it means the
post-launch priority is the **daily-value loop**, not more onboarding polish:

- the nightly briefing (`venue-briefing`) that makes them open the app tomorrow
- labor cost against revenue, which their POS leaves at 0.0%
- call-out rescue when someone no-shows on a Friday
- capacity guarding when the kitchen is drowning

Onboarding wins the customer. **The daily loop is what stops them leaving**, and
at three venues, one churn is a third of the business.

---

## What the build already got right — do not "fix" these

Reviewed on `feat/template-to-product` (69 files, Builds 1–5):

- **`supabase/tests/venue_profile_isolation.sql` — 224 lines, shipped *with* the
  renderer**, not after an incident. This was the doc's central non-negotiable
  and it held.
- **`enrich-business` pulls no photos.** It uses the Google Places API for
  name, address, phone, hours and rating only. This avoids industrialising the
  exact problem hit on 2026-07-31, when Jigsy's site used guest-uploaded Google
  photos and the hero screenshot had to be pulled from wisensellc.com. Scraped
  photos would have put someone else's images on every venue's site under our
  footer. **Keep photos owner-supplied, permanently.**
- **`venue_assets_org_scoped_writes.sql`** — asset storage scoped per org, so one
  venue cannot write into another's bucket path.
- **`get_public_venue_profile`** as the renderer's only data path, per the seam
  rule.

---

## Revisions

### 1. Declare Apex the menu source of truth — and say so to the venue

Unresolved in both plans. The renderer reading live tables fixes site-vs-app
drift, but not **Apex-vs-Square drift**: a venue changes a price in their POS and
the ordering page is stale. At one venue this is a month-three annoyance; at N
venues it happens N times and most will never be reported.

**Decision to make explicit:** Apex is authoritative for the online menu. Say it
in the wizard, in plain words, at the moment they confirm their menu — *"prices
here drive your website; update them here."* A one-line expectation set at
onboarding costs nothing and prevents the class of complaint that erodes trust
quietly.

Square catalog sync stays a later feature, not a v1 promise.

### 2. Instrument the wizard funnel from the first venue

"Business #2 goes live with Nick watching, not touching" is only measurable if
you know where they stalled. Per-step `enter / complete / abandon` events, from
the first real run.

Cheap now, impossible to backfill, and with only three venues in year one
**every abandonment is a large fraction of the data you will ever have.**

### 3. Resolve the onboarding-philosophy contradiction

The two plans pull opposite directions at the conversion step:

- Gameplan: *"They invest 15 minutes of labor into Apex. They now feel complete
  ownership."*
- Gap map: *"The wow is the absence of a form."*

**Resolution: AI drafts, owner confirms.** Never a blank form, never a passive
watch. Every screen arrives pre-filled and every screen requires a tap to accept.
They touch every field — that is the ownership — but they never face emptiness.

The gap map's `todo → ai_draft → needs_you → done` state machine already models
this; it just needed deciding out loud so the two documents stop disagreeing.

### 4. Rotate Jigsy's `public_token` before any QR code is printed

Audit carryover, still open, and it has a hard deadline that is easy to miss: a
token in a database rotates in a second, a token on printed QR posters does not
rotate at all. Do it before the Order Link Pack ships.

### 5. The AI guide is the product, not a feature

The stated goal — *"walked through the app, from building their site to actually
running their business"* — is the difference between "I want this" and "I need
this." Nothing else in the stack is hard to copy; a POS vendor can ship online
ordering. What they will not ship is a guide that knows this venue's numbers and
tells the owner what to do next.

That argues for the AI surface extending past onboarding into operations:
tonight's labor cost, tomorrow's schedule gap, this week's slow night. The
onboarding wizard is chapter one of that, not a separate feature.

Keep the gap map's v1 discipline — **one assistant, one question, answered well:
"what do I do next?"** — and let it grow into operations after launch, not
before.

---

## Prioritisation, given three venues

**Before launch:** token rotation · funnel instrumentation · menu-authority copy
in the wizard.

**At launch:** the pay-at-pickup floor, so go-live is not gated on OAuth,
printing, or owner availability (see
[[projects/JIGSYS_PILOT_LAUNCH_STRATEGY_2026-07-31]]).

**After the first paying venue:** the daily-value loop — briefing, labor cost,
call-out rescue. This is what earns month two.

**Not yet:** custom domains, vertical pack #2, Square catalog sync. All are
correct eventually, none move the year-one number.
