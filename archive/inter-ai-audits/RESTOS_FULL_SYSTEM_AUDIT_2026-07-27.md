# restOS — Full System Audit

**Date:** 2026-07-27
**Status:** Complete — 4 blocking defects before first paying pilot
**Repo:** [nicholaswittle/restOS](https://github.com/nicholaswittle/restOS) @ commit `ffd8f5e` (archive snapshot: apex_v2 app + jigsys_site + archived jigsy)
**Related:** [[APEX_V2_AUDIT_2026-07-27]], [[RESTAURANT_OS_BUILD_PLAN]]
**PDF:** ![[Restaurant OS — Full System Audit 2026-07-27.pdf]]

> Full-system companion to the apex_v2 code audit. Reads all 4 design docs, all 6 migrations, all 3 edge functions, and the OS-tier feature code (ordering, call-out, capacity, labor-vs-revenue), plus live competitor research.

---

## 1. The Verdict, Up Front

**Yes — well designed, and yes, it can actually work.** Architecture is sound, strategy is unusually disciplined, and the competitive analysis in the design docs checks out against real 2026 pricing and review data. The "nobody else sells this" claim is substantially true for the independent-restaurant segment.

**One gap matters more than everything else combined:** the flagship feature — "one number tells you if the night made money" — currently only sees **online-order revenue**, a small fraction of actual sales. Until POS integration or a daily revenue entry exists, the labor-vs-revenue dashboard shows a misleading number. Fixable (the plan already defers POS integration deliberately), but it's the difference between a demo and a product an owner trusts.

Four smaller defects need fixing before a paying pilot (Section 4).

---

## 2. How It All Works Together, in Plain Language

### One database, one app, many faces

Everything — schedule, menu, orders, clock punches, tips, log book — lives in **one Supabase (Postgres) database**. Every row is tagged `organization_id`; RLS policies isolate restaurants from each other. Best architectural decision in the project: the OS emerges for free because all data was always co-located.

**One Flutter app** (iOS, Android, web) with a module registry — each feature is a plug-in. What you see depends on **tier** (Free / Pro $25 / OS $99 / Multi $199) and **role** (owner, manager, server, kitchen, readonly). Entitlements are per-module, so a venue can buy one piece à la carte.

### The three pillars

**Pillar 1 — Staff side (what Apex already is).** Employee-first dashboard (next shift, hours, estimated pay + tips, shift notes, one-tap clock-in). Manager scheduling with photo import + AI parse, labor-law guardrails, swaps, time-off. QR wall clock-in + offline punch queue.

**Pillar 2 — Customer side (ordering).** Public menu link via unguessable `public_token`, no login. Menu + modifiers + cart + pickup orders. Staff console: accept/reject, pause, sold-out. Realtime order feed.

**Pillar 3 — OS bridge (the part nobody else has).** Because pillars 1–2 share one database:

- **Labor vs revenue:** punches × rates vs order totals, same night, same screen.
- **Smart capacity:** ordering checks who's clocked in; understaffed → longer-wait warning or auto-pause; restaffed → auto-resume.
- **No-show call-out engine:** "can't make it" → edge function finds eligible coworkers (role match, not on time-off, not already working) → SMS + push fan-out → first claim fills → capacity adjusts if unfilled.

Supporting all three: **notification router** (push → Twilio SMS fallback, quiet hours, per-user prefs) — built because unreliable notifications are the #1 competitor complaint.

### Built vs. designed vs. deferred

| Status | Items |
|---|---|
| **Built** | All of Pillar 1 (16 features), ordering platform, labor-vs-revenue dashboard, call-out engine + SMS fan-out, capacity engine, tip pools, log book, chat, notification routing, 3 edge functions, 6 migrations, demo mode, 28 passing tests |
| **Designed, deferred** | POS integration (6 tables incl. idempotency + dead-letter queue), ML prep-time (data capture only), floor plan, multi-location |
| **Not yet designed** | Online payment (orders are `payment_mode: 'manual'` — pay at pickup), payroll export beyond CSV |

---

## 3. Is It Well Designed?

Yes, specifically:

1. **The build order is the design.** Every phase ships something standalone before any OS connection exists. Docs explicitly analyze why past projects died and organize to avoid it.
2. **One backend, org-scoped from day one** — smart capacity is ~230 lines because the data was already co-located.
3. **RLS as a first-class design surface** — helper functions, idempotent migrations with rationale, partial unique index (one open call-out per shift), unique index against double tip payouts, confirmation-token pattern so anon customers never need SELECT on orders.
4. **Careful money math** — largest-remainder tip split (cents always reconcile), full timestamps never clock-face strings, integer cents.
5. **Multi-vertical thinking correctly restrained** — "a vertical is a pack, not a fork," but refuse to build vertical two until vertical one pays.
6. **Demo mode at the HTTP layer** — sales demos run the real screens.

---

## 4. Would It Actually Work? The Honest Gaps

### 4.1 The revenue denominator problem (the big one)

Labor-vs-revenue computes `revenue` **only from `online_orders`**. Dine-in/bar sales flow through the POS and never touch this database → labor-% reads ~120% instead of ~20%. An owner who sees that once stops trusting the dashboard.

**Fixes, cheapest first:** (a) manual daily revenue entry ("what did the POS say tonight?") — one column, one tap, makes the number real immediately; (b) later, Phase 4 POS integration (Square first — free API, common in independents; Toast charges $50/mo for API access). Ship the manual stopgap **with** the dashboard.

### 4.2 Public checkout likely broken as written (verify before pilot)

Anon customers insert orders directly. The `order_items` insert policy checks `exists (select 1 from online_orders ...)`, but Postgres applies `online_orders` RLS **inside** that subquery — anon has no SELECT policy → EXISTS always false → every order_items insert rejected. **A customer can create an empty order but cannot add food to it.** Fix with a `place_order` RPC (security definer) inserting order + items + modifiers atomically — also fixes 4.3.

### 4.3 Order totals computed by the customer

Cart sends its own `subtotal_cents`/`total_cents`; server trusts them. Payment is manual (pay at pickup), so a tampered client orders a $24 tray for $0.01, or order-bombs the kitchen — the anon insert policy only checks "not paused." The RPC should recompute prices from menu tables server-side, enforce pickup windows, rate-limit.

### 4.4 The "killer feature" isn't wired up

`CapacityEngine.autoAdjust()` — the code that actually auto-pauses ordering — **is never called anywhere.** The capacity screen calls `check()` only. And the engine counts **all** clocked-in staff (servers too) against capacity, while the design rule is "orders per **cook**" — 4 servers + 0 cooks reads as full capacity. Fixes: trigger autoAdjust server-side (scheduled edge function or DB webhook on `time_entries`, not a screen someone must open), and count kitchen roles only.

### 4.5 Carried over from the apex_v2 audit

Still open in this snapshot: staff keyed by display name, clock-QR forgeable by photo all day, tip publish can wedge mid-transaction, offline queue can jam on one bad row, `messages`/`has_role` RLS issues. All touch money or identity — fix before payroll/payouts get real. Detail: [[APEX_V2_AUDIT_2026-07-27]].

### 4.6 Smaller notes

- **Payments manual-only** — fine for pilot; label it in the pitch, don't surprise.
- **SMS costs scale** with call-out fan-out (~$0.008/msg — trivial at 1 venue, worth a per-org cap at 50).
- **Edge functions now in repo** (earlier audit flagged their absence) — good; keep versioned with migrations.
- **`restaurants` public SELECT exposes every venue's row** — low harm; scope to lookup-by-token RPC.

---

## 5. The Competition, Checked Against Reality

Design docs' claims verified against 2026 pricing trackers and user reviews. Claims hold up well.

### Pricing reality (2026, verified)

| Product | Real cost, single independent venue | What you get |
|---|---|---|
| **7shifts** | Free tier; Essentials ~$40–45/loc/mo; Pro ~$80–90; Premium ~$135–150 | Scheduling + time clock. **Tips $49.99/mo add-on; Log Book $14.99/mo add-on** on lower tiers |
| **Homebase** | Free (≤10 emp); Essentials $24/loc/mo; Plus $56; payroll +$39 + $6/emp | General SMB — no restaurant tip pooling or POS-driven labor forecasting |
| **Deputy** | ~$5–9 **per user**/mo → $100–180/mo for 20 staff | Multi-industry; per-user pricing punishes restaurant headcount |
| **When I Work** | ~$5 **per user**/mo | Generic; no restaurant features |
| **Toast** | ~$1,000+/mo with hardware lock-in; API access $50/mo | Enterprise full stack |
| **Square** | POS + Online + Appointments are **separate products that don't talk** | Pieces exist, connection doesn't |
| **restOS Pro / OS** | **$25 / $99 flat** | Scheduling + clock + tips + log book + labor cost (Pro); + ordering + smart capacity + call-out + labor-vs-revenue (OS) |

**A venue wanting 7shifts Essentials + tips + log book pays ~$105–125/mo for what restOS Pro does at $25.**

*Sources: restauranttools.ai (verified 2026-07-02), checkthat.ai (2026-03), stackscored.com (2026-04), shifton.com, ontheclock.com (2026-02). 7shifts rebranded plans in 2026 and trackers disagree on exact tier contents — confirm live pricing before quoting in sales material.*

### Their pain points vs. restOS features

| Documented competitor pain | restOS answer | Assessment |
|---|---|---|
| **Unreliable notifications** — #1 complaint across 7shifts ("shift pool notifications don't always work"), Deputy ("staff turning up to unconfirmed shifts"), Homebase ("do not receive notifications") | Push → SMS fallback router, delivery logging, quiet hours, per-user prefs | Targets the right wound. **Strongest marketing angle.** |
| **Nickel-and-diming** — add-ons, price increases, per-user pricing | Flat $25/$99, everything included | Real and provable. Put the math on one slide. |
| **Disconnected systems** — Square's pieces don't integrate; 7shifts can't do ordering; ordering cos can't do scheduling | One database; capacity/call-out only possible *because* it's one system | Structurally true. 7shifts would need to build or buy an ordering company to copy smart capacity. |
| **Labor forecasting at high tiers** — 7shifts Pro/Premium ($80–150) + POS API fees | Labor-vs-revenue at $99 *including ordering* | True — but competitors read **real POS sales**; restOS reads online orders only until 4.1 is fixed. Their expensive feature is currently more accurate than your cheap one. |
| **Clunky UX** — Deputy/7shifts mobile complaints, separate 7punches app for clock-in | Employee-first, 3-tap-max, one-screen dashboard | Plausible, unproven until daily real-world use. |
| **Auto-scheduling ignores availability** (documented Deputy limitation) | Advisory guardrails before publish (overtime, minor hours, consecutive days) | Arguably better fit for 15-person restaurants where the manager knows everyone. |

### Where the moat claims are overconfident

- "No competitor does this": true today for SMB tools, but 7shifts integrates 65+ POS systems and could approximate capacity-throttling via a partner. The durable moat isn't the feature — it's **owning both sides of the data**, so the version is real-time and free. Say that instead.
- Free tier competes with legitimately good free tiers (7shifts Comp, Homebase). The wedge is the **pilot relationship + upgrade path**, not the free tier itself.

---

## 6. What It Needs — Prioritized

**Before the first paying pilot (blocking):**
1. Manual daily revenue entry so labor-% is real (half a day, fixes 4.1)
2. `place_order` RPC: atomic order+items, server-recomputed totals, rate limiting (fixes 4.2, 4.3)
3. Wire `autoAdjust()` to a server-side trigger; count kitchen roles only (fixes 4.4)
4. Test public checkout end-to-end against a real Supabase project, not demo mode

**Before scale:**
5. apex_v2 carryovers: staff `user_id` migration, tip transaction, QR rotation, offline-queue hardening
6. Online payment (Stripe) alongside manual
7. Square POS integration (pull Phase 4 forward once pilot proves value — makes the OS number automatic)
8. CI: analyze + tests on every PR

**Later (plan already says this — correctly):**
9. Prep-time ML from captured snapshots
10. Multi-location tier
11. Vertical two — only when vertical one pays

---

## 7. Bottom Line

Better design than most funded startups produce: one database, RLS everywhere, phased build order, honest docs, OS features competitors can't replicate without an acquisition. Pricing ($25/$99 flat vs. a real ~$105–125/mo 7shifts equivalent) is aggressive and defensible.

It would work. The risks are not architectural — they're the four pre-pilot defects plus the normal one-person-company support risk. Fix the four, run the Jigsy's pilot, measure the labor-cost delta the docs promise to measure, and the pitch writes itself with real numbers.
