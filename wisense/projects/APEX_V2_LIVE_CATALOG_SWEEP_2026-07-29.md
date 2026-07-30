---
type: audit
title: "Apex v2 — Live Catalog Sweep 2026-07-29"
tags: [audit, security, rls, apex-v2, supabase, privilege-escalation, drift]
date: 2026-07-29
status: completed
target_project: "apex_v2"
supabase_ref: "pqkremkwfkudrhtxasdj"
git_head: "db7d9ed"
---

# Apex v2 — Live Catalog Sweep

**Not an audit report.** This is what came out of querying the running database
directly, after the Supabase connector was authorized. Everything here was
invisible to the four audits that preceded it, for one reason:

> **None of it exists in the repository.** Three audits read the migrations and
> found nothing wrong, because the migrations do not contain these objects. The
> fourth trusted a summary table. The repo describes what was *meant* to be
> built; the catalog describes what is *running*.

The prompt for the sweep was `shifts staff claim open` — audit 3's CRITICAL,
which turned out to be in no migration file. That one orphan implied others.

---

## Findings, in order of severity

### 1. 🔴 Anyone could become Owner of any venue

`apex_grant_membership(p_user uuid, p_org uuid, p_role text)` is
`SECURITY DEFINER`, inserts directly into `organization_members`, and **performs
no authorization check of any kind**. `EXECUTE` was granted to `anon`.

```
select apex_grant_membership('<uid>', '<any org id>', 'Owner');
```

Callable from a browser console with the publishable anon key that ships in the
web build. Every other control in the system — manager gates, `hourly_rate`
column grants, the shift write lockdown, pay privacy — sits behind a role this
function handed out for free.

It was written as an internal helper for `apex_redeem_invite`, which is itself
`SECURITY DEFINER` and therefore calls it as the definer regardless of grants.
The grant was never needed. **Fixed:** revoked from `public, anon, authenticated`.
Verified afterwards that both functions are owned by `postgres` and the owner
retains EXECUTE, so invite redemption still works.

### 2. 🔴 Every pending invite in the database was rewritable

Policy `org invite consume` on `organization_invites`:
`UPDATE`, `USING (code is not null and used_at is null and not expired)`,
`WITH CHECK (true)`.

The `USING` clause never scoped to the caller's organization and the `WITH CHECK`
accepted any resulting row, so a single statement could rewrite every unused
invite in the system — set `role` to Owner, set a `code` of your choosing — and
then redeem it.

Nothing used the policy: the client only reads invites, and redemption goes
through `apex_redeem_invite`, which bypasses RLS as definer. Pure attack surface.
**Fixed:** dropped.

### 3. 🔴 Paid tiers settable by anyone

`apex_set_subscription_status(uuid, text)` — same shape as #1: no caller check,
`EXECUTE` granted to `anon`. This is the tier-enforcement gap in its most direct
form; it does not even require the client. **Fixed:** revoked.

### 4. 🟠 The `org all` pattern was on six tables, not one

`shifts org all` was fixed in `20260802210000`. The identical policy —
`for all using (coalesce(organization_id, apex_current_org_id()) =
apex_current_org_id())`, membership as the entire check — was live on five more:

`availability`, `sidework`, `swaps`, `time_entries`, `time_off_requests`

**`time_entries` is the one that matters.** It holds clock-in and clock-out
records: what people are paid from, and what a wage dispute is decided on. Any
employee could edit their own hours upward or delete a colleague's record, with
nothing to show it happened.

**Fixed** in `20260802230000`: dropped all five, added manager-write policies on
`has_role`. Dropping alone would have broken managers — the surviving narrow
policies are Owner-only through `apex_current_role()`, the pre-membership helper
that reads a person's *primary* venue and so fails an owner at their second one.
Verified beforehand that no client code deletes from any of these tables and
that swap approval is already a manager action.

### 5. 🟠 Billing state readable without an account

`v_paying_venues` is a `SECURITY DEFINER` view — RLS on the underlying tables
does not apply — with `SELECT` granted to `anon`. **Fixed:** revoked.

### 6. 🟡 `messages_update` had no `WITH CHECK`

`USING` allowed updating your own message; nothing constrained what it became,
so `user_id` could be set to a colleague's. Their name, your words, in the venue
chat. **Fixed:** added a `WITH CHECK`.

### 7. 🟡 Four more internal helpers exposed to `anon`

`apex_provision_venue_for_org`, plus trigger functions
`apex_sync_profile_membership`, `apex_organizations_after_insert`,
`apex_clear_active_org`, and legacy v1 `consume_invite_code`. **Fixed:** revoked.

### 8. 🔵 Two definer functions with mutable `search_path`

`apex_slug_public_token`, `apex_allowed_job_roles`. **Fixed:** pinned to
`public`.

---

## Verified after the fixes

All confirmed by catalog query, not by assumption:

- All seven revoked helpers: `anon = false`, `authenticated = false`.
- `apex_redeem_invite` and `place_order` still public — intended, they are the
  signup and guest-ordering entry points.
- `org invite consume` gone; `v_paying_venues` not selectable by `anon`;
  both `search_path` values pinned.
- Six `org all` policies gone; manager-write policies present on all five tables;
  `messages_update` now constrains writes.
- `auth.users` has exactly one trigger (`apex_handle_new_user`), so the orphan
  `handle_new_user` is dead code, not a second signup path.

## Still open

- **21 functions in production that no migration in the repo creates.** Named in
  the session; most are ordinary v2 functions applied out-of-band, but they are
  unowned by definition.
- **Two money-guard triggers on `online_orders`** (`apex_guard_order_money` and
  `apex_protect_online_order_money`), both `BEFORE UPDATE`. Redundant, possibly
  conflicting. Not investigated.
- **Tier enforcement** remains UI-only for Pro features (audit 4, finding 2).
  Finding #3 above was one route around it; the RLS policies on `tip_pools` and
  friends are another.
- **`create-guest-payment` and `stripe-os-webhook`** rated safe by audit 4, never
  independently verified.
- **Migration history drift** — the root cause of this entire class of problem.
  `supabase migration list` shows every local migration as un-applied remotely.
  Until that is reconciled, the repo cannot answer "what is in production", and
  reviewing the repo will keep producing false assurance.

## The lesson

Four audits, ~20 findings, and the three most severe issues in the system were
all invisible to every one of them. Not because the auditors were careless —
because they were reading the wrong artifact. Reviewing migrations tells you
about intent. Only the catalog tells you about production.

See also [[APEX_V2_AUDIT3_2026-07-29]], [[APEX_V2_AUDIT4_2026-07-29]].
