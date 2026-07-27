---
title: NOW — Weekly Scorecard
tags: [meta, tasks, scorecard, now]
aliases: [NOW, Weekly Scorecard, This Week]
updated: 2026-07-24
---

# NOW — Weekly Scorecard & Task Board

> First-class execution layer. Agents: after [[hot]], read this before planning work. Overwrite the “This week” section each Monday (or when focus shifts). Keep history in the archive section or daily logs.

## Company truth (one glance)

| Field | Value |
|-------|-------|
| **Stage** | Pre-revenue → pilot + store launch |
| **Focus apps** | Apex v2 (active build) + Apex v1 (ship Friday); COMMS LINK parked |
| **#1 metric this week** | Apex v2: log book + tip management built |
| **This week's bet** | Ship Apex v1 to stores + build v2 features with Claude/Cursor |
| **Working stack** | Claude CLI + Ollama + Cursor — see [[Working Stack — Claude CLI and Ollama]] |

## Scorecard (update weekly)

| Metric | Target | Current | Notes |
|--------|--------|---------|-------|
| Apex RLS on staging | Applied + smoke-tested | Pending (human) | Migration `20260720000000_launch_blockers_rls.sql` in repo |
| COMMS LINK store packaging | Keystore + AAB built | Not started | [[output/Gate C — Android Packaging & Store Listings 2026-07-20]] |
| Apex store packaging | Keystore + AAB built | Not started | Same Gate C note |
| Jigsy pilot interviews logged | ≥1 owner note in `customers/` | 1 interest note | Website liked; ordering idea planted; no pilot approval yet |
| Experiments with outcomes | Keep/kill filled | See [[business/Experiment Log]] | |
| WiSense Agency Pipeline | Loom Zoom Boom Active | See [[business/Client Acquisition Strategy — Loom Zoom Boom & FIG Portfolio]] | |

## This week — next actions

### Human-only (Nicholas)

1. [ ] Apply Apex RLS migration on **Supabase staging** → smoke-test org isolation → **prod**
2. [ ] Gate C assets: app icon 512, feature graphic 1024×500, screenshots (both apps)
3. [ ] Host privacy policy URLs (COMMS LINK + Apex)
4. [ ] Create Android upload keystores (commands in Gate C note)
5. [ ] **Apex iOS bundle ID changed** → add `com.nicholaswittle.apex://` to Supabase allowed redirect URLs, or iOS login will not return to the app ([[DECISIONS]] 2026-07-21)
6. [ ] QA branch `feat/apex-plan-2026-07-21` on the iPhone (`flutter clean && pub get`, `pod install`, re-select signing team) — then decide whether to push/merge. See [[Apex — Feature Plan Implementation 2026-07-21]]
7. [ ] Create Android upload keystores for both apps — see [[output/Keystore Setup Instructions 2026-07-20]] (run the keytool commands, save passwords)
8. [ ] Buy Google Play Developer account ($25 one-time) and Apple Developer Program ($99/yr) — payday Friday

### Side income — revenue plays (2026-07-21)

7. [ ] Fiverr: 2 gigs live (military resumes + website building) — add gig images from jigsyssite.vercel.app screenshots
8. [ ] Jigsy's: owner liked the website and the online-ordering idea has been
   planted. Next: show the isolated demo as **one website with optional
   ordering** - pause during a rush hides the ordering buttons; reopen during a
   slower period. Leave the five-page owner handout, ask what printer/device
   they use, and request a small reversible pilot only if interested. Demo:
   https://jigsys-ordering-demo.nicholaswittle.chatgpt.site — see
   [[business/Jigsys Ordering Demo — Build Record 2026-07-23]]
9. [ ] Set up intake form (Google Forms) + auto-reply on nicholaswittle@wisensellc.com for website inquiries — see [[business/Website Business Setup Guide 2026-07-21]]
10. [ ] Post in 3-5 veteran Facebook groups about resume service (free, zero-cost client acquisition)
11. [ ] Next off-weekend: build a cold demo for a real business with a bad website → send live link as pitch

### Agent-ready (when asked)

1. [ ] Apex v2: Build manager log book + tip management (prompts ready for Claude/Cursor — see [[Apex v2 — Restaurant OS Build]])
2. [ ] Apex v2: Build labor cost dashboard (after log book + tips)
3. [ ] After RLS applied: help smoke-test checklist / update [[Apex Scheduler]] + [[hot]]
4. [ ] Gate C: assist Play Console listing from copy already in Gate C note
5. [ ] New Horizon: commit or gitignore untracked `AGENTS.md` / `CLAUDE.md` / `.cursor/mcp.json`; push `main` if still ahead
6. [ ] Monthly [[VAULT_LINT]] pass if due

### Parked (not this week)

- iOS / Codemagic (needs Apple account)
- New Horizon commercial Duffel/Viator expansion
- Stripe for Apex (deferred — [[DECISIONS]])

## Blocked on

| Item | Blocker | Owner |
|------|---------|-------|
| Apex launch security | DB access to apply RLS | Nicholas |
| Play Store submit | Graphics + privacy URLs + keystore | Nicholas |

## Links

- Launch: [[output/Launch Readiness — COMMS LINK and Apex 2026-07-20]]
- Packaging: [[output/Gate C — Android Packaging & Store Listings 2026-07-20]]
- Keystore setup: [[output/Keystore Setup Instructions 2026-07-20]]
- Customers: [[customers/_Index]]
- Experiments: [[business/Experiment Log]]
- Decisions: [[DECISIONS]]

Related: [[hot]], [[index]], [[agents]], [[Home]]
