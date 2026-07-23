---
title: Jigsy's Website Concept
tags: [jigsys, web, concept, portfolio, design, vercel, github]
aliases: [jigsysite, Jigsy Website, Jigsy's Pizza Site]
date: 2026-07-22
updated: 2026-07-22
stage: concept-deployed
---

# Jigsy's Website Concept

An independent redesign concept for **Jigsy's Old Forge Pizza / Brewpub**
(Enola, PA) — the same business that is the [[Jigsys Brewpub|Apex pilot
customer]]. Built as a practice template / portfolio piece; **not** the
official Jigsy's site.

## Where it lives

| | |
|---|---|
| **Source** | `C:\development\projects\jigsys_site\` |
| **GitHub** | https://github.com/nicholaswittle/jigsysite |
| **Live (Vercel)** | https://jigsyssite.vercel.app — team **wi-sense-llc** |
| **Vercel project** | `jigsys_site` (CLI deploy; not Git-integrated yet) |

Redeploy after local changes:

```
vercel --cwd C:\development\projects\jigsys_site deploy --prod --yes --scope wi-sense-llc
```

Or: push to GitHub once Git → Vercel is connected (still open).

## What it is

Static site — single `index.html` + `images/` + README. No build step, no npm.

- **Identity = Old Forge style** — steel pan trays, square cuts, red / white /
  double white, ordered **3 / 6 / 12**.
- **The Board** (menu): tray matrix · specialty/gourmet ledgers · wings spotlight · plate lists · sticky jump nav
- **Call It In** — phone-order / dine-in only (no cart)
- Live Open/Closed from Summer 2026 hours
- Head: title/meta/OG, favicon, `Restaurant` JSON-LD + `aggregateRating` (Google 4.5 · 553)
- Light/dark theme toggle; reduced-motion safe

## Session changelog (2026-07-22)

### Pass A — foundation
- HTML5 head, schema, Call as primary CTA
- Full-bleed photo hero (later replaced by food)
- Pulled early assets from `jigsyspizza.com/graphics`

### Menu → The Board
- Replaced generic cut-cards with tray matrix / ledger / plates so layout matches how Old Forge is ordered

### Social proof (no scraped feeds)
- **Neighborhood says** — short guest notes + TripAdvisor / Yelp / Google chips
- **Chick Fil “J”** signature band
- Visit amenity strip; Locals also order
- TripAdvisor research: 4.5 · 76 · #3 of 26 Enola · ~78 photos (photo opportunity)

### Pass C — Google pull
- Google is strongest proof: **4.5 · 553** trust chip + schema rating
- Sharper voice notes (family / house beer); kept Chick Fil J quote
- **Peanut Butter Pie** dessert band (confirm still served — praise is older)
- Locals strip by Google topics: Wings · Stromboli · Antipasto · Pierogi · Double White · Pie · Chick Fil J
- **No live music** (outdated). Downstairs = **skill games + TV lounge** for sports / parties

### Pass B — food-first (then corrected)
- Tried Google guest food photos as hero + big photo band
- Nicholas: tray/wings band shots were weak and too large → removed fat band
- Hero kept as compressed pizza close-up (`hero_pizza.jpg`)
- Story uses lounge (`band_lounge.jpg`); exterior backup kept

### Pass D — Owner.com playbook
Source: [21 Best Pizza Websites](https://www.owner.com/blog/best-pizza-websites)

**Stole:** What’s good here · style education · FAQ · local SEO copy  
**Skipped:** online cart, loyalty popups, Domino’s-style ordering

Shipped:
1. **What’s good here** — Chick Fil J · Wings · Peanut Butter Pie (text tiles → deep links)
2. **Why Old Forge** — tightened explainer (not NY / not Chicago)
3. **FAQ** — how to order, patio, downstairs, catering
4. Killed photo band
5. Amenity strip → 3-col grid; lone mobile item centered

### Niche / portfolio angle
Website refresh for local restaurants is a possible Fiverr/service niche.
Pitch: before/after case study (old jigsyspizza.com → this concept),
*phone-order first, no fake review widgets*. Food photos from owner are the
real unlock for client handoff.

## Current images (`images/`)

| File | Use |
|------|-----|
| `hero_pizza.jpg` | Full-bleed hero (compressed) |
| `band_lounge.jpg` | Story — downstairs TV / skill games |
| `band_exterior.jpg` | Backup exterior (not in main flow) |
| `banner_collage.png` | Backup venue collage |
| `square-logo-full.png` | Favicon |
| `img_2956.jpg` | Legacy catering shot (unused in main flow) |

**Handoff rule:** Google guest photos are for concept demo only — replace with
owner originals before any client delivery.

## Review platform facts (research)

| Platform | Rating | Count | Emily mentions |
|----------|--------|-------|----------------|
| Google | 4.5 | 553 | None |
| TripAdvisor | 4.5 | 76 | None |
| Yelp | 4.0 | ~107 | None |

## Open gaps

- Owner-approved tray / wings / pie / patio photos
- Confirm peanut butter pie still on menu
- Identity break (cream/terracotta/Georgia still AI-default-adjacent)
- Theme persist (`localStorage`)
- **Connect GitHub → Vercel** so `git push` redeploys
- Custom domain later if they adopt the concept

## Why it matters

Portfolio / service-line proof: a real deployed restaurant refresh for a
business Nicholas already knows. Practice template for “revamp a small-business site.”

**Productized (2026-07-22 → recurring 2026-07-23):** this niche is now a live
service on [wisensellc.com](https://wisensellc.com) — the **"Website, Handled"
care plan ($79/mo + $299 setup**, client keeps domain, WiSense hosts +
maintains). Pricing evolved fast: launched as a $1,499 one-time sprint →
pulled the price (too high to anchor) → moved to the recurring care plan after
a second-opinion check. Plus a filterable Work showcase and an **anonymized**
before/after (the pizzeria concept, *no Jigsy's branding*, to protect the pilot
relationship). Model rationale: [[business/Web Redesign — Recurring Model Proposal 2026-07-23]].

Related: [[Jigsys Brewpub]], [[Apex Scheduler]], [[NOW]], [[index]]
