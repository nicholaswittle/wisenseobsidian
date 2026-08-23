---
title: Decisions Register
tags: [decisions, adr, rationale, governance, history]
aliases: [Decisions, ADR, Decision Log]
date: 2026-07-20
---

# ⚖️ Decisions Register

> Append-only record of **settled decisions** — the *why* behind the current state. Any agent (Hermes, Claude, Cursor, Codex, Gemini) checks here before re-opening a question or "helpfully" undoing something deliberate. Newest at top. Never edit a past entry; supersede it with a new one and mark the old `SUPERSEDED`.

Entry format: **date · decision · status · rationale · consequences**. Status ∈ ACTIVE / SUPERSEDED / REVERSED.

## 2026-08-23 · Apex v3 becomes **Apex Core**; the icon says **Apex**; identifiers do not move — `ACTIVE`
- **Decision**: The product formerly called Apex v3 is **Apex Core** in writing, and the app's home-screen name is simply **Apex**. The second app is **Apex Comms**. "Core" never appears in front of a customer. The bundle id `com.wisense.apexV3`, the Dart package `apex_v3`, the repo folder, and the Supabase and Vercel project names are **deliberately unchanged**.
- **Rationale**: Renaming identifiers costs real things and buys nothing anyone can see. The bundle id is bound to the App Store record, the TestFlight builds and a Tap to Pay entitlement request in flight with Apple — changing it creates a *different app*: new store record, testers lost, installed builds orphaned. The Dart package rename alone is ~170 files of churn. The Apex Comms builder reached the same conclusion independently.
- **Consequences**:
  - The home-screen label was `Apex V3` on iOS and literally `apex_v3` on Android and the web build — a package name, shown to customers, on the filming phone and headed for both pilots. All three now read **Apex**.
  - A test fails if the display name ever contains a digit or an underscore. It was checked to reject `Apex V3`, `apex_v3` and `Apex 2`, and accept `Apex`.
  - Expect a permanent, harmless mismatch between the product name and every identifier. That is intended; do not "fix" it.

## 2026-08-23 · Apex Comms acts inside Core as a revocable **service user**, never as the owner — `ACTIVE`
- **Decision**: When a venue links Apex Comms, Core creates a real auth user for that org — a visible member named "Apex Comms" at **Manager**, never owner — and Comms acts as that user through a hashed, scoped, revocable partner key. Comms may call only already-registered Core operations (`create_request`, `send_quote`, `accept_quote`, `propose_day`, `book_request`). Availability lives in Comms; what is booked lives in Core; both read each other.
- **Rationale**: Comms is a webhook answering a text at 3am with no human in the request, and Core's guards correctly refuse an unauthenticated caller. The alternative — acting as the owner — would put a machine's 3am writes under a person's name. A distinct actor is one a venue can point at and revoke, and every audit row names it. Manager rather than owner means a partner can never change the plan or remove people. **No new Core operations were needed**, which is the strongest evidence the operations layer was built right.
- **Consequences**:
  - Booking is **four calls, not one** — Core refuses to book without a price *and* a day agreed. That is the discipline working, not overhead to route around.
  - The org is resolved *from the key*, never the request body; a key that can book cannot refund; the plaintext is shown once.
  - Core side deployed 2026-08-23. **Not built**: the Comms side that consumes the key, the live link against a real venue, and the venue-facing Disconnect screen.
  - See [[projects/APEX_CORE_AND_COMMS_2026-08-23]] and `apex_v3/docs/COMMS_CORE_SEAM.md`.

## 2026-08-21 · A venue sets its own deposit cap, and the reason is recorded — `ACTIVE`
- **Decision**: Each business sets the largest deposit it may ask for and the job size it applies above, **with a required written reason**, owner-only. Apex ships Pennsylvania's HICPA figure (a third, above $1,000) as the default and **states on screen that it cannot look up what any state allows**.
- **Rationale**: A bar-certified attorney (a friend of Nicholas) advised on 2026-08-20 that leaving the percentage to the venue's own choice is fine and leaves the liability with the venue. Nicholas reaffirmed this on 2026-08-21 when it was questioned a second time. **Treat it as settled and do not re-open it.** The prior state was worse in a way nobody had noticed: the cap was read in four places and writable in none, so every business ran on Pennsylvania's number whether or not Pennsylvania governed it — including businesses in other states and outside home improvement entirely.
- **Consequences**:
  - The required note is the point, not paperwork around it: a cap the business cannot be *shown* to have chosen is a cap Apex appears to have picked, and that distinction is the whole basis of the liability sitting with the venue. The change emits an event carrying the previous values.
  - A test fails if any refusal on that screen contains "legal", "law", "statute" or "hicpa" — software that names a statute is software giving advice.
  - **Not covered by that conversation**: terms and privacy for a product holding other businesses' customer data, the SMS consent language behind a live public opt-in page, and the Stripe Connect platform posture behind the 1.5% fee. Those remain open.

## 2026-08-21 · Tap to Pay requires **iOS 18**, above its own 16.4 floor — `ACTIVE`
- **Decision**: The "Take a card" act refuses below iOS 18, in plain words naming cash and the payment link as alternatives. The act stays **visible and enabled** on every iPhone — Apple's checklist forbids hiding it on iOS.
- **Rationale**: Apple requires a "How to Tap" education overlay before review, and the only way to present Apple's own is `ProximityReaderDiscovery`, which is iOS 18+. Below that the call returned *success* and displayed nothing — a merchant handed a live card reader having been taught nothing, with the setup screen carrying on as though they had been. That is the house defect class (a failure returning the reassuring answer) wearing an Apple lanyard. Stripe's guidance is to supply a fallback UI; improvising Apple's payment education is not a thing to improvise.
- **Consequences**: costs merchants on iOS 16–17 until a fallback education screen exists, at which point this drops back to 16.4. Tap to Pay does **not** gate launch either way — the store build ships without the entitlement.


## 2026-08-06 · The Toddler Bar is Apex's UX acceptance standard — `ACTIVE`
- **Decision**: Every Apex screen is held to one bar before it ships: **a first-time user with no computer or technology ability must be able to work it unaided.** Nicholas's phrasing, after watching the first outside user nearly uninstall the app: *"I should be able to give it to my 2 year old and be able to do it."* Operationally, five tests: (1) one obvious next action, named as a verb, always visible — never a bare status to interpret; (2) no jargon — no "Stripe", "quoted", "publish", "sync" without a plain translation in the same breath; (3) no blank boxes — every empty field gets a suggestion, example, or prefill; (4) backing out lands on a signpost — nothing dead-ends or silently resets; (5) if the app can do it for them, it does, then lets them correct it — *suggest, never commit*. Applies to v2 now and is inherited by v3.
- **Rationale**: The 2026-08-06 cold-user test was the first time a genuine stranger touched the onboarding flow, and it only completed because Nicholas stood next to him and translated. The target customer is a tradesperson, not a software user. The standing rule from the test handoff — *every question the user asks is a screen that failed* — produced six failures in one sitting, and four of them were pure interface: blank fields with no starting point, a "publish" step that routed through the literal first screen of setup, a payment provider named instead of described, and a wizard that stranded him after every step. None were bugs. All were the app assuming competence it must not assume.
- **Consequences**:
  - Shipped the same day: AI **Suggest** on tagline/about/service blurbs (`draft-site-copy` edge function), "Review & publish" routed to the review form instead of the business search, a "Next: <first unfinished step>" button on the launch checklist, plain-language payment copy, the quote board rebuilt around **"Your move"** with a named button per card, plain-English status chips, a services owner home screen, and a pinned back bar on every screen. See [[projects/APEX_COLD_USER_TEST_AND_TODDLER_BAR_2026-08-06]].
  - **The `vertical_pin_test` restaurant-wording pin was deliberately updated**, not worked around. Pins exist to catch accidental drift; the cold-user test is the evidence that justifies a deliberate change.
  - **Still failing the bar and known**: the launch wizard remains a checklist of seven nouns rather than a one-question-per-screen conveyor; payments onboarding hands off to Stripe's hosted forms; "86 Board", "QR Wall" and "Sidework" are insider names; Get Found and the share card have never been held to the bar.
  - Future UX disagreements resolve against this bar, not against taste. A screen that is defensible to an operator but opaque to a first-timer fails.

## 2026-08-05 · Apex v3 is rebuilt on a new Supabase project, fixed phase-by-phase, reviewed by parallel scoped agents — `ACTIVE`
- **Decision**: Apex v3 is a ground-up rebuild in `C:\development\projects\apex_v3` against a **new** Supabase project (`apex-v3-prod`, `fnsonnhumcvxdnyarguv`), pulling from v2 only where it saves time without carrying defects. Each phase is fixed to completion **before** the next begins ("fix each phase as we build it to prevent drift"), and each phase closes with an adversarial review run as **four parallel scoped agents** — money/payroll, guest/anon + tenancy, CI/test honesty, and a seams pass — rather than one serial generalist. Sequencing set 2026-08-05: database tier → edge-function tier → UI.
- **Rationale**: Five review rounds on Phase 0 established that the bottleneck was never care, it was *method*. Four consecutive rounds each introduced the next round's defect because fixes were reasoned about rather than proven, and every CI gate tested a rule's **text** instead of its **reachability** — the money-guard gate stayed green with the trigger dropped, and the RLS suite's failure detector counted NULL as a pass. Serial generalist reviews also cost ~11 min each plus 20–35 min fix cycles; the parallel format delivers roughly 4× the coverage for the same tokens, and scoped briefs go deeper. Cold context is retained deliberately — a reviewer handed a warm summary inherits the author's blind spots, and the worst finding of round four (staff minting a full day of paid hours) surfaced only because the reviewer re-derived the function from scratch.
- **Consequences**:
  - **Every fix ships with an assertion watched FAIL before the change and PASS after.** No exceptions.
  - Six standing rules carry into every later phase: decide each new table's write path explicitly (RPC-only, or direct DML with pinned columns + audit trigger); use `revoke all` then re-grant, never enumerate privileges; test the **victim** of a control, not the attacker; for every operation locked down, prove a legitimate actor can still complete it; ask what a user with **entirely legitimate credentials** can abuse and whether anyone could tell afterward; audit the **catalog**, not the repo.
  - Rule 4 exists because closing the tip-table write policies left a manager with **no route back** from a wrong split. Rule 3 exists because a rate-limiter test asserted the attacker was refused and never that a real customer still got through — which is the actual property, and its absence hid a QR-code DoS.
  - Phase 0 is **not closed** as of 2026-08-05. Do not port UI.
  - **UI is gated on Nicholas's review** — design system + screen flows before any screen is ported. v2's layout was not user-friendly; cohesion is the explicit goal.
  - Open product decision, not yet made: public menus also expose every ordering-enabled venue's address, phone, coordinates, tenant id and full pricing **with no token** — effectively the platform's customer list.
  - Full record: [[projects/APEX_V3_BUILD_AND_STATUS_2026-08-05]].

## 2026-08-02 · Multi-vertical Apex is one app, not a forked clone — `ACTIVE`
- **Decision**: If Apex ever serves a second vertical (landscaping / field services), it does so as **one codebase where the vertical is data-driven configuration**, selected at *organization creation* — not at login, because a user may belong to both a restaurant org and a services org. An `apex_services` fork of the repo is **rejected**. `organizations.vertical` added 2026-08-02 (`20260901000000`) as the canonical field; `venue_site_profile.vertical` (which already existed since `20260809000000:20`) mirrors it for the renderer.
- **Rationale**: A fork means every security fix is applied twice, by hand, forever, by one person on shift work — and the last week alone carried 56 anon `EXECUTE` revokes, the identity migration, two tier-gate closures and two AI meters. This workspace has already lived the smaller version of that failure: the vendored `wisense_core` / `wisense_ui` divergence needed a dedicated reconciliation project, and Apex's `wisense_ui` fork is *still* an open decision (see 2026-07-21 below). The genuinely shared ~45% is precisely the hardened part — auth, orgs, RLS, membership, payments, the module registry — so forking means forking the code that most needs to be byte-identical. A clone would be defensible if services were a different product for a different buyer; it is not, it is the same buyer profile: a small local business with hourly staff that needs a website and a schedule.
- **Consequences**:
  - `organizations.disabled_modules` **already wins over everything** (`20260730210000:41–44`), so a services org is expressible today with **zero further schema change**. `vertical` drives defaults, onboarding and user-facing nouns — not gating. The registry key may not need extending at all.
  - **Reuse is ~45%, not the 90% three external audits claimed.** Two mapping rows break outright: the "geofenced clock-in" does not exist (clock-in is a daily-rotating wall QR, `lib/core/clock_qr.dart`; there is no geolocation code in `lib/` at all), and the quote lifecycle cannot ride `online_orders`, which is sub-hour by construction.
  - Honest sizing: landscaping **site** 15–30 hrs; services vertical **MVP** 150–250 hrs (4–7 months of evenings). Earlier "3–4 weeks" estimates were ~5x optimistic.
  - **Gate**: the Master Plan's own rule stands — *"do not build vertical two until vertical one is paying"* (`docs/WiSense Restaurant OS Master Plan 2026-07-27.md:156`). Vertical one has one customer against a target of three.
  - Deferred and named so they do not creep in: quote state machine, geofenced clock-in (a new feature, not an adaptation), a job entity for crews, client records, recurring jobs, estimate→invoice, materials. **HVAC is far further away than landscaping** — its core is dispatch, parts inventory and service agreements, ≈0% built. Do not treat "field services" as one market.
  - Plans: [[projects/APEX_MULTI_VERTICAL_PLAN_2026-08-02]] and [[projects/APEX_DECOUPLE_PILOT_2026-08-02]].

## 2026-08-02 · Guest review follow-up ships symmetric, never gated — `ACTIVE`
- **Decision**: Apex may send a post-pickup SMS offering **both** a Google review link and a private note to the owner, presented identically to **every** guest on **every** completed order. Apex must never learn or infer sentiment before deciding what to show, and must never select who receives the message. The proposed "5-Star Review Magnet / Feedback Shield" — sentiment-routed, negative feedback diverted private — is **rejected**. The names are rejected with it; the ship name is **Guest Follow-Up**.
- **Rationale**: Sentiment routing is review gating, which produces a public review corpus that misrepresents guest experience. Two exposures, and the smaller one is the real risk: the FTC route is Section 5 deception (reference case Fashion Nova, 2022, $4.2M) and is a low-probability tail risk at our size, while **Google's policy prohibits gating outright and enforces by removing reviews or flagging the profile — no case, no notice, no appeal**. The realistic loss is a venue's review history disappearing on our advice, at our only customer. Symmetric solicitation is not a workaround for the rule, it *is* the rule. Same doctrine as the Payroll Lite decision below: where the honest artifact and the safe artifact are the same artifact, build that one. Little is given up — asking everyone at a venue guests already like skews positive on its own, and 200 reviews at 4.6 outsells 40 at 5.0.
- **Consequences**:
  - Four boundaries, three of them easy to cross by accident: **send to 100% of completed orders automatically** (manager picking recipients, or skipping comped/complained tickets, rebuilds the gate one step upstream); **no sentiment question first**; **equal visual weight** on both options; **no incentives** (separately barred by Google regardless of sentiment).
  - **"5-Star Review Magnet" and "Feedback Shield" must not appear in code, UI, marketing or commit messages.** Marketing copy is read as evidence of intent — a feature whose name says it shields the venue from bad reviews has documented its own purpose.
  - **Blocked on paperwork, not code.** This is *marketing* SMS; the existing `notify-order-event` guest rail is *transactional* and its consent basis does not extend. Requires express written consent captured at checkout (own unchecked box) plus A2P 10DLC registration for the marketing use case. Start the registration early — it queues.
  - Not on the 14-day launch path. Build after Jigsy's is live and stable.
  - Feature has **no prior provenance**: it entered the record inside an external audit on 2026-08-02, labelled "Compliance-Clean" with no analysis. Zero hits in `lib/`, `supabase/`, or the vault. The archived `ReviewGuard` concept is unrelated — it drafts *responses to existing reviews*, never solicits, never branches.
  - Full design, spec and boundaries: [[projects/APEX_GUEST_FOLLOW_UP_PLAN_2026-08-02]].

## 2026-07-31 · New Horizon canonical package dependencies confirmed — `ACTIVE`
- **Decision**: Close the New Horizon package-fork question. `wisense_new_horizon` now depends on `../../packages/wisense_core` and `../../packages/wisense_ui`; its vendored package copies are deleted.
- **Rationale**: The canonical dependency build is clean, so the old warning that switching paths would break New Horizon is false.
- **Consequences**: This supersedes the stale interpretation of the 2026-07-20 fork-reconciliation entry. The separate Apex `wisense_ui` fork remains an open, unrelated decision.

## 2026-07-31 · Apex will replace Square as the venue POS — staged — `ACTIVE`
- **Decision**: The Square payment rail is a **wedge, not the destination**. Long-term Apex replaces Square at Jigsy's. Sequence: (1) ship the Square rail on their existing hardware, (2) own the kitchen ticket via a CloudPRNT printer, (3) own front-of-house with a native app + Stripe Terminal + offline queue, (4) the long tail — timeclock, close-out, tax reports.
- **Rationale**: Getting in the door requires bending to hardware they already own; replacing it requires earning the right. Each stage is independently valuable and sellable, so stalling at stage 2 still leaves a real product.
- **Consequences**: Three walls, all hardware/platform rather than software. Their **Star TSP143IIU is USB** — a browser cannot address it, so replacing Square's printing means **replacing the printer** (~$250–350, the only unavoidable hardware cost). Card-present needs a reader we own (~$249–349) and Terminal's SDK does not run in a browser. Offline resilience alone forces a native app. **The recurring decision is when to go native** — everything past stage 2 hits that same wall. Apple Developer account purchased 2026-07-31. See [[projects/APEX_PAYMENTS_AND_POS_STRATEGY_2026-07-31]].

## 2026-07-31 · Tap to Pay (SDK) struck from the backlog — `ACTIVE`
- **Decision**: Remove Tap to Pay on iPhone/Android from the Apex roadmap entirely. Not deprioritised — removed.
- **Rationale**: *"Tap to Pay isn't available on iPads."* Android excludes tablets, and under Square for Restaurants it is restricted to two Samsung phone models. Jigsy's runs iPads in a Square Stand.
- **Consequences**: Jigsy's wired contactless reader is **card-present hardware inside Square POS** — a different thing, already working, not ours to touch. Do not conflate the two; doing so mis-sizes the roadmap by an order of magnitude (the SDK feature needs a native app + an Apple entitlement + App Review).

## 2026-07-31 · Jigsy's named publicly as first client — `ACTIVE`
- **Decision**: With their permission, name Jigsy's Brewpub & Restaurant on wisensellc.com as WiSense's first client, with a real before/after case study.
- **Rationale**: The prior disclaimer said this was spec work "not a commissioned project" — now false, and it undersold the work.
- **Consequences**: Deliberately **not** claimed: that the site is live (it has not replaced jigsypizza.com), that Apex is in production (Emily is still testing on a blank slate — Apex reads "First Venue Onboarding"), or anything about online ordering. Those claims get checked. See [[customers/Jigsys Brewpub]].

## 2026-07-31 · Guest-upload photography is not ours to republish — `ACTIVE`
- **Decision**: Photos sourced from public Google guest uploads must not appear on WiSense marketing property. The Jigsy's hero screenshot was **deleted from the repo**, not merely unreferenced.
- **Rationale**: Guest-uploaded photos belong to the guests who took them. Acceptable on a demo behind a disclaimer; not acceptable republished under our brand on a commercial site selling services.
- **Consequences**: **For a static host, "unpublish" means delete the file** — anything under Next's `public/` is served whether a page links it or not, so an orphaned asset stays fetchable at its URL. Verified 404 in production. Blocker #1 in `jigsys_site/LAUNCH_CHECKLIST.md` is replacing the site's photography with owner originals before launch; the hero screenshot gets re-captured then.

---

## 2026-07-26 · Restaurant SaaS Lead Offer = $0 Setup + $99/mo (Starter Tier) — `ACTIVE`
- **Decision**: WiSense adopts a 2-tiered commercial offer for the restaurant website + online ordering platform: Tier 1 (Lead Pitch) = **$0 Setup Fee + $99/month**, Tier 2 (Pro Value) = **$299 Setup Fee + $79/month**.
- **Rationale**: Market research across 9 competitors revealed that setup fees ($299+) create massive upfront sales friction for cash-strapped local restaurants. $99/mo eliminates setup risk while keeping price below the $100/mo psychological threshold.
- **Unit Economics**: A single client at $99/mo pays 100% of WiSense AI agent tool costs and infrastructure ($100/mo total overhead). Every subsequent client yields a 95%+ net profit margin because AI agent templates allow deployment in <30 minutes.
- **Consequences**: Outreach to the 408 local Enola prospects will lead with the $0 Setup / $99/mo offer. See [[Restaurant Website SaaS — Master Pitch Model and Strategy 2026-07-26]].

## 2026-07-23 · Jigsy's = free core + 99¢ per accepted online order; pay at pickup; no Stripe — `ACTIVE`
- **Decision**: The Jigsy-specific offer is a free core website/order demo plus a **$0.99 Online ordering fee on each accepted order**. Jigsy's collects the food, tax, and fee in person at pickup. WiSense does not process customer payments through Stripe or hold customer/restaurant funds.
- **Rationale**: Nicholas is not comfortable asking Jigsy's for the broader `$299 setup + $79/month` price. The 99-cent model keeps the relationship simple, charges only when the system produces an accepted order, and preserves Jigsy's existing cash/card-at-counter workflow.
- **Settlement**: The system records accepted orders and produces a monthly statement: `accepted orders × $0.99`. Jigsy's pays WiSense separately by check, cash, or bank transfer. Rejected/abandoned requests do not count.
- **Product consequences**: Staff workflow is intentionally **Accept & Print** only, plus pause/reopen ordering, prep time, sold-out controls, and ticket reprint. The ticket must disclose the fee and say payment is collected at the counter.
- **Boundary**: This decision is Jigsy-specific. The general WiSense `$299 + $79/month` website offer may remain for other clients. The current demo is browser-local and is not a live cross-device ordering system. See [[Jigsys Ordering Demo — Build Record 2026-07-23]].

## 2026-07-21 · AI in Apex = parsing, not scheduling optimization — `ACTIVE`
- **Decision**: LLMs in Apex are for turning **unstructured input into structure** (photo of a paper schedule → rows; later, free-text availability). They are **not** for building or optimizing schedules. Assignment logic stays deterministic in `SuggestionEngine` / `staff_ranker.dart`.
- **Rationale**: scheduling is a constraint problem — LLMs are non-deterministic and weak at it, while rules are testable and free. Extraction from messy real-world input is the opposite: exactly what vision models are best at and what no amount of Dart solves. Cost is **not** the constraint — ~1 schedule photo/week on Opus 4.8 (`$5/$25` per MTok, high-res vision at 2576px) is roughly **$0.08/photo ≈ $4/year**. The real constraints are privacy and engineering time.
- **Consequences**: `44adeac` shipped the deterministic half (staff ranking) first — no dep, no API, no privacy question. Feature A (photo import) remains the only planned LLM surface. **Privacy middle path preferred**: run on-device ML Kit OCR, then send only the *recognized text* — never the photo — to the LLM for structuring. Cheaper, and no image of a staff board leaves the device, which matters given WiSense's on-device/zero-retention positioning on [[COMMS LINK]].
- **Sequencing (agreed)**: (1) ship what's built — RLS → phone QA → Gate C; (2) staff ranking *(done, `44adeac`)*; (3) Feature A last. **Kill criterion**: if the OCR+verify flow doesn't cut a week's entry below ~5 minutes, drop the photo path and keep `copyPreviousWeek` + suggestions.
- **Related risk**: [[COMMS LINK]] was parked because an AI dependency made it unshippable. Do not repeat that on Apex before it has a paying user.

## 2026-07-21 · Labor cost excluded from staff ranking — `ACTIVE`
- **Decision**: `staff_ranker.dart` does **not** factor `hourly_rate` into its score, though the field is available and `LaborCostPanel` already displays it.
- **Rationale**: ranking people by how cheap they are is a judgment for the human, not a default the app makes silently. The admin can see cost in the existing panel and weigh it themselves.
- **Consequences**: one-line change to reverse if Nicholas decides margin should drive ranking. Flagged as an open question rather than assumed.

## 2026-07-21 · Apex vendors its own `wisense_ui` — fork NOT fully reconciled — `ACTIVE`
- **Finding** (not yet a decision — needs ratification): Apex's `pubspec.yaml` depends on `packages/wisense_ui` (v1.0.0, **2 files**: `spacing.dart`, `loading_indicator.dart`), not canonical `C:\development\packages\wisense_ui` (v0.1.0, **18 files** incl. `text_styles.dart`, `error_banner.dart`). This is a **second fork**, separate from New Horizon's.
- **Why it matters**: System Architect Directive §2 mandates `WiSenseTextStyles` as the text-scale base for all apps. `WiSenseTextStyles` **does not exist in Apex's dependency**, so §2 is unsatisfiable there and every Apex widget hardcodes `fontSize`. The 2026-07-20 entry below is marked SUPERSEDED because its consequence line ("the Known fork caveat is now historical") is false — it reconciled New Horizon only.
- **Not fixed deliberately**: canonical `WiSenseTextStyles` derives from `WiSenseThemeText` (travel-app theme), semantically wrong for Apex's brewpub palette. Switching Apex to the canonical package is a Phase 2 refactor, not a drive-by. **Open decision.**

## 2026-07-21 · Tripartite Protocol breached on the Apex feature branch — `ACTIVE`
- **What happened**: Section 0 + Features B/C were designed, implemented, committed **and pushed** without the protocol. Skipped: startup read of the System Architect Directive + `global_status.md`; the Judicial audit (Claude self-reviewed with `flutter analyze`, which the protocol explicitly forbids); the Completion Report and Delivery Gate.
- **Root cause**: no reachable auditor. There is no audit tooling in `C:\development\scripts\`, no `secrets\` directory, and no Gemini/Groq/OpenRouter credential path — the Judicial branch is currently unimplementable by the agent, so it silently degrades to self-audit.
- **Consequences**: `feat/apex-plan-2026-07-21` is **unaudited and unmerged**; a Completion Report exists marked BLOCKED with MCA/MDT NOT RUN. Remediation commit `594b4be` fixed the Directive §2 conformance defects that self-review missed. **`main` must not take this branch until an external audit runs.** Also: `global_status.md` is stale (2026-07-03, describes deleted `my_ai`), so the Directive's hand-off chain is broken.

## 2026-07-21 · Apex iOS bundle ID = `com.nicholaswittle.apex` — `ACTIVE`
- **Decision**: iOS `PRODUCT_BUNDLE_IDENTIFIER` (6 spots) + the `Info.plist` Supabase auth redirect scheme become `com.nicholaswittle.apex`. Android `applicationId` **stays** `com.wisense.apex`.
- **Rationale**: `com.wisense.apex` was registered to another Apple team; the Mac was building with a throwaway `com.nicholaswittle.apex.local` under a free Personal Team. Android's ID is a separate namespace already registered with Firebase — changing it would break push for no gain.
- **Consequences**: iOS and Android bundle IDs legitimately differ; `docs/LAUNCH_CHECKLIST.md` records the split. Firebase iOS registration must use the new ID. **Supabase auth must allow `com.nicholaswittle.apex://` as a redirect or iOS login will not return to the app.** Xcode signing needs re-selecting after pulling. See [[Apex — Feature Plan Implementation 2026-07-21]].

## 2026-07-21 · Sentry dropped from Apex — `ACTIVE`
- **Decision**: Remove `sentry_flutter` entirely rather than upgrade to 9.x — out of `pubspec.yaml`, `sentryDsn` out of `app_config.dart`, `error_monitoring.dart` reduced to a plain `FlutterError.onError` handler.
- **Rationale**: It had no `SENTRY_DSN`, so it reported nothing in production, and 8.x does not build under current Xcode. Upgrading would have paid a migration cost for a feature that was inert.
- **Consequences**: **Apex currently has no crash reporting** — re-evaluate before store launch if crash visibility matters. Sentry also dropped out of the Linux/macOS/Windows generated plugin registrants. Pods must be regenerated on the Mac. See [[Apex — Feature Plan Implementation 2026-07-21]].

## 2026-07-20 · Vault AI performance layer — `ACTIVE`
- **Decision**: Add execution + founder-memory layer without reviving auto-wiki: [[NOW]], `customers/`, [[business/Experiment Log]], [[VAULT_LINT]]; thin [[CLAUDE]] to point at [[agents]]; single boot chain.
- **Rationale**: Research (Karpathy LLM Wiki + founder OS) showed status drift and missing tasks/customer truth hurt agents more than missing folders. See research canvas / [[log]] 2026-07-20.
- **Consequences**: Boot order is now [[hot]] → [[NOW]] → [[index]] → note. Live status only in hot/NOW (not duplicated in CLAUDE). Monthly lint via [[VAULT_LINT]]. Customer truth replaces retired `/crm`.

## 2026-07-20 · Vault is a curated static reference — `ACTIVE`
- **Decision**: Treat this vault as hand-written, cross-linked knowledge. Retire the MindStudio auto-synthesis pipeline; delete empty `wiki/`, `journal/`, `crm/` folders and the sync scripts/task.
- **Rationale**: The `raw→wiki` loop never ran in 6 months; all real value came from hand-written manifests. Empty scaffolding actively misled agents.
- **Consequences**: Codification is manual/on-request. Knowledge lives as **root notes**. Boot order extended by AI performance layer decision above. `WiSenseVaultAutoSync` + sync scripts removed. See [[log]] 2026-07-20.

## 2026-07-20 · Fork reconciliation complete — `SUPERSEDED` (see 2026-07-21 · Apex vendors its own wisense_ui)
- **Decision**: Promote New Horizon's vendored `wisense_core` (47 files) + `wisense_ui` (19 files) to canonical `C:\development\packages\`; New Horizon consumes canonical via path deps; delete the vendored copies.
- **Rationale**: Ends the long-standing package divergence documented in [[Fork Reconciliation]].
- **Consequences**: All green — core 69/69, UI 21/21, NH 117/117, HV2 7/7. `CLAUDE.md`'s "Known fork" caveat is now historical.

## 2026-07-19 · Deleted dead projects — `ACTIVE`
- **Decision**: Remove `wisense-os`, `my_ai`, `local-agent-work-center`, `command_center` from active status.
- **Rationale**: Abandoned/superseded; see [[Abandoned Projects — Lessons]].
- **Consequences**: Do NOT treat as live. `wisense-os` port 5050 daemon no longer exists (invalidated `wisense-engine-probe`).

## 2026-07-19 · Working stack = Claude CLI + Ollama — `ACTIVE`
- **Decision**: Standardize on Claude Code CLI for build + local Ollama (`11434`) for on-device/model work; abandon other agent platforms as primary.
- **Rationale**: See [[Working Stack — Claude CLI and Ollama]].
- **Consequences**: Community AI plugins route through local Ollama; no unencrypted cloud sync without consent.

## 2026-08-01 · Apex venue website is free forever — `ACTIVE`
- **Decision**: The self-serve venue website publishes on the free tier, permanently. Task 2.2 of [[APEX_V2_SELF_SERVE_OS_GAMEPLAN_2026-07-28]] — the "OS Tier Publish Gate" — is closed as **WON'T DO**. Online ordering remains gated to OS ($99/mo).
- **Rationale**: A restaurant website has no pricing power. Square, Wix and Google Business Profile all give one away, so charging $99 for it means competing on a commodity against free. The site is the shop window, not the shop — what carries price is what happens after a guest taps Order: the ticket on the iPad, capacity auto-pause, labor vs revenue, the support agent. The free site's job is acquisition: getting into ten conversations so three convert at $99, which is the $300/month target. The 1.5% order fee will not carry the business at pilot volume; subscriptions will.
- **Consequences**:
  - Do **not** implement a publish gate. The current behaviour (site free, ordering paid) is intended, not an unfinished checkbox — it was discovered during testing on 2026-08-01 and deliberately kept.
  - Pricing must be stated explicitly and early: *"Your website is free, forever. Online ordering is $99/month."* The resentment risk is ambiguity, not generosity — a venue that assumed everything was free and later gets a bill feels tricked.
  - Planned: a "Powered by Apex" footer on free-tier sites, removable on a paid tier. Turns the free tier into distribution in a warm-introduction market, and creates a modest upgrade reason that is not "we are taking your website away".
  - This supersedes the "Build Free, Pay OS to Publish" framing in the 2026-07-28 gameplan, which remains accurate about tiers but not about publishing.

## 2026-08-02 · Payroll Lite is hours-and-tips only — no pay figures — `ACTIVE`
- **Decision**: Apex's payroll export reports **hours and tips**, never computed pay. The `base_pay_cents`, `overtime_pay_cents`, `gross_estimate_cents` and `tip_credit_shortfall_cents` columns come out of `apex_payroll_register()` and the CSV. Apex hands clean data to a real payroll provider (Gusto, ADP, Paychex, QuickBooks) and stops there.
- **Rationale**: Two things collided. The product line held throughout is *"Apex gives them what they need to run payroll easily; it does not run it for them"* — and the register's own disclaimer already says "Hours and tips only — Apex is not a payroll provider." But the code contradicted that: it emitted real dollar figures, and the tipped-overtime figure was roughly a third low. Fixing the arithmetic would have meant owning payroll maths forever, on money someone is actually paid. Removing the columns makes the wrong arithmetic unreachable and matches what was always promised. The cheaper fix is also the honest one.
- **Consequences**:
  - Strip the four money columns from `apex_payroll_register()` and from `lib/core/payroll_export.dart` before `feat/payroll-export` merges. The overtime rate bug then needs no fix — it becomes dead code.
  - **Exposure is latent, not live**: nothing is shipped, nobody at Jigsy's has exceeded 40 hours, and `time_entries` holds exactly **one** punch for Jigsy's (three across all orgs). This is a decision made before harm, not after. That singularity is also why "reconcile a full pay period by hand, once" cannot be done yet — it needs real clock-ins first.
  - The staff-facing "your week" estimate stays **out** until overtime and tip credit are provably right. A server seeing $847 and being paid $612 is the failure this avoids.
  - Does not change the sales answer. "It does everything up to the taxes" is still true — hours and tips *are* the work; the arithmetic was never the valuable part.
  - Supersedes the pay-figure columns described in the 2026-08-02 payroll plan (never filed as a separate note). Research and plan otherwise stand.

## (undated) · Stripe deferred for Apex pilot — `ACTIVE`
- **Decision**: Do not integrate Stripe billing for the initial Apex Scheduler pilot.
- **Rationale**: Out of scope for pilot validation. See [[Stripe]].
- **Consequences**: Revisit post-pilot. *(Confirm/adjust date — carried over from vendor notes.)*

---

Related: [[Home]], [[index]], [[log]], [[hot]], [[company/WiSense Governance — Rules and Protocols]], [[Fork Reconciliation]], [[Abandoned Projects — Lessons]]

## 2026-08-06 · Apex v3 — the AI is a peer actor, and the contract that enforces it — `ACTIVE`
- **Decision**: Every capability in Apex v3 must exist as a named operation callable by the app and by an assistant identically — same RLS, same guards, same metering, never a service-role shortcut. Nick's framing: *"AI could do this whole thing itself but we are using a human to do it… they feel smart but in reality AI is doing the heavy lifting."*
- **Rationale**: If any capability lives only in a widget's tap handler, the assistant cannot perform it and the app silently stops being AI-operable — discovered only once the guiding feature ships and stalls halfway. Close to free at a dozen operations; brutal at a hundred.
- **Consequences**: The confirmation tier is **derived from four risk facts, never declared**, so it cannot be forgotten (D28). Where a tier feels wrong the honest moves are *change reality or accept the ceremony*, never re-answer the fact (D29) — which is why an undo was built rather than an exception granted, twice. A CI gate fails the build on any unregistered mutation in `lib/`. Guests turned out to be a **third actor class** the registry cannot describe, declared separately. Full detail in `apex_v3/docs/decisions.md` D26–D30 and `docs/operations_contract.md`.

## 2026-08-06 · Apex v3 — search is an accelerator, not the onboarding path — `ACTIVE`
- **Decision**: Keep Google Places business search in onboarding, but treat the manual questions as the main path for services businesses.
- **Rationale**: First live run, two real businesses. Jigsy's — established, listed — was found correctly. JR Property Maintenance, a working landscaping business with no online presence, returned nothing. **The businesses search helps are the ones that need Apex least**; the target owner is frequently the one Google cannot find.
- **Consequences**: "I'm just starting out" must read as normal rather than as failure, since for the target owner it is the *first* thing that happens. Search pays most in the restaurant vertical, where listings and hours are reliable — design that flow around it, do not assume the same lift for services.
