---
title: Hermes 3 Agent Memory Architecture & Vault Integration
tags: [hermes, nous-research, memory, RAG, agentic, architecture]
aliases: [Hermes 3 Architecture, Hermes Agent Memory]
date: 2026-07-19
---

# 🧠 Hermes 3 Agent Memory Architecture & Vault Integration

> Technical reference detailing how **Nous Research's Hermes 3** model and **Hermes Agent** framework utilize an Obsidian Vault as an autonomous long-term memory disk, procedural skill store, and tool execution engine.

---

## 🏛 13-Point Architectural Alignment Matrix

| # | Architectural Mechanism | Role in Hermes Ecosystem | Obsidian Vault Implementation | Alignment Status |
|---|---|---|---|---|
| **1** | **Procedural Skill Library (`skills/`)** | Dynamically loads task recipes (`agentskills.io` standard) on demand. | `skills/HERMES_PROCEDURAL_SKILLS.md` | 🟢 **100% Installed** |
| **2** | **Internal Monologue Store (`scratchpad/`)** | Persists `<scratch_pad>` reasoning for multi-session auditability. | `scratchpad/HERMES_SCRATCHPAD_PROTOCOL.md` | 🟢 **100% Installed** |
| **3** | **Dynamic Context Pointer Layer** | Prevents token window overflow using a 1-page catalog. | `00_AI_AGENT_MANIFEST.md` + `index.md` | 🟢 **100% Installed** |
| **4** | **Zettelkasten Graph Memory** | Navigates technical concepts via Wikilinks (`[[Note]]`). | `wiki/` folder + `kepano/obsidian-skills` rules | 🟢 **100% Installed** |
| **5** | **Structured XML Tool Registration** | Hermes 3 expects tools defined in clean XML/JSON schemas (`<tools>`). | `API_CONTRACT_REGISTRY.md` | 🟢 **100% Installed** |
| **6** | **Self-Correcting Error Katas** | Provides 1-line recovery recipes when command execution fails. | `TROUBLESHOOTING_KATAS.md` | 🟢 **100% Installed** |
| **7** | **Deduplicated Intake Pipeline** | Stages raw documents, extracts concepts, archives sources. | `raw/` $\rightarrow$ `raw/processed/` + `sync_obsidian_vault.ps1` | 🟢 **100% Installed** |
| **8** | **Environment State Grounding** | Provides explicit active network ports, URLs, and token paths. | `ENVIRONMENT_MAP.md` | 🟢 **100% Installed** |
| **9** | **Git Version Control & Diff Audit** | Turns every agent modification into a reviewable Git commit diff. | `git init` + `github.com/nicholaswittle/wisenseobsidian` | 🟢 **100% Installed** |
| **10** | **Persona-Scoped Agent Steering** | Adapts system prompts dynamically based on active role directives. | Section 7 in `agents.md` (`Architect`, `Security`, `Travel`, `Flutter`) | 🟢 **100% Installed** |
| **11** | **Automated Background Refresh** | Periodically syncs intake, document conversion, and Git commits. | `register_vault_task_scheduler.ps1` (`WiSenseVaultAutoSync`) | 🟢 **100% Installed** |
| **12** | **Live Codebase & Repo Linkage** | Connects vault Markdown notes directly to software codebase roots. | Live Codebase Index in `index.md` | 🟢 **100% Installed** |
| **13** | **Local Vector Similarity & RAG** | Performs semantic retrieval over non-keyword-matching notes. | `OBSIDIAN_PLUGINS_MANIFEST.md` + Ollama `11434` probes | 🟢 **100% Installed** |

---

Related: [[Home]], [[00_AI_AGENT_MANIFEST]], [[agents]], [[index]], [[HERMES_PROCEDURAL_SKILLS]], [[HERMES_SCRATCHPAD_PROTOCOL]]
