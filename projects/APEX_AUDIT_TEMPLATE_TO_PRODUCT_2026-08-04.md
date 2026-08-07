---
title: "Audit — feat/template-to-product (ChatGPT branch)"
tags: [apex, audit, security, services, merge]
date: 2026-08-04
branch: feat/template-to-product
head: f313b6d
---

# Audit — `feat/template-to-product` @ `f313b6d`

Five commits branched from `7c769e3`. **48 files, +3,707 / −539**, 12 new
migrations, 4 new edge functions, 5 new test files.

Verified by running it, not reading it: worktree checkout, `flutter analyze`
clean, **180 tests pass** (mine: 166), `site/` builds on Next 16.

---

## Grade: B+ as a body of work · **D on shipping risk**

The code is good — better than mine in a couple of places. The problem is
that **none of it is deployed, and it cannot be deployed in pieces.**

| Dimension | Grade | Why |
|---|---|---|
| Security posture | **A−** | The anon allowlist is the best idea on the branch |
| Architecture | **A−** | Client writes replaced by guarded RPCs; upload intents replace an open anon storage policy |
| Test discipline | **A−** | +14 tests, including source-grep pins that fail if a screen goes back to raw table writes |
| Correctness vs live DB | **D** | 12 migrations unapplied; branch is inert and partially self-contradictory until they land |
| Coordination | **C** | Branched from `7c769e3`, so it silently reverts five fixes made after |

---

## What it does well

**The anon allowlist (`20260915000400`).** Enumerates every SECURITY DEFINER
function anon can execute, revokes everything not on a 13-entry list, and
`raise exception` if any unexpected one survives. That is a standing guard,
not a one-time sweep — the next function that accidentally grants anon fails
the migration. Genuinely better than anything I wrote in this area.

**Client writes replaced by guarded RPCs.** `apex_transition_request`,
`apex_save_request_quote`, `apex_schedule_request`. The inbox and crew sheet
no longer write `requests`/`shifts` directly, so status transitions are
server-validated rather than "whatever the client sent". The accompanying
pin test greps the source and fails if a screen reverts to
`from('shifts').insert` — a cheap, durable way to hold an architectural line.

**Photo uploads moved to signed intents.** My version left an anon INSERT
policy on the storage bucket, fenced by a two-hour window. Theirs issues a
server-side upload intent instead. Strictly tighter, and it removes the
open-bucket surface I flagged as the weak point of my own design.

**`submit_public_request` moved from an anon RPC to an edge function** —
correctly accompanied by removing it from the allowlist *and* switching
`site/lib/services.ts` to `functions.invoke("submit-public-request")`.
Coherent end to end; I checked both halves.

**Permission tightening on payments:** `create-request-payment` now requires
`has_role(manager)` rather than `is_member`. Right call — any member could
previously mint a payment link.

**Next 15 → 16** with the `middleware.ts` → `proxy.ts` rename handled. Builds
clean, all six routes present including `/hicpa` and `/pricing`.

---

## What is wrong

### 1. Nothing is applied — and it must land atomically 🔴

Checked live: `service_catalog_items`, `request_photo_upload_intents`, the
membership-authority functions — **none exist**. Latest applied migration is
still `20260901000000`. The four new edge functions are not deployed.

The ordering is not optional:

- **Migrations without edge functions** → `submit_public_request` is revoked
  from anon while `submit-public-request` does not exist. **The public quote
  form stops working entirely.**
- **Edge functions without migrations** → they reference tables that do not
  exist.
- **App without both** → the inbox calls `apex_transition_request`, which is
  not there.

This is a single-window change: 12 migrations, 4 function deploys, one site
deploy, one app build, in that order. Given the migration ledger is already
drifted (`db push` is unsafe), each migration goes individually.

### 2. It reverts five fixes made after it branched 🔴

Branch point is `7c769e3`; `370cb08` is not an ancestor. Absent from this
branch:

| Fix | State on this branch |
|---|---|
| Deposit cap `33` → `100/3` | **Still `33`** — $825 instead of $833.33 on a $2,500 quote |
| "Customer accepted" gating + confirm | Ungated, still a full-width mis-tap |
| Payment link card (Copy/Open) | Still `SelectableText` of the raw ~200-char URL |
| Quote form shows all services | Still filters to `cta === "request"` — 2 of Bradley's 4 |
| Photos reachable | Still `questions.length > 0 ? "qualify" : "done"` — step 2 skipped, photos unreachable |

The deposit cap one matters most: it is money, it is statutory, and it now
has a test on my branch that this branch does not have.

### 3. Merge surface is small but both files are load-bearing

Only two files were touched by both sides since the split:

- `lib/features/requests/request_inbox.dart` — theirs: RPC mutations; mine:
  accept gating + link card. Different regions, mergeable with care.
- `supabase/functions/create-request-payment/index.ts` — theirs:
  `has_role(manager)`; mine: cap constant + return URL. **Both are correct
  and both must survive.**

---

## Recommended merge order

1. Rebase `feat/template-to-product` onto `370cb08` (or merge `370cb08` in).
2. Resolve the two files keeping **both** sides — manager check *and* the
   `100/3` cap.
3. Re-run: analyze, 180+166 union of tests, site build.
4. Then the atomic window: 12 migrations individually → 4 edge functions →
   site deploy → app build 12.
5. Re-verify the public form end to end afterwards, because step 4 changes
   how submission works.

**Do not deploy either branch's migrations before the merge.** The five
reverted fixes include a live money bug.

---

Related: [[projects/Apex v2 — Restaurant OS Build]] · [[projects/APEX_V2_TEMPLATE_TO_PRODUCT_GAP_MAP_2026-07-31]] · [[projects/APEX_MASTER_PLAN]] · [[hot]] · [[NOW]] · [[index]]
