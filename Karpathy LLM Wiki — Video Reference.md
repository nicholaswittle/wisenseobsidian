---
title: Karpathy LLM Wiki — Video Reference
tags: [reference, youtube, karpathy, llm-wiki, obsidian, claude-code]
aliases: [Jamie LLM Wiki, iXd0t60YmMw, Teachers Tech LLM Wiki]
---

# Karpathy LLM Wiki — Video Reference

**Source:** https://www.youtube.com/watch?v=iXd0t60YmMw
**Channel:** Teachers Tech (Jamie)
**Topic:** Karpathy's LLM Wiki pattern — building a persistent AI knowledge base that compounds

## The problem this solves

Standard RAG (retrieval augmented generation) starts from zero every time. You upload documents, ask a question, the AI searches chunks and answers. Ask the same question tomorrow — it does ALL of that work again. Nothing saved. Nothing compounds. Every question starts from scratch.

Karpathy's LLM Wiki flips this: the AI reads your documents ONCE and builds a structured wiki out of them. When you ask a question, the AI reads the pre-built wiki, not the raw documents. The synthesis is already done. Connections are already made.

## Karpathy's framing

> "Think of Obsidian as the IDE, the LLM as the programmer, and the wiki as the codebase. You rarely write the wiki yourself. The AI does the writing and organizing. You focus on what goes in and what questions to ask."

## The 3-layer structure

| Layer | Folder | Purpose | Who writes |
|-------|--------|---------|------------|
| 1. Raw sources | `raw/` | Original documents (PDFs, articles, meeting notes). Read-only — AI reads but never changes | You |
| 2. Wiki | `wiki/` | Markdown pages the AI creates and maintains — index, concept pages, entity pages, comparisons, all interlinked | AI |
| 3. Schema | `CLAUDE.md` | Rules document — tells AI how to structure the wiki, handle new sources, format pages | You (with AI help) |

## What the schema (CLAUDE.md) contains

1. **Purpose** — what the knowledge base is about (one line you customize)
2. **Folder structure** — where raw sources go, where wiki output goes
3. **Ingest workflow** — when you add a source: read document → extract key concepts → create/update wiki pages → update index → log changes
4. **Page formatting rules** — every page has summary at top, claims reference sources, pages link to related concepts
5. **Question answering behavior** — consult wiki first, cite sources, flag uncertainty

## How it works in practice

1. Drop a source into `raw/` (PDF, markdown, text — Claude reads all natively)
2. Tell Claude: "I just added a new source to the raw folder. Please read it and update the wiki."
3. Claude reads the document, proposes new wiki pages + updates to existing pages
4. You approve, Claude writes the pages with `[[links]]` between concepts
5. Next source: Claude doesn't just create new pages — it updates existing pages with new information
6. Ask a question: Claude reads the wiki (not raw docs), connects dots across multiple pages, cites specific wiki pages

## Lint (wiki maintenance)

Like a code linter checks code for problems, periodically ask Claude to lint the wiki. It checks for:
- Contradictions between pages
- Outdated claims
- Orphan pages (no links pointing to them)
- Concepts mentioned but don't have their own page yet
- Broken links

Command: "Please lint the wiki." Claude gives a report and offers to fix issues.

## What makes this different from RAG

| RAG | LLM Wiki |
|-----|----------|
| Searches raw documents every question | Reads pre-built wiki |
| Starts from zero every time | Compounds over time |
| No memory between questions | Persistent knowledge base |
| Synthesis done per question | Synthesis already done |
| Chunks may miss connections | Pages explicitly link related concepts |

## Use cases from the video

- **Students/researchers** — build wiki as you read papers. End up with structured knowledge base, not pile of highlighted PDFs.
- **Teachers** — feed curriculum docs, PD materials, articles. Personal teaching wiki that grows.
- **Business** — feed meeting notes, call transcripts, project docs. New team members browse organized wiki instead of digging through Slack.
- **Curious people** — track learning from books, podcasts, articles. Personal encyclopedia.

## Limitations (honest)

1. **Personal scale** — Karpathy talks about wikis of ~100 articles. Tens of thousands of pages need more infrastructure than markdown files.
2. **Garbage in, garbage out** — wiki is only as good as sources you feed it. You still curate.
3. **Needs a coding agent** — Obsidian alone does nothing. The AI (Claude Code, Codex, etc.) is the engine.
4. **AI can make mistakes** — miscategorization, wrong connections. That's why lint exists. Review early output.

## How this maps to your vault

**You're already running this exact pattern.** Your vault at `C:\Users\nikwi\Notes\` has:

- `raw/` folder ✓ — staging area for unprocessed sources
- Root notes (40+) ✓ — the wiki itself (governance, project references, decisions, vendor docs)
- `output/` folder ✓ — final deliverables (audits, plans, checklists)
- `CLAUDE.md` ✓ — the schema file with purpose, structure, ingest workflow, rules
- claude-obsidian plugin ✓ — automates the ingest + lint process
- This very note ✓ — a source I just ingested from `raw/` (your YouTube URL) into a wiki page with links

**What this video confirms you're doing right:**
- The 3-folder structure (raw → wiki → output) matches Karpathy's pattern exactly
- The CLAUDE.md in vault root is the schema
- Cross-linking with `[[wikilinks]]` is the connection layer
- Lint is available via the claude-obsidian plugin

**What you could do that the video shows:**
- Run "Please lint the wiki" periodically — the plugin supports this and it would find orphan notes, broken links, and gaps in your 40+ notes
- Feed more raw sources into `raw/` and let Claude ingest them — the system is set up for it, you just haven't used the raw folder much yet

Related: [[KJ AI Second Brain — Video Reference]], [[Claude Code Agentic OS — Video Reference]], [[Obsidian Essentials — Video Reference]], [[Mason — Scalable AI Brain with Hermes]], [[Home]]