---
title: Apex v3 — 2026-08-07 build session
tags: [apex, apex_v3, build, session, services, quotes, operations, ui]
updated: 2026-08-07
---

# Apex v3 — 2026-08-07

The day v3 stopped being a rebuild and started being a product with an
opinion. Cold-user testing on the onboarding, a strategic argument about
whether to port v2 wholesale, a super-admin fleet console, and then the
services pack: schema, eleven operations, and the quote screen.

Related: [[APEX_V3_SESSION_2026-08-06]] · [[APEX_V3_OPEN_ITEMS_2026-08-06]] ·
[[APEX_COLD_USER_TEST_AND_TODDLER_BAR_2026-08-06]] ·
[[APEX_SERVICES_BUILD_PLAN_CANONICAL]] · [[NOW]] · [[DECISIONS]]

---

## What we are doing

**Porting v2's product knowledge into v3's architecture.** Not cloning v2's
code, and not writing v3 from a blank page. Nick's words: *"we aren't cloning
v2 onto v3 wiring, we are improving based on what we've learned."*

The split that settles every argument about this:

| v2 is the authority on | v3 is the authority on |
|---|---|
| What to build, and what it costs to get wrong | How it is wired |
| Wording that worked on real users | Authorization, RLS, money guards |
| Legal and carrier constraints learned the hard way | The operation contract (D26/D28) |
| Which screens confused people | Audit trail and CI gates |

Kept from v3 by Nick's explicit call: **the onboarding** (*"no I really like
this onboarding"*), **the colour scheme**, and **the schedule** (*"I like the
schedule mode on v3 better — don't change anything we already built in v3
until I review it side by side"*).

## How we are doing it

1. **Read v2's version of a thing before writing v3's.** Every time. This is
   rule 10 and it has paid for itself repeatedly — see the findings below.
2. **Fix each phase as it is built**, rather than accumulating a remediation
   list. Drift is cheaper to prevent than to audit out.
3. **Red-then-green, live.** Every gate is watched failing before it passes.
   A check that has only ever been green has not been tested.
4. **The toddler bar governs the UI.** Zero-tech-ability users must succeed
   unaided: one named verb per screen, no jargon, no blank boxes, no dead
   ends.
5. **Trade-neutral is a premise, not a preference.** Landscaping was the
   example on Nick's screen; it is not the target. Any noun that only makes
   sense outdoors is a bug.

---

## The strategic question, answered

*"How do we feel about bringing v2 into v3 instead of making it scratch?"*

**As a source of requirements, v2 is the best asset the project has. As a
source of code, it would have hurt us three times in one hour.** Both halves
were demonstrated on the same table:

Would have hurt us:

- v2's `mode` column would have ported straight across and **forced every
  services business to be either quote-only or fixed-price**. Nick would have
  hit it the first time a landscaper wanted "$60 mowing" beside "land
  clearing, depends."
- v2's deposit cap shipped `33` where the statute says one third —
  **$825 collected on a $2,500 job instead of $833.33** — and its tests
  passed the whole time because they were handed the right number by hand.
- `draft-quote-from-photos` is keyed to `p_restaurant_id` for businesses with
  no restaurant.

Could never have been invented from scratch:

- **A2P 10DLC consent**, three columns, captured at the form or never. From
  scratch we would have discovered it when a carrier blocked the texts and
  the fix required re-contacting every customer.
- **Town at intake, street later** — a conversion finding, not a derivation.
- **Repeat customers matched on phone number.**
- **The labour screens refusing to show a percentage they cannot compute** —
  better than what v3 shipped, which was a hardcoded 22.4%.

**No third audit.** `apex_v2/docs/` already holds the full scorecard, a
payments/ordering security audit and a Fable audit, and those produced the v3
plan. What they did not cover — what the SCREENS actually do — is what Nick's
screenshots filled in, now written down in `apex_v3/docs/v2_module_inventory.md`.

Found along the way: **`PAYROLL_LITE_SCOPE_2026-08-02.md`**. Payroll was
scoped and never built. When it comes up, that doc is the starting point, not
a blank page.

---

## The one place v3's UI already beats v2

Nick, on the restaurant home with everything unlocked: *"when Emily is logged
into her account everything is everywhere because of the online ordering and
everything."*

That screen has **one** operational item — "1 order awaiting payment" — under
three setup cards: Launch your website, a four-step *How online ordering
works* explainer, and Flagship setup in progress. The explainer is still
there after all four steps are ticked and the link is live.

Two failures, the second structural:

1. **Setup content is permanent furniture.** It taught something once and
   never left.
2. **The home grows a card per unlocked module.** The services home reads
   cleaner mostly because that account has LESS switched on — not because it
   was designed better. Unlocking features makes v2's home worse.

v3's D31 already caps the hub at three destinations and forbids a scrolling
list of every feature. **Keep it.** When porting v2's home, take NEEDS YOU
NOW and leave the accumulation.

---

## Built today

### Super-admin fleet console
Nick's ask: *"super admin is only for me so I can turn on/off specific
features within the app and see where an issue may lie inside a business."*
`apex_admin_fleet` (read), `apex_admin_set_tier`, `apex_admin_set_module` —
the last with three states, on/off/**null = inherit the plan**.

### Entitled is not the same as built
"What you have" was ticking Payroll export, Time clock, Team chat and Log
book for Pro businesses — none of which exist in v3, and payroll never
existed in v2 either. A tick beside a feature is a claim about value
received, on the screen that names what somebody pays for. Now split: *what
you have* vs *in your plan, not built yet*.

### The services pack — schema
Five tables, live. `service_offerings`, `service_requests`,
`service_request_items`, `request_quotes`, `request_ai_runs`.

**The structural departure from v2:** price-fixedness is a property of the
**service**, not the business. `price_cents` nullable; `requires_quote` is
generated over it so a second flag can never disagree with the price. One
pipeline — a fully-priced job simply starts further along it.

**Not reusing `menu_items`**, though a fixed-price service is shaped like a
menu item and the proven payment chain would come free: that flow is built
around *pickup*, and a booking needs a day and a time.

`request_quotes` is append-only and **the current price is the latest row**.
"Who offered what, when" is what gets asked when a customer disputes a
number, and one mutable column cannot answer it.

### The services pack — eleven operations
All registered, all proven live red-then-green (28 assertions,
`scripts/services_pipeline_gate.sql`). Refuses, by name: booking before
anything is agreed, a day before a price, $0, anything over $1,000,000,
re-pricing a booked job, another business's service on your job, staff
pricing a job or reading the customer's phone number, and another business
doing either.

**Booking is last** — `propose_day` is unreachable before the price is
agreed, in the database, not just in the UI.

### The services pack — the quote screen
The part Nick asked to be redesigned: *"the quote function is something I want
redesigned for sure, I find it very confusing."*

**One control that changes as the job moves. Editable until you send, locked
after.** His rule, verbatim: *"once the text is sent it locks the button, can't
go back, but as long as you don't hit send text it can still be edited."*

- The number lives in the button — "Send $180 to Emily", never "Send".
- The price field **disappears** once a price is sent, rather than going grey
  while still showing an editable old number.
- Two second-buttons, both earned: decline on a countered job (countering is
  uncapped — it is the customer's money — so the exit must not be silence)
  and call-it-off on a booked one.
- On a counter, typing a different number turns the **same** control into a
  fresh offer instead of adding a third button.

---

## The finding worth remembering

**v2 had already reached this exact conclusion, written it down, and it did
not save the screen.**

`lib/features/requests/quote_actions.dart`, v2's own comment: *"the screen
used to render every possible action at once — Save quote, Deposit link,
Balance link, Send quote to customer, Copy the link instead, They said yes —
and left the owner to infer the order of operations from six button labels.
There is only ever one next thing."*

Correct. And `nextQuoteStep` governs **one** button in `request_inbox.dart`,
rendered directly beneath "Propose a day (optional)" and directly above "Copy
the link instead", "They said yes", "Put it on the schedule" and "Mark
complete".

The fix was real in one place and absent in the place it was written for.
That is the same shape as almost every serious defect this week — the fixture
menu answering for a real one, a comment claiming a guard the next line
removed, a fallback fixed in one of two readers, a test handed `100/3` while
production used `33`.

**So v3's version returns EVERY control the screen may draw, and the screen
draws nothing else.** A test asserts that exactly two states carry a second
control. Adding a button means changing the rule file, where the reasoning
and the tests live.

## And a test that failed for the wrong reason

The cross-organization boundary check reported FAIL and the code was fine —
it had picked `nicholaswittle` as "another business", and that account is a
**super-admin**, which bypasses by design.

The gate now asserts, first, that the outsider it chose is genuinely not a
super-admin. **A check whose subject can silently be the one account exempt
from it proves nothing either way.**

---

## Where it stands

467 tests, analyzer clean, every CI gate green, three migrations applied
live. Next: the public request intake, then deposits (port the statute
arithmetic deliberately — see the $8.33 above), then the photo scope draft.

**The AI drafts SCOPE from photos and never names a price.** A price depends
on the owner's rates, travel and how busy they are, none of which is in a
photograph. There is deliberately no price column in `request_ai_runs` for it
to put one in.
