# WiSense AI Vault — Claude Entry

Thin always-on schema. Full protocol lives in [[agents]]. Project status lives in [[hot]] and [[NOW]].

## Boot order (mandatory)

1. [[hot]] — ~500-word recent-context cache  
2. [[NOW]] — this week’s tasks, scorecard, human blockers  
3. [[index]] — pointer catalog  
4. Relevant note only after the above  

Do **not** start from [[Home]] alone — Home is a human dashboard; agents use hot → NOW → index.

## What this vault is

Curated static reference for cross-project decisions, launch planning, governance, and lessons. Not source code. Codification from `raw/` is **manual / on-request** only.

## Canonical files

| File | Role |
|------|------|
| [[agents]] | Operating protocol (personas, syntax, codify rules) |
| [[00_AI_AGENT_MANIFEST]] | Karpathy rules + project path map |
| [[DECISIONS]] | Settled decisions — check before reopening |
| [[VAULT_LINT]] | Monthly health checklist |

## Active apps (paths only — status in [[hot]])

| App | Path |
|-----|------|
| COMMS LINK | `C:\development\projects\wisense_decompression` |
| Apex Scheduler | `C:\development\projects\apex\apex` |
| New Horizon | `C:\development\projects\wisense_new_horizon` |
| Shared packages | `C:\development\packages\` |

Deleted — never treat as live: wisense-os, my_ai, local-agent-work-center, command_center. See [[Abandoned Projects — Lessons]].

## Governance (before code changes)

[[WiSense Governance — Rules and Protocols]] — Builder proposes plan+diff; Reviewer (Gemini) for Medium/High; Architect (Nicholas) ratifies. No self-audit.

## Write-back rules

- Audits / launch checklists → `output/` + link from project note  
- Decisions → [[DECISIONS]]  
- Tasks / weekly focus → [[NOW]]  
- Customer truth → `customers/`  
- Experiments → [[business/Experiment Log]]  
- Daily summary → `YYYY-MM-DD.md`  
- After structural changes → update [[hot]], [[index]], append [[log]]

## Hard no

- No code copies or secrets in the vault  
- No deleting notes (deprecate + link replacement)  
- No inventing live project status — read [[hot]] / [[NOW]]
