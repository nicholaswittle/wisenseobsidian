---
title: Obsidian Essentials — Video Reference
tags: [reference, youtube, obsidian, fundamentals]
aliases: [Obsidian 80% guide, z4AbijUCoKU, Ideaverse]
---

# Obsidian Essentials — Video Reference

**Source:** https://www.youtube.com/watch?v=z4AbijUCoKU
**Topic:** The essential 80% of Obsidian features — capture, link, organize

## Why Obsidian

Three reasons to pick it over other note apps:
1. Near-zero friction to capture your own ideas
2. Connect ideas using links and backlinks
3. Improve thinking/learning/writing with an offline, local-first format you own

**Key point:** Obsidian is just a folder of markdown files on your computer. Even if Obsidian disappears, you can open your notes in any markdown reader. You own your data.

## The basics

- **Vault** = a folder on your computer. Obsidian just views it.
- **Notes** = individual `.md` files. Future-proof markdown format.
- **Sync** = use any cloud storage (Dropbox, iCloud) or Obsidian Sync (paid, end-to-end encrypted, version history)

## Creating and linking notes

- Type `[[Note Name]]` to create a link. If the note doesn't exist yet, it shows as a placeholder (uncreated) in the graph view.
- `Cmd+click` (Mac) / `Ctrl+click` (Windows) on a placeholder link to create the actual note.
- Highlight text + `[[` to wrap a selection in a link.
- `![[Note Name]]` embeds an entire note inside another.
- `![external link]` displays an image if the link is one.
- Backlinks panel (bottom right) shows every note that links TO the current note.

## Critical settings (do these first)

1. **Settings → Files and Links → "Automatically update internal links"** — ON. Without this, renaming a note breaks all links to it.
2. **Default folder for attachments** — set to a dedicated folder (avoids cluttering the sidebar with images).
3. **Appearance → Themes** — install a theme (AnuPuchin is the base for the "Soft Paper" theme recommended in the video).

## Gotchas to avoid

1. **Don't import everything from your old notes app.** Start fresh. Link your own thoughts. Don't drown in old clutter.
2. **Keep it simple with plugins.** Don't start with advanced tables, kanban boards, dataview dashboards on day one. Focus on linking first.
3. **Don't over-folder your ideas/knowledge.** Structure must be earned. Keep things in big buckets until patterns naturally show up. Categories for ideas are fuzzy and brittle.
4. **Don't put off learning hotkeys.** The faster you work, the more you'll enjoy it.

## Essential hotkeys (Mac → Windows: Cmd=Ctrl, Option=Alt)

| Action | Mac | Windows |
|--------|-----|---------|
| Bold | Cmd+B | Ctrl+B |
| Italic | Cmd+I | Ctrl+I |
| Find on page | Cmd+F | Ctrl+F |
| New tab | Cmd+T | Ctrl+T |
| Close tab | Cmd+W | Ctrl+W |
| Quick switcher (find/open note) | Cmd+O | Ctrl+O |
| Command palette | Cmd+P | Ctrl+P |
| Navigate back/forward | Cmd+Option+←/→ | Ctrl+Alt+←/→ |
| Add property | Cmd+; | Ctrl+; |

## Markdown basics

- `# Heading`, `## Subheading` — headings
- `**bold**`, `*italic*` — formatting
- `~~strikethrough~~`, `==highlight==` — more formatting
- `> quote` — blockquote
- `- item` — bullet list
- `1. item` — numbered list
- `- [ ] task` — checklist
- `---` — horizontal divider
- `` `code` `` — inline code
- Triple backtick — code block
- `[link text](url)` — external link

## Organizing notes (Ideaverse structure)

The video recommends a 3-folder top-level structure:

| Folder | Purpose | Examples |
|-------|---------|---------|
| **Atlas** | Timeless ideas and knowledge | Concepts, principles, reference notes |
| **Calendar** | Time-based notes | Daily notes, journal entries |
| **Efforts** | Time-bound work | Projects, tasks, productivity |

**Key principle:** Build folder structure only as it's earned. Make the biggest-picture categories possible. Use links, quick switcher, and graph view to find notes — don't rely on deep folder hierarchies.

**Maps of Content (MOCs):** Notes that organize and link other notes together by topic/theme. These create strong connective hubs in the graph view. Prefer these over tags for organization.

## Note formats

- Standard text markdown
- Embed images (drag and drop)
- Attach PDFs, audio, documents
- Audio recorder core plugin for voice memos
- Embed YouTube videos and tweets
- Tables (Cmd+P → "insert table")
- Templates core plugin for reusable note formats
- Properties (Cmd+;) — metadata without cluttering content (created date, checkboxes, links, text, numbers)
- Bases — sort/update notes by type
- Canvas — virtual whiteboard for brainstorming

## AI integration

Obsidian doesn't have built-in AI. You choose how it integrates:
- Claude for asking questions, talking to notes, deep research, auto-populating properties
- Always back up notes before trying new AI features
- Keep a separation between your original thinking (Ideaverse) and AI-generated notes

## How this maps to your vault

Your vault at `C:\Users\nikwi\Notes\` already follows the key principles:
- Links and backlinks ✓ (40+ cross-linked notes)
- Start fresh, don't import clutter ✓
- Maps of Content pattern ✓ (Home.md is your MOC)
- Daily notes ✓ (2026-07-19.md)
- AI integration ✓ (Claude via claude-obsidian plugin, Hermes, CLAUDE.md)

**What you could add from this video:**
- The Atlas/Calendar/Efforts folder structure (if you want more organization than flat root notes)
- Maps of Content notes for major topics (a "Travel App MOC" linking all New Horizon notes, a "Launch Prep MOC" linking all three app launch checklists)
- Properties on notes (Cmd+;) for status tracking beyond the frontmatter we already use

Related: [[KJ AI Second Brain — Video Reference]], [[Claude Code Agentic OS — Video Reference]], [[Home]]