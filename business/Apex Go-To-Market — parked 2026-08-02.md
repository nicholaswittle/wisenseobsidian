---
title: Apex Go-To-Market — parked
tags: [business, apex, pricing, parked]
updated: 2026-08-02
status: parked — revisit when there is a second customer
---

# Apex go-to-market — parked 2026-08-02

Worked through with Claude and Antigravity on 2026-08-02. Parked deliberately:
the strategy is sound, and none of it can be validated without a second
customer. Revisit then, not before.

## The model as it stands

| Tier | What | Price |
|---|---|---|
| Free wedge | Generic template site on `apex-venue-site.vercel.app` | $0 forever |
| One-time | "Un-generic" site — their photos, palette, copy, theme | $499–799 |
| Recurring | Restaurant OS + 1.5% of card orders | $99/mo + 1.5% |

**The reframe that made it work:** sell a *website*, not "setup". Once the site
is free forever, "I ran an importer and did a Zoom call" is a weak thing to
invoice for. A visibly better site is something an owner can see and show off.
Web agencies charge $2–5k for the same thing.

**100% remote is a constraint, not a preference.** Full-time PSP trooper, five
kids. No in-person onboarding, ever. Menu import by AI, QR cards shipped direct
from Vistaprint, 15-minute onboarding call, Loom training video.

## The correction that matters most

The paid site must NOT be bespoke code. `jigsys_site` — the model for
"un-generic" — is **1,663 lines of hand-written HTML plus 1,268 of JS**. Twenty
venues that way is twenty codebases maintained by one person with a day job.

**Instead: one renderer, five parameterised themes** — brewpub, pizza, diner,
café, upscale. The renderer already takes `palette`, `hero_photo_urls`,
`sections`, `highlights`, `vertical`, so the machinery exists; only one theme
does. That converts the $499 deliverable from days of code into ~90 minutes of
configuration, and scales to 200 venues without a new line.

**This is the one item here with an engineering consequence rather than a
business one.** Estimated 2–4 days. It also fixes the free tier's real weakness:
right now every free site looks identical, which advertises Apex badly.

## Realities to hold onto

- **Onboarding is 3–4 hours for venues #2–5**, not 30 minutes. Fuzzy menu OCR,
  owners who don't know their tax rate, Stripe bank details. Falls toward ~90
  minutes by venue #10. Still a good rate for $499 — just not the happy path.
- **The 1.5% is paid by the guest, not the venue.** Verified against real
  orders: `subtotal + tax + fee + tip = total`, and the fee comes off the
  venue's side of the Direct Charge. On a $24.50 order it is 39¢. Raising it
  raises the *guest's* price, which is a conversion question, not a
  venue-tolerance one.
- **Never take a cut of tips.** The fee is computed on subtotal + tax only. One
  test order had a $100 tip and Apex correctly took 39¢.
- **Support boundary, in writing, before invoicing:** *"…30 days of launch
  support. Ongoing support is provided via in-app diagnostic tools and email."*
  Do **not** contractually promise 24/7 AI support — that agent became
  reachable from the dashboard on 2026-08-02, has three allowlisted actions, and
  has never been used by a real venue manager.
- **PSP secondary employment.** Verify the outside-business disclosure is filed
  and approved before invoicing a commercial client. Not assumed — checked.

## The thing that is actually blocking everything

All of the above is about how to *serve* venue #2. Nothing addresses how to
*find* venue #2.

**The referral mechanism is decorative.** `ReferralInviteTile` generates a code,
sits on the launch checklist — a screen visited once, during setup — and there
is **no referrals table in the database at all**. Nothing records whether a
referral ever converted.

Three things worth doing when this is unparked:
1. **Ask Emily directly** which two owners she would introduce. She is the
   channel: a brewpub owner in Enola knows the pizza place and the diner, and
   she is the only person who can say "I use this."
2. **Move the referral tile** off the launch checklist to somewhere a happy
   owner sees it — the dashboard, after a good first week.
3. **Add the table.** Two hours, and it turns a decoration into a measurable
   channel.

## Scoreboard at parking

| | |
|---|---|
| Paying customers | 1 (Jigsy's — pilot, unpriced) |
| Real guest orders | ~0 (the 22 paid are our own tests) |
| Referrals tracked | none — no table |
| Themes built | 1 of 5 |
| $499 price validated | no |

**Unpark when:** a second venue is in conversation, or the five themes are built
— whichever comes first.

Related: [[NOW]], [[Apex v2 — Restaurant OS Build]], [[Client Acquisition Strategy]]
