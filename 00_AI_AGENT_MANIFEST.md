---
title: AI Agent Manifest & Operating Rules
tags: [ai, manifest, rules, entrypoint, karpathy]
aliases: [AI Manifest, Agent Rules, Start Here]
---

# 🤖 00 AI Agent Manifest & Operating Rules

> **CRITICAL**: Every AI session (Antigravity, Cursor, Claude Code, Codex, Gemini) MUST read this note before starting work or modifying files in `C:\development\`.

---

## 🗺 Active Project Topology & Root Map

| Project | Location | Primary Tech Stack | Status |
|---|---|---|---|
| ~~WiSense OS~~ | ~~`C:\development\projects\wisense-os`~~ | ~~Python 3.14 · Flask · Flutter Desktop~~ | **DELETED 2026-07-19** — see [[Abandoned Projects — Lessons]] |
| **Apex Scheduler** | `C:\development\projects\apex\apex` | Flutter · Supabase · FCM · Sentry | Merge Conflict Fix Needed |
| **COMMS LINK** | `C:\development\projects\wisense_decompression` | Flutter · On-Device Gemma 2B-IT | 100% Green (59/59 tests) |
| **New Horizon** | `C:\development\projects\wisense_new_horizon` | Flutter · Supabase · Duffel API | 100% Green (117/117 tests) |
| **Shared Packages** | `C:\development\packages\` | `wisense_core`, `wisense_ui` | Canonical Master Source |

---

## ⚖️ Andrej Karpathy 4 Core LLM Operating Principles

1. **Think Before Coding**: State all assumptions explicitly in the plan draft. If requirements are ambiguous, ask for clarification instead of guessing.
2. **Simplicity First**: Implement the simplest possible solution. Avoid speculative abstractions, unused features, or unnecessary wrapper code.
3. **Surgical Changes**: Restrict modifications strictly to requested files. Never refactor, reformat, or "clean up" unrelated code outside the requested scope.
4. **Goal-Driven Execution**: Define explicit, test-verifiable acceptance criteria before writing code. Verify that unit tests pass before committing.

---

## 🛑 Hard Safety Rules (No-Touch Zones)

- **Secrets & Credentials**: Never hardcode or output API keys (`sk-...`, `ghp_...`, `AIza...`) into code or logs. Redact secrets automatically.
- **Unapproved File Mutations**: Never create or modify `.wisense/CONTEXT.md` or configuration files on disk without explicit user approval.
- **Isolated Testing**: Always run tests in isolated project roots (`.venv\Scripts\python -m pytest` or `flutter test`). Never bypass failed test gates.
- **Model Truthfulness**: Never invent uninstalled local models or fake token costs. Use actual registered profiles in `ModelRegistry`.

---

## 🔗 Related Notes

- [[WiSense Governance — Rules and Protocols]]
- [[ENVIRONMENT_MAP]]
- [[API_CONTRACT_REGISTRY]]
- [[TROUBLESHOOTING_KATAS]]
- [[CANONICAL_PACKAGE_MAP]]
