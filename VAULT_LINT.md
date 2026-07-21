---
title: Vault Lint Checklist
tags: [meta, lint, health, maintenance]
aliases: [VAULT_LINT, Vault Health, Lint Ritual]
date: 2026-07-20
cadence: monthly
---

# Vault Lint Checklist

> Monthly (or after large vault edits). Run as a deliberate agent task — not automatic. Goal: stop status drift and dead links before they poison agent context.

## Cadence

- **When**: First weekend of the month, or after any bulk ingest
- **Who**: Any agent when Nicholas asks “lint the vault”
- **Output**: Append findings + fixes to [[log]]; refresh [[hot]] if status changed

## Checklist

### 1. Boot consistency
- [ ] [[hot]], [[NOW]], [[00_AI_AGENT_MANIFEST]], [[index]] agree on active project **status**
- [ ] [[CLAUDE]] still thin and points to hot → NOW → index (not a second status table)
- [ ] No note claims deleted apps (wisense-os, my_ai, etc.) are live

### 2. Stale claims
- [ ] Search for outdated phrases: `merge conflict`, `needs push`, `fork reconciliation open`, port `5050` as live
- [ ] Launch blockers on [[Apex Scheduler]] / [[COMMS LINK]] match [[NOW]]
- [ ] Business roadmaps (e.g. [[business/WiSense Service as a Software Execution Strategy]]) don’t contradict [[hot]]

### 3. Link health
- [ ] No wikilinks to deleted `wiki/`, `journal/`, `crm/` as if they still exist
- [ ] New notes linked from [[index]] (and [[Home]] if human-facing)
- [ ] Orphans: notes with zero inbound links either get a parent or an explicit “reference-only” home

### 4. Execution layer
- [ ] [[NOW]] updated within last 7 days (or marked stale)
- [ ] Open human blockers still accurate
- [ ] [[business/Experiment Log]] outcomes filled for closed experiments
- [ ] `customers/` has any new pilot feedback since last lint

### 5. Intake hygiene
- [ ] `raw/` items older than 30 days: codify, move to `raw/processed/`, or delete with note in [[log]]
- [ ] No secrets in vault files

### 6. Optional deeper pass
- [ ] Flat-root cluster candidates (`projects/`, `governance/`, `vendors/`) — propose only, don’t move without ask
- [ ] Video references with durable lessons → 1-page principle notes

## Last run

| Date | Runner | Result |
|------|--------|--------|
| 2026-07-20 | Cursor (vault AI research implement) | Baseline created; status sync + NOW/customers/experiments added |

Related: [[agents]], [[hot]], [[NOW]], [[log]], [[index]]
