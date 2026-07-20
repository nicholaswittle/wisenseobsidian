---
title: Obsidian Plugins Manifest & Security Registry
tags: [obsidian, plugins, manifest, governance, releases]
aliases: [OBSIDIAN_PLUGINS_MANIFEST, Plugin Registry]
---

# 🔌 OBSIDIAN PLUGINS MANIFEST & SECURITY REGISTRY

> Official audit registry of installed and verified Obsidian community plugins, based on the **`obsidianmd/obsidian-releases`** standard.

---

## 🏛 Verified Installed Community Plugins

| Plugin ID | Display Name | Category | Primary Job | Provider Routing | Audit Status |
|---|---|---|---|---|---|
| `obsidian-smart-connections` | **Smart Connections** | Retrieval | Local vector embeddings & semantic search side panel | Local Ollama / On-Device | Verified (`obsidian-releases`) |
| `obsidian-copilot` | **Copilot for Obsidian** | Chat & QA | Interactive chat sidebar over vault context | Local Ollama (`11434`) | Verified (`obsidian-releases`) |
| `obsidian-smart-composer` | **Smart Composer** | Composition | Cursor-style in-editor diff generation | Local Ollama (`11434`) | Verified (`obsidian-releases`) |
| `dataview` | **Dataview** | Indexing | High-performance metadata query & table engine | Local Native | Verified (`obsidian-releases`) |
| `obsidian-kanban` | **Kanban** | Tasks | Markdown-based visual project boards | Local Native | Verified (`obsidian-releases`) |

---

## 🔒 Security & Verification Rules

1. **Official Registry Requirement**: Every community plugin installed MUST be verified and listed in the official `obsidianmd/obsidian-releases` directory.
2. **Local-First Privacy**: Community AI plugins must route through local Ollama (`http://localhost:11434`) or local embedding models. No raw unencrypted cloud sync without consent.
3. **Manifest Integrity**: Any custom WiSense plugins must include a valid `manifest.json` adhering to Obsidian's official schema (`id`, `name`, `version`, `minAppVersion`, `isDesktopOnly`).

---

## 🛠 Custom WiSense Plugin Manifest Spec (`manifest.json`)

```json
{
  "id": "wisense-os-bridge",
  "name": "WiSense OS Native Bridge",
  "version": "1.0.0",
  "minAppVersion": "1.0.0",
  "description": "Native bridge connecting Obsidian notes to WiSense OS local AI engine on port 5050.",
  "author": "WiSense AI Team",
  "isDesktopOnly": true
}
```

Related: [[00_AI_AGENT_MANIFEST]], [[agents]], [[ENVIRONMENT_MAP]]
