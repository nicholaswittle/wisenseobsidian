---
title: Web Redesign — Recurring Model Proposal
tags: [business, pricing, web, recurring, mrr, proposal, for-review]
date: 2026-07-23
status: ADOPTED · live on site 2026-07-23
---

# Web Redesign — Recurring Model Proposal (2026-07-23)

> **✅ ADOPTED (2026-07-23).** Second opinion confirmed the pay model. **Live on
> [wisensellc.com](https://wisensellc.com)** as the "Website, Handled" care plan:
> **$79/mo + $299 one-time setup**, client keeps their domain, WiSense hosts on
> Vercel + maintains, cancel anytime. (Superseded the "72-Hour Sprint · Free
> quote" card; the $1,499 one-time was pulled the same day.) Commit `dcce76e`.
> The proposal detail below is retained as the rationale of record.

## The question

Switch the website-redesign service line from a **one-time fee** to a
**monthly recurring** model (hosting + maintenance). Nicholas's instinct;
this note captures the proposed shape and open decisions.

## The ownership model (decided)

- **Client keeps their own domain** (their name, SEO, email — the assets that
  matter to them).
- **WiSense builds the site and hosts it on WiSense's own Vercel.** Client
  points their domain (A / CNAME) at WiSense's deployment.
- **Monthly fee = hosting + maintenance + content updates.**
- **WiSense owns the build/code;** the monthly licenses its use + hosting +
  upkeep. This is what makes the recurring model defensible.

## Recommended structure (starting point)

**$299 setup + $79/mo — single plan** ("we build it, host it, keep it
updated"). One plan, not tiers, to reduce decision friction for a brand-new
business. Introduce **Essentials $49 / Plus $99** tiers later once there's a
client base and known demand.

- Setup fee protects build labor against early churn without the $1,499
  sticker shock.
- $79/mo is an easy local-owner yes.
- **Buyout option:** if a client leaves and wants the code, one-time fee →
  hand over the repo. Clean, non-hostage exit.

## Why recurring fits this business

- **Lower barrier than one-time** — solves the same problem that pulling the
  $1,499 did, but better (small monthly vs. four-figure check).
- **Predictable MRR** vs. one-time feast/famine; compounds over time
  ($79/mo ≈ $2,800 / 3yr vs. one $1,499).
- **Restaurant niche fits perfectly** — menus, specials, hours, events change
  constantly; "we keep it updated" is genuine value, not a fake upsell.
- **Retention/moat** — site lives on WiSense infra; low casual churn.
- Aligns with the [[business/WiSense Service as a Software Execution Strategy|Service-as-a-Software]] direction.

## Economics

- **Vercel Pro ($20/mo) is required** — the free Hobby plan is non-commercial
  per ToS; hosting paying clients on it violates it. **Pro hosts many client
  sites/domains under one account**, so hosting cost stays ~$20 flat.
- Margin: $79/mo × N clients − $20 flat Vercel ≈ near-pure margin. 10 clients
  = ~$790/mo revenue on ~$20 cost.

## Operational must-dos (where new agencies get burned)

1. **Vercel Pro for commercial hosting** (above).
2. **Never break their email on DNS change** — only change web records
   (A / CNAME) to Vercel; **leave MX records alone.** A careless DNS edit kills
   their email and loses the client on day one.
3. **Code-ownership clause in writing** — "WiSense owns the build; monthly
   licenses use + hosting + maintenance."
4. **Define maintenance scope** — cap updates (e.g., "up to X/month, Y-business-
   day response") or "maintenance" becomes unbounded free support.

## Risks / cons (be honest)

- **Ongoing solo support load** — Nicholas is also launching Apex + COMMS LINK.
  Monthly means "my site's down Friday 8pm" is now his problem. Scope discipline
  is essential.
- **Revenue depends on retention** — local businesses close / change owners.
- **Scope creep** on "updates" if not capped.

## Open decisions (for the second opinion)

1. Numbers: is **$299 setup + $79/mo** right, or different? Lower setup / higher
   monthly? True $0-down?
2. **Single plan vs. tiers** at launch?
3. **Buyout** — advertise on the site, or keep for the sales conversation?
4. **Minimum term / contract** (e.g., 3-month min) or month-to-month?
5. Where does the funnel land? Currently "Get a Free Quote" → `/partner` form.

## Next step

Once numbers are confirmed: rebuild the wisensellc.com pricing card around the
care-plan model + deploy (Claude can do this in one pass). Draft card copy
first for review.

Related: [[business/Pricing Strategy]], [[business/WiSense Service as a Software Execution Strategy]], [[Jigsys Website Concept]], [[log]], [[NOW]]
