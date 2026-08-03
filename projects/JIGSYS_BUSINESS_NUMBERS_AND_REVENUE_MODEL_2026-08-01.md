---
title: Jigsy's Business Numbers and What They Say About the Revenue Model
tags: [jigsys, apex, revenue, pricing, strategy, square]
date: 2026-08-01
---

# Jigsy's Business Numbers and the Revenue Model

Observed from their Square reporting dashboard, 2026-08-01. Trailing-period
figures with year-over-year comparison.

Related: [[projects/JIGSYS_PILOT_LAUNCH_STRATEGY_2026-07-31]],
[[projects/APEX_PAYMENTS_AND_POS_STRATEGY_2026-07-31]], [[customers/Jigsys Brewpub]]

---

## The numbers

| Metric | Value | YoY |
|---|---|---|
| Gross sales | $293,232 | −4% |
| **Net sales** | **$291,668** | −5% |
| Comps & discounts | ($618) | −39% |
| **Tips** | **$39,813** | +2% |
| Average sale | $48 | **+11%** |
| **Orders** | **6,171** | **−14%** |
| Customers | 5,296 | −12% |
| Refunds | ($136) | — |
| **Labor % of net sales** | **0.0%** | no change |
| New / returning customers | 2,033 / 3,263 | both down |

Top categories: Pizza $112,051 (6,725 qty, +4%) · Wings $29,638 (−11%) ·
Appetizers $24,485 (−9%) · Salads $22,970 (−3%) · Beer $20,207 (−11%) ·
Liquor $16,752 (−9%).

---

## What the numbers say

**Traffic is the problem, not spend.** Orders −14% and customers −12%, while
average sale is **+11%**. Fewer people, each spending more. Revenue holds at
−5% only because ticket size is masking the decline.

**This is the pitch.** Not "modernise your website" — *recover order count*.
They are the only restaurant within five miles without online ordering, and
their own report says traffic is what they are losing. Pizza is the only
category up (+4%); everything else is down.

**Tips are 13.6% of net sales** ($39,813). Staff have real skin in the tip
flow, which makes the tipping configuration worth getting right rather than
treating as a detail.

**Refunds are $136 on $292k.** A clean operation — no meaningful dispute or
comp problem to solve.

---

## The finding that reframes the business model

**Labor % of net sales: 0.0%.** They track no labor in Square at all. No
scheduling, no timeclock, no labor cost, against a $292k revenue base.

Apex has all three, and Square is not going to fill that gap.

### Ordering fees are not a business

At 1.5% of online volume, against $291,668 net sales:

| Online capture | Annual volume | Fee revenue | Per month |
|---|---|---|---|
| 10% | $29,167 | $437 | **$36** |
| 25% | $72,917 | $1,094 | **$91** |
| 40% | $116,667 | $1,750 | $146 |

25% is optimistic for a brewpub with a dining room and a bar. Realistic
expectation from Jigsy's ordering fees: **$40-90/month.**

### On Stripe, small venues may be net-negative

If WiSense is on Stripe's "platform sets pricing" Connect model, the platform
pays $2/mo per active account, 0.25% + 25c per payout, and 0.25% of payout
volume.

Net is approximately **1.0% of volume − $2 − payout fees**. With daily payouts
that is roughly `0.01 × V − $9.50` per month, so **break-even sits near
$950/month of online volume.** Below that a Stripe venue *costs* money every
month.

**Square carries no such per-account charges** — `app_fee_money` comes off the
payment and that is the end of it. So Square is materially better economics for
small venues, which is most of the target market. That is a second, independent
reason to favour the Square rail beyond the free printing.

This makes the Connect pricing-model check (`scripts/check_connect_pricing.mjs`)
urgent rather than optional: it is the difference between "small but positive"
and "negative on every quiet venue."

### Where the money actually is

**One venue on a $79/month subscription is $948/year** — more than double the
fee revenue at optimistic online capture, with no volume risk and no
payment-processor overhead.

**Reframe: online ordering is the wedge, Apex-the-scheduler is the business.**
Ordering gets in the door and proves delivery; recurring revenue comes from the
gap their POS leaves empty. The 1.5% is not wrong — it is guest-funded, costs
the venue nothing, and makes the ordering pitch free to say yes to. It simply
must not be expected to fund the company.

---

## Operational notes captured the same day

**Tipping config** (inherited automatically by the hosted checkout via
`allow_tipping`): Collect Tips ON · Smart Tip Amounts · calculated **after**
taxes · custom amounts allowed · "ask for tips before payment" OFF.

**In-person tipping runs on paper receipts.** Square warns that tips on printed
receipts must be entered within 36 hours or the transaction settles with no tip.
Online tips bypass this entirely — captured at checkout, no window. A small
genuine improvement worth mentioning to them.

**Two duplicate "Appetizers" categories** exist in their Square Library (13
items and 6 items). Harmless today; would produce duplicates if menu sync from
Square is ever built.
