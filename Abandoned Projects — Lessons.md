---
title: Abandoned Projects — Lessons
tags: [decision, postmortem, hygiene]
aliases: [killed projects, postmortem]
status: closed
---

# Abandoned Projects — Lessons

Killed 2026-07-19 (and shortly after). Not a failure — clarified the real workflow: [[Working Stack — Claude CLI and Ollama]].

## What was removed / abandoned

| Project | Path | Why |
|---|---|---|
| **WiSense OS** | `projects/wisense-os` | Custom desktop agent OS; overbuilt for the job; Claude CLI is enough |
| **my_ai** | `projects/my_ai` | Earlier agent / Command Center experiments; not the daily path |
| **Local Agent Work Center** | `projects/local-agent-work-center` | Claude↔Codex watcher theatre; audits move to vault + per-repo `audit/` |
| **command_center** | `projects/command_center` | Parallel UI experiment; not a launch product |
| **markdown_practice** | `projects/markdown_practice` | Scratch; already gone |

**Disk note:** `local-agent-work-center` and `wisense-os` may leave locked `.pytest_*` / `.git` leftovers until an elevated delete or reboot. Treat them as dead either way.

## Keep these ideas (don't rebuild the apps)

- **Digest / explicit approval before writes** — never freeform "go ahead" as a write gate
- **Cloud honesty** — don't invent local builders or Autopilot until hardware + qualification exist
- **Tenant + RLS discipline** — apply immediately to [[Apex Scheduler]] (see [[Apex Security Audit 2026-07-19]])
- **Audits as markdown artifacts** — vault + `repo/audit/`, not a separate agent product

## Still active (do not treat as abandoned)

- [[COMMS LINK]] — `wisense_decompression`
- [[Apex Scheduler]] — `projects/apex/apex` (own git repo)
- [[New Horizon]] — `wisense_new_horizon`
- Shared packages under `C:\development\packages` (canonical) — see [[Fork Reconciliation]]

Related: [[Working Stack — Claude CLI and Ollama]], [[Parent Repo Cleanup]], [[Home]]
