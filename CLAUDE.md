# WiSense AI Vault — Claude Entry

Thin always-on schema. Full protocol lives in [[agents]]. Project status lives in [[hot]] and [[NOW]].

## Boot order (mandatory)

1. [[hot]] — ~500-word recent-context cache  
2. [[NOW]] — this week’s tasks, scorecard, human blockers  
3. [[index]] — pointer catalog  
4. Relevant note only after the above  

Do **not** start from [[Home]] alone — Home is a human dashboard; agents use hot → NOW → index.

## What this vault is

Curated static reference for cross-project decisions, launch planning, governance, and lessons. Not source code. Codification is **manual / on-request** only — see [[agents]].

## Canonical files

| File | Role |
|------|------|
| [[agents]] | Operating protocol (personas, syntax, codify rules) |
| [[meta/00_AI_AGENT_MANIFEST]] | Karpathy rules + project path map |
| [[DECISIONS]] | Settled decisions — check before reopening |
| [[meta/VAULT_LINT]] | Monthly health checklist |

## Active apps (paths only — status in [[hot]])

| App | Path |
|-----|------|
| Apex v2 | `C:\development\projects\apex_v2` |
| Apex v2 build/site | `C:\development\projects\apex_v2_build` |
| Apex Scheduler (dormant) | `C:\development\projects\apex\apex` |
| COMMS LINK (dormant) | `C:\development\projects\wisense_decompression` |
| New Horizon (dormant) | `C:\development\projects\wisense_new_horizon` |
| Shared packages | `C:\development\packages\` |

Deleted — never treat as live: wisense-os, my_ai, local-agent-work-center, command_center. See [[Abandoned Projects — Lessons]].

## Governance (before code changes)

[[WiSense Governance — Rules and Protocols]] — Builder proposes plan+diff; Reviewer (Gemini) for Medium/High; Architect (Nicholas) ratifies. No self-audit.

## Write-back rules

- Audits / launch checklists → `archive/completed-deliverables/` + link from project note  
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
