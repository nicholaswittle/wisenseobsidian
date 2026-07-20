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
- **Apex Scheduler** (`apex\apex`) — merge finished (`13971b0`) + **race/timezone fixes** (`9f723a5`), both local. Gate B **BLOCKED on RLS**: repo docs contradict (README "applied to remote" vs MIGRATION_INVENTORY "SQL pending"); RLS SQL not in repo; needs live Supabase truth before writing multi-tenant policies. See [[output/Launch Readiness — COMMS LINK and Apex 2026-07-20]], [[output/Apex Security Audit 2026-07-19]].
- **New Horizon** (`wisense_new_horizon`) — 117/117 tests pass; fork reconciliation COMPLETE; README still boilerplate.
- **DELETED (not active):** wisense-os, my_ai, local-agent-work-center, command_center.

## Recent Changes
- **Fork Reconciliation COMPLETE (2026-07-20)** — vendored wisense_core (47 files) and wisense_ui (19 files) promoted to canonical at `C:\development\packages\`. New Horizon switched to path deps. Vendored `packages/` dir removed. All tests green: core 69/69, UI 21/21, NH 117/117, HV2 7/7.
- **Vault audit & cleanup (2026-07-20)** — 55 notes, 412 wikilinks. Fixed stale fork references in [[Fork Reconciliation]], [[index]], [[Home]]. Rewrote [[CANONICAL_PACKAGE_MAP]] for single-source-of-truth. Cleaned dead wisense-os refs from 3 manifests. Filled [[Stripe]] stub. Linked 2 unreachable notes. Created [[2026-07-20]] daily log.
- **DECISION EXECUTED (2026-07-20):** Vault formally declared a **curated static reference**. Retired dormant MindStudio synthesis pipeline — deleted empty `wiki/`, `journal/`, `crm/` folders.

## Active Threads
- **LAUNCH PLAN (2026-07-20):** [[output/Launch Readiness — COMMS LINK and Apex 2026-07-20]] — Android-first, no Mac yet. **COMMS LINK pushed ✅** (next = Android packaging). **Apex:** Gate A merge ✅ (`13971b0`), Gate B races+timezone ✅ (`9f723a5`), **RLS ⛔ blocked — awaiting Nicholas to confirm live Supabase RLS state** (repo docs contradict). Then per-query org-scoping + Gate C package. iOS via Codemagic later.
- Open: New Horizon untracked `AGENTS.md`, `CLAUDE.md`, `.cursor/mcp.json` — commit or .gitignore.
- Open: New Horizon `main` ahead of `origin/main` by 1 commit — needs push.
- Boot order for any agent: [[hot]] -> [[index]] -> relevant note.