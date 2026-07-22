---
title: Jigsy's Website Concept
tags: [jigsys, web, concept, portfolio, design, vercel]
aliases: [jigsysite, Jigsy Website, Jigsy's Pizza Site]
date: 2026-07-22
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
| **Source** | `C:\development\projects\jigsys_site\` (single self-contained `index.html` + README) |
| **Repo** | https://github.com/nicholaswittle/jigsysite |
| **Live (Vercel)** | https://jigsyssite.vercel.app — deployed under the **wi-sense-llc** Vercel team |
| **Artifact** | https://claude.ai/code/artifact/85f5d407-0824-480e-863b-dab0d6fedc65 (private) |

Deploy is a **direct Vercel CLI push, not a Git integration** — it does *not*
auto-redeploy on `git push`. Re-run `vercel --cwd <dir> deploy --prod --yes
--scope wi-sense-llc`, or connect the repo in the Vercel dashboard for
auto-deploys.

## What it is

Single static HTML file, no build step, no dependencies, no external assets
(Artifact CSP blocks remote fonts/images, so everything is inline).

- **Identity built on Old Forge style** — rectangular trays cut into squares,
  red / single-white / double-white, 3/6/12 cuts. The tray-cut grid is the
  structural system; hero art is a CSS/SVG top-down tray. Deliberately avoids
  the red-green-white Italian cliché and the AI cream+serif+terracotta default.
- **Real menu** (transcribed from Jigsy's published **Nov 2025** menu images
  at `jigsyspizza.com/graphics/`): Old Forge pizza, 11 Specialty, 5 Gourmet,
  Stromboli & Flatbreads, Wings (5/10/20/30 + all flavors), Starters, Salads,
  Subs & Platters, soups, drinks, dressings. Prices real; subject to change.
- **Live Open/Closed status** computed from the real Summer 2026 hours; hours
  table auto-marks "today."
- **"Call It In" section** — honest phone-order flow (see decision below).
- Light/dark themes with toggle; all motion respects `prefers-reduced-motion`;
  progressive-enhancement safe (visible with JS off).

## Decisions & feedback applied

- **No "Build Your Tray" builder.** An earlier version had an interactive
  tray builder. **Nicholas's call:** Jigsy's is *phone-order and dine-in
  only* — a builder implies an online cart they don't offer, which misleads.
  Replaced with a "Call It In" section (three steps + big tap-to-call number +
  live status). Lesson: match the interaction to how the business actually
  takes orders.
- **Hero "hard to load" fixed** — removed the ambient steam canvas, the
  load-fade that hid the hero until JS ran, and the sticky-bar backdrop blur
  (mobile compositing cost). Hero now paints instantly.

## Open gap

- **Photo-less.** No real food photography (CSP blocked remote images; none
  supplied). Hero uses CSS/SVG tray art. This is the one thing between
  "sharp concept" and "looks like the real site" — needs real photos wired in
  (image slots or data-URI embeds).

## Why it matters

Doubles as a **portfolio / service-line proof**: a real, deployed restaurant
site for a business Nicholas already has a relationship with. Relevant to the
web-design / Fiverr gig direction explored earlier. Also a reusable practice
template for the "revamp a small-business site" play.

Related: [[Jigsys Brewpub]], [[Apex Scheduler]], [[NOW]], [[index]]
