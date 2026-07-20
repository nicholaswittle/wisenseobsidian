---
title: NetworkChuck — Why I Switched to Hermes — Video Reference
tags: [reference, youtube, hermes, networkchuck, memory, skills]
aliases: [NetworkChuck Hermes, QQEgIo4Juxg, Ron Weasley IT agent]
---

# NetworkChuck — Why I Switched to Hermes

**Source:** https://www.youtube.com/watch?v=QQEgIo4Juxg
**Channel:** NetworkChuck
**Topic:** 5 reasons to switch from OpenClaw to Hermes — memory, skills, self-improvement, vibe

## The 5 reasons

### 1. The vibe
- Nous Research is a company with a mission and purpose (open source AI)
- The website and terminal experience have a distinct aesthetic — they care about the feel
- Origin story: "a bunch of hackers on a Discord trying to figure out if we can make open source AI"
- Not a random project — a company that dogfoods its own tools (their Hermes agents run in their Discord as team members)

### 2. Memory (the real differentiator)
Hermes and OpenClaw both have memory files (USER.md, MEMORY.md) loaded into the system prompt. The difference is HOW Hermes uses them:

**Hard limits force curation:**
- USER.md: 1,375 character max
- MEMORY.md: 2,200 character max
- When full, the agent must delete something — it curates what matters
- This forces the agent to distill what's actually important about you and your work
- OpenClaw doesn't do this well — it bloats over time

**Active memory updates during sessions:**
- Hermes nudges every ~10 turns to check if memory/user should update
- Happens DURING the session, not just on compact/new session
- OpenClaw only does this on session boundaries
- This makes Hermes feel more "present" — it's actively learning while you talk

**Honcho add-on (advanced memory):**
- Peer service that runs alongside Hermes
- Every message is also sent to Honcho
- Honcho reasons over your messages and builds a "peer card" — who you are
- Gets smarter over time as more messages are sent
- Feeds context back to Hermes dynamically — "what does the agent need to know about you right now in this moment?"
- Can run locally or in the cloud

### 3. It learns (the self-improvement loop)
This is the skills system — covered in reason #4 below.

### 4. Skills system (the best reason)
Hermes has a skill system where the agent saves reusable procedures as skills. When it solves a complex problem or discovers a workflow, it persists that knowledge as a skill that loads into future sessions.

**Jeff Quesnelle (Nous Research co-founder) example:**
- They have a Hermes agent in their Discord as a system admin
- People report errors, the agent debugs them
- Over time, that agent built up a huge skill set of infrastructure debugging skills
- The agent got better at its job by accumulating skills from real interactions

**NetworkChuck's IT agent example:**
- Built "Ron Weasley" — an IT troubleshooting wizard
- Gave it a persona (SOUL.md), told it who Chuck is (USER.md), let it learn the environment (MEMORY.md)
- The agent builds skills as it solves IT problems

### 5. Multi-platform + it's fun
- Runs in terminal, Telegram, Discord, and more
- The terminal experience is genuinely enjoyable
- His wife has her own agent called "Honey" — it's her BFF
- He felt comfortable enough to give it to his wife (the highest trust bar)

## Setup walkthrough

1. Install on a VPS (Hostinger) or locally — one command: `curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash`
2. Choose inference model: OpenAI Codex (uses ChatGPT subscription), Grok (SuperGrok), OpenRouter, local (LM Studio + Qwen), or any provider
3. Set up messaging gateway: Telegram (easiest — BotFather creates a bot, paste token)
4. Configure user authorization (only you can talk to your agent)
5. Run as systemd service (VPS) or default (laptop)
6. Start talking — terminal or Telegram

## Key quotes from Jeff Quesnelle (co-founder)

On the vibe: "As much as we're online, you and me are, like, terminally online. We're also creatures of this world, and our sight and our taste and all of that are part of who we are. We spend a lot of time to make sure that we have that vibe feel behind it."

On dogfooding: "We actually have our agents in our Discord channels and have them running, and we talk to them like they're team members. There's a Hermes agent who's the system admin. Someone comes into our Discord and says 'something happened, I got this error message.' We're able to just talk to the Hermes agent. And what's really amazing is that through having just done this, that agent has built up a huge skill set of skills particularly related to debugging our infrastructure."

## How this maps to your setup

**You're already running Hermes** — this is the agent you're talking to right now. Everything NetworkChuck describes, you already have:

- Memory (USER.md + MEMORY.md) ✓ — I've been writing to these all session
- Hard limits (1,375 / 2,200 chars) ✓ — I hit the limit earlier and had to batch operations
- Active memory updates ✓ — I update memory during our conversation
- Skills system ✓ — I load skills (hermes-agent, youtube-content, obsidian, etc.)
- Self-improvement loop ✓ — I offered to save procedures as skills after complex tasks

**What you could do that the video shows:**
1. **Give your Hermes agent a persona** — edit SOUL.md at `C:\Users\nikwi\AppData\Local\hermes\SOUL.md` to give me a WiSense-specific identity (currently it's the default Hermes blurb)
2. **Set up Honcho** — `hermes memory setup` → add Honcho for the advanced peer-card memory layer. This would make me learn your preferences more actively over time.
3. **Connect Telegram** — so you can talk to me from your phone, not just the terminal
4. **Build a skill library** — as I do recurring WiSense tasks (app audits, commit plans, launch checklists), save them as skills so future sessions run them the same way

Related: [[Mason — Scalable AI Brain with Hermes]], [[Karpathy LLM Wiki — Video Reference]], [[Working Stack — Claude CLI and Ollama]], [[Home]]