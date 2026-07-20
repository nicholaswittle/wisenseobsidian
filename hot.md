---
type: meta
title: "Hot Cache"
tags: [meta, hot-cache, context]
updated: 2026-07-20T00:00:00
---

# Recent Context

> ~500-word cache for fast boot. Any agent/session reads this FIRST for recent state before crawling [[index]]. Overwrite completely each update — cache, not journal.

## Last Updated
2026-07-20. Ran a grounded audit of this vault (verified claims against disk), then reconciled [[index]] and rebuilt this cache.

## Key Recent Facts
- **Vault is a strong static reference, but the "self-synthesizing" loop is dormant.** `wiki/`, `journal/`, `crm/` folders are empty; the `raw/ → wiki/` intake pipeline has essentially never run.
- **Real value comes from static manifests**, not automation: [[ENVIRONMENT_MAP]] (ports 5050/11434/54321), [[API_CONTRACT_REGISTRY]], [[TROUBLESHOOTING_KATAS]], [[CANONICAL_PACKAGE_MAP]], [[00_AI_AGENT_MANIFEST]], [[index]].
- **Auto-sync is NOT running.** Windows task `WiSenseVaultAutoSync` is not registered; the only auto-sync commits were a 14-min burst on 2026-07-19 (22:43–22:57). Vault content froze ~then until this session.
- Git mirror is live and verified: `github.com/nicholaswittle/wisenseobsidian` (origin/main).

## Active Project Status (per [[00_AI_AGENT_MANIFEST]])
- **COMMS LINK** (`wisense_decompression`) — 59/59 tests pass; main ~10 commits ahead of origin (needs push).
- **Apex Scheduler** (`apex\apex`) — merge conflict fix needed + security audit findings (RLS, claim races). See [[output/Apex — Merge Conflict Resolution Plan]], [[output/Apex Security Audit 2026-07-19]].
- **New Horizon** (`wisense_new_horizon`) — 117/117 tests pass; README boilerplate; [[Fork Reconciliation]] open.
- **DELETED (not active):** wisense-os, my_ai, local-agent-work-center, command_center.

## Recent Changes
- **Fork Reconciliation COMPLETE (2026-07-20)** — vendored wisense_core (47 files, full travel engine) and wisense_ui (19 files) promoted to canonical at `C:\development\packages\`. New Horizon switched to path deps (`../../packages/`). Vendored `packages/` dir removed. All tests green: core 69/69, UI 21/21, NH 117/117, HV2 7/7.
- **DECISION EXECUTED (2026-07-20):** Vault formally declared a **curated static reference**. Retired the dormant MindStudio synthesis pipeline — deleted empty `wiki/`, `journal/`, `crm/` folders.
- Reconciled topology drift across all governance files: [[CLAUDE]], [[agents]], [[Home]], [[index]] now agree on `raw/ → root notes` (manual codification), no auto-synthesis loop. [[agents]] is the canonical protocol; codification collapsed to one on-request section.
- Updated: [[index]] — struck deleted `wisense-os` from Live Codebase; added [[hot]] to boot order.
- Created: this [[hot]] cache (was empty before today).

## Active Threads
- Open: `WiSenseVaultAutoSync` task is unregistered — now consistent with the static-reference decision, but the two sync scripts in `C:\development\scripts\` are dead code; retire them if you want full cleanup.
- Boot order for any agent: [[hot]] → [[index]] → relevant note.
