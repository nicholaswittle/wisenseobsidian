# Phase 0 — open items and their owners

Everything the master plan put in Phase 0 that is **not** closed by code in this
repo, with an explicit owner. Nothing here is allowed to be "assumed handled" —
that assumption is what produced v2's audit findings.

Status legend: **DONE** · **BLOCKED (needs Nick)** · **DEFERRED (decided, with a date)**

---

## 1. Security and correctness

| Item | Status | Notes |
|---|---|---|
| Baseline schema applied and verified | DONE | 29 migrations, live on `apex-v3-prod`, repo↔ledger parity gated in CI |
| Review BLOCKERs B1–B4 | DONE | Fixed and each proved by an attack test |
| Review HIGHs H1–H6 | DONE | H1/H2 clock, H3 revenue, H4 delete gap, H5 invite entropy, H6 capacity authz |
| RLS / authorization suite | DONE | `supabase/tests/rls_suite.sql`, run against prod, caught 2 further regressions (D22, D23) |
| Money-guard completeness | DONE | `status` + `payment_mode` now pinned; gate rewritten to assert pinning, not mention |
| Anon surface | DONE | 6 functions, each individually justified; zero anon table writes |
| **H9 — founder super-admin** | **DONE** | Email allowlist removed from the signup trigger (the v2 hole). Account created 2026-08-05, email confirmed, elevated by service-role step, grant recorded as a `platform.super_admin_granted` domain event. 35 policies referencing `is_super_admin()` now have a holder. |

## 2. Operations

| Item | Status | Notes |
|---|---|---|
| CI gates (replay, lint, grants, money guard, policy/table snapshots) | DONE | `.github/workflows/ci.yml` + `scripts/` + `ci/allowlists/` |
| Migration ledger ↔ repo filename parity | DONE | B1; regenerated from the applied ledger |
| **Nightly drift alarm on v2 prod** | **DEFERRED** | Script and workflow can be written, but the job needs a repo secret holding a **read-only** credential for the v2 project. Until v2 is frozen this alarm is the early warning that someone changed prod under the port. Ask: add `V2_READONLY_DB_URL` as a GitHub Actions secret. |
| **Sentry** | **BLOCKED (needs Nick)** | The Sentry MCP connector is not authorized in this environment, and this session cannot run an OAuth flow. Authorize the Sentry connector (claude.ai connector settings, or `claude mcp` in an interactive terminal), then instrumentation can be wired. |
| **Secrets manager** | **BLOCKED (needs Nick)** | v2 kept `secrets.rtf` / `twilo keys.rtf` as plaintext files on disk (gitignored, but plaintext). v3's `.gitignore` refuses that shape. Before Phase 1 deploys any edge function, the Twilio / FCM / Anthropic / Gemini / Stripe / Square credentials must be re-issued into Supabase function secrets (and the v2 copies rotated at cutover). No key should ever be pasted into this repo or a chat. |
| GitHub remote | **DONE** | `github.com/nicholaswittle/apex_v3`, private. Pushed 2026-08-05; CI runs on every push. `apex_v2` was also made private the same day — it was public and carried the live system's schema, RLS model and security audit. |
| Restore drill | **DEFERRED to Phase 3** | Master plan schedules it before cutover; nothing to restore yet (zero business rows). |

## 3. Known gaps carried into Phase 1 (decided, not forgotten)

| Gap | Where it came from | Decision |
|---|---|---|
| Shifts can be assigned to staff marked unavailable | UI competitive research — Deputy has the identical validation gap | The `availability` table exists and nothing consults it. Deliberately NOT a hard database constraint: managers legitimately override ("can you come in anyway?"). Phase 1 surfaces it as a scheduling **warning** in the builder, alongside the labor guardrails, where an override is a conscious act. |
| Swap approval model | UI research: When I Work's claim-based swaps are praised for having no approval chain | v3 currently ships claim → manager approve for swaps, while open shifts are claim-and-go. This is a product decision, not a technical one — it goes to the UI kickoff for Nick, not settled by schema default. |
| Fair Workweek / predictive scheduling | UI research — Deputy supports it, PA does not have it | Out of launch scope, but the parity matrix must not silently overclaim compliance breadth. Footnote added to master plan §2. |
| `_shared/square.ts` drift inside v2 prod | Edge-function reconciliation | Three versions live simultaneously; `square-webhook` runs one three deploys stale with a hardcoded refund idempotency key. v3 adopts the newest version once, for every function, at Phase 1. Not fixed in v2 — v2 is frozen to fixes-only, and this is a latent bug, not an active incident. |
| Port hazards PH1–PH7 | Phase 0 review | Enumerated in `edge_function_reconciliation.md`. PH1 and PH3 fail *silently*; the standing rule is that no ported function is done until exercised against the v3 schema with a real payload. |

---

## The one rule this document exists to enforce

An item is closed when someone has **seen it work**, not when it has been
written. Every DONE above has an artifact: a test that ran, a query that
returned the expected row, or an attack that was refused. Every BLOCKED names
the human and the exact action. Nothing is implied.

---

## Status as of 2026-08-05, end of Phase 0 remediation

**CI is green on all seven gates**, running on every push to
`github.com/nicholaswittle/apex_v3` (private). This is the line between
"we built gates" and "gates protect us" — everything above marked DONE was
previously verified only by hand.

| Gate | Proven by |
|---|---|
| migration-replay | every migration rebuilds from empty on postgres:17 |
| migration-lint | ordering, duplicates, future dates, revoke-completeness, policy roles, **and every migration on disk tracked by git** |
| migration-ledger-drift | repo prefixes diffed against the live ledger with a read-only credential |
| grant-allowlist | anon/authenticated function + table privileges and policy roles diffed against checked-in snapshots |
| money-guard-completeness | 18 columns proven pinned inside the immutability condition |
| rls-suite | 70 assertions, both directions, role-impersonated |
| detect-flutter / flutter | correctly skipping until `pubspec.yaml` exists |

**What the gates caught on their first working runs** — this is the argument
for having built them:

- `migration-ledger-drift`, first successful run: found a migration applied to
  production with no file in the repo (B1's failure mode, live, again).
- `migration-lint`'s new tracked-files rule: found that `.gitignore`'s
  `*secret*` pattern had silently swallowed
  `..._clock_secret_for_new_orgs.sql` — a migration. `git status` never shows
  ignored files, so the working tree looked clean while CI checked out one
  fewer migration and the clock-secret trigger was never created.
- The `pipefail` fix: before it, a failed database connection produced an
  *empty* ledger and the gate blamed the repo for 23 phantom drifts. It now
  reports the actual error.

**Still open, honestly:**

| Item | Status |
|---|---|
| Sentry | BLOCKED — connector not authorized; this session cannot run OAuth |
| Secrets manager | BLOCKED — Twilio/FCM/AI/Stripe/Square keys must be re-issued into Supabase function secrets before Phase 1 deploys any function |
| Nightly drift alarm on **v2** prod | DEFERRED — v3's drift gate is live; the v2 watch matters only while v2 is still authoritative, and needs a separate read-only credential |
| Restore drill | DEFERRED to Phase 3 — nothing to restore yet (1 org, 1 profile) |
| Third adversarial review | PENDING — the actual verdict on whether Phase 0 closes |

## Fifth review — carried forward (2026-08-05)

| Item | Status |
|---|---|
| **H-DML** — clients hold direct table DML while the protections live in RPCs | **DONE (sixth review, 2026-08-06).** See the sixth-review section at the end of this file; the detail below is retained as the record of what was found. |
| **H-REVENUE** — refunds never decrement `daily_revenue` | **DONE (sixth review, 2026-08-06).** `apex_refund_order`, `refunded_at`/`refunded_by`, compensating `order_refunds` row. |
| `pg_cron` / `pg_net` not installed | OPEN. `apex_prune_guest_order_attempts()` and `apex_sweep_expired_orders()` exist and are correct but nothing calls them on a schedule. The guest hot path no longer carries a `random()` prune dice roll, so pruning of `guest_order_attempts` is now *entirely* dependent on this. |
| `domain_events` drain | OPEN — out of scope for this stage. |
| 27 edge functions, `config.toml` / `verify_jwt`, deploy + drift gates | OPEN — explicitly next stage. `refund-order` now has a documented DB-side contract to build against (`docs/decisions.md`, H-REVENUE). |

### H-DML, recorded precisely so the next pass does not have to re-derive it

`authenticated` holds INSERT/UPDATE/DELETE through PostgREST on
`time_entries`, `online_orders`, `organization_members`, `daily_revenue`,
`server_tips`, `swaps` and `organization_invites`. Every clamp built over
five review rounds lives in RPCs that these paths never touch. Confirmed
against the live catalog during this pass:

- **manager/owner can rewrite or DELETE any employee's `time_entries` row**
  (`time_entries_manager_update` / `time_entries_manager_delete`), moving
  hours between employees (`user_id` is unpinned), inflating their own, or
  erasing a punch entirely — no audit row, no event, nothing retained. The
  16-hour CHECK added this pass bounds the *duration* of such a row but says
  nothing about who it belongs to.
- **owner can `UPDATE organization_members SET user_id = <stranger>`** —
  USING and WITH CHECK both pass because they own the org before and after —
  grafting a stranger in and exposing their name/email/phone/DOB via
  `shares_org_with`.
- **owner can forge `tip_pool_eligible_confirmed_by`** by PATCHing it
  directly. *Partially mitigated this pass*: the audit row now records
  `auth.uid()` rather than trusting that column, so the trail is honest even
  though the column itself is still writable.
- **`server_tips` are self-mutable and self-deletable** with a
  client-supplied `updated_at` (no trigger maintains it). Declared tips are a
  tax record.
- **`daily_revenue.created_by`** is client-supplied with no default or
  trigger.
- **manager can drive `swaps` to `Approved` without the shift moving**
  (`swaps_manager_update` is column-unrestricted), bypassing
  `apex_approve_swap`'s tenant pin, membership recheck, `shifts.user_id`
  update and event.

The required shape, and the reason it was not rushed: **every operation
closed must ship with the legitimate path that replaces it, proven working.**
That is the specific lesson of H-CORRECTION in this same review — the
previous round dropped the tip-split write policies and left managers with no
way to correct a wrong total at all, which is how a security fix became an
operational outage. Closing seven tables' worth of DML without building and
proving each correction path first would repeat that failure at seven times
the scale. It needs `time_entry_revisions` (or equivalent) written by an
AFTER UPDATE OR DELETE trigger capturing old/new plus `auth.uid()` with no
client DML grant, immutable-column BEFORE triggers where a direct write must
remain, and an RPC per closed operation.

---

## Sixth review — 2026-08-06

| Item | Status |
|---|---|
| **H-DML** — clients hold direct table DML while the protections live in RPCs | **DONE.** Eleven operations closed, each with its replacement path built and proved working first. `record_revisions` is the provenance layer. 41 new assertions, each watched red then green. Full table of operation to legitimate path in `docs/decisions.md` S-2. |
| **H-REVENUE** — refunds never decrement `daily_revenue` | **DONE.** `apex_refund_order` + `refunded_at`/`refunded_by` + a compensating `source='order_refunds'` row. The DB-side contract the edge function must be written against is in `docs/decisions.md`, H-REVENUE. |
| `pg_cron` / `pg_net` not installed | OPEN, unchanged. `apex_prune_guest_order_attempts()` and `apex_sweep_expired_orders()` are correct and uncalled. |
| `domain_events` drain | OPEN — next stage, unchanged. |
| 27 edge functions, `config.toml` / `verify_jwt`, deploy + drift gates | OPEN — next stage, unchanged. |

### Discovered during this pass, recorded rather than fixed

| Item | Status | Detail |
|---|---|---|
| **The `service_role` exemption is now load-bearing across five guards** | **OPEN — the first thing the edge-function stage must address.** | `apex_guard_order_money`, `apex_guard_daily_revenue`, `apex_guard_membership_identity`, `apex_guard_swap` and `apex_guard_record_revisions` all return early for `service_role`. Every edge function runs as `service_role`. So an edge function can move hours between employees, graft a member, rewrite a refund, or prune the append-only revision log — and because `auth.uid()` is null under the service key, nothing it writes names a human. This was true before this pass for the money guard alone; it is now true for the whole H-DML layer, which raises the stakes rather than creating the hole. Deliberately not narrowed here: doing it correctly means giving each function a scoped identity, and that decision belongs with the functions themselves. In the meantime the mitigation is procedural and written down: **an edge function calls an RPC, it does not write a table.** |
| `docs/decisions.md` carries the "Fifth review" section twice | OPEN (documentation debt) | The two copies are NOT identical — only the second contains H-CORRECTION — so neither was deleted blind during this pass. Reconciling them needs someone to confirm nothing else diverges. |
| `apex_correct_time_entry` cannot move an entry's `shift_id` or `venue_id` | OPEN (deliberate, small) | The RPC corrects times only. Re-associating a punch with a different shift is not a task anyone has asked for yet, and adding parameters nobody uses widens the surface. If the UI needs it, it is an added parameter, not a new path. |
| No throttle on `apex_refund_order` | OPEN (accepted) | A compromised manager JWT can issue many small refunds. Each is bounded by `total_cents`, each emits an event and each writes a revision, so the action is bounded and fully attributed — which is the property that matters. Rate limiting belongs at the gateway, the same conclusion BL-1 reached for guest ordering.

### What the red-then-green discipline caught this round

Two things, both worth recording because they are the failure modes this
project keeps hitting:

* **A probe encoded the bug, and the code was right.** The first
  `apex_correct_time_entry` positive assertion failed. The RPC was correct —
  the probe had asked a manager to set a punch-out three hours in the future.
  Had the assertion been trusted over the code, the future-punch guard would
  have been removed to make a bad test pass. The test was fixed; the code was
  not touched; and the rejection is now its own assertion (TE-9).
* **A swallowed exception produced an EMPTY result set, not a green one.** An
  early probe wrapped its whole `do` block in `exception when undefined_function
  then null`, which rolled back every `insert into res` alongside the error.
  The suite's "a NULL assertion is vacuous" rule is the same lesson at the row
  level; the block-level version is that a *missing* assertion looks like
  success unless you count them.

### Three findings this pass declined to implement as written

Recorded because "the review said so" is not evidence:

1. **`server_tips` "self-mutable — a staffer can move the row onto another
   employee".** False. The `server_tips_own_write` WITH CHECK already pinned
   `user_id = auth.uid()`; the assertion passed before any migration. The real
   gaps (client-supplied `updated_at`, untraced DELETE) are closed.
2. **`online_orders` direct DML.** `authenticated` holds the grants but there
   is no INSERT or DELETE policy, so RLS denies both, and the sole UPDATE
   policy is already fully constrained by `apex_guard_order_money`. No
   lockdown was needed and none was added.
3. **Replacing `organization_members` DELETE with an RPC.** It already emits
   `membership.left`, and it now also writes a full `record_revisions` row.
   Replacing it would have removed a working offboarding path for no gain —
   exactly the H-CORRECTION mistake.

---

## Edge-function stage — 2026-08-06

The half of Phase 0 that had never been reviewed. Entering this stage: 27
functions in the repo, **zero deployed**, no `config.toml`, no gate of any kind
over `supabase/functions/`.

### Closed

| Item | Status |
|---|---|
| **The `service_role` contract** — "an edge function calls an RPC, it does not write a table" | **MECHANICAL.** `scripts/check_function_writes.py` + `ci/allowlists/edge_function_writes.txt`. Found **42** direct mutations across 16 files. `refund-order` and `reconcile-pending-payments` — the two that wrote `online_orders`, the table `apex_guard_order_money` protects — now call RPCs. The rest are allowlisted with a per-file justification; all but four are in functions that cannot be deployed anyway. |
| `supabase/config.toml` | **DONE.** Explicit `verify_jwt` for all 27, with the justification for every `false` in the file. `scripts/check_function_config.py` fails if a directory has no entry, if an entry has no directory, or if an open function's own validation disappears from its source. |
| Deployment drift gate | **DONE** (`scripts/check_function_drift.sh`). Blocked on a secret — see below. |
| `deno check` in CI | **DONE** — `edge-function-typecheck` job. |
| Functions deployed | **10 of 27.** See "Why only 10" below — this is the stage's main finding. |
| `pg_cron` / `pg_net` | **INSTALLED** (1.6.4 / 0.20.4). `cron` schema revoked from anon/authenticated. |
| Scheduled jobs | **DONE, and observed running.** `apex-sweep-expired-orders` `*/5 * * * *`, `apex-prune-guest-order-attempts` `7 * * * *`, `apex-prune-domain-events` `17 3 * * *`. First real pg_cron run recorded `succeeded` in `cron.job_run_details`. Asserted by `scripts/check_cron_schedules.sql` in `migration-replay`. |
| `domain_events` drain | **DONE** — `domain_event_consumers` cursor registry + `apex_drain_domain_events` (exactly-once per consumer, proven) + `apex_system_prune_domain_events` (90-day retention that a lagging consumer holds back, proven both ways). No business consumer was invented; there is no requirement for one. |

### Why only 10 of 27 were deployed — that stage's main finding

> Superseded in part on 2026-08-06: five of the blocking tables now exist and
> the count is **15 of 28 deployed**. See "The money path" below. The analysis
> here is kept because it is why the schema-refs gate exists.

**Sixteen of the 27 functions name a table or RPC that does not exist in v3.**
Twelve tables (`restaurants`, `requests`, `request_payments`,
`venue_site_profile`, `request_ai_runs`, `request_notification_outbox`,
`call_outs`, `call_out_notifications`, `square_oauth_credentials`,
`provider_refunds`, `apex_billing_events`, `support_escalations`) and seven
RPCs. Verified against `pg_catalog`, not inferred.

They were retrieved verbatim from v2 prod and never adapted
(`docs/edge_function_reconciliation.md`: "pending Phase 1 adaptation"). This
fails at **runtime**, not at deploy and not at type-check — a PostgREST 404 the
caller sees as a generic 500. Deploying them would have put 16 broken
endpoints in production, including guest-facing `create-guest-payment` and
three provider webhooks, and every one of them would have *looked* deployed.

`scripts/check_function_schema_refs.py` now proves this mechanically, resolving
every `.from()` and `.rpc()` against a manifest derived from the replayed
migrations. It propagates through `_shared/` imports — which is how it caught
that `refund-order` inherited a broken Square rail via
`_shared/square-credentials.ts`. `ci/allowlists/functions_not_deployed.txt`
records the 17 undeployed with the exact blocking object.

### Still open

| Item | Status |
|---|---|
| **`SUPABASE_ACCESS_TOKEN` repo secret** | **BLOCKED (needs Nick).** `edge-function-drift` fails loudly without it, exactly as `migration-ledger-drift` did before `APEX_V3_DB_URL` was added. A Supabase personal access token with read access to `fnsonnhumcvxdnyarguv`; the gate makes one GET and never deploys. |
| **Function secrets** | **BLOCKED (needs Nick), unchanged.** No secrets are set on this project. Every deployed function now announces its missing secrets by name in its boot log and, where a missing secret makes it inert, returns a 503 naming them (`supabase/functions/_shared/env.ts`). See the report's per-function checklist. |
| **Webhook idempotency** | **DONE (money-path pass, 2026-08-06).** The three ledgers exist (`provider_refunds`, `apex_billing_events`, `square_oauth_credentials`) and the webhooks land through validated security-definer RPCs. Replay safety is asserted by applying the SAME event twice and proving the second moves nothing: `MP/W-2`, `MP/P-2`, `MP/B-2`, `MP/R-3b`, `MP/REV-2` in `supabase/tests/rls_suite.sql`. Each assertion was watched FAIL against a build with its guard removed before it was trusted — see "Red-then-green" below. |
| **`check-capacity` / `stripe-connect-onboard` direct writes** | OPEN. Both write `restaurant_settings` — pausing a venue's ordering, and setting where its money lands — with no domain event and no revision. Allowlisted, not converted: the right shape (a provenance trigger on `restaurant_settings` covering several writers, vs. an RPC each) is a schema decision. |
| **Scheduling the two edge-function sweeps** | OPEN. `check-capacity` and `reconcile-pending-payments` are HTTP, so pg_cron would need `pg_net` carrying the service-role key inside `cron.job.command` in clear text, readable by anyone with database access. That is a secret-handling decision (Vault vs. Supabase's own scheduler), not one to settle in a migration. |
| **Square tip refunds** | OPEN (narrowing, recorded). `online_orders_refund_within_charge` caps `refunded_cents` at `total_cents`, which excludes the Square tip. `refund-order` now refuses rather than moving money the ledger cannot represent. |

### Frozen services pack — non-normative, per-function missing objects

Nine functions are **frozen pending redesign**, not awaiting a port. They encode
v2's services model, which is landscaping-shaped and venue-keyed
(`p_restaurant_id` for a business that has no restaurant, `p_town` as free
text). The replacement services schema pack is being designed **trade-neutral** —
plumbers, HVAC, cleaners, electricians. If these files become the reference for
that design, the rebuild's premise is void.

Each now carries a `FROZEN — NON-NORMATIVE` header naming its own missing
objects, and the set is committed at `ci/allowlists/functions_frozen.txt`. All
missing objects below were verified against `pg_tables` / `pg_proc`, not assumed.

| Function | Missing from the v3 schema |
|---|---|
| `submit-public-request` | RPC `submit_public_request_from_gateway` |
| `create-request-photo-upload-intent` | RPC `reserve_public_request_photo_upload` |
| `draft-quote-from-photos` | `requests`, `request_ai_runs` |
| `create-request-payment` | `requests`, `request_payments`, `restaurants`, `venue_site_profile` |
| `request-payment-webhook` | `requests`, `request_payments` |
| `notify-request` | `requests`, `request_notification_outbox`, `restaurants`, `venue_site_profile` |
| `notify-request-scheduled` | `requests`, `restaurants`, `venue_site_profile` |
| `sweep-request-reminders` | `request_payments`, `restaurants`; RPCs `apex_claim_request_reminder`, `apex_settle_request_reminder` |
| `sweep-request-notifications` | RPC `apex_claim_request_notifications` |

**`route-callout` is deliberately NOT in this set**, despite appearing on one
circulated list of it. It is staff shift call-outs — a coworker calls out sick
and eligible coworkers are notified — categorised `notifications` in
`docs/edge_function_reconciliation.md`, present in the v2 working tree, and
unrelated to the services vertical. It is blocked on `call_outs` /
`call_out_notifications` being ported, which is a real port task for a feature
that is wanted as it stands. Freezing it as a non-normative services artifact
would point its next author at a redesign that does not apply to it. The same
applies to `venue-briefing` (`call_outs`), `create-guest-payment`
(`restaurants`, `square_oauth_credentials`), `square-webhook`
(`provider_refunds`), `square-connect-oauth` / `square-test-order`
(`square_oauth_credentials`), `stripe-os-webhook` (`apex_billing_events`) and
`venue-support-agent` (`support_escalations`) — eight blocked-on-port functions,
listed in `ci/allowlists/functions_not_deployed.txt` only.

The drift gate reads both files and reports the two counts separately, because
"frozen, do not port" and "wanted, needs its tables" call for opposite actions.

---

## Vertical-neutrality preparation — 2026-08-06

Core preparation only. **The services pack was NOT built**, `lib/` was not
touched, no `pubspec.yaml` was created and no secret was read or set. Eight
migrations (57 total), all against empty tables — 1 organization, 1 profile, 0
venues, 0 shifts, 0 time entries, 0 orders. This is the cheapest this work will
ever be, and after launch three of these six items become impossible rather than
merely expensive.

Full reasoning for every choice is in `docs/decisions.md` under
"Vertical-neutrality preparation", items VN-1 to VN-7.

### Closed

| Item | Status | Proven by |
|---|---|---|
| **1. Work-unit reference on `time_entries`** | **DONE.** `work_ref uuid` (un-FK'd, polymorphic) + `work_kind text` FK'd to a new `work_kinds` registry, both-null-or-both-set. `apex_clock_in` carries it (6-arg signature dropped, not overloaded); `apex_set_time_entry_work` is the manager-only correction path, shipped in the same migration. Covered by the existing `record_revisions` trigger. | VN/WU-1 … WU-13. WU-8 computes labour cost for one work unit from the database alone and proves it survives a time correction. |
| **2. Module registry as data** | **DONE.** `tiers` + `modules` tables replace two hardcoded CASE expressions; `apex_org_has_module_internal` joins them and now reads `organizations.vertical`. Seeded byte-identically, then `onlineOrdering` / `smartCapacity` / `noShowEngine` marked `restaurant` in a separate migration. | VN/MOD-1 … MOD-9. MOD-2 is a 5-tier x 16-module identity matrix watched green before AND after. MOD-8 registers `dispatch` with an INSERT and proves a services org gets it and a restaurant org does not. |
| **3. `vertical` stops failing open; `trade` added** | **DONE.** Column default dropped; `apex_handle_new_user` leaves `vertical_chosen_at` NULL when the metadata is absent or unrecognised; nullable `trade` added without widening the `vertical` CHECK. `vertical` joined `apex_guard_org_entitlements` as a one-way door, because marking modules made it an entitlement lever. | VN/V-1 … V-10. V-1/V-2 are the restored v2 tell. V-6 and V-7 prove the door and prove the evidence cannot be erased. |
| **4. `venues` vs a neutral location** | **DONE, option (b).** `venues` keeps the core location role and gains address / locality / region / postal code / country / lat / lon / `service_radius_m`. `public_token` moved to `venue_guest_endpoints` (restaurant pack), minted by a pack-owned trigger ONLY for `vertical = 'restaurant'`. | VN/LOC-1 … LOC-13. LOC-2 is the point of the item; LOC-4 proves guest ordering still resolves end to end through the moved token. |
| **5. `crew_id` on `shifts`** | **DONE** (nullable, no FK, no `crews` table). | VN/CREW-1 … CREW-3. |
| **6. `venue_clock_secrets` → `org_clock_secrets`** | **DONE.** Both reading functions recreated (a rename does not rewrite plpgsql bodies; they fail at RUN time). | VN/OCS-1 … OCS-3. OCS-3 re-proves that a punch completes with venue, shift and work unit all NULL. |
| **7. Frozen services artifacts** | **DONE (documentation).** Reinforced in `docs/decisions.md` VN-7. | — |

### Decided and written down rather than built

| Item | Decision |
|---|---|
| **Recurrence** | **Core, not per-pack.** No rrule and no parent-schedule FK exists anywhere; an HVAC twice-yearly maintenance visit has no representation today. Nothing is built (a recurrence table with no reader is a guess), but the LOCATION is now settled so restaurant and services cannot grow two incompatible models — precisely the v2 failure. `docs/decisions.md` VN-5. |
| **`domain_events.venue_id`** | Left as-is, deliberately. Under option (b) it references the core location concept, which is now neutral. Renaming the single location column on an append-only log buys a nicer noun for the same edge-function blast radius that ruled out option (a). |
| **`shifts.is_event`** | Restaurant-shaped (a private party) and left in place: relocating it means deciding where the restaurant pack's shift extensions live, and no such table exists. Known debt, not a discovery. `shifts.zone` generalises honestly and stays. |
| **`notification_preferences`** | **NOT generalised, with reasons.** A category map needs a notification-category registry whose contents are determined by `apex_notify_user`'s taxonomy and the settings UI, neither of which exists. Building it first would create a second source of truth, and the wrong one would be the one in the database. No security or money consequence; adding a column later is cheap and non-breaking. `docs/decisions.md` VN-6. |
| **`tipManagement` / `serviceRequests` not marked vertical-specific** | The marking criterion is written into `20260806015136_mark_hospitality_only_modules.sql`. `tipManagement` lives outside the restaurant pack; marking `serviceRequests` as `services` would be designing the unbuilt pack from a module name. |

### Carried forward — unchanged by this pass

`SUPABASE_ACCESS_TOKEN` (edge-function-drift), function secrets, Sentry,
webhook idempotency, the `service_role` guard exemptions, and the v2 nightly
drift alarm all remain exactly as recorded above, with one correction:
**`edge-function-drift` now PASSES.** It was documented above as blocked on a
missing `SUPABASE_ACCESS_TOKEN` repo secret and was expected to fail on this
run; it did not, on either CI run for this branch. The secret has evidently
been added since that section was written. Nothing in this pass touched it --
recorded here so the next reader does not chase a gate that is already green.

### One thing this pass got wrong, and what caught it

The module-registry migration copied a `grant execute ... to authenticated` onto
`apex_org_has_module_internal`, widening an ungated entitlement lookup that
answers for any organization. Nothing in the schema depends on that grant, and
`ci/allowlists/authenticated_functions.txt` does not list the function — which
is how it was found within minutes.
`20260806015051_module_registry_restore_internal_grant_boundary.sql` revokes it
and is kept as its own migration so the mistake stays in the record. The
allowlist gate has now caught a real widening on its own terms, which is the
first time it has done so.

---

## Edge-secret health and the status contract — 2026-08-06

### Closed

| Item | Status |
|---|---|
| **Function secrets — "are they set, and do they look right?"** | **DONE (mechanism).** `supabase/functions/health-secrets` reports `missing` / `malformed` / `present` per secret, names and states only, no provider call. Deployed to `fnsonnhumcvxdnyarguv`, ACTIVE, `verify_jwt = true` plus a service-role bearer check. Refusal proven live: no token 401, junk bearer 401, **valid anon JWT 401 `service_role_required`**. |
| **The expected-secret inventory** | **DONE.** `ci/allowlists/function_secrets.txt` — 25 secrets, 3 required / 19 optional / 3 platform, each with its readers and structural rule. `scripts/check_function_secrets.py` diffs it against every `Deno.env.get` / `secret()` in the tree, proves the embedded copy is byte-identical, and asserts both are tracked by git. |
| **GEMINI_API_KEY / GOOGLE_AI_API_KEY** | **RESOLVED — one credential, two names, in ONE function.** Standardised on `GOOGLE_AI_API_KEY`, the name actually set on the project. `docs/decisions.md` ES-2. |
| **Validation errors returning 5xx (and server faults returning 200)** | **DONE across all 11 deployed functions.** `_shared/http.ts` fault vocabulary + `scripts/check_function_status_contract.py`: 27 violations before, 0 after. `docs/decisions.md` ES-5. |
| **`.gitignore` `*secret*` swallowing repository files** | **DONE.** `!ci/**` added; the manifest gate now asserts git-tracking. Second occurrence of this exact class. `docs/decisions.md` ES-4. |

### What the audit found that was NOT on anyone's list

**Four deployed functions were reading `profiles` columns that v3 deleted.**
Verified against `pg_catalog`, not inferred. Fixed in this pass; full table in
`docs/decisions.md` ES-6. Summarised because it changes what "10 deployed
functions" was worth:

* `notify-order-event` — refused **every** caller, as a 401.
* `stripe-connect-onboard` — **500 to everyone**; nobody could connect Stripe.
* `route-notification` — **500 on every call**.
* `check-capacity` — degraded **silently at HTTP 200**, measuring a
  restaurant's whole clocked-in roster instead of its cooks. A venue with two
  servers and no cook cleared a threshold of two and kept taking food orders.

No customer was affected, for one reason only: `lib/` is empty and there is no
client yet. This is the last moment these are free to fix.

### Still open

| Item | Status |
|---|---|
| **`APEX_V3_SERVICE_ROLE_KEY` repo secret** | **BLOCKED (needs Nick).** `.github/workflows/secret-health.yml` calls `health-secrets` daily and fails loudly without it, the same discipline as `migration-ledger-drift`. It is deliberately NOT in `ci.yml` — it measures the project's configuration, not the commit's correctness, and no commit can make it pass (`docs/decisions.md` ES-7). Until the secret exists, the scheduled run is red by design and the push CI is unaffected. |
| **A COLUMN-level schema-refs gate** | **OPEN, and it is the gap ES-6 walked through.** `check_function_schema_refs.py` resolves `.from()` and `.rpc()` names only, so four deployed functions named columns that do not exist and every gate stayed green. The fix is to parse each `.select("a, b, c")` and resolve the columns against the replayed catalog. Not done here: it goes red across the 17 undeployed functions too, and deciding its scope is its own piece of work rather than a rider on this one. |
| **`send-push-notification` does not exist in this repo** | **OPEN.** `route-notification` fetches `${SUPABASE_URL}/functions/v1/send-push-notification`, which v2 had and was never ported. Push is now recorded as `skipped` with detail `push_transport_unported` rather than `failed`, because "Firebase rejected this token" and "the transport was never built" want opposite actions from whoever reads the receipts. The SMS fallback still runs. `route-notification` is repaired against the v3 schema but has NOT been exercised end to end — no client exists to call it. |
| **A typo'd secret name is invisible** | **OPEN (accepted).** `health-secrets` reports the EXPECTED inventory. Detecting `ANTROPIC_API_KEY` sitting next to a missing `ANTHROPIC_API_KEY` needs `Deno.env.toObject()`, which reads every value — a worse trade than the gap. The `missing` verdict on the correctly-spelled name is the signal. |
| **`malformed` is structural, never validation** | **OPEN by design.** A present, well-formed, REVOKED key reads as `present`. Only the provider can say otherwise, and asking is a billable call from a health check. `docs/decisions.md` ES-1. |
| Sentry, webhook idempotency, the `service_role` guard exemptions, the v2 nightly drift alarm | Unchanged. |

### RESOLVED 2026-08-06 — the five applied-but-untracked migrations are now committed

Closed by the money-path pass, which owns them. Repo↔ledger is **62 / 62**, and
parity was verified by **md5 against the ledger**, not by filename: each of the
five files is byte-identical to
`md5(array_to_string(statements, E';\n\n'))` for its row in
`supabase_migrations.schema_migrations`, after the single normalization
`fetch_missing_migration.py` applies (terminating `;` plus one trailing
newline).

Two honest limits of that check, recorded rather than glossed:

* For the other 57, the ledger's `statements` array is a **re-serialization**,
  so byte equality is not expected and is not the test. What was verified for
  those is that the **executable SQL is identical** once comments, whitespace
  and non-ASCII are normalized away — 53 match on the cheap comparison, and the
  remaining 9 (the renamed baseline set) match once SQL comments are stripped.
  The difference in those 9 is prose only: the repo files carry expanded lesson
  notes written when they were regenerated, and one file has an em dash that a
  Windows round-trip flattened to a hyphen.
* `migration-ledger-drift` in CI compares **version lists**, not checksums. The
  md5 comparison above was run by hand for this pass; making it a gate is a
  separate piece of work.

The original finding follows, kept for the record.

---

`migration-ledger-drift` went red on the previous pass's CI run and it was right
to. Verified against the live ledger at the time: **62 applied, 57 tracked**.
These five were in `supabase_migrations.schema_migrations` on
`fnsonnhumcvxdnyarguv` with no file in the repository:

    20260806025854_record_revisions_action_vocabulary
    20260806030028_provider_refunds_ledger
    20260806030129_provider_payment_capture_rpcs
    20260806030159_apex_billing_events_ledger
    20260806030238_square_oauth_credentials

The files exist, untracked, in the shared working tree — another session's
work, in flight, applied to production before being committed. This is
BLOCKER-1's exact shape and the precise reason this gate was built: *the code
you review is not the code that runs.* Two of them create the idempotency
ledgers three provider webhooks are blocked on (`provider_refunds`,
`apex_billing_events`) and one creates `square_oauth_credentials`, so they are
plainly wanted — but they are unreviewed by this pass and they move payment
infrastructure.

**Deliberately not committed by that pass.** Staging is by explicit path, and
sweeping another author's unreviewed payment migrations into an unrelated commit
to turn a gate green is how a gate stops meaning anything. The gate was telling
the truth and stayed red until their author committed them — which is what the
money-path pass did, in a commit of their own, on 2026-08-06.

Every other job on that run was green (11 of 12), including all three gates that
pass added.

---

## The money path — what closed on 2026-08-06, and what did not

### Deployed (15, up from 10)

`create-guest-payment`, `stripe-os-webhook`, `square-webhook`,
`square-connect-oauth` joined the deployed set; `refund-order` was redeployed
with the reservation flow. Repo-vs-prod was checked in **both** directions and
is 1:1 at 15.

### The refund race — closed

`refund-order` used to compare-and-set `refunded_cents` around the provider
call, and **the loser returned HTTP 200 with a warning field**. That shape is
the dangerous one: success-shaped, so nothing alerts, while the ledger
under-reports money that actually left — which then permits further refunds
against a balance that no longer exists.

The fix is reserve-before-provider. `apex_reserve_order_refund` claims the
balance under `select ... for update` on the order and counts outstanding
reservations, so the loser is refused with `refund_exceeds_order_total`
**before any money moves**, and the function returns **409**, never a 2xx. The
processor idempotency key is now the server-minted reservation id rather than
`refunded_cents` — the very value the race made unreliable. Correction path
shipped with it (`apex_void_order_refund_reservation`), because a reservation
whose processor call failed would otherwise reduce the refundable balance
forever.

### Red-then-green — how the assertions were trusted

Six deliberate regressions were applied to a replayed database, one per fix, and
the suite was run against each. Every one was caught by the assertion that names
it; the suite returned to 273 pass / 0 fail / 0 NULL after each was reverted.

| Fix removed | Assertion that went RED |
|---|---|
| Provider-refund replay guard | `MP/W-2`, `MP/W-3`, `MP/W-5` |
| Outstanding-reservation count (the race) | `MP/R-2`, `MP/R-3b` |
| Settle replay guard | `MP/R-3b`, `MP/R-4` |
| Revenue reversal | `MP/REV-1`, `MP/REV-2`, `MP/W-1`, `MP/R-3a`, `H-REVENUE/1`, `H-REVENUE/2` |
| Payment-capture replay guard | `MP/P-2` |
| Payment account binding | `MP/P-4` |

### Signatures — verified structurally, not by a live call

All three open money endpoints refuse to run on an absent **or empty** signing
secret rather than verifying with an empty key. `_shared/env.ts` treats a
whitespace-only secret as missing, and the check runs before any body is read,
so the refusal is a 503 that names the secret.

* `stripe-os-webhook` — `STRIPE_WEBHOOK_SECRET` required; `stripe-signature`
  header required (400); `constructEventAsync` verifies. The optional
  `STRIPE_WEBHOOK_SECRET_CONNECT` is `.filter(Boolean)`-ed, so an unset connect
  secret can never become an empty key everything verifies against.
* `square-webhook` — `SQUARE_WEBHOOK_SECRET` **and** `SQUARE_WEBHOOK_URL` both
  required. The URL is not optional: Square signs URL+body, so without it a
  genuine event is rejected as forged forever and the log blames an attacker
  instead of a missing secret.
* `square-connect-oauth` — `SQUARE_OAUTH_STATE_SECRET` required. An empty HMAC
  key would let anyone mint an authorization state and bind their own Square
  merchant to any organization.

### Still NOT proven, and by which named secret

`STRIPE_SECRET_KEY` is set. **`STRIPE_WEBHOOK_SECRET`,
`STRIPE_WEBHOOK_SECRET_CONNECT`, `SQUARE_WEBHOOK_SECRET` and
`SQUARE_OAUTH_STATE_SECRET` are not.** Consequently:

* No provider webhook has ever been delivered to this project. Signature
  ACCEPTANCE is unexercised; only the database layer beneath it is proven.
  Pending `STRIPE_WEBHOOK_SECRET` / `SQUARE_WEBHOOK_SECRET`.
* No Square merchant has ever been connected. The OAuth callback path is
  unexercised end to end. Pending `SQUARE_OAUTH_STATE_SECRET` plus
  `SQUARE_PRODUCTION_APP_ID` / `SQUARE_PRODUCTION_CLIENT_SECRET`.
* No guest has paid. `create-guest-payment` can reach Stripe now that
  `STRIPE_SECRET_KEY` is set, but no venue on this project has completed Stripe
  Connect onboarding, so no charge has been made.

What IS proven is everything below the provider boundary: replay safety, the
refund race, the revenue correction, the authorization split, and that every
webhook write lands through an RPC rather than a table.

### KNOWN ARCHITECTURAL DEBT — v3 shares v2's Stripe platform account

**Decided by Nick, 2026-08-06, with eyes open. Recorded here so it is not
rediscovered as a surprise.**

v3 and v2 are both Connect platforms on the **same Stripe account**
(`acct_1TqfSbHeXj7HLVbu`, application `ca_UyH1RaCP8BJyeAYU6qlptLVCepSL0eXC`).
Stripe fans a connected-account event out to **every** Connect webhook endpoint
registered under that application, regardless of which deployment created the
charge. There is no per-endpoint filter for "only my own connected accounts".

Concretely, a guest paying a v3 venue emits one `checkout.session.completed`
that Stripe delivers to all three of:

| Endpoint | Deployment | URL |
|---|---|---|
| `we_1U1PjN` (`apex-v3-stripe-connect`) | **v3** | `fnsonnhumcvxdnyarguv…/stripe-os-webhook` |
| `we_1Tykth` | v2 | `pqkremkwfkudrhtxasdj…/stripe-os-webhook` |
| `we_1U12Z3` | v2 | `pqkremkwfkudrhtxasdj…/request-payment-webhook` |

So **v3 cannot take a payment without touching v2's production deployment.**
For the 2026-08-06 end-to-end proof this was accepted as a read-only hit: v2's
`stripe-os-webhook` finds no matching order and skips its write;
`request-payment-webhook` returns before any DB access. Every endpoint above is
`livemode: false`, so v2's live paying venue sits on a separate live-mode
endpoint set and was never in scope.

**Why this is debt and not a settled design:**

* The isolation is *behavioural*, not *enforced*. It holds because v2's handlers
  happen to no-op on ids they do not recognise. Nothing in Stripe, in v2, or in
  v3 prevents a future v2 handler — or a v2 handler for an event type nobody
  analysed — from writing on a v3-originated event. The safety property is "we
  read the other system's code once and it looked fine", which is exactly the
  class of assumption this project keeps getting burned by.
* It blocks proof work. The refund leg of the 2026-08-06 proof was **not**
  executed against live Stripe for this reason: `charge.refunded` is subscribed
  on v2's guest-orders Connect endpoint, and v2's refund handler is the one that
  *does* write `online_orders`. Confirming it would be safe requires reading v2,
  which the standing rule forbids. The refund path was proven at the database
  layer instead.

  Re-confirmed 2026-08-06 from v2's source at
  `apex_v2/supabase/functions/stripe-os-webhook/index.ts:324-339`, and it is
  worse than "a handler that happens to no-op": that branch issues a
  service-role `UPDATE online_orders … WHERE stripe_payment_intent_id = <pi>`
  keyed on the payment intent **alone**. Unlike the `payment_intent.succeeded`
  branch immediately below it, which does check `event.account` against the
  venue's connected account, the refund branch has **no account binding at
  all**. It no-ops today only because a v3 payment intent id happens not to
  exist in v2's table. That is a coincidence of id-space, not a guard.

* Not every event type fans out. `payment_intent.succeeded` is subscribed by
  v3's two endpoints and by **no** v2 endpoint, so a proof driven through
  PaymentIntents touches v2 zero times. `checkout.session.completed` and
  `charge.refunded` are the two that do reach v2. Useful when scoping future
  money-path proofs under the standing v2 rule.
* It couples cutover. Rotating, disabling or reconfiguring the shared platform's
  Connect endpoints at v2 sunset will affect v3 unless they are separated first.

**Accepted correct fix: give v3 its own Stripe platform account**, with its own
`ca_…` application id, its own Connect endpoints and its own signing secrets.
Then v3's connected accounts emit events only to v3. Not done now because it
means re-onboarding every connected account, and v3 has no real venues yet —
which is also precisely why doing it **before** launch is cheap and after launch
is not.

### Found during the 2026-08-06 guest-checkout proof — v3 returns paying guests to **v2**

**OPEN.** `create-guest-payment` builds `success_url` / `cancel_url` from
`returnBase`, which resolves in this order:

1. `restaurant_settings.site_url` for the venue (null for any venue that has not
   set one — i.e. all of them today), then
2. `APEX_APP_URL`, which is **not set** on `fnsonnhumcvxdnyarguv`, then
3. the hardcoded fallback at `supabase/functions/create-guest-payment/index.ts:59`:
   `https://apex-v2-ten.vercel.app`.

So the live Checkout session created during the proof carried
`cancel_url = https://apex-v2-ten.vercel.app/?token=…&paid=0` — observed on the
real event payload in Stripe's delivery record, not inferred. A guest who pays a
**v3** venue is handed back to the **v2** web app, which knows nothing about the
v3 order and cannot show them their confirmation.

This is not a v2 *write* — it is a redirect the guest's own browser follows — but
it is a customer-facing break and one more v2 coupling. `stripe-connect-onboard`
carries the identical fallback at `index.ts:44`.

Not fixed in this pass: the right fix is setting `APEX_APP_URL` /
`APEX_VENUE_SITE_URL` on the project (a secret/config action) and deciding what
v3's guest site actually is — neither of which is a code change, and the second
is a product decision that does not belong in a proof run.

### Still open on the money path

| Item | Status |
|---|---|
| **Square tip refunds** | OPEN (narrowing, recorded). Unchanged: `online_orders_refund_within_charge` caps `refunded_cents` at `total_cents`, which excludes the Square tip, so `refund-order` returns a deliberate 501 for a Square-paid order rather than moving money the ledger cannot represent. A counter refund taken in the Square dashboard IS recorded automatically by `square-webhook`. |
| **Edge-function CONTENT drift is ungated** | OPEN. `check_function_drift.sh` compares deployed **slugs** against repo directories; it cannot see a deployed function whose source differs from the file under review. This pass closed the gap by hand — every money-path function was redeployed from disk, and three of the five came back with an unchanged `ezbr_sha256`, which is positive proof they already matched. A gate that does this continuously does not exist yet. |
| **The `service_role` exemption** | Unchanged and still the largest structural weakness. Every edge function runs as `service_role`, `auth.uid()` is null, and all five guards return early for it. The mitigation stays procedural-made-mechanical: `check_function_writes.py` enforces "an edge function calls an RPC, it does not write a table". The money-path RPCs additionally bracket their writes with `apex.order_payment_rpc` / `apex.order_refund_rpc` so they are legal on the guard's **own** terms and therefore testable under an impersonated role. |

## SECURITY DEFINER functions omit `pg_temp` from search_path (repo-wide)

Found while porting `apex_charge_ai_call` (2026-08-06). The house style across
every migration is `set search_path = public`; the documented hardening shape
for SECURITY DEFINER is `set search_path = public, pg_temp`, which pins the
temp schema LAST. With it omitted, `pg_temp` is implicitly searched FIRST for
relation and type names (never for functions/operators), so a session that can
create a temp table named after a core table could shadow an unqualified
reference inside a definer body.

Why this is recorded as debt rather than an emergency: the only client surface
is PostgREST, which offers no way to run `CREATE TEMP TABLE` — the attack needs
arbitrary-SQL access, at which point far worse is available. Most v3 bodies
also schema-qualify (`public.`) their references, which is immune regardless.
It is still the wrong default: any future surface that executes caller-shaped
SQL (a reporting endpoint, a support tool) inherits the hole silently.

Remediation, when picked up (one migration + one gate, not piecemeal):
1. `alter function ... set search_path = public, pg_temp` for every
   SECURITY DEFINER function (no body recreation needed).
2. A CI assertion over `pg_proc.proconfig` that every `prosecdef` function's
   search_path ends in `pg_temp`, so the next new function cannot regress it.

`apex_charge_ai_call` (20260806190500) is written in the hardened shape
already and is the template.
