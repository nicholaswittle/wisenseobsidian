---
title: Mason — Scalable AI Brain with Hermes + Obsidian — Video Reference
tags: [reference, youtube, hermes, obsidian, memory, google-drive]
aliases: [Mason Hermes Brain, I9W6NhFkGAI, Dancing Chicken Media]
---

# Mason — Scalable AI Brain with Hermes + Obsidian

**Source:** https://www.youtube.com/watch?v=I9W6NhFkGAI
**Topic:** Building a scalable memory brain for Hermes agent using Obsidian + Google Drive

## The problem this solves

AI agents (Hermes, OpenClaw) forget things you just talked about. No rhyme or reason — they just lose context. A memory brain in Obsidian fixes this by indexing all context (clients, emails, YouTube videos, research) into a connected graph that the agent reads before responding.

## The setup (Mason's stack)

1. **Hermes agent** — running on a VPS (he has a separate setup video for this)
2. **Obsidian** — the memory layer, installed locally
3. **Google Drive** — cloud backup + sync layer. Vault lives inside a Google Drive folder so it's backed up automatically and accessible from anywhere
4. **Google Cloud OAuth** — connects the Hermes agent to Google Drive so it can read/write the vault

## Step-by-step setup

### 1. Google Drive setup
- Create a **separate Gmail account** for the agent (not your personal one). Name it something like "Hermes" or your business name.
- Create a folder called `do not delete` in that Google Drive
- Inside it, create a folder for your vault (e.g. `vault` or your company name)
- Install Google Drive for desktop so the folder syncs locally

### 2. Obsidian setup
- Open Obsidian → "Create new vault"
- Point it at the Google Drive folder you just created
- The vault is now synced to Google Drive automatically

### 3. Google Cloud OAuth (connect agent to Drive)
- Go to Google Cloud Console
- Create a new project (name it after your agent)
- Enable Google Drive API
- Set up OAuth consent screen (external, add yourself as test user)
- Create OAuth client ID (application type: Desktop app)
- Download the JSON credentials file
- Share the Google Drive folder with the new Gmail account (editor access)
- Run the OAuth authorization flow in Hermes — it gives you a localhost URL to paste back

### 4. Hermes agent setup
- Give Hermes the setup prompt (he posts it in the video description)
- The agent finds the Google Drive folder, creates the folder structure, and starts building the memory graph
- It creates: security rules, capture rules, agent instructions, Google Drive index, and customizes based on what the agent already knows

## Key insights from the video

1. **The agent is smart — you just have to guide it.** Half the work is troubleshooting and understanding how to talk to it. People get frustrated and give up, but it can do anything if you give it the right tools.

2. **Memory compounds over time.** Over months, the agent indexes clients, emails, videos, research — building a "neural network of data" where one topic connects to another, just like human memory.

3. **Ask the agent to improve itself.** Message your agent: "How can we improve this memory even more? We want to set up our memory to be really scalable over the next couple years and not break when we get a ton of information." The agents can improve themselves if you set them up correctly and talk to them the right way.

4. **Use a separate Google account.** Don't connect your agent to your personal Google Drive. Create a dedicated account for security isolation.

5. **Google Drive as the sync layer.** The vault lives in a Google Drive folder so it's backed up automatically and the agent (on a VPS) can access it via the Drive API.

## How this maps to your setup

**What you already have:**
- Hermes agent ✓ (you're talking to it right now)
- Obsidian vault ✓ (`C:\Users\nikwi\Notes\` with 40+ notes)
- Memory layer ✓ (governance, project references, code references, audit history)
- claude-obsidian plugin ✓ (automates organization)

**What Mason does that you don't:**
- **Google Drive sync** — your vault is local only. If your machine dies, the vault is gone. Mason's vault lives in Google Drive so it's backed up + accessible from his VPS.
- **Separate Google account for the agent** — security isolation. Your agent doesn't have a dedicated Google account.
- **OAuth connection** — his Hermes agent can read/write the vault via Google Drive API from his VPS. Your vault is only accessible from your local machine.

**What you could do (if you want cloud backup + VPS access):**
1. Create a dedicated Gmail account for WiSense
2. Install Google Drive for desktop
3. Move (or copy) your vault into the Google Drive folder
4. Set up Google Cloud OAuth so Hermes can access it remotely
5. This gives you: automatic backup, access from anywhere, and the ability to run Hermes on a VPS that reads the same vault

**Alternatively (simpler):** Just back up the vault to a cloud service you already use. You have Dropbox installed (I saw it in your home directory). You could symlink or copy `C:\Users\nikwi\Notes\` to a Dropbox folder for automatic backup without the full Google Drive + OAuth setup.

Related: [[KJ AI Second Brain — Video Reference]], [[Claude Code Agentic OS — Video Reference]], [[Obsidian Essentials — Video Reference]], [[Working Stack — Claude CLI and Ollama]], [[Home]]