---
title: New Horizon
tags: [app, launch-priority, new-horizon]
aliases: [wisense_new_horizon, Alignment Engine]
---

# New Horizon

Travel booking Alignment Engine — ToolRegistry, Duffel proxy, Consensus Deck. The most architecturally rich WiSense app.

| | |
|---|---|
| **Repo** | `C:\development\projects\wisense_new_horizon` |
| **Stack** | Flutter · Supabase · Duffel API · Viator/Expedia affiliate · google_generative_ai |
| **Platforms** | iOS · Android · Web |
| **Last commit** | 14 min ago (active) |

## Architecture

- `lib/core/ai/agent_bridge.dart` — ToolRegistry routing point (governed by Travel Data Integrity MDT extension)
- `lib/core/governance/` — governance log store + orchestrator
- Features: alignment, auth, debug, governance, governor, itinerary, search, transit, vault

## Launch blockers

- [x] **README boilerplate** — replaced with real README matching Apex/COMMS LINK format (2026-07-20)
- [x] **Fork Reconciliation COMPLETE (2026-07-20)** — vendored packages promoted to canonical; New Horizon now uses path deps on `../../packages/wisense_core` and `../../packages/wisense_ui`
- [x] **28 cursor branches pruned (2026-07-20)** — 25 merged deleted, 3 unmerged archived as tags
- Untracked `AGENTS.md`, `CLAUDE.md`, `.cursor/mcp.json` — commit or .gitignore
- Push local `main` branch to `origin/main` (ahead by 1 commit)

Related: [[Priority launches|Priority Launches]], [[Fork Reconciliation]], [[Tripartite Protocol]], [[Audit Findings Loop]]
- **Repo Location**: `C:\development\projects\wisense_new_horizon`
- **Git State**: 3 untracked files (`.cursor/mcp.json`, `AGENTS.md`, `CLAUDE.md`); `main` is ahead of `origin/main` by 1 commit.
- **Analyzer**: **0 issues found** (100% clean).
- **Unit & Widget Tests**: **117 / 117 passed** (100% pass rate in 7s).
- **Verdict**: Architecture & Test Suite 100% Green. Next step: Commit governance files, push to origin, update README.

Related: [[Home|Priority launches]], [[Fork Reconciliation]], [[Tripartite Protocol]], [[Audit Findings Loop]]