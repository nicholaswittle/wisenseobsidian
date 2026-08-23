---
type: meta
title: "Hot Cache"
tags: [meta, hot-cache, context]
updated: 2026-08-23
---

# Recent Context

> **DISPATCH FROM [[projects/APEX_MASTER_PLAN]].** It is the single ordered view of state, constraints and the work queue. If it disagrees with a tool's memory, the master plan wins.

## Apex Core + Apex Comms — START HERE — 2026-08-23

**Canonical note:** [[projects/APEX_CORE_AND_COMMS_2026-08-23]].
**Live status is NOT in this vault** — it is `apex_v3/docs/STATE.md`, machine
refreshed, and CI goes red when it is over seven days stale. Read that before
asserting anything about state. Everything below is orientation.

**Apex v3 is now Apex Core.** The icon says just **Apex**; "Core" only
distinguishes it from **Apex Comms** in writing. The folder and bundle id keep
the `v3` spelling deliberately (App Store record + a live Apple entitlement
request).

**Two apps.** Core is the business's own app — quotes, jobs, a free hosted
website, three ways to take money. Comms answers a customer's **text at 3am**
and books the job into Core through a scoped, revocable partner key. The Core
half of that seam is deployed; the Comms half that consumes it is not built.

**Honest numbers, 2026-08-23:** five real-money transactions in Core's entire
history, **under $17 total**. Zero paying customers. Not launched. Two pilots
committed, not started. ~1165 tests and ~293 migrations are engineering
evidence, **not traction** — an outside audit graded it B overall, A− on
engineering, **C+ on UX for a non-technical owner**.

**Nobody who is not Nicholas has ever used it.** The 12–18 Aug venue week
proved the machine runs seven days; he operated it himself. The
[[projects/APEX_COLD_USER_TEST_AND_TODDLER_BAR_2026-08-06]] fixes have never
been re-tested on a cold user. That is the single biggest open risk.

**What gates launch: legal only.** Terms/privacy, SMS consent, Connect
posture. Tap to Pay does not gate it — the store build ships without the
entitlement and the card button refuses in plain words.

**The defect class to know before reading any Apex code:** *a failure
returning the reassuring answer.* Seven found in one week, each writing a
false reason into a ledger somebody would later read. Now CI-gated.

---

## Apex v3 remediation plan — HISTORY — 2026-08-09

**Plan:** [[projects/APEX_V3_REMEDIATION_PLAN_2026-08-09]] ← execute from this.
**Audit:** [[projects/APEX_V3_VS_V2_AUDIT_2026-08-09]]

**Grades:** Build A · Completeness B- · UX C+ · Launch C. Through-line holds.
**Remediation Weeks 1–4 shipped 2026-08-09** — PR https://github.com/nicholaswittle/apex_v3/pull/4 · `feat/v3-remediation-ux-parity` · ask-apex live.
**Nick left:** W1f device/Vercel twin check · FCM for real push · optional full support-agent.

## 🔴 THE TODDLER BAR — governing UX rule, 2026-08-06

Full note: [[projects/APEX_COLD_USER_TEST_AND_TODDLER_BAR_2026-08-06]].

The first genuine outside user (a landscaper, minimal instructions) worked
through Apex onboarding on 2026-08-06. **Nicholas: "if I wasn't there to get
him through, he would have just uninstalled the app."**

The bar that came out of it, in his words: *"Can a first time user that has no
computer or technology abilities work this app? I should be able to give it to
my 2 year old and be able to do it."* Five operational tests — one named verb
per screen, no jargon, no blank boxes, no dead ends, do-it-for-them and let
them correct. **Applies to v2 and is inherited by v3.**

Shipped the same day against his findings: AI **Suggest** on tagline/about/
service blurbs (`draft-site-copy`), "Review & publish" no longer routed through
the *first* setup screen (it literally was), a "Next: <step>" button on the
checklist, plain-language payment copy, the quote board rebuilt around **"Your
move"** with a named button per card, a services owner home screen, one-round-
trip quoting, customer counter-offers, and a pinned back bar on every screen.

✅ **What landed, and it matters commercially:** he was *super excited* to see
his own website, and liked **online pay** on sight, unprompted. The demo is the
product — the site preview is the hook and online pay is the want. Both
currently sit *behind* the checklist that nearly lost him. Get to the site
preview earlier; sell the outcome ("get paid by card"), never the provider.

⚠️ **Still failing the bar:** the launch wizard is a checklist of nouns, not a
one-question-per-screen conveyor. That is the biggest remaining swing.

🔴 **Migration files from 08-06 are NOT in the repo** — applied via MCP,
recorded in the ledger, `.sql` files never written (classifier blocked the
directory). With the pre-existing ledger drift, **`supabase db push` stays
forbidden** and the repo no longer describes production.

## Apex v3 rebuild — Phase 1 MET, services pack built, 2026-08-07

Full note: [[projects/APEX_V3_BUILD_AND_STATUS_2026-08-05]]. New repo
`C:\development\projects\apex_v3`, new Supabase project **apex-v3-prod**
`fnsonnhumcvxdnyarguv`. v2's project is a different live product — off-limits.

🟢 **Phase 0 CLOSED** (met its written gate 08-05, remediation through 08-06).
All three round-5 BLOCKERs proved by attack tests; H-DML closed; money path
deployed; `pg_cron`/`pg_net` installed; `domain_events` drain done; repo↔ledger
62/62 by md5; 12 CI gates green.

🟢 **Phase 1 MET (2026-08-07)** — all four gate clauses proven live, not just
declared:
- **Money leg** — real Stripe Connect charge in test mode, webhook flipped to
  "Paid online", full + partial refunds verified (action retires / stays active).
- **Assistant clause** — the assistant placed/accepted/ready/completed an order
  through the exact `ci/operations.json` registry; DB stamped it `assistant`,
  zero `system` (no service-role lane). **The peer-actor architecture is real.**
- **Photo → published schedule** — MET, three real defects found live.
- **parse-menu scored** — 0 wrong prices on 35/37 items, scoring the deployed fn.

**Services pack built (08-07):** five tables (`service_offerings`,
`service_requests`, `service_request_items`, `request_quotes`, `request_ai_runs`),
**eleven operations** (28 assertions red-then-green), and the redesigned quote
screen. Price-fixedness is a property of the SERVICE, not the business. Booking
is last, enforced in the database. Super-admin fleet console (on/off/inherit).
**467 tests, analyzer clean, every CI gate green.**

**Strategic question settled (08-07):** v2 is the authority on WHAT to build and
what it costs to get wrong; v3 is the authority on how it's wired. Proven on one
table — v2's `mode` column would have forced every services business to be
quote-only or fixed-price, while v2's A2P 10DLC consent and town-at-intake could
never have been invented from scratch. See [[projects/APEX_V3_SESSION_2026-08-07]].

⚠️ **Still open / largest structural weakness:** the `service_role` exemption is
load-bearing across five guards. Shared Stripe platform with v2 still behavioral
isolation. **Blocked on Nick:** Sentry connector, function secrets re-issue,
`APEX_V3_SERVICE_ROLE_KEY`, v2 nightly drift alarm (`V2_READONLY_DB_URL`).
**Phase 2 (Jigsy's employee loop) is next** — the retention test.

**Grade (08-07): A- to A as a Phase-1 foundation.** The assistant clause being
met is the milestone that makes v3 genuinely different from v2 — a peer-actor
system, not a rebuild. Authoritative plan: `apex_v3/docs/MASTER_PLAN.md`
(canonical 08-06; its top Status table is stale — repo is ahead). Open items:
private vault `Notes/projects/APEX_V3_OPEN_ITEMS_2026-08-06.md`.

## Last Updated

2026-08-09. v3-vs-v2 product/UX audit filed — Build A / UX C+ / Launch C. See audit section above. Prior: 08-07 Phase 1 MET, services pack, foundation A-.

## Services vertical + build 10 — 2026-08-03 (evening)

`feat/services-vertical` at `38d7387`, pushed. 165 tests green, analyze clean.
**Build 10 is LIVE on TestFlight** (2026-08-03 evening): build 9 (`0ec3460` is an
ancestor) plus the whole services vertical in one build. First build carrying
the `apex://` scheme and the signup business-type picker — neither has been
exercised on a real device yet. `pubspec.yaml` now carries `version: 0.1.0+10`
— earlier builds passed `--build-number` by hand, which is why it was absent.

Second business type alongside restaurants, chosen at signup: jobs board with
open-slot sign-up, quote requests, customer-facing quote pages
(`/[token]/quote/[id]`), needs-scheduling queue, pipeline console with revenue
tiles, Get Found (GBP) checklist. Restaurant side unchanged, pinned by
`test/vertical_pin_test.dart`.

🟢 **The services site is LIVE: https://wisense-apex.vercel.app** (2026-08-03,
`4ba0851`). /hicpa and /bradley-sons-landscaping serve publicly; deployment
protection is previews-only. App links point at it from build 11 (+11 pushed).

🔴 **Enrichment re-applied the California identity AGAIN tonight** — it went
public for a few minutes before being caught and re-scrubbed. The launch
wizard applies "Find your business" results over an existing profile with no
confirm step. Until that gets one, do NOT run Find your business on Bradley's.
Durable fix logged. Was: **the services public site is NOT deployed.**

## Fable 5 deep audit — 2026-08-03

Full write-up: [[projects/APEX_AUDIT_FABLE5_2026-08-03]]. Overall **C+** — code
craft is A-range, restaurant half is a working B+, but the services vertical is
invisible to its customers and payments have one test file. "A product is graded
on what a customer can touch."

Its structural claim, worth arguing with rather than filing: **services is the
better market than restaurants.** Free incumbents (7shifts, Homebase, Square)
cap restaurant pricing at ~$0–40, while services tolerates $99 flat and has a
villain to position against (per-seat pricing, Angi lead resale). It recommends
customers #2 and #3 be a landscaper and an HVAC/plumbing shop — inverting the
master plan's "do not build vertical two until vertical one is paying."

Also found: the services 1.5% is **contractor-paid** (deducted from the deposit)
while the restaurant 1.5% is **guest-paid** — same number, opposite payer, and
an accident of which function was written first.

## ⚠️ Two live branches, diverged — 2026-08-04

`feat/services-vertical` @ `370cb08` (mine) and `feat/template-to-product` @
`f313b6d` (ChatGPT) both branched from `7c769e3`. Full audit:
[[projects/APEX_AUDIT_TEMPLATE_TO_PRODUCT_2026-08-04]].

The GPT branch is good work — anon SECURITY DEFINER allowlist that fails the
migration if an unexpected grant survives, client table-writes replaced by
guarded RPCs, photo uploads moved to signed intents, Next 15→16. Analyze
clean, 180 tests, site builds.

🔴 **None of its 12 migrations are applied and its 4 edge functions are not
deployed.** It must ship atomically: migrations without the functions revokes
`submit_public_request` from anon while `submit-public-request` does not exist
— the public quote form stops working.

🔴 **It reverts five fixes from `370cb08`**, including the deposit cap
(`33` → `100/3`; $825 vs $833.33 on a $2,500 quote). Merge `370cb08` in FIRST,
keeping both sides in the two overlapping files (their `has_role(manager)`
check AND my cap constant).

## Services build map — 2026-08-03

[[projects/APEX_SERVICES_COMPETITIVE_BUILD_MAP]]. Ordered by "can we compete out
the gate". Thesis: you cannot out-feature Jobber and do not have to — the wedge
is flat pricing with unlimited crew, which their per-seat model structurally
cannot answer.

🟢 **Phase 1 BUILT 2026-08-03** (`f5ea759`, `edff6ad`): job_series + daily materializer cron (probe-verified: skips stick, ending keeps history), collect-the-balance lane + Job done flow, notify-request-scheduled (email; SMS post-A2P), repeat-client history by phone. Also fixed: claim guard blocked claiming next week's visit of the same job. Was: **recurring jobs did not exist anywhere in the codebase** (verified: no
`recurring`/`rrule`/`repeat_every` in lib/ or migrations). The vault filed it
under "deliberately not built", which was right when services was a side bet and
is wrong now. Weekly mowing, fortnightly cleaning and quarterly pest control ARE
the services revenue base — a landscaper who must hand-enter 40 visits leaves in
week two. It is a week of work, not an evening, and it is the item most likely
to be underestimated.

Phase 0 (deploy site/, build 11, one real request) → Phase 1 floor (recurring,
balance collection, customer notification, client history) → Phase 2 wedge
(quote-page referral loop, flat-pricing story, photo-to-quote, HICPA page) →
Phase 3 Front Desk once A2P clears. Grade trajectory C+ → B− → B+ → A−.

## Traps hit 2026-08-03 — read before repeating

- **`supabase config push` pushes whole sections, fills CLI defaults for every
  key absent from config.toml, and does NOT prompt.** An `[auth]` block holding
  two keys silently disabled MFA and email confirmations, cut OTP 8→6 and the
  send rate limit 1m→1s, and dropped the wildcard redirect entries. All
  restored; `[auth]` now states every value explicitly. `[storage.vector]`
  pinned off — it defaults on, needs a paid tier, and 402s *after* auth has
  already applied, so the error reads like total failure when it is not.
- **Auth Site URL was `apex-scheduler.vercel.app`**, a project name predating
  v2. That was the blank page after tapping Confirm in a signup email. Now the
  v2 app, with `apex://login-callback` registered — `ios/Runner/Info.plist` had
  no `CFBundleURLSchemes` entry at all.
- **`on_auth_user_created` calls `apex_handle_new_user`, not
  `handle_new_user`.** The latter is dead code. Editing it shipped a
  business-type picker that passed analyze, tests and on-screen checks and
  wrote nothing to the database.
- **`enrich-business` writes a real third party's identity wholesale.** A
  California company's phone, address, website and 4.9-star claim were on
  Bradley & Sons; Duke's Riverside Bar & Grille is still on the "test rest"
  org. Check `venue_site_profile.contact` before publishing anything.
- **Two notions of "member" disagree** — `profiles.organization_id` vs
  `organization_members`. Produced two separate wrong behaviours in one day.
- **Committed-but-unpushed cost three round trips** with the Mac agent in one
  night. Verify `git ls-remote`, not local log.

## Security — DONE 2026-08-02

🟢 **anon EXECUTE revoked on 56 SECURITY DEFINER functions.** Only 8 guest-ordering functions remain anon-callable. Repo is public, so function names and signatures were published — this was the cheapest risk reduction available.

🟢 **apex_set_role privilege escalation closed.** `has_role(org,'owner')` returns true for any manager; was used as an owner gate. Fixed: `is_owner(org)` for owner-only, `can_administer(org)` for owner-or-admin.

🟢 **Trigger functions locked down.** `apex_protect_*`, `handle_new_user`, etc. revoked from public/anon/authenticated. Triggers unaffected (run as table owner).

🟢 **Identity moved to user_id.** `shifts.user_id` backfilled (27 live rows), sync triggers keep it in step. Sidework RLS policy still has a name fallback that fails open on ambiguity — see [[NOW]] known issues.

## Bugs fixed 2026-08-02

- **`.order()` was descending everywhere** — postgrest-dart defaults to `ascending: false`. All 26 unqualified calls sorted backwards. Next shift, menu order, every list.
- **Worked hours came from the schedule, not punches.** Emily's 13-hour scheduled shift clocked at 2.84 hours reported 13 hours worked. "Est. pay" overstated 4.6x. Fixed: `hoursWorkedFromEntries`, 6 tests pin the real row.
- **tip_allocations realtime subscription never existed.** Filtered on `organization_id` but that table has no such column. Realtime rejected the binding silently. Now scoped by `user_id`.
- **Admin excluded by hand-rolled role checks.** Six screens tested `role == 'owner' || role == 'manager'` — Admin is neither. Nav offered the tile, screen refused entry. All six now go through `StaffRole`.
- **Share sheet white-on-white.** Same class as the clock button. Fixed with explicit ink constant.
- **Guest site scrolled sideways.** `1fr` grid with `nowrap` labels pushed past viewport on mobile. Fixed: `minmax(0, 1fr)` with wrapping labels.
- **Monitor's Twilio alert was silent.** `notify()` checked secrets were set but not whether the send succeeded. Twilio pending approval = every send rejected, looked identical to delivered. Now fails loudly.

## Outside monitor — LIVE

GitHub Actions cron, headless browser (Playwright). Checks: venue page 200, menu opacity > 0, item count > 0, order page renders cart, next-shift date is nearest future shift, `apex_venue_health` not paused during opening hours. Five secrets set.

**Two corrections verified 2026-08-02:** GitHub throttles `*/15` schedules — it fired once in 2h15m, not every 15 min. And SMS cannot deliver: Twilio account pending approval. Until it clears, the real alarm is the GitHub Actions failure email. Detection latency is hours, not minutes.

## Tip pool — LIVE 2026-08-03

Migrations `20260831000000`/`0001` applied **after** build 9 was confirmed installed, per the ordering rule. Hardened: all twelve tip functions were anon-callable (Supabase default privileges grant EXECUTE to anon; the migration revoked PUBLIC on only four), closed by revoking from both anon AND PUBLIC and re-granting to authenticated (`20260831000002`/`0003`). First split reconciled by hand: $100.00, Emily 2.84 h, zero unallocated cents, `created_by` Robin.

Live compliance: tip pools include managers/owners, which PA law bars when a tip credit is taken. Seeded eligible: Emily, Avi, Courtney, Dana, Kim, Marsha, Morgan. Not eligible: Robin (owner, never on floor).

## Payroll Lite — decided, not shipped

**Hours report, not a pay calculator.** The tipped overtime rate was wrong (1.5x cash wage instead of 1.5x min wage minus tip credit). A 45-hour tipped week returned $22.50 where ~$33.13 was owed. Decision: remove the money columns rather than fix the rate. See [[DECISIONS]] 2026-08-02 and `apex_v2/docs/PAYROLL_LITE_SCOPE_2026-08-02.md`.

`feat/payroll-export` is stale — predates the 08-02 work, would delete the monitor, five migrations and six test files. Needs rebase before merge.

## Business context

🎯 **Target: $300/mo from three paying OS venues.** Not thirty. Self-serve is the product, not an acquisition necessity. Jigsy's network is the year-one plan. Retention is the hard part — one churn is a third of the business.

💵 **Fee model: direct charges + 1.5% guest-paid service fee.** On $32.96: guest pays $35.46, venue nets $33.61, WiSense earns 52¢. Break-even on Stripe Connect is ~$950/mo online volume; Square has no per-account Connect fees (better for small venues). See [[projects/APEX_PLATFORM_FEE_ECONOMICS_2026-07-30]].

💰 **Jigsy's numbers:** net sales $291,668, orders -14%, customers -12%, average sale +11%. Traffic is the problem. Labor % of net sales = 0.0% — they track no labor in Square. Apex has scheduling + timeclock + labor cost. That gap is the recurring revenue pitch. See [[projects/JIGSYS_BUSINESS_NUMBERS_AND_REVENUE_MODEL_2026-08-01]].

## AI policy

**Haiku for everything, Opus only for image OCR.** Two calls removed that only reprosed existing data. `polish-labor-warnings` should be cut to templates (warnings are already deterministic in `labor_guardrails.dart`). `venue-briefing` at 134 calls/30d is the largest AI line item — template 90% of it. See `apex_v2/docs/AUDIT_2026-08-02_FABLE.md`.

## Standing warnings

🔴 **Migration ledger was repaired 2026-08-03** — matches the catalog through `20260901000000`. Still do not run `supabase db push` blindly; the earlier drift was fixed, but confirm the ledger vs catalog before any bulk replay.

⚠️ After any deploy, **hard-refresh** — the Flutter service worker serves the old bundle.

⚠️ `apex/apex/supabase/config.toml` still points at Horizon's project.

🔴 **Square Sandbox cannot test the value proposition.** No POS, no Restaurants, no application-fee reporting. Sandbox de-risks the code, not the product.

❌ **Tap to Pay is struck** — not available on iPads. Jigsy's wired contactless reader is card-present hardware inside Square POS, a different thing.

## Active threads

- Build 9 shipped + verified (tip-pool routing, hours fix, Admin gates, monitor fixes)
- Phase A testing: real clock-in, real pay-now order, real pay period (see [[NOW]])
- Tip-pool migrations applied + hardened in same window as build 9
- Parked: payroll export (stale branch, needs rebase), services vertical (gated on vertical one paying)

[[NOW]] · [[index]] · [[projects/Apex v2 — Restaurant OS Build]]

## Apex v3 — as of 2026-08-06
- Phase 1 roughly half built. Onboarding, staff orders, guest ordering, schedule authoring, operation contract all in. Dashboard, admin, capacity, entitlements, refunds not started.
- **No public site exists for a services business.** Every anon RLS policy is restaurant-pack; there is none on organizations or venues. Largest open gap.
- Restaurant onboarding is deliberately BLOCKED before the vertical write — that column allows exactly one write and gates which modules an org can ever unlock.
- Next: three cold-user tests, then the adversarial review of what a confident-but-wrong assistant can do with the owner's rights.
