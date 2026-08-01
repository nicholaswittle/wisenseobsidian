---
title: Tripartite Audit Chain
tags: [governance, audit, judicial, reference]
aliases: [Audit Chain, Tripartite Audit, External Auditor]
---

# Tripartite Audit Chain

Source: `C:\development\WI-SENSE_TRIPARTITE_PROTOCOL.md`, `C:\Users\nikwi\CLAUDE.md`

The Judicial branch is always fulfilled by an external AI — Codex/Claude/Hermes may never self-audit. This applies regardless of how the task arrives.

## The 3-Tier Audit Chain (attempt in order)

| Tier | Auditor | Model | Trigger |
|---|---|---|---|
| 1 | Gemini | `gemini-2.5-flash` | Primary hand-off — initial generation, logical structure, architectural intent |
| 2 | Groq | `llama-3.3-70b-versatile` | Gemini unavailable or over quota — high-speed secondary validation, syntax, logic flow |
| 3 | OpenRouter | `openrouter/auto` | Groq fails — final comprehensive ratification (frontier model: Claude 3.5 Sonnet, GPT-4o) |

## Execution Requirements

- Every task executed by any agent must pass through this Tripartite Audit before final ratification.
- If any tier fails, the task is flagged as `Failed` in the result artifact and sent back to the queue for human review or re-generation.
- Dry runs or system heartbeat tasks must simulate this chain and explicitly log success/failure to `global_status.md`.
- Log the auditor used and any hand-off reason in the Completion Report.

## Audit Outcomes

- **PASS** — output is forwarded to the Executive for the Completion Report.
- **FAIL** — findings are returned to the Legislative branch with specific, actionable notes. The cycle repeats until PASS.

## Hand-off Protocol

If the primary architect (Claude) encounters a token limit, API error, or communication failure, write the reason and full task context to `global_status.md`. The receiving auditor reads `global_status.md` to resume without loss of state.

Related: [[WiSense Governance — Rules and Protocols]], [[MCA and MDT]], [[Travel Data Integrity]]