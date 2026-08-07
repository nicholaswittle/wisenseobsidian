---
title: "Apex v2 — Fable 5 Deep Audit"
tags: [apex, audit, pricing, competitive, ai, fable]
date: 2026-08-03
commit: 38d7387
---

# Apex v2 — Fable 5 Deep Audit

Run against `feat/services-vertical` @ `38d7387`, 165 tests green, analyze clean.
40,278 lines under `lib/`, 20 test files / 157 `test()` calls, 33 migrations,
22 edge functions.

> **Caveat Fable stated up front:** its Supabase/Stripe/Vercel connectors were
> unauthorized, so it could not query the live catalog. Claims below are from
> source, the vault, and stated ground truth. Where I (Opus) verified against
> the live database afterwards, it is marked **[verified]** or **[corrected]**.

---

## Overall grade: C+ (with an asterisk)

The reasoning, stated explicitly: code craft is A-range — the comments alone
are a maintenance asset most funded teams don't have. The restaurant half is a
functioning B+. But **the vertical shipped yesterday is invisible to its
customers**, the flagship customer-facing artifact (the quote page) lands on a
sign-in screen, and payments have one test file.

A product is graded on what a customer can touch. Deploy `site/`, fix the two
link-builders, add the deposit-cap test, and this is a **B+ within a week of
evenings**.

---

## Feature grades

### Services vertical

| Area | Grade | Note |
|---|---|---|
| `core/vertical.dart` | **A** | "The best file in the codebase." Nouns vs content separated, restaurant byte-identical to pre-fork, verticals structural on `OsFeature`. Survives a third vertical without surgery. Leave it alone. |
| Request inbox / pipeline | **B+ design, D reachability** | Lanes named by owner's next action beats Jobber's pipeline. Money read from `request_payments`, not inferred from status. But the quote link is broken (below). |
| Jobs board | **A−** | Open slots as unassigned `shifts` rows so claiming *is* writing the shift — swaps, clock, call-outs keep working free. Race handled by `apex_claim_open_shift`. |
| Public services site | **A written, F served** | Phone-before-form, credentials high for HICPA, SMS consent suppressed for SHAFT businesses, quotes `noindex`. Sitting in a directory never deployed. |
| Service editor | **B** | Free-text prices right for trades. Appropriately thin. |
| `create-request-payment` | **A−** | HICPA one-third cap server-side, cumulative-payment aware, fee clamped below amount. Most legally-aware code in the repo — and **zero tests**. |
| Get Found | **B−** | Honest self-reported checklist, correctly free. Weakness: static advice. |

### Restaurant vertical

| Area | Grade | Note |
|---|---|---|
| Ordering stack (7,277 lines) | **B+** | Hardened by July payment audits. `staff_console_screen.dart` at 2,001 lines with one indirect test is the structural risk. |
| Schedule + photo import | **A−** | Opus-for-image / Haiku-for-text split is measured, not assumed. |
| Tips | **B+** | Pool eligibility server-side, anon EXECUTE revoked. |
| Labor cost / vs revenue | **B** | Correct, but value depends on order volume that doesn't exist yet. |
| Dashboard / shell | **A−** | Group-ordered nav with a pinning test. |
| Admin console + view-as | **A−** | Read-only enforced by DB, unmissable banner. Solo-operator support tooling done right. |

### Two concrete bugs it found by reading source

1. **[corrected — worse than previously reported]** `_quoteLink`
   (`request_inbox.dart:597–604`) builds `https://apex-v2-ten.vercel.app/<token>/quote/<id>`.
   Root `vercel.json` rewrites every path to Flutter's `index.html`; Flutter
   reads only `?token=`. A path-style URL has no query param, so the customer
   lands on the **Apex sign-in screen** — not the wrong menu. The entire
   quote→accept loop is unreachable by any customer.
2. **[verified and fixed 2026-08-03, commit `9bb9cab`]**
   `jobs_board_screen.dart:754` read `'\$n spots open'` — escaped dollar, so a
   job with three open slots literally displayed "**$n spots open**" on the
   first screen a crew member sees.

---

## Pricing

### The asymmetry nobody had noticed

On restaurants the 1.5% is **guest-paid**, added on top. On services,
`create-request-payment` deducts it as `application_fee_amount` from the
deposit — **the contractor pays it**. Same headline number, opposite payer.
Defensible (a service-fee line on a home-improvement deposit reads badly and
brushes surcharge rules), but it should be a decision, not an accident of
which function was written first.

### What the services 1.5% actually yields

Landscaper at ~$250k/yr, ~120 jobs at ~$2,000 average, deposits capped at
one-third (~$667):

- Optimistic — half of jobs deposit through Apex: 60 × $667 = $40k → **$50/mo**
- Realistic — phone-agreed work, checks, Venmo: 20 jobs → **$17/mo**

Compare a restaurant at Jigsy's $292k with 10% online: ~$36/mo, guest-paid.
So the services fee is **smaller in expectation and more resented**, because
the contractor watches it leave their deposit.

**Recommendation:** keep it, price it differently — *"$99/mo, and 1.5% only on
money we collect for you."* Frame it as the cost of the collections rail, not
a platform tax.

### Flat-per-location for services? Mostly no

Restaurant OS at $99 includes ordering console, KDS, capacity, 86 board, tips.
Services OS at the same $99 includes the request inbox and jobs board.
**The services customer gets materially less for the same price**, and their
reference point is Jobber Core at $39/mo.

But per-seat is Jobber's #1 complaint — $29/user pushing bills past $500. So
the wedge is **flat price, unlimited crew, positioned explicitly against
per-seat**. "Your whole crew, one price" is a sentence Jobber structurally
cannot say.

- **Services Pro $49/mo** — inbox, quote pages, deposit links, jobs board, unlimited crew
- **Services OS $99/mo** — adds Get Found automation and the site
- A 3-crew landscaper needs Jobber Connect at ~$149/mo. At $99 flat you
  undercut *and* out-simplify. At $49 you make no money — don't go there.

### Grandfathering

Grandfather **exactly one** customer: Jigsy's, forever, in writing, as the
reference account — Emily's introductions are the only distribution channel.
**Do not grandfather customer #2.** Three customers at three legacy prices is
a bookkeeping tax on a one-man operation, and every SaaS that grandfathers
early pricing spends year three unwinding it. Use annual prepay (2 months
free) instead — it converts churn risk into cash now.

### Money left on the table / too greedy

- **Left on table: the $499–799 site build for services.** Agencies charge
  trades $2–5k, and a PA services business has a *legal* reason to buy —
  HICPA requires the registration number in advertising and the renderer
  already puts credentials above the fold. One site build = 5–8 months of
  subscription. At this scale, one-time cash beats MRR.
- **Too greedy to support: Multi at $199.** Zero prospects, and every Multi
  customer is a support surface that can't be staffed. Remove from the public
  list; quote privately if asked.
- Target check: 3 × $99 = $297 + fees ≈ **$330–350/mo**. Achievable *if* the
  services tier survives a Jobber-priced market — which needs the
  unlimited-crew story attached.

---

## Competitive analysis

### Restaurant — defending Jigsy's, not conquering

| Competitor | Good at | Real complaints | Price | Apex |
|---|---|---|---|---|
| 7shifts | Restaurant-native scheduling, labor forecasting | Slow loads, random logouts during service | Free ≤30 staff; $39.99+/location | **Loses** — their free tier covers Jigsy's entirely |
| Homebase | Cheap, broad SMB base, payroll | Cancellation friction, post-cancel charges | Free ≤20 at one location; ~$24/location | Wins only on "one system, one login" |
| ChowNow | Direct ordering brand, marketplace | Rising renewals; an unauthorized $9.99 diner fee | $229–449/mo + setup + 2.95% | **Wins decisively on price** |
| Owner.com | Marketing automation, 4.8 G2 | 5% on *direct* orders | $249/mo + 5% | Wins on fee honesty; loses on marketing muscle |
| Toast / Square | The POS *is* the install base | Per-order fees, hardware lock-in | Bundled | **The real competitor is "do nothing, Square already does this."** Be its missing layer, don't fight it |

Honest conclusion: in restaurants Apex wins only as a *bundle at a mom-and-pop
price*. It loses on integrations, install base and support. Fine for three
venues by warm intro. **Does not support cold acquisition** — nothing should be
built as if it does.

### Services — the open flank, and the better market

| Competitor | Good at | Real complaints | Price | Apex |
|---|---|---|---|---|
| Jobber | Category default; quote→invoice→payment loop | Per-user $29/ea, tier jumps at 5/10/15, add-ons ($99 AI receptionist) past $500 | $39–599/mo | Loses on maturity. **Wins on flat pricing + crew self-claim** — Jobber dispatch is top-down only |
| Housecall Pro | Marketing polish | Add-on cost creep; $110 for a second user; billing-after-cancel | $79–329/mo | Counter-story: no add-ons, ever |
| ServiceTitan | Enterprise depth | Explicitly not for ≤3 techs; $245–398/tech/mo, $5k–50k implementation | — | Irrelevant except as proof trades pay real money |
| Square Appointments | Free-ish, trusted | Appointment-shaped not job-shaped — no quotes, crews or deposits | Free–$69 | Apex's quote/deposit/crew loop is outside its shape |
| Thumbtack / Angi | Demand generation | Ghost leads (~75% reported), leads resold to 3–8 contractors, FTC complaint 2023, VT AG settlement 2025 | ~$288/yr + $15–85/lead | **Not competitors — the villain in the pitch** |

> **The structural read:** the restaurant market punishes Apex (free
> incumbents, POS bundling); the services market rewards exactly what Apex is
> (flat price, no per-seat, owner-owned lead capture, deposit rails with a
> statutory cap built in). **The competitive set didn't just double — it
> tilted. Services is the better half.**

---

## AI — past the current doctrine

Current: Haiku everywhere, Opus only for image parsing, Gemini Flash fallback
on menu parse, `venue-briefing` deliberately model-free, support agent pinned
with three allowlisted repairs. The doctrine ("drafting layer on the money
rail, suggest never commit") is right and not relitigated. What changed: the
services vertical gives Apex an **inbound phone problem**, and inbound phone
is where SMBs demonstrably pay.

**A. Apex Front Desk — the one to build.** ~74% of home-services calls go
unanswered. Jobber sells an AI Receptionist add-on at **$99/mo**; the
standalone market clusters at $99–299/mo. Apex already owns the Twilio number
(A2P pending), the `requests` table and the quote link. Missed call → instant
SMS with the quote link → a Haiku SMS agent that conversationally fills the
request form and writes a `requests` row. **Creates a request, never a
booking** — doctrine intact. Cost: number $1.15/mo, SMS ~0.8¢, Haiku turns
fractions of a cent — **under $5/mo per busy tenant**, chargeable at **$29–49**.

**B. Photo-to-quote drafting.** Opus vision drafts scope line items from
customer photos, flags what it can't price. Owner edits, nothing auto-sends.
~$0.05–0.15 per quote.
*(Photo upload shipped 2026-08-03, commit `0e0f41f` — the input half of this
now exists.)*

**C. Get Found, automated.** Post-A2P: after a paid `request_payments` row,
SMS the client a review ask with the GBP link; draft (never post) review
replies. Converts a static checklist into a loop.

**D. Cut the model call in `route-callout`.** Ranking ~8 replacement
candidates is a deterministic sort. The Haiku call adds latency and
nondeterminism to a list a human scans in two seconds.

**E. Don't build a voice agent yet.** SMS-first captures most of the value
with none of the "your robot hung up on my customer" risk.

Total at 3 tenants with all of the above: **under $20/mo** against $87–147/mo
of add-on revenue.

---

## New ideas, ranked by impact × solo-feasibility

1. **The quote page is a distribution channel.** Every accepted quote is read
   carefully by a homeowner who hires other trades. One quiet line in the
   QuoteView footer: *"Powered by Apex — quote pages for your business."*
   Calendly/Stripe-invoice mechanics: the customer's paperwork recruits the
   next customer. One evening, including a referral-code param into the
   existing `referrals` table.
2. **The "stop paying Angi" wedge** (marketing, not code). The complaint
   corpus is enormous and documented. Pitch: *"$99/mo flat. Your leads land in
   your inbox, from your link, and nobody resells them to three competitors."*
   Landing page + PA contractor Facebook groups. Asynchronous — fits a
   full-time job.
3. **HICPA compliance as the PA sales hook.** Registration numbers required in
   advertising; quote pages and site already surface credentials. A wedge no
   national competitor bothers with, in the exact geography where the warm
   network lives.
4. **Uncomfortable: demote restaurant to maintenance.** The master plan says
   "don't build vertical two until vertical one is paying" — but vertical two
   is built, and it landed in the better market. **Jigsy's is the reference
   customer and the lab; customers #2 and #3 should be a landscaper and an
   HVAC/plumbing shop.** Emily's network supplies them too — brewpub owners
   know their contractors.
5. **Retention: the Monday "what came in" SMS.** Template, no model:
   *"3 requests last week ($4,200 quoted). 1 still needs a price."* The owner
   who gets that text does not churn, because churning means the requests stop.
6. ✅ **DONE 2026-08-03 (`a1a9b8c`):** partial unique indexes on
   `square_merchant_id` AND `stripe_account_id` (copy-cat rows cleaned,
   Jigsy's keeps its credential); HICPA cap extracted to
   `_shared/deposit_cap.ts`, six-case test written and verified under node,
   handler refactored onto the tested module and redeployed (v3).

**Do first, in order:** deploy `site/` + fix the link bases → ~~fix `$n spots
open`~~ *(done, `9bb9cab`)* → quote-page footer referral line → deposit-cap
test + merchant-id constraint → Apex Front Desk once A2P clears.

---

## Verified against the live catalog after the audit ran

Fable could not reach the database. These were checked afterwards:

- **`referrals` table exists** (added 2026-08-03). Older vault notes calling
  the referral mechanism "decorative — no table" are stale.
- **`growth_tasks` exists** — Get Found persistence.
- **Three orgs share one `square_merchant_id`** (jigsys, moe's, test rest).
  Harmless while nothing is live; must be true before a real card runs.
  This is the concrete case for idea #6.
- **Bradley's has its own Stripe Connect account** (`acct_1U0X…`), distinct
  from Jigsy's — it did not need Jigsy's copied onto it, only the
  `stripe_charges_enabled` flag set.
- **Nothing is live.** No real customers, no real money, Stripe in test mode,
  simple passwords deliberate. "Untested but honest" — not "exposed".

## Sources

[servicestorm.io](https://servicestorm.io/articles/jobber-pricing) ·
[myquoteiq.com](https://myquoteiq.com/jobber-pricing-breakdown-2026/) ·
[tooleduppro.com](https://tooleduppro.com/guides/housecall-pro-pricing/) ·
[projul.com](https://projul.com/blog/housecall-pro-pricing-analysis-2026/) ·
[procured.us](https://procured.us/articles/servicetitan-pricing) ·
[serviceagent.ai](https://serviceagent.ai/blogs/servicetitan-pricing-explained/) ·
[savullc.com](https://savullc.com/thumbtack-pro-reviews/) ·
[adaptdigitalsolutions.com](https://adaptdigitalsolutions.com/articles/homeadvisor-vs-angieslist-vs-houzz-vs-porch-vs-thumbtack-vs-yelp-vs-bark/) ·
[selecthub.com](https://www.selecthub.com/employee-scheduling-software/homebase-vs-7shifts/) ·
[stackscored.com](https://www.stackscored.com/pricing/employee-scheduling/compare/7shifts-vs-homebase/) ·
[get.chownow.com](https://get.chownow.com/blog/restaurant-marketing-reviews/) ·
[getsauce.com](https://www.getsauce.com/post/chownow-review) ·
[trustpilot.com](https://www.trustpilot.com/review/chownow.com) ·
[trillet.ai](https://trillet.ai/blogs/best-ai-receptionist-for-small-business-2026) ·
[agentzap.ai](https://agentzap.ai/blog/ai-receptionist-pricing-complete-cost-guide-2025)

---

Related: [[projects/Apex v2 — Restaurant OS Build]] · [[projects/APEX_SERVICES_COMPETITIVE_BUILD_MAP]] · [[projects/APEX_V3_VS_V2_ASSESSMENT_2026-08-05]] · [[hot]] · [[NOW]] · [[index]]
