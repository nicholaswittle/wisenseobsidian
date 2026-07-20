---
type: meta
title: "Hot Cache"
tags: [meta, hot-cache, context]
updated: 2026-07-20T01:00:00
---

# Recent Context

> ~500-word cache for fast boot. Any agent/session reads this FIRST for recent state before crawling [[index]]. Overwrite completely each update — cache, not journal.

## Last Updated
2026-07-20. Fork reconciliation complete + vault audit and cleanup performed.

## Key Recent Facts
- **Vault is a curated static reference** — the dormant MindStudio synthesis pipeline was retired 2026-07-20. Knowledge lives as hand-linked root notes.
- **Real value comes from static manifests**: [[ENVIRONMENT_MAP]] (ports 5050/11434/54321), [[API_CONTRACT_REGISTRY]], [[TROUBLESHOOTING_KATAS]], [[CANONICAL_PACKAGE_MAP]], [[00_AI_AGENT_MANIFEST]], [[index]].
- Git mirror is live and verified: `github.com/nicholaswittle/wisenseobsidian` (origin/main).

## Active Project Status (per [[00_AI_AGENT_MANIFEST]])
- **COMMS LINK** (`wisense_decompression`) — 59/59 tests pass; **pushed to origin/main 2026-07-20 (in sync)**. Next: Android packaging (keystore, .aab, store assets).
- **Apex Scheduler** (`apex\apex`) — **code-complete & pushed** (origin/main in sync): merge `13971b0`, races+timezone `9f723a5`, **RLS migration `a66b039`** (`20260720000000_launch_blockers_rls.sql` — per-org RLS + atomic clock-in index, idempotent). All green. **ONLY remaining step: apply that migration on Supabase staging→prod** (default-deny; needs Nicholas's DB access). Then Android packaging. See [[output/Launch Readiness — COMMS LINK and Apex 2026-07-20]].
- **New Horizon** (`wisense_new_horizon`) — 117/117 tests pass; fork reconciliation COMPLETE; README still boilerplate.
- **DELETED (not active):** wisense-os, my_ai, local-agent-work-center, command_center.

## Recent Changes
- **Fork Reconciliation COMPLETE (2026-07-20)** — vendored wisense_core (47 files) and wisense_ui (19 files) promoted to canonical at `C:\development\packages\`. New Horizon switched to path deps. Vendored `packages/` dir removed. All tests green: core 69/69, UI 21/21, NH 117/117, HV2 7/7.
- **Vault audit & cleanup (2026-07-20)** — 55 notes, 412 wikilinks. Fixed stale fork references in [[Fork Reconciliation]], [[index]], [[Home]]. Rewrote [[CANONICAL_PACKAGE_MAP]] for single-source-of-truth. Cleaned dead wisense-os refs from 3 manifests. Filled [[Stripe]] stub. Linked 2 unreachable notes. Created [[2026-07-20]] daily log.
- **DECISION EXECUTED (2026-07-20):** Vault formally declared a **curated static reference**. Retired dormant MindStudio synthesis pipeline — deleted empty `wiki/`, `journal/`, `crm/` folders.

## Active Threads
- **LAUNCH PLAN (2026-07-20):** [[output/Launch Readiness — COMMS LINK and Apex 2026-07-20]] + [[output/Gate C — Android Packaging & Store Listings 2026-07-20]]. Android-first, no Mac yet. **COMMS LINK & Apex both code-complete & pushed ✅.** Both apps have release signing wired. **Remaining is ops/human-only:** (1) apply Apex RLS migration on Supabase staging→prod; (2) Gate C = run keystore + `flutter build appbundle`, supply graphic assets (icon 512, feature 1024×500, screenshots), host privacy URLs, Play Console listing (copy already written in the Gate C note); (3) iOS via Codemagic once Apple account exists.
- Open: New Horizon untracked `AGENTS.md`, `CLAUDE.md`, `.cursor/mcp.json` — commit or .gitignore.
- Open: New Horizon `main` ahead of `origin/main` by 1 commit — needs push.
- Boot order for any agent: [[hot]] -> [[index]] -> relevant note.