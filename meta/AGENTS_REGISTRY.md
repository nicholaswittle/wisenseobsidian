---
title: Agent Routing Registry
tags: [agents, routing, coordination, hermes, governance]
aliases: [Agent Registry, Routing Table, Who Does What]
date: 2026-07-20
---

# 🧭 Agent Routing Registry

> Machine-readable "who does what" for the WiSense multi-agent system. **Hermes reads this to route work.** Any agent picking up a task checks here first to confirm it's the right owner and to hand off correctly. Source of truth for delegation — supersedes role prose scattered elsewhere.

---

## 🤝 Governance Roles (WiSense Tripartite Protocol)

| Branch | Role | Held by | Rule |
|---|---|---|---|
| **Executive (Builder)** | Proposes plan + diff, implements | **Claude Code** | No file change without a plan/diff first |
| **Legislative (Reviewer)** | Mandatory review before delivery | **Gemini / Antigravity** | No delivery without sign-off (High risk = required) |
| **Judicial (Auditor)** | Independent audit — never self-audit | **External AI** (Gemini → Groq → OpenRouter) | Audit chain must be a *different* agent than the builder |
| **Architect** | Final ratification on major decisions | **Nicholas** | Silence ≠ approval |

See [[company/WiSense Governance — Rules and Protocols]], [[Tripartite Audit Chain]], [[Head of Team Directive]].

---

## 🤖 Agent Capability Table

| Agent | Model / Runtime | Primary Role | Route here for | Do NOT | Notes |
|---|---|---|---|---|---|
| **Hermes** | Hermes 3 (Nous) | Coordinator + vault/memory keeper | Task routing, vault maintenance, multi-session continuity, delegation | Ship product code unreviewed; self-audit its own coordination | Reads this registry + [[hot]] to dispatch. Uses `skills/` + `scratchpad/`. |
| **Claude Code** | Claude Opus 4.8 | Builder (implementation) | Feature code, refactors, vault edits, diffs, this workspace | Deliver without a plan/diff; self-audit | Governed by [[CLAUDE]] + [[agents]]. |
| **Cursor** | (editor-hosted) | UI polish | Swipeable cards, map integration, widget/visual work | Architectural/package changes | Routed via `cursor_inbox`. |
| **Codex** | (OpenAI) | External auditor + watcher | `audit_findings` review, second-opinion audits | Act as sole builder | See [[Audit Findings Loop]]. |
| **Antigravity (Gemini)** | Gemini | Research + dispatcher + reviewer | Web research, rate-limit failover, Legislative review | Final ratification | See [[Antigravity — Brain Sessions and Knowledge]]. |
| **Groq** | llama-3.3-70b-versatile | Secondary auditor | Fast Judicial audit when Gemini is rate-limited | Primary build work | Failover in the audit chain. |

---

## 🔀 Routing Heuristics (for Hermes)

1. **Code change requested** → Claude (Builder) proposes plan+diff → Gemini reviews → external audit if Medium/High risk → Nicholas ratifies High risk.
2. **UI/visual polish** → Cursor.
3. **Research / unknown facts** → Antigravity (Gemini). File results in `raw/` for codification.
4. **Audit / second opinion** → Codex or Groq (never the agent that built it).
5. **Vault upkeep** (index/hot/log/codification) → Hermes or Claude.
6. **Ambiguous or High-risk architectural decision** → stop, escalate to Nicholas. Silence ≠ approval.

---

Related: [[Home]], [[index]], [[CLAUDE]], [[agents]], [[company/WiSense Governance — Rules and Protocols]], [[Head of Team Directive]], [[Hermes 3 Agent Memory Architecture]]
