---
title: Apex v2 Template-to-Product Gap Map
tags: [apex, strategy, self-serve, build-doc]
aliases: [Template-to-Product Gap Map, Gap Map 2026-07-31]
date: 2026-07-31
---

# Apex Template-to-Product Gap Map

Date: 2026-07-31
Status: build doc — the bridge from "Nick + AI builds each client" to "the product builds each client"

## The thesis

Everything Jigsy's has — website, guest ordering, staff console, the Apex app,
payments, printer routing — was built by one founder with AI tools in about two
weeks. That proves two things: the template works, and every part of it is
parameterizable. The endgame is not more features. It is deleting Nick from the
loop: a business owner types their info, and the product stamps out the same
stack the Jigsy build produced, in one sitting.

This doc maps every piece of the Jigsy build to what it is today (Nick-powered)
and what it becomes (a button), then specs the five builds that close the gap.

The gap is not invention. It is three moves:

1. The template stops living in Nick's editor and starts living in the product
   (one renderer that reads venue rows).
2. The steps Nick ran with AI tools become product screens with buttons.
3. The legally-human steps (payments KYC, OAuth taps) get framed as guided
   moments, not removed.

---

## The gap map

| Piece | Exists today as | How it happens now | The v1 button | Build required |
|---|---|---|---|---|
| Venue identity | `venues` row + `public_token` | Nick creates it | Wizard step 1: "Type your business name" | Scrape-and-confirm (Build 2) |
| Branding / theme | Jigsy site's hardcoded colors, logo, photos | Nick hand-tuned the redesign | "Drop your logo and 3 photos" → palette + theme generated | Photo intake + palette (Build 4) + `venue_site_profile` fields |
| Website | The Jigsy site (reference implementation) | Nick built it as a codebase | "Here's your site" — published preview on night one | Multi-tenant renderer (Build 1) |
| Menu / services | `menus` / `menu_items`, `parse-menu` edge function | Nick ran imports | "Photograph your menu" → parsed → confirm | Already 80% built; wire `parse-menu` into the wizard |
| Guest ordering | Token-scoped public ordering page + `create-guest-payment` + `check-capacity` | Live at Jigsy's | Comes free with the rendered site — same renderer, order route | Build 1 (order route) |
| Payments | `square-connect-oauth`, `stripe-connect-onboard`, app-fee rails | Nick walked the venue through it | Week-1 coronation: "Connect your Square" — guided OAuth inside the wizard | Wizard step + framing copy only; rails already exist |
| Printer / POS routing | Square printer profile + `square-test-order` + day-one live validation checklist | Nick went on-site | "Send test order" button with a pass/fail checklist the owner self-serves | Extend `SQUARE_DAY_ONE_LIVE_VALIDATION.md` into an in-app checklist |
| Staff console | Role-based console in the app | Nick provisioned accounts | Week 3: "Add your team" — QR poster, invite flow | Invite polish (org/invite plumbing already exists) |
| Call-out rescue | `route-callout` + notifications | Built, untested in anger | Works out of the box once staff are in | Nothing — pilot proves it |
| Nightly briefing | `venue-briefing` edge function | Configured per venue | Week 4: turns on automatically after 7 days of data | Cron default per venue |
| Growth loops | None | — | "Powered by Apex" footer, before/after card, referral month | Build 5 |
| Multi-vertical | Template is restaurant-shaped | — | Vertical pack per industry (labels, menu schema, site sections) | Later — after restaurant vertical is self-serve |

**Read the right column:** of twelve pieces, only five require real build work,
and two of those are small. The rest already exists and just needs to be
reachable through the wizard.

---

## The template contract

The renderer can only be zero-touch if a complete venue profile is well-defined.
This is the contract — everything the product must know about a business to
render its entire stack with no human input:

**Already in the schema**
- `venues`: name, `public_token`
- `venue_hours`
- `menus` / `menu_items` (with availability / 86 state)
- Payment connection state (Square/Stripe credential rows, `square_environment`)
- `organization_members` (staff + roles)

**New — one table, `venue_site_profile` (or jsonb column on `venues`)**
- `display_name`, `tagline`, `about_blurb` (AI-drafted, owner-confirmed)
- `logo_url`, `hero_photo_urls[]`
- `palette` (primary / accent / background — generated from logo, overridable)
- `contact`: phone, address, maps link, socials
- `sections`: which site blocks are on (menu, about, gallery, hours, order CTA)
- `vertical`: `restaurant` for now; the field exists from day one so vertical #2
  is a data problem, not a rewrite

**Rule:** the renderer reads the contract and nothing else. If a field is
missing, the site renders a sensible default — the site can never be "broken,"
only "plain." The wizard's job is filling the contract; the renderer's job is
never needing Nick.

---

## Build 1 — the multi-tenant renderer (start now; zero pilot input needed)

One Next.js app that turns any venue row into the full Jigsy-grade site.

- **Routes:** `/{public_token}` → home; `/{public_token}/order` → guest
  ordering. Custom domains later (CNAME → Vercel), not v1.
- **Data path — the non-negotiable seam:** the renderer never queries tables
  with the anon key. It calls one `security definer` RPC,
  `get_public_venue_profile(token)`, that returns exactly the template contract
  for that one venue. No anon SELECT on `venues`, `venue_site_profile`, or
  `menus` — that is the no-cross-venue-bleed guarantee, enforced in the
  database, not in app code.
- **Theming:** palette + fonts injected as CSS variables from `venue_site_profile`.
  The Jigsy site is the visual reference; its sections become the component set.
- **Living-site property:** menu and hours render from live tables, so an 86'd
  item disappears from the public site the moment staff 86 it. This is the
  demo-able magic — the website is a view over the OS, not a second system.
- **Isolation test, written before launch:** seed two fake venues with
  different data; assert token A can never render venue B's menu, hours,
  branding, or order state, including error paths and unknown tokens.
- **Prerequisite from the security audit:** rotate the Jigsy `public_token`
  (still short/legacy) and enforce strong tokens for all venues before any
  public renderer goes live. Do this before QR codes get printed.

## Build 2 — scrape-and-confirm (night-one wow)

Owner types their business name → edge function pulls what's publicly known
(Google/Yelp/Facebook: address, hours, phone, photos, rating) → pre-fills the
template contract → owner confirms or corrects on one screen.

- The wow is the *absence* of a form: "we already know most of this — is it right?"
- Falls back gracefully to manual entry when scrape finds nothing.
- Every confirmed field lands in the contract, so the preview site at the end of
  night one is already 80% correct before they've typed anything.

## Build 3 — the wizard shell + AI guide

The wizard lives in the Apex app, where the owner already is. It is a checklist
of the gap-map buttons with a state machine per step:

`todo → ai_draft → needs_you → done`

- `ai_draft` means the system did the work and the owner only confirms
  (branding, blurb, menu parse).
- `needs_you` is reserved for the legally-human steps: payments KYC/OAuth,
  picking a payout account. These get framed as the coronation — "connect your
  Square and you're taking real orders" — not a chore.
- The AI guide is one assistant with the venue's contract as context answering
  a single question well: "what do I do next?" No chatbot sprawl in v1.

## Build 4 — photo intake + palette

- Logo upload → extract palette → generate theme (overridable swatches).
- Hero/food photos → compress, store, slot into site sections.
- Menu photo → existing `parse-menu` edge function → owner confirms items and
  prices on one screen.
- All three feed the contract; all three already have analogues in the codebase.

## Build 5 — growth loops (the app selling itself)

- **"Powered by Apex" footer** on every rendered site and ordering page —
  removable on a paid tier, which also gives the free tier a job.
- **Before/after card:** auto-generated side-by-side (old web presence vs the
  Apex site) the owner can post. One tap, branded.
- **Referral month:** owner's venue gets a month free per activated referral;
  tracked via a `referred_by` field on `venues`.
- **The guest ordering page is the demo.** Every customer who orders from any
  Apex venue touches the product; the footer tells them they can have it.

---

## The one-month journey (what the owner experiences)

| When | Moment | What fires |
|---|---|---|
| Night one | "Wow, that's my business" | Scrape-confirm → logo/photos → menu parse → **published preview site** before they close the app |
| Week 1 | The coronation | Guided Square/Stripe OAuth; first test order to their own printer (`square-test-order` pattern, self-served) |
| Week 2 | First real order | Share Order Link Pack (QR + social link); `check-capacity` guards on from minute one |
| Week 3 | Staff onboarded | QR poster + invites; call-out rescue armed |
| Week 4 | The habit | First automatic nightly briefing via `venue-briefing` — the loop that makes it indispensable |

## Build order

**Start now (no pilot input required):**
1. `venue_site_profile` schema + `get_public_venue_profile` RPC + isolation test
2. Renderer skeleton rendering the Jigsy venue from the contract (dogfood: if
   the renderer can reproduce the Jigsy site from data alone, the contract is
   right)
3. Token rotation + strong-token enforcement (audit carryover)

**Shaped by the pilot (build in parallel, let pilot friction edit the copy):**

4. Wizard shell + state machine
5. Scrape-and-confirm
6. Photo/palette intake wired to `parse-menu`
7. Self-serve version of the Square day-one checklist

**After the first self-serve client lands:**

8. Growth loops (footer, before/after, referral)
9. Custom domains
10. Vertical pack #2 — only after restaurants onboard without Nick

## Non-negotiables (seam discipline)

Every failure shipped so far was a seam failure, not a component failure. This
build's seams:

- The renderer's only data path is the definer RPC — day one, not a hardening pass.
- The two-venue isolation test ships with the renderer, not after an incident.
- Token strength is enforced at creation; the legacy Jigsy token rotates before
  public traffic.
- The wizard never marks a `needs_you` step `done` on the owner's behalf —
  payments activation is confirmed by webhook state, not by button taps.
- One venue's outage, bad data, or abuse can never degrade another venue's site.

## Definition of done for v1

Business #2 downloads Apex, and goes from zero to live site + taking orders in
one sitting, with Nick watching — not touching. When that happens, the template
has become the product, and the only remaining question is how fast word travels.

Related: [[wisense/projects/APEX_V2_SELF_SERVE_OS_GAMEPLAN_2026-07-28]], [[wisense/projects/APEX_V2_SELF_SERVE_FUNNEL_AUDIT_2026-07-28]], [[Apex v2 — Restaurant OS Build]], [[customers/Jigsys Brewpub]]
