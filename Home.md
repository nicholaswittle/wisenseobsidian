---
title: Home
tags: [home, index]
aliases: [Dashboard, Index]
---

# WiSense Notes

Personal vault for cross-project decisions, launch planning, vendor research, and lessons learned. Kept separate from code repos on purpose — this is for thinking, not for shipping files.

## 🧠 Vault Architecture (curated static reference)

Hand-written, cross-linked knowledge — not an automated intake machine. The 7-folder synthesis pipeline (`wiki/`, `journal/`, `crm/`) was retired 2026-07-20 (never used). See [[agents]] for the full protocol.

- **`raw/`** — Intake inbox for clipped articles, transcripts, and prompt drops awaiting manual codification.
- **`raw/processed/`** — Sources moved here after they've been codified into root notes.
- **`output/`** — Final deliverables (audit reports, launch checklists, decision docs).
- **Root notes** — The knowledge layer: governance, project refs, manifests, decisions, daily logs.
- **[[hot]]** — ~500-word recent-context cache; read first for fast boot.
- **[[agents]]** — Master protocol governing all AI operations in this vault.
- **[[index]]** — Pointer catalog read first before retrieval.
- **[[log]]** — Append-only audit trail of structural changes.

## 🤖 AI Agent Entrypoint & Context Maps

- [[00_AI_AGENT_MANIFEST]] — 1-page AI agent manifest & Karpathy operating rules
- [[ENVIRONMENT_MAP]] — local port registry & environment key map
- [[API_CONTRACT_REGISTRY]] — cross-app JSON payload & data model contracts
- [[TROUBLESHOOTING_KATAS]] — known error signatures & 1-line resolution commands
- [[CANONICAL_PACKAGE_MAP]] — shared package map & sync protocol

## Governance — rules, protocols, and delegations

- [[WiSense Governance — Rules and Protocols]] — root index of all governance
- [[AGENTS_REGISTRY]] — agent routing table: who does what, how Hermes delegates
- [[DECISIONS]] — append-only register of settled decisions + rationale
- [[Mythos 5.5 Persona]] — the agent persona every session uses
- [[Head of Team Directive]] — role assignments + conflict resolution + human veto
- [[Team Workflow]] — 8-step session structure
- [[Governance Protocol]] — 3 pillars, risk levels, ratification rules
- [[Tripartite Audit Chain]] — Gemini → Groq → OpenRouter external audit
- [[MCA and MDT]] — Minimalist Conformance Audit + Material Defect Triage
- [[Travel Data Integrity]] — MDT extension for travel/booking code
- [[System Architect Directive]] — shared package enforcement, design system, testing gate
- [[Workspace Architecture]] — C:\development layout, apps, scripts, shared packages
- [[Task Protocol]] — pre-execution validation + artifact generation
- [[Advisor Profile]] — Nicholas's founder profile + virtual board of advisors
- [[Master Status]] — intelligent dispatcher task history
- [[Antigravity — Brain Sessions and Knowledge]] — Gemini Antigravity memory / skills
- [[Tripartite Protocol]] — quick reference (Executive/Legislative/Judicial)
- [[Audit Findings Loop]] — where audits live now (vault + per-repo `audit/`)

## Priority launches

- [[COMMS LINK]] — on-device decompression, zero cloud. Push `main` + store assets.
- [[Apex Scheduler]] — Jigsy's Brewpub staff scheduling. **Launch blocker: tenancy/RLS + claim/clock races** — see [[Apex Security Audit 2026-07-19]].
- [[New Horizon]] — travel Alignment Engine, Duffel + Supabase. README boilerplate still needs replacing. Fork reconciliation COMPLETE.

## Cross-project decisions

- [[Fork Reconciliation]] — ~~New Horizon's vendored wisense_core/wisense_ui vs canonical~~ **COMPLETE (2026-07-20)** — promoted to canonical, path deps in place.
- [[Parent Repo Cleanup]] — C:\development parent git hygiene after project kills.

## Vendors

- [[Duffel]] — flight booking API (Horizon V2, New Horizon)
- [[Supabase]] — auth + database (Apex, Horizon V2, New Horizon)
- [[Stripe]] — billing, deferred for Apex pilot
- [[WiSense LLC — About Page]] — company info & about page content

## Code reference (for AI agents + quick lookup)

- [[COMMS LINK — Code Reference]]
- [[Apex Scheduler — Code Reference]]
- [[New Horizon — Code Reference]]
- [[Code Reuse Analysis]]

## Video references

- [[KJ AI Second Brain — Video Reference]] — two-layer system (strategy + projects)
- [[Claude Code Agentic OS — Video Reference]] — 3-step agentic OS (architecture + memory + observability)
- [[Obsidian Essentials — Video Reference]] — the 80% of Obsidian features (linking, settings, hotkeys)
- [[Mason — Scalable AI Brain with Hermes]] — Hermes + Obsidian + Google Drive memory brain
- [[Karpathy LLM Wiki — Video Reference]] — the LLM Wiki pattern your vault is based on
- [[NetworkChuck — Why I Switched to Hermes]] — 5 reasons to switch: memory, skills, self-improvement, vibe
- [[Jack — Hermes Agent Masterclass]] — advanced usage: SOUL.md, cron jobs, goals, model routing, delegation

## Final deliverables (output/ folder)

- [[output/Apex Security Audit 2026-07-19]] — Apex multi-tenancy + claim/clock race findings
- [[output/Apex — Merge Conflict Resolution Plan]] — 13 conflicts across 6 files, High risk
- [[output/COMMS LINK — Commit Plan]] — 9 atomic commits, Judicial PASS, executed

## Daily log

- [[2026-07-19]] — vault created; projects audited; agent platforms abandoned; Claude CLI + Ollama chosen; Apex security audit captured.
- [[2026-07-20]] — fork reconciliation complete; vault audit & cleanup.
