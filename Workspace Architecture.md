---
title: Workspace Architecture
tags: [governance, architecture, workspace, reference]
aliases: [C Development Workspace, WiSense Workspace]
---

# WiSense Workspace Architecture

Source: `C:\development\CLAUDE.md`

## Startup
Read `SYSTEM_ARCHITECT_DIRECTIVE.md` and `global_status.md` before starting any task in this workspace. See [[System Architect Directive]].

## Shared Packages — check before building anything new

| Package | Path | Contents |
|---------|------|----------|
| `wisense_core` | `packages/wisense_core/` | `Result<T>`, `ApiException`, `WiSenseApiClient` (http wrapper), affiliate providers, flight models, travel search controller, repositories |
| `wisense_ui` | `packages/wisense_ui/` | `WiSenseLoadingIndicator`, `WiSenseErrorBanner`, `WiSenseSpacing`, `WiSenseTextStyles`, `glass_panel`, `snack_message`, flight UI components |

Before adding a new loading spinner, error display, spacing constant, or HTTP utility — read these packages first. See [[System Architect Directive]] mandate #1.

**Known fork:** New Horizon vendors its own copies — see [[Fork Reconciliation]].

## Apps (active)

| App | Path | Auth | Notes |
|-----|------|------|-------|
| COMMS LINK | `projects/wisense_decompression/` | None | On-device Gemma 2B-IT, zero-retention |
| Apex Scheduler | `projects/apex/apex/` | Supabase | Firebase push, brewpub scheduling |
| Horizon V2 | `projects/wisense_horizon_v2/` | Supabase | Duffel API, travel booking |
| New Horizon | `projects/wisense_new_horizon/` | Supabase | Alignment Engine — ToolRegistry, Duffel proxy, Consensus Deck |

## Apps (deleted — remove all references)

- wisense-os — deleted 2026-07-19 (not working as intended)
- my_ai — deleted 2026-07-19 (not working as intended)

## Other projects on disk

- `projects/helix/` — Flutter project, has its own repo. Status unclear (is it still active?)
- `projects/agent_test_playground/` — test playground, has its own repo
- `projects/local-agent-work-center/` — offline local-AI agent harness (Python). NOT its own repo — tracked inside the parent `C:\development` git repo. See [[Audit Findings Loop]].

## Workspace Scripts

```
.\scripts\analyze_workspace.ps1          — flutter/dart analyze across all projects
.\scripts\run_tests.ps1                  — run all package tests
.\scripts\run_tests.ps1 --update-goldens — regenerate golden baselines
.\scripts\diagnose_apex.ps1              — run environment checks, analysis, and tests for Apex Scheduler
.\scripts\watch_and_run.ps1              — monitor PLAN_FOR_CLAUDE.md and claude_inbox; execute Claude tasks
.\scripts\watch_cursor_inbox.ps1        — monitor cursor_inbox; route claude-target tasks; notify on cursor-ready work
.\scripts\correlate_changes.ps1          — match analyze issues to recent git changes
.\scripts\generate_docs.ps1              — dart doc → docs/<package>/
```

## Standards

All work follows the WiSense Tripartite Protocol at `C:\Users\nikwi\CLAUDE.md`.
Minimalist: no speculative features, no premature abstractions, no what-comments.

Related: [[WiSense Governance — Rules and Protocols]], [[System Architect Directive]], [[Fork Reconciliation]], [[Parent Repo Cleanup]]