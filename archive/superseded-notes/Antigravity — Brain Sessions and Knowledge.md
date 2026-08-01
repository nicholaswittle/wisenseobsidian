---
title: Antigravity — Brain Sessions & Knowledge
tags: [governance, antigravity, gemini, memory, reference]
aliases: [Gemini Brain, Antigravity Sessions, Antigravity Memory]
---

# Antigravity — Brain Sessions & Knowledge

The Gemini Antigravity IDE stores session memory in `~/.gemini/antigravity/brain/<session-id>/`. Each session has:
- `task.md` — task checklist
- `walkthrough.md` — narrative summary of what was done
- `implementation_plan.md` — detailed plan (when applicable)

## Known brain sessions (10 with content)

| Session ID | Title | Project |
|---|---|---|
| 05973de1 | Cloud Ask Before Changes (ABC) Usability & Reliability | wisense-os |
| 21cb8159 | Dynamic Curation Map & Layover Visualizer | New Horizon |
| 2891d406 | Phase 2 & GPT Bridge Implementation Walkthrough | COMMS LINK |
| 3198858b | Duffel Payments Integration | New Horizon |
| 3bfeb842 | Antigravity Phone Connect Setup | Phone Connect |
| 3f884d89 | Web UI Upgrades and Codex Implementation Plan | local-agent-work-center |
| 4f29a867 | Command Center 2D NPC & Router Dispatch | my_ai (deleted) |
| 556e0d15 | WiSense LLC Command Center | Command Center |
| 99a51d15 | Comparative Audit Remediation Complete | Cross-project audit |
| ed49dead | New Horizon Hub UI Dispatched | New Horizon |

## Global Agent Rules (.gemini/config/AGENTS.md)

Antigravity has its own global agent rules:
1. **Active Project Root Detection** — resolve paths relative to active workspace
2. **Automated Dev Loop & Intelligent Dispatcher (Claude Bridge)** — for code-intensive tasks, do NOT modify source directly. Instead, write task instructions to `C:\development\.tasks\claude_queue\` and update `MASTER_STATUS.md`. Research/strategy tasks stay in Gemini's buffer.
3. **Rate-Limit Failover** — if Claude Code is rate-limited or unavailable, Antigravity takes over directly, implements the plan, runs tests, and delivers the completion report.
4. **Automated Startup Loop** — on session start, verify `watch_and_run.ps1` is running. If not, launch it as a persistent background process.

## Antigravity scratch projects

- `scratch/wisense_command_center/` — Python server + HTTP serving + `/api/audit` powershell wrapper
- `scratch/dropship/`
- `scratch/internal_os/`

## Gemini skills (50+ GCP data skills)

`~/.gemini/skills/` and `~/.gemini/config/skills/` contain 50+ Google Cloud data skills (AlloyDB, BigQuery, Cloud SQL, Dataflow, Firestore, Spanner, etc.). These are GCP-specific and not directly relevant to WiSense Flutter work, but are available when GCP infrastructure questions arise.

## Gemini config plugins

`~/.gemini/config/plugins/` — android-cli, chrome-devtools, firebase, flutter, google-antigravity-sdk, googlecloudtools, modern-web-guidance, science.

Related: [[WiSense Governance — Rules and Protocols]], [[Master Status]], [[Audit Findings Loop]]

---

## Audit Chain Replacement (2026-07-21)

Gemini CLI free tier died ~July 2026 (`IneligibleTierError: This client is no longer supported for Gemini Code Assist for individuals`). The CLI is still installed (`@google/gemini-cli@0.50.0`) and the Google account (`nicholaswittle@gmail.com`) is still configured, but it can't authenticate.

**Antigravity (agy) is the replacement.** Same Google account, same OAuth, different client.

### How to run agy for headless audits

```bash
# From within the workspace directory (so agy can read files)
cd /path/to/project
/c/Users/nikwi/AppData/Local/agy/bin/agy.exe \
  --dangerously-skip-permissions \
  --model gemini-2.5-flash \
  --print-timeout 240s \
  -p "Your audit prompt here. Tell agy to read a specific file in the workspace."
```

Key flags:
- `--dangerously-skip-permissions` — auto-approve all tool calls (required for headless mode)
- `--model gemini-2.5-flash` — same model as Gemini CLI, free tier via Antigravity
- `--print-timeout 240s` — extend timeout for long prompts (default 5m)
- `-p` — non-interactive print mode (headless)

### Limitations

- Argument list too long for large diffs passed inline. Solution: have agy read the diff from a file in the workspace (`-p "Read audit_diff_temp.txt and audit it..."`), then delete the temp file after.
- No `--yolo` flag (use `--dangerously-skip-permissions` instead).
- `--yolo` and `--approval-mode` are mutually exclusive — use one or the other.

### Audit chain status

1. Antigravity (agy) with gemini-2.5-flash — WORKING (primary)
2. Groq (llama-3.3-70b-versatile) — no API key configured yet (fallback)
3. OpenRouter (openrouter/auto) — no API key configured yet (fallback)

To configure fallbacks: set `GROQ_API_KEY` and `OPENROUTER_API_KEY` environment variables.