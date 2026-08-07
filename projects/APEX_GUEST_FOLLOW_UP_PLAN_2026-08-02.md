---
title: Apex Guest Follow-Up — Plan
tags: [apex, sms, reviews, compliance, growth, build-doc]
date: 2026-08-02
---

# Apex Guest Follow-Up — Plan

Post-pickup SMS that asks every guest, identically, for a Google review or a
private note to the owner. Symmetric by design: the branch is what would make
it illegal, and the branch is worth almost nothing.

Related: [[projects/Apex v2 — Restaurant OS Build]] · [[DECISIONS]], [[projects/JIGSYS_PILOT_LAUNCH_STRATEGY_2026-07-31]],
[[projects/APEX_AI_SUPPORT_AGENT_PLAN_2026-08-01]], [[NOW]]

---

## 0. Provenance — this is new, not recovered

The feature entered the record on 2026-08-02 inside an external
"Master Executive Audit" as **"AI 5-Star Review Magnet & Feedback Shield,"**
described as sentiment-routing and labelled *Compliance-Clean* with no analysis
behind the label.

It has **no prior vault or code provenance.** Searched 2026-08-02: zero hits for
`google review`, `review link`, `review_url`, `write review` across `lib/`,
`supabase/functions/`, `supabase/migrations/`. The one vault hit —
`archive/stale-plans/AI Side Hustles Remote Only 2026-07-21` — is **ReviewGuard**,
an unrelated standalone $149/mo SaaS that drafts *responses to reviews that
already exist*. It never solicits and never branches, so it shares no
regulatory surface with this.

Recording that because the audit presented the feature as existing scope. It
was not. Treat this note as its origin.

## 1. The decision

**Ship the symmetric version. Never ship the branching version.**

Every guest gets one message with both options at equal weight. Apex never
learns sentiment before choosing what to show, because learning it first is the
whole offence.

This is not a workaround. Symmetric solicitation is not evading the rule — it
*is* the rule. Framing it as a loophole invites drift toward the edge; framing
it as the design keeps it clean.

## 2. Why the branching version is off the table

**Review gating** — predicting or asking sentiment, then routing happy guests to
a public review and unhappy ones to a private channel — produces a public review
corpus that misrepresents actual guest experience.

Two enforcement paths, and the smaller one is the real risk:

| | Exposure | Realistic? |
|---|---|---|
| **FTC** | Section 5 deception. Reference case **Fashion Nova, 2022, $4.2M**, for suppressing negative reviews. The 2024 rule (16 CFR Part 465) is *not* the primary hook — it targets fake reviews, bought reviews, insider reviews, and suppression by threats/intimidation. Gating runs through §5. | Low-probability tail risk at our size |
| **Google** | Platform policy prohibits gating outright. Enforced by removing reviews or flagging the profile — no case, no notice, no meaningful appeal. | **This is the live one** |

The loss that actually lands is a venue's review history disappearing, on our
recommendation, at our only customer. An earlier note in this vault overstated
the FTC rule as the primary exposure; corrected here.

Same doctrine as [[DECISIONS]] 2026-08-02 on Payroll Lite: where the honest
artifact and the safe artifact are the same artifact, build that one and stop.

## 3. The four boundaries

Three of these are easy to cross by accident.

### 3.1 Who receives it — the one that catches people

**Send to 100% of completed orders, automatically, with no human in the loop.**

Gating reintroduced one step upstream is still gating. All of these rebuild it:

- a manager choosing who gets the text
- skipping guests who complained, or whose ticket was comped
- filtering by order value, repeat-customer status, or anything correlated
  with satisfaction

Automatic-and-universal is most of the compliance story in a single rule, and
it is the one to write a test for.

### 3.2 No sentiment question first

"How was everything?" followed by different screens is textbook gating even
when both destinations exist. **Do not ask.** No star prompt, no thumbs, no
"rate your experience" step, no LLM scoring the order history to decide the
message.

### 3.3 Symmetric presentation

Two options, equal visual weight, neutral copy. A large green *Leave a Google
Review* beside grey 11px *message the owner* is steering, and the design file
becomes the evidence of intent.

### 3.4 No incentives

Anything offered in exchange for a review is prohibited by Google regardless of
sentiment, and conditioning it on positivity is an independent FTC problem.
No discount, no free item, no loyalty points.

## 4. The names are part of the compliance

**"5-Star Review Magnet"** and **"Feedback Shield"** must not survive into code,
UI, marketing, or commit messages.

Regulators and plaintiffs read marketing copy as evidence of intent. A feature
whose own name announces that it shields the venue from bad reviews has
documented its purpose in advance. This costs nothing to fix and is the single
largest liability in the concept.

**Ship name: Guest Follow-Up.** Table name, function name, screen name, sales
deck — all of it.

## 5. Spec

```
Thanks for your order at Jigsy's! We'd love to hear how it went.
Leave a Google review: <link>
Message the owner directly: <link>
Reply STOP to opt out.
```

- Sent ~20 minutes after pickup, on every completed order, no exceptions
- Both links plain text, equal prominence, neither pre-qualified
- Private note routes to the owner's existing notification path
- Google link is the venue's own review URL, stored per-org (not hardcoded —
  same rule as the wage constants in the payroll plan)

## 6. Prerequisite: this is marketing SMS, not transactional

The blocker on shipping is paperwork, not code.

`supabase/functions/notify-order-event/index.ts` already sends guest SMS on
Accept / Reject / Ready via Twilio, normalizes `customer.phone`, and carries
`Reply STOP to opt out`. **That consent basis does not extend to this.** A guest
gave a number to be told their food was ready — that is transactional. A review
request twenty minutes later is marketing.

Required before first send:

1. **Express written consent at checkout** — a real unchecked box with its own
   language about marketing/follow-up messages. Not buried in terms, not
   pre-ticked, not bundled with the order-status consent.
2. **A2P 10DLC campaign registration** for the marketing use case. The existing
   transactional registration does not cover it.
3. STOP/HELP handling verified against the marketing campaign specifically.

## 7. Why the compliant version is also the better product

Almost nothing is lost by going symmetric. At a venue guests already like,
asking *everyone* skews positive on its own — the volume lift is where nearly
all the value sits, and symmetric keeps it. What is given up is suppressing the
genuinely bad experience, and 200 reviews at 4.6 outsells 40 at 5.0.

The private-note path also has standalone worth: it routes a complaint to the
owner that would otherwise have gone public or gone nowhere. That is the part
worth building, and it survives untouched.

## 8. Not in scope

- Sentiment analysis of any kind, at any point in the flow
- Selective sending on any criterion
- Incentives
- AI-drafted *responses* to inbound reviews — that is ReviewGuard, a separate
  product concept, archived and unrelated
- Anything that reads or writes a venue's Google Business Profile

## 9. Sequencing

**Not on the 14-day launch path.** The consent + 10DLC work is a week of
paperwork with no code, and the current runway already carries an unshipped app
icon, an untested clock-in, no custom SMTP, and zero money/auth tests. Start
the 10DLC registration early because it queues; build the feature after
Jigsy's is live and stable.
