---
title: AI Agent Manifest & Operating Rules
tags: [ai, manifest, rules, entrypoint, karpathy]
aliases: [AI Manifest, Agent Rules, Start Here]
updated: 2026-07-20
---

# 00 AI Agent Manifest & Operating Rules

> Every AI session MUST boot [[hot]] → [[NOW]] → [[index]] before modifying files under `C:\development\` or this vault. Status below is a **path map**; live status is always [[hot]] / [[NOW]].

---

## Active Project Topology & Root Map

| Project | Location | Primary Tech Stack | Status (see [[hot]]) |
|---|---|---|---|
| **COMMS LINK** | `C:\development\projects\wisense_decompression` | Flutter · On-Device Gemma 2B-IT | Code-complete & pushed; next = Android packaging (Gate C) |
| **Apex Scheduler** | `C:\development\projects\apex\apex` | Flutter · Supabase · FCM · Sentry | Code-complete & pushed; **human:** apply RLS migration on Supabase, then Gate C |
| **New Horizon** | `C:\development\projects\wisense_new_horizon` | Flutter · Supabase · Duffel API | Tests green; fork reconciliation complete; README still boilerplate |
| **Shared Packages** | `C:\development\packages\` | `wisense_core`, `wisense_ui` | Canonical master source |
| ~~WiSense OS~~ | ~~`...\wisense-os`~~ | — | **DELETED 2026-07-19** — [[Abandoned Projects — Lessons]] |

---

## Andrej Karpathy 4 Core LLM Operating Principles

1. **Think Before Coding**: State assumptions explicitly. If ambiguous, ask — don’t guess.
2. **Simplicity First**: Simplest solution. No speculative abstractions.
3. **Surgical Changes**: Only requested files. No drive-by cleanup.
4. **Goal-Driven Execution**: Testable acceptance criteria; verify tests before commit.

---

## Hard Safety Rules

- Never hardcode or output API keys into code or logs.
- Never invent live project status — read [[hot]] / [[NOW]].
- Run tests in isolated project roots; never bypass failed gates.
- Protocol details: [[agents]]. Governance: [[WiSense Governance — Rules and Protocols]].

---

## Related Notes

- [[hot]] · [[NOW]] · [[agents]] · [[CLAUDE]] · [[VAULT_LINT]]
- [[ENVIRONMENT_MAP]] · [[API_CONTRACT_REGISTRY]] · [[TROUBLESHOOTING_KATAS]] · [[CANONICAL_PACKAGE_MAP]]
