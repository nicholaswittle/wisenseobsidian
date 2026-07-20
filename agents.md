---
title: AI Vault Agent Protocol
tags: [agents, protocol, mindstudio, karpathy, second-brain]
aliases: [agents.md, Vault Agent Instructions]
---

# 🤖 agents.md — AI Vault Operating Protocol

> This file contains the master instructions governing how AI agents (Antigravity, Cursor, Claude Code, Codex, Gemini) interact with this Obsidian vault (`C:\Users\nikwi\Notes`).

---

## 📂 Vault Architecture & Folder Roles

1. `/raw` — **Intake Queue**: Unprocessed web clips, transcripts, research, and prompt drops land here.
2. `/raw/processed` — **Audit Trail**: After ingesting a file from `/raw`, move it here to prevent duplicate processing.
3. `/wiki` — **AI Knowledge Layer**: Atomic concept notes, architecture specs, and tool pages cross-linked to source clips.
4. `/journal` — **Grounded Reflection**: Daily session logs and decision reflections, grounded in vault context.
5. `/crm` — **Contacts & Relationships**: Markdown files per founder, team member, advisor, or vendor contact.
6. `index.md` — **Master Index Pointer**: Catalog of all vault contents, read first before querying.
7. `log.md` — **Append-Only Audit Trail**: History of every AI ingest, query, and synthesis action.

---

## ⚙️ AI Agent Operating Rules

### 1. Intake & Ingestion (`/raw` → `/wiki`)
- Check `/raw` for new `.md` files.
- Extract atomic concepts, technical patterns, tools, and entities.
- Create or update notes in `/wiki/` with back-links `[[source-file]]`.
- Move processed source files from `/raw/` to `/raw/processed/`.
- Append an entry to `log.md` and update `index.md`.

### 2. Querying & Synthesis
- Read `index.md` first to understand existing vault context.
- When answering a user question, cite specific notes using `[[Note Name]]`.
- If a question synthesizes new insights, save a new note in `/wiki/` capturing the answer so the vault grows from questions!

### 3. Karpathy 4 Core Rules Enforcement
- **Think Before Coding**: State all assumptions explicitly.
- **Simplicity First**: Implement the simplest solution. Avoid speculative abstractions.
- **Surgical Changes**: Restrict edits strictly to target files.
- **Goal-Driven Execution**: Verify acceptance criteria and tests pass before committing.

---

Related: [[00_AI_AGENT_MANIFEST]], [[index]], [[log]]
