---
title: Apex Services — Canonical Build Plan
tags: [apex, verticals, services, build-doc, canonical]
date: 2026-08-02
---

# Apex Services — Canonical Build Plan

**Reconciled from two independent plans (Claude + Fable 5), 2026-08-02.** This
is the document to follow. Research detail lives in
[[projects/APEX_SERVICES_VERTICAL_FULL_BUILD_2026-08-02]].

---

## 1. Thesis

Apex's core — people show up somewhere at a time, somebody pays, everyone needs
telling what changed — is industry-agnostic. A second business type is a
**configuration**, not a fork. Restaurant stays the focus. Services is the
ability to say **yes** when a warm introduction hands you a business that isn't
a restaurant.

## 2. Settled — do not re-open

One app · vertical chosen at org creation · **the wedge is getting paid**, not
job margin · **GBP is the front door**, the site is the second room ·
phone-first, form-second, booking-third · 5-field form, capture-then-qualify ·
A2P 10DLC is the moat, SHAFT bars firearms from SMS · free site forever ·
$499–799 build · **$99/mo flat, unlimited users** · 1.5% on collected payments ·
never collect 4473 data · never generate HICPA contracts · never install an
accessibility overlay · *do not build vertical two until vertical one is paying*.

## 3. The parallelism rule

> **New objects are safe alongside restaurant work. Modified shared objects are not.**

---

## 4. ⭐ The decision that shapes everything: do not touch `menu_items`

My first draft proposed making `menu_items.price_cents` nullable and adding
`price_type` — a schema change to the most load-bearing table in the restaurant
product, to express "call for estimate."

**Unnecessary, and it broke my own rule.** The public site renders from
`get_public_venue_profile`, so a **`services` jsonb column on
`venue_site_profile`** — a new object — expresses the entire services catalog
with zero shared-table risk. Deposits attach to `requests`, not to catalog rows.

`menu_items` adaptation waits until a paying services org needs the in-app
Flutter catalog. At zero services customers that is not a requirement.

This removes the single riskiest change from the critical path and leaves
**exactly four shared-object edits in the whole plan**, batched into one
deliberate window (§5.2b).

---

## 5. Phases

### Phase 0 — done ✅
`organizations.vertical` (`20260901000000`, inert by design) · venue name
de-hardcoded from order SMS · showcase link moved to env.

---

### Phase 1 — The services site · ~55–70 hrs · **safe now, fully parallel**

`site/` is a separate Vercel deployable. Nothing here can reach the Flutter
build on Emily's iPad.

**1a · Schema — new objects only (~6 hrs)**

Additive jsonb on `venue_site_profile`, inheriting existing RLS: `services`,
`credentials`, `how_it_works`, `service_area`, `theme`, `form_config`, `faq`,
`legal`. Extend `sections` with `services / how_it_works / credentials /
request_form / faq`. Adding keys to `get_public_venue_profile`'s return shape
is additive — the restaurant renderer ignores what it doesn't know.

```sql
create table requests (
  id uuid primary key default gen_random_uuid(),
  organization_id uuid not null references organizations(id),
  mode text not null check (mode in ('point_of_service','quote_to_invoice')),
  client_name text not null, client_phone text not null, client_email text,
  service_town text,          -- town at capture, not street
  service_address text,       -- filled later
  services jsonb not null default '[]',
  urgency text check (urgency in ('asap','this_week','flexible')),
  detail jsonb not null default '{}',   -- step-2 answers
  notes text,
  status text not null default 'requested'
    check (status in ('requested','quoted','approved','scheduled',
                      'in_progress','complete','cancelled')),
  quoted_cents int, deposit_cents int, final_cents int,
  scheduled_at timestamptz,
  sms_consent_at timestamptz, sms_consent_source text,
  source text not null default 'site',
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);

create table request_payments (
  id uuid primary key default gen_random_uuid(),
  request_id uuid not null references requests(id),
  organization_id uuid not null references organizations(id),
  kind text not null check (kind in ('deposit','balance','invoice')),
  amount_cents int not null check (amount_cents > 0),
  provider text not null check (provider in ('stripe','square')),
  provider_object_id text,
  status text not null default 'pending'
    check (status in ('pending','paid','refunded','void')),
  paid_at timestamptz, created_at timestamptz not null default now()
);

create table request_reminders (
  id uuid primary key default gen_random_uuid(),
  request_id uuid not null references requests(id),
  channel text not null check (channel in ('sms','email')),
  template text not null, sent_at timestamptz not null default now()
);
```

RLS: org-member read/write mirroring `venue_site_profile`. **No anon table
grant** — public inserts go only through a `security definer`
`submit_public_request(public_token, payload)` with per-token rate limiting,
matching the `get_public_venue_profile` posture.

**1b · The generated theme system (~30–40 hrs)** — §6 in full. The largest line
and the acquisition instrument.

> **De-risking option:** ship skeleton **A only** (~12 hrs, the source exists in
> Dimmsville) plus the derivation engine (~10 hrs). B and C follow on demand.
> That lands Phase 1 nearer **35–45 hrs** without weakening the wedge.

**1c · Form + notify (~10 hrs)** — 5 fields, one column, `tel:`/`email`
inputmodes, trade fields behind step 2 **after name+phone land**. Owner
notified by a new edge function: **email always**; SMS only post-10DLC and
never for a firearms org.

**1d · Legal furniture (~5 hrs)** — auto-generated privacy policy per tenant
(CalOPPA, no small-business exemption) · HICPA registration number in the
credentials block and footer · WCAG 2.1 AA in the skeletons (contrast-clamped
palettes, focus states, `prefers-reduced-motion` guard on scroll reveal).

**Exit:** the brother-in-law or Dimmsville live on their own domain, a real
request in `requests`, an email arriving. Gates nothing; gated by nothing.

---

### Phase 2 — Get found · ~15–20 hrs · **safe now, parallel**

Where the customers actually come from. GBP outranks the website for discovery
and neither of us should have left it to last.

1. **GBP claim + optimize as an onboarding checklist** — category, hours,
   service area, photos, description. Eight of the top ten local-pack factors
   are GBP-native.
2. **Tracked number** → *"your Google profile sent you 14 calls this month."*
   Google killed GBP call history in 2024, so **nobody has this.**
3. **Referral table** — the tile in the app records nothing because no table
   exists. ~2 hrs, and word of mouth is the only channel.

---

### Phase 3 — The money rail · ~50–65 hrs · **gated on Jigsy's live and stable**

**3a · New objects — parallel-safe (~35 hrs)**
- Requests inbox + quote composer in Flutter. Only shared-file touch is one row
  in `moduleRoutes` and one `OsModule.requests` case — a two-line additive diff.
- `create-request-payment` edge function — Stripe payment link for
  deposit/balance/invoice, 1.5% application fee, same Connect posture.
- **New** `request-payment-webhook`, *not* an extension of `stripe-os-webhook`.
  A new function cannot regress order payments.
- Reminder engine: pg_cron over unpaid `request_payments` → cadence rows →
  send. **Cadence and scheduling are deterministic**; only wording is AI-drafted.
  Recipient-timezone quiet hours and STOP handling live here. `mode` switches
  the template set — Mode A gets pre-appointment deposit/reminder, Mode B gets
  overdue-invoice.
- **PA guardrail: cap the deposit at one-third** over $1,000 in the quote
  composer UI, not a check constraint — the statute has conditions a constraint
  can't see.
- **PA service sales tax by category** — lawn care, window cleaning, janitorial
  taxable; plumbing and salon not.
- **Start A2P 10DLC registration the day Phase 3 opens.** Zero engineering
  hours, multi-week carrier latency, gates all customer SMS. Do not discover
  this late.

**3b · The shared-object window (~8 hrs, one deliberate evening)**
Four edits, batched, shipped together, verified against Jigsy immediately after
— honoring the runbook lesson that a server change must never land before the
client that can handle it:

1. `venue_auto_bootstrap` — branch on `vertical`; a services org gets no
   `restaurants` row, or a shell if the `venue_site_profile.venue_id →
   restaurants.id` FK makes that cheaper. **Decide against the live catalog.**
2. `apex_onboard_venue` — same branch; services writes profile + org only.
3. `check-capacity` — exclude services orgs. It sweeps every
   `restaurant_settings` row service-role today.
4. `venue-briefing` — same skip.

**3c · Label layer (~12 hrs)** — the ~100–150 shared strings. Deferred to the
moment a services org actually opens the Flutter app; until then it is polish
on an empty room.

**Exit:** one services customer with a paid deposit through the system.

---

### Phase 4 — Job margin · ~25–35 hrs · **gated on money flowing**

- **Hard precondition:** `shifts`, `time_entries`, `profiles` are in **no
  migration**. Dump their live columns from `pg_catalog` before writing a line.
- Additive nullable `shifts.job_id uuid references requests(id)` — goes through
  the same deliberate-window treatment as 3b.
- One tap on arrival, one on departure. Not a timesheet. Reuses time-clock
  rails; wall QR remains the clock-in story.
- Completion margin line, then the 20-job insight.

**Month-three retention feature. Do not let it creep forward.**

---

### Phase 5 — demand-gated
Trade-specific step-2 field packs · Square as a second request-payment provider
· GBP review-response tooling · services variant of the support-agent prompt.

---

## 6. Making the free site not look generated

**Generate per tenant from a seed. Do not pick from presets.** Five presets is
a picker, and pickers converge — by the third customer in one county two sites
share a skin and the acquisition instrument reads as a template farm.

**Three skeletons × deterministic per-tenant derivation.** Same build cost,
unbounded output.

**The tells that read as template:** symmetric everything · default radii and
Inter-600 · no dark band · stock photos or SaaS-purple gradients · copy without
specificity · no operational furniture. Dimmsville avoids all six —
`--radius: 3px`, headings at weight 800 uppercase with **−0.035em tracking**, a
`#0E1014` hero band, **zero photos** (layered gradients), and *"Who made the
firearm — not who you bought it from."*

**What derives automatically** from name + category + one photo + a palette
anchor:
- **Type** — hash the name to a stable seed; category picks the pairing family,
  seed picks within it (≈6 self-hosted variable pairings, modular scale
  1.25–1.333, display weight 700–850, tracking −0.01 to −0.04em).
- **Colour** — `ThemeRoot` already derives 17 tokens from 3 inputs. Two
  upgrades: derive the anchor from the photo (dominant-colour quantization at
  publish, cached into `theme`), and **contrast-clamp every derived pair to AA
  programmatically**. That clamp is what makes generated palettes shippable
  rather than a liability.
- **Layout** — seed picks hero variant, card treatment, ticker on/off, radius
  2–6px (never default-rounded).
- **Copy** — AI-drafted from name/category/town, owner confirms.

**Cannot derive:** credentials, real hours, the photo (owner-supplied
permanently), final say on copy.

**Three skeletons:** **A — Storefront** (trades, mechanic, gun shop, detailer;
Dimmsville *is* this) · **B — Studio** (salon, photographer, groomer, tutor;
light, photo-forward, serif display) · **C — Crew** (cleaner, caterer,
contractor, mover; how-it-works leads, quote CTA dominant, trust strip above
the fold). 2–3 hero variants each = 7–9 distinct structures before derivation
multiplies them.

Non-negotiable in all three: sticky header with tappable `tel:` · sticky bottom
call bar after the hero scrolls · 5-field one-column form · AA enforced by the
derivation · mobile-first · reduced-motion-safe reveals.

---

## 7. AI

**AI is a drafting layer on the money rail.** It writes words a human approves,
and the one place it earns Opus money is turning a paper customer list into
verified, geocoded, deduplicated records.

**Doctrine:** suggest never commit · never guess between two people · a failed
call bills like a success · probe a model ID before trusting it · score against
a confirmed answer, more than twice · a cap must never become a staffing or
safety failure.

| Existing | Verdict |
|---|---|
| `parse-schedule`, `parse-log-summary`, `route-callout`, `enrich-business` | **carry unchanged** |
| `parse-menu` | variant → `parse-pricelist` (+ customer list, route sheet) |
| `venue-support-agent` | gate off for services until a variant exists |
| `venue-briefing` | **no model at all for services** — new requests, tomorrow's jobs, unpaid balances, days since oldest invoice are three queries and a template. Do not rebuild the restaurant mistake in vertical two. |
| `polish-labor-warnings` | **cut** (already sentenced) |

**The onboarding wedge, honestly:** OCR is table stakes; **verified geocoded,
deduplicated records** are the product. The model does layout understanding and
field segmentation only. **Geocoding goes to a geocoder, never the model** — a
model invents plausible addresses, and a wrong service address is a truck in
the wrong driveway. Dedup follows `name_matcher`: strong matches merge,
ambiguity goes to a review queue. *"62 imported, 3 need your eye"* is the demo
moment; *"65 imported"* with a silent bad merge is how trust dies.

**Build:** quote draft from intake (Phase 3) · overdue-collection wording
(Phase 3) · site copy (Phase 1) · review-response drafts (Phase 5, start
paste-in before GBP OAuth).

**Do not build — urgency triage.** The form's fifth field *is* the urgency,
stated by the customer. A model re-deriving what a radio button captured is
decorative AI. Deterministic sort: ASAP first, then age.

**Never:** autonomous send · free-form customer chatbot · pricing authority ·
identity resolution by guess · scheduling commits · money actions · any SMS for
a firearms org.

**Cap-degrade matrix:** collection nudge → **static template, never silent** ·
quote draft → blank composer · review draft → nothing. Nothing in services is
safety-critical under a cap, and that property is worth preserving — a reason
the support agent comes last.

**Cost:** ~80 Haiku calls/tenant/month ≈ $0.08, **×3 worst-case retry = $0.24**
against $99 revenue — 0.25%. One-time onboarding (Opus imports + copy) ~$0.40.
The per-org DB cap stays mandatory anyway: the story holds at 3 tenants and 300
only if a loop-spender cannot exist.

---

## 8. Deliberately not built

GPS geofence · `menu_items` schema change · recurring schedules · route
optimisation · materials markup · change orders · client CRM · invoices
spanning requests · HVAC dispatch/parts/service agreements · 4473 data · HICPA
contract generation · accessibility overlay · autonomous scheduler · free-form
chatbot · urgency triage · forked app · $25 tier · per-user pricing.

## 9. Totals

| Phase | Hours | Gate |
|---|---|---|
| 1 — Services site | 55–70 *(35–45 with skeleton A only)* | **none** |
| 2 — Get found | 15–20 | **none** |
| 3 — Money rail | 50–65 | Jigsy's live |
| 4 — Job margin | 25–35 | money flowing |

**~145–190 hrs end to end. ~50–90 hrs of it is spendable today**, in parallel,
with zero restaurant risk.

## 10. Open

1. Do `shifts` / `time_entries` have the assumed columns? **No migration
   defines them — check `pg_catalog` first.**
2. Does a services org need a shell `restaurants` row for the
   `venue_site_profile.venue_id` FK, or is the FK repointed? Decide against the
   live catalog.
3. Which tracked-number provider, and does it survive 1.5% economics?
4. Does monthly-invoice billing break anything in the existing Stripe flow?
