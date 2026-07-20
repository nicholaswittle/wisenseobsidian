---
title: Jack — Hermes Agent Masterclass — Video Reference
tags: [reference, youtube, hermes, masterclass, memory, cron, goals, models]
aliases: [Jack Hermes Guide, k5NhsF7t68M, Hermes Power User]
---

# Jack — Hermes Agent Masterclass

**Source:** https://www.youtube.com/watch?v=k5NhsF7t68M
**Topic:** Advanced Hermes usage — memory, SOUL.md, Obsidian connection, cron jobs, goals, model routing, delegation

## 1. Memory (the differentiator)

**Memory architecture:**
- `MEMORY.md` — plain markdown file with everything about your environment
- Peer cards — one card per person, covers time and preferences
- Fuzzy index — search across past sessions
- 1-hour prompt cache — keeps token costs lower, remembers recent conversations
- Can now recall what you discussed on specific days ("what were we talking about on Sunday?")

**The 3 memory upgrades Jack recommends:**

### a) SOUL.md — give your agent a soul
- A file that explains who you are, where you live, your context manual for life
- Also how you want it to behave — should it challenge your thinking? Look around corners?
- Without SOUL.md, your agent is generic. With it, it understands your world.

### b) Connect to Obsidian memory system
- Link Hermes to your Obsidian vault folder so it can search your notes dynamically
- Example: "In my Obsidian memory system, I talk a lot about YouTube strategy. Pull me one great insight for intros."
- Hermes searches your local Obsidian files and answers from your own knowledge base
- This is exactly what we've set up — your vault at `C:\Users\nikwi\Notes\`

### c) Connect to meeting notes + email
- Meeting note tools: Granola (his favorite), Grain, Fireflies, Fathom
- Email: connect via Zapier MCP — configure exactly what the agent can do (add labels, archive, draft, but maybe not send)
- Lets you ask "what was the last meeting I was in and one interesting point I made?"

## 2. Background tasks

Use `/background` to run tasks without blocking the conversation. Stack multiple background tasks — they never collide. Example: research the most untapped industry for AI automations while you ask another question.

## 3. Cron jobs (scheduled tasks)

Set up scheduled tasks via `/cron`. Jack's morning brief example:
- Every day at 8am: weather, motivational quote from top entrepreneurs, today's meetings
- At 6am (before the brief): a "dreaming sequence" — Hermes reviews all conversation history, meetings, files, and recommends 3 core actions + 1 non-negotiable for the day

**The dreaming concept:** Hermes grows with you. The more it knows, the better it gets. Having it "dream" overnight means it proactively thinks about what you should do next.

Also: "remind me in 30 minutes to take the dog out" — simple natural language reminders.

## 4. Goals + Super Goals

**`/goal`** — persists for 20 messages. Hermes keeps working until it solves the problem. Like a North Star for the session.

**Super Goals** (Jack's custom system) — adds a human-AI handshake:
- Standard goals have no back-and-forth — Hermes just goes until done
- Super Goals break the goal into bite-size chunks and assign actions to EITHER you or Hermes
- Example: "Launch course to 500 sign-ups" breaks into 6 tasks — some Hermes does (build opt-in engine, write course sequence), some you do (record promo videos)
- The handshake means Hermes knows when it needs YOU to do something, not just itself

**Key:** goals must be specific. Don't say "run my business." Say "I want a thumbnail that looks like this" or "a document that hits these criteria."

## 5. Model routing (use the right model for the job)

Jack's model stack:
- **OpenAI Codex** — daily driver (uses ChatGPT subscription)
- **Claude Opus 4.7** — for design, expensive but the king for presentation
- **Claude Sonnet** — nearly as good, cheaper
- **Grok (xAI)** — 1M context window + can search X/Twitter. "What are the most viral tweets about Hermes in the last 48 hours?"
- **DeepSeek V4** — delegate deep research to it (cheaper)
- **Gemini (via Antigravity CLI)** — multimodality, reading videos/images, long context

**The principle:** Don't do deep research with expensive models. Delegate research to cheaper models (DeepSeek), use expensive models (Opus) for design and quality output. Tag the best model for the job.

## 6. Antigravity CLI (replaces Gemini CLI)

The Gemini CLI is deprecated — now it's Antigravity. Install the Antigravity CLI, Google OAuth, and you can use Gemini models within Hermes. Best for: multimodality (reading videos, images, long documents).

You already have Antigravity installed — this is the `AGY` tool Jack mentions.

## 7. Mission Control / Dashboard

Jack built a visual dashboard for Hermes that shows:
- Usage tracking (which models, how much)
- Open connections (all your integrations)
- Long-term goal planning (visual interface)
- Memory status
- GitHub backup status

## 8. GitHub backup

Daily cron job that takes a complete snapshot of your Hermes config and backs it up to a private GitHub repo. If you change computers, you can restore your entire Hermes setup.

**This is the backup solution for your vault too** — instead of Dropbox, you could back up `C:\Users\nikwi\Notes\` to a private GitHub repo daily.

## 9. Where to run Hermes

Jack's recommendation: **run it on your own computer**, not a VPS.
- Safer — it's in your room, not exposed to attacks
- Easier — all your memory systems (Obsidian, files) are local
- If you want isolation, use Docker (sealed container that can't access the rest of your computer)
- VPS works but adds complications (tunneling, security, remote memory access)

You're already running locally — this is the right call.

## 10. Key commands

| Command | Purpose |
|---------|---------|
| `/goal` | Set a persistent goal for the session |
| `/background` | Run task in background without blocking |
| `/handoff` | Switch model persona or platform |
| `/cron` | Schedule recurring tasks |
| `/clear` | Clear context |
| `/steer` | Steer the agent in a direction without interrupting its task |
| `/resume` | Pick up a different session |
| `/kanban` | Multi-agent swarm boards |
| `/curator` | Skill maintenance |
| `/stop` | Kill background processes |
| `/model` | Switch models manually |

## How this maps to your setup

**What you already have:**
- Memory (MEMORY.md + USER.md) ✓
- Obsidian vault connected ✓
- Local installation ✓
- Multiple models available ✓
- Cron capability ✓
- Goals capability ✓
- Background tasks ✓
- Antigravity (Gemini) ✓

**What you're NOT doing yet (from this video):**
1. **SOUL.md** — your agent identity is still the default Hermes blurb. Jack says this is the #1 thing to fix. Give me a WiSense-specific soul.
2. **Cron jobs** — no scheduled tasks running. A morning brief + dreaming sequence would be powerful for launch planning.
3. **GitHub backup** — no daily snapshot of your Hermes config. This is the safer backup than Dropbox.
4. **Model routing** — you're using GLM-5.2 for everything. You could delegate research to cheaper models and use stronger models for design/quality.
5. **Meeting notes + email integration** — not connected. Would let you ask "what did I commit to in that meeting?"
6. **Super Goals** — the human-AI handshake for longer-term goals like "launch COMMS LINK to App Store"

Related: [[NetworkChuck — Why I Switched to Hermes]], [[Mason — Scalable AI Brain with Hermes]], [[Working Stack — Claude CLI and Ollama]], [[Home]]