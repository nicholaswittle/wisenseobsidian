---
title: NOW — Weekly Scorecard
tags: [meta, tasks, scorecard, now]
aliases: [NOW, Weekly Scorecard, This Week]
updated: 2026-07-28
---

# NOW — Weekly Scorecard & Task Board

> First-class execution layer. Agents: after [[hot]], read this before planning work. Overwrite the “This week” section each Monday (or when focus shifts). Keep history in the archive section or daily logs.

## Company truth (one glance)

| Field | Value |
|-------|-------|
| **Stage** | Pre-revenue → pilot + store launch |
| **Focus apps** | Apex v2 (OS Phase 2–3 built locally) + Apex v1 (Assign Days live; ship Friday); COMMS LINK parked |
| **#1 metric this week** | Jigsy live ordering pilot with Emily · ship Apex v1 to stores |
| **This week's bet** | Pilot ops (stock/pause/accept+print) · vault+repos synced · Apex v1 keystore/accounts |
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

1. [x] Apex v2: manager log book + tip management + labor cost — **DONE 2026-07-27**
2. [x] Apex v2: AuthGate + sign-in wiring — **DONE 2026-07-27**
3. [x] Apex v2: foundation migration applied on live Apex DB — **DONE 2026-07-27**
4. [x] Apex v2: schedule week view — **DONE 2026-07-27**
5. [x] Apex v2: team chat + swaps + time-off + schedule publish — **DONE 2026-07-27**
6. [x] Apex v2 Reimagined #5–10: Assign Days · QR+offline · push→SMS · photo import · PA labor guardrails · calendar export — **DONE 2026-07-27** (`8f64bf8`; real + demo live)
7. [x] Apex v2 Online ordering (Flutter + Supabase) — **DONE 2026-07-27** (`d3a218e`; migration applied; seed `jigsys`)
8. [x] Apex v2 Labor vs revenue + Call-Outs + Smart capacity — **DONE 2026-07-27** (`2b08e5f` · `71152c6` · `1b74a4e`; local, **push pending**)
9. [x] Apex v1 Assign Days port + Vercel — **DONE 2026-07-27** (`03a62e6`; https://apex-scheduler-theta.vercel.app)
10. [x] restOS archive push — **DONE 2026-07-27** (`ffd8f5e`); **refresh 2026-07-28** afternoon (`a6cb554`)
11. [x] Jigsy Order online + full board + extras — **DONE 2026-07-28** (see [[Jigsy Online Ordering — Live Status 2026-07-28]])
12. [x] Push apex_v2 + jigsysite + vault — **DONE 2026-07-28** (`dc40da8` / `4505cd4` / restOS `a6cb554`)
13. [x] Kitchen Accept & print + menu stock toggles; no online alcohol — **DONE 2026-07-28**
14. [ ] Optional thermal printer; re-enable `auto_pause_enabled`; confirm topping $ with kitchen
15. [ ] Apex v2 product audit against live apps (deferred intentionally)
16. [ ] After store accounts: help smoke-test checklist / update [[Apex Scheduler]] + [[hot]]
17. [ ] Gate C: assist Play Console listing from copy already in Gate C note
18. [ ] New Horizon: commit or gitignore untracked `AGENTS.md` / `CLAUDE.md` / `.cursor/mcp.json`; push `main` if still ahead
19. [ ] Monthly [[VAULT_LINT]] pass if due

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
