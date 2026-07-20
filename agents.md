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

### 1. The Gardener & Soil Paradigm
- **Human User (Gardener)**: Drops raw thoughts, articles, YouTube clips, and prompt notes into `/raw`.
- **AI Agent (Soil)**: Handles structure, atomic note synthesis in `/wiki/`, deduplication into `/raw/processed/`, indexing in `index.md`, and logging in `log.md`.

### 2. Obsidian Native Syntax Standards (`kepano/obsidian-skills`)
- **Wikilinks**: Always link notes using `[[Note Name]]` or `[[Note Name|Alias]]` instead of raw URL markdown links.
- **Callouts**: Use GitHub / Obsidian callouts (`> [!NOTE]`, `> [!WARNING]`, `> [!TIP]`, `> [!IMPORTANT]`) for visual callouts.
- **YAML Frontmatter**: Every note created in `/wiki/` MUST begin with valid YAML frontmatter:
  ```yaml
  ---
  title: Note Title
  tags: [topic, concept, category]
  aliases: [Alternative Title]
  date: YYYY-MM-DD
  ---
  ```
- **Dataview Compatibility**: Use standard key-value pairs (`key:: value`) or YAML fields for queryability.

### 3. Intake & Ingestion Pipeline (`/raw` → `/wiki`)
1. Scan `/raw` for unprocessed `.md` files.
2. Extract core concepts, tools, architectures, and entities into atomic pages in `/wiki/`.
3. Ensure all generated `/wiki/` pages contain explicit back-links `[[source-clip-name]]`.
4. Move processed raw files from `/raw/` to `/raw/processed/`.
5. Append an event entry to `log.md` and update `index.md`.
6. Create a Git commit (`git commit -m "feat(vault): ingest [clip-name]"`) to preserve an auditable diff.

### 4. Continuous Synthesis
- When responding to complex user queries, check if the response contains reusable architectural insights.
- If so, automatically synthesize a new note in `/wiki/` capturing the insight, cross-linking it to related notes so the vault grows from every question asked!

### 5. Karpathy 4 Core Rules Enforcement
- **Think Before Coding**: State all assumptions explicitly before taking action.
- **Simplicity First**: Implement the simplest solution. Avoid speculative abstractions.
- **Surgical Changes**: Restrict edits strictly to target files.
- **Goal-Driven Execution**: Verify acceptance criteria and tests pass before committing.

---

Related: [[00_AI_AGENT_MANIFEST]], [[index]], [[log]]
