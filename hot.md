---
type: meta
title: "Hot Cache"
tags: [meta, hot-cache, context]
updated: 2026-07-20T20:40:00
---

# Recent Context

> ~500-word cache for fast boot. Any agent/session reads this FIRST, then [[NOW]], then [[index]]. Overwrite completely each update — cache, not journal.

## Last Updated
2026-07-20 (evening). Vault AI performance layer implemented (NOW, customers, experiments, lint, thin CLAUDE, status sync).

## Key Recent Facts
- **Vault = curated static reference** — no auto-wiki. Boot: [[hot]] → [[NOW]] → [[index]] → note.
- **Execution layer:** [[NOW]] (scorecard + human blockers). Customer truth: [[customers/_Index]]. Experiments: [[business/Experiment Log]]. Health: [[VAULT_LINT]].
- **Schema:** thin [[CLAUDE]]; protocol in [[agents]]; paths in [[00_AI_AGENT_MANIFEST]] (status always here / NOW).
- Git mirror: `github.com/nicholaswittle/wisenseobsidian` (origin/main).

## Active Project Status
- **COMMS LINK** (`wisense_decompression`) — 59/59; **pushed**, origin in sync. Next: Gate C Android packaging.
- **Apex Scheduler** (`apex\apex`) — **code-complete & pushed**: merge `13971b0`, races+tz `9f723a5`, RLS `a66b039`. **ONLY human step:** apply `20260720000000_launch_blockers_rls.sql` on Supabase staging→prod. Then Gate C. See [[output/Launch Readiness — COMMS LINK and Apex 2026-07-20]].
- **New Horizon** (`wisense_new_horizon`) — 117/117; fork reconciliation COMPLETE. Open: untracked agent files; may need push.
- **DELETED:** wisense-os, my_ai, local-agent-work-center, command_center.

## Active Threads
- **LAUNCH:** Both apps code-complete. Remaining = ops/human: (1) Apex RLS apply; (2) Gate C keystore/AAB/graphics/privacy/Play listing — [[output/Gate C — Android Packaging & Store Listings 2026-07-20]]; (3) iOS later via Codemagic.
- **This week board:** [[NOW]]
- Working stack: Claude CLI + Ollama — [[Working Stack — Claude CLI and Ollama]]
