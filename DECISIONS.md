---
title: Decisions Register
tags: [decisions, adr, rationale, governance, history]
aliases: [Decisions, ADR, Decision Log]
date: 2026-07-20
---

# ⚖️ Decisions Register

> Append-only record of **settled decisions** — the *why* behind the current state. Any agent (Hermes, Claude, Cursor, Codex, Gemini) checks here before re-opening a question or "helpfully" undoing something deliberate. Newest at top. Never edit a past entry; supersede it with a new one and mark the old `SUPERSEDED`.

Entry format: **date · decision · status · rationale · consequences**. Status ∈ ACTIVE / SUPERSEDED / REVERSED.

---

## 2026-07-20 · Vault AI performance layer — `ACTIVE`
- **Decision**: Add execution + founder-memory layer without reviving auto-wiki: [[NOW]], `customers/`, [[business/Experiment Log]], [[VAULT_LINT]]; thin [[CLAUDE]] to point at [[agents]]; single boot chain.
- **Rationale**: Research (Karpathy LLM Wiki + founder OS) showed status drift and missing tasks/customer truth hurt agents more than missing folders. See research canvas / [[log]] 2026-07-20.
- **Consequences**: Boot order is now [[hot]] → [[NOW]] → [[index]] → note. Live status only in hot/NOW (not duplicated in CLAUDE). Monthly lint via [[VAULT_LINT]]. Customer truth replaces retired `/crm`.

## 2026-07-20 · Vault is a curated static reference — `ACTIVE`
- **Decision**: Treat this vault as hand-written, cross-linked knowledge. Retire the MindStudio auto-synthesis pipeline; delete empty `wiki/`, `journal/`, `crm/` folders and the sync scripts/task.
- **Rationale**: The `raw→wiki` loop never ran in 6 months; all real value came from hand-written manifests. Empty scaffolding actively misled agents.
- **Consequences**: Codification is manual/on-request. Knowledge lives as **root notes**. Boot order extended by AI performance layer decision above. `WiSenseVaultAutoSync` + sync scripts removed. See [[log]] 2026-07-20.

## 2026-07-20 · Fork reconciliation complete — `ACTIVE`
- **Decision**: Promote New Horizon's vendored `wisense_core` (47 files) + `wisense_ui` (19 files) to canonical `C:\development\packages\`; New Horizon consumes canonical via path deps; delete the vendored copies.
- **Rationale**: Ends the long-standing package divergence documented in [[Fork Reconciliation]].
- **Consequences**: All green — core 69/69, UI 21/21, NH 117/117, HV2 7/7. `CLAUDE.md`'s "Known fork" caveat is now historical.

## 2026-07-19 · Deleted dead projects — `ACTIVE`
- **Decision**: Remove `wisense-os`, `my_ai`, `local-agent-work-center`, `command_center` from active status.
- **Rationale**: Abandoned/superseded; see [[Abandoned Projects — Lessons]].
- **Consequences**: Do NOT treat as live. `wisense-os` port 5050 daemon no longer exists (invalidated `wisense-engine-probe`).

## 2026-07-19 · Working stack = Claude CLI + Ollama — `ACTIVE`
- **Decision**: Standardize on Claude Code CLI for build + local Ollama (`11434`) for on-device/model work; abandon other agent platforms as primary.
- **Rationale**: See [[Working Stack — Claude CLI and Ollama]].
- **Consequences**: Community AI plugins route through local Ollama; no unencrypted cloud sync without consent.

## (undated) · Stripe deferred for Apex pilot — `ACTIVE`
- **Decision**: Do not integrate Stripe billing for the initial Apex Scheduler pilot.
- **Rationale**: Out of scope for pilot validation. See [[Stripe]].
- **Consequences**: Revisit post-pilot. *(Confirm/adjust date — carried over from vendor notes.)*

---

Related: [[Home]], [[index]], [[log]], [[hot]], [[WiSense Governance — Rules and Protocols]], [[Fork Reconciliation]], [[Abandoned Projects — Lessons]]
