---
title: Audit Findings Loop
tags: [process, audit, vault]
aliases: [audits, audit handoff]
---

# Audit Findings Loop

**Updated 2026-07-19:** The Local Agent Work Center handoff folder is **retired** (see [[Abandoned Projects — Lessons]]). Audits live in two places now.

## Where audits go

1. **This vault** — durable summaries agents and Nicholas will re-read (example: [[output/Apex Security Audit 2026-07-19]])
2. **Per-repo `audit/`** — full dated write-ups next to the code (example: `C:\development\projects\apex\apex\audit\AUDIT_2026-07-19.md`)

## Conventions

- Findings-first; exact file paths and severity
- Security / payment / RLS treated as launch blockers until verified
- No real API or money-spending commands unless the note explicitly authorizes
- Prefer Claude CLI + Ollama for the audit session ([[Working Stack — Claude CLI and Ollama]])

## Active audit pointers

| Project | Vault note | Repo artifact |
|---|---|---|
| Apex | [[output/Apex Security Audit 2026-07-19]] | `apex/apex/audit/AUDIT_2026-07-19.md` |
| COMMS LINK | [[COMMS LINK]] (live status) | — |
| New Horizon | [[New Horizon]] (live status) | — |

## Retired

- `C:\development\projects\local-agent-work-center\audit_findings\` — do not write new notes there

Related: [[Apex Scheduler]], [[COMMS LINK]], [[New Horizon]], [[Tripartite Protocol]], [[Home]]
