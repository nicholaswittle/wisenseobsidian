---
title: NOW — Weekly Scorecard
tags: [meta, tasks, scorecard, now]
aliases: [NOW, Weekly Scorecard, This Week]
updated: 2026-07-20
---

# NOW — Weekly Scorecard & Task Board

> First-class execution layer. Agents: after [[hot]], read this before planning work. Overwrite the “This week” section each Monday (or when focus shifts). Keep history in the archive section or daily logs.

## Company truth (one glance)

| Field | Value |
|-------|-------|
| **Stage** | Pre-revenue → pilot + store launch |
| **Focus apps** | COMMS LINK + Apex (Android-first); New Horizon secondary |
| **#1 metric this week** | Apex RLS applied on Supabase staging (yes/no) |
| **This week’s bet** | Clear human launch blockers so both apps can enter Gate C packaging |
| **Working stack** | Claude CLI + Ollama — see [[Working Stack — Claude CLI and Ollama]] |

## Scorecard (update weekly)

| Metric | Target | Current | Notes |
|--------|--------|---------|-------|
| Apex RLS on staging | Applied + smoke-tested | Pending (human) | Migration `20260720000000_launch_blockers_rls.sql` in repo |
| COMMS LINK store packaging | Keystore + AAB built | Not started | [[output/Gate C — Android Packaging & Store Listings 2026-07-20]] |
| Apex store packaging | Keystore + AAB built | Not started | Same Gate C note |
| Jigsy pilot interviews logged | ≥1 owner note in `customers/` | 0 | Template: [[customers/_Index]] |
| Experiments with outcomes | Keep/kill filled | See [[business/Experiment Log]] | |

## This week — next actions

### Human-only (Nicholas)

1. [ ] Apply Apex RLS migration on **Supabase staging** → smoke-test org isolation → **prod**
2. [ ] Gate C assets: app icon 512, feature graphic 1024×500, screenshots (both apps)
3. [ ] Host privacy policy URLs (COMMS LINK + Apex)
4. [ ] Create Android upload keystores (commands in Gate C note)
5. [ ] **Apex iOS bundle ID changed** → add `com.nicholaswittle.apex://` to Supabase allowed redirect URLs, or iOS login will not return to the app ([[DECISIONS]] 2026-07-21)
6. [ ] QA branch `feat/apex-plan-2026-07-21` on the iPhone (`flutter clean && pub get`, `pod install`, re-select signing team) — then decide whether to push/merge. See [[Apex — Feature Plan Implementation 2026-07-21]]

### Agent-ready (when asked)

1. [ ] After RLS applied: help smoke-test checklist / update [[Apex Scheduler]] + [[hot]]
2. [ ] Gate C: assist Play Console listing from copy already in Gate C note
3. [ ] New Horizon: commit or gitignore untracked `AGENTS.md` / `CLAUDE.md` / `.cursor/mcp.json`; push `main` if still ahead
4. [ ] Monthly [[VAULT_LINT]] pass if due

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
- Customers: [[customers/_Index]]
- Experiments: [[business/Experiment Log]]
- Decisions: [[DECISIONS]]

Related: [[hot]], [[index]], [[agents]], [[Home]]
