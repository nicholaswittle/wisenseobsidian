---
title: Experiment Log
tags: [business, experiments, validation, lean]
aliases: [Experiment Log, Experiments, Keep Kill]
date: 2026-07-20
---

# Experiment Log

> Hypothesis → result → keep/kill. Prevents building without learning. Link ideas from [[Ideas Log]]; link customers from [[customers/_Index]].

## Active / recent

| ID | Hypothesis | App | Metric | Result | Decision | Date |
|----|------------|-----|--------|--------|----------|------|
| E001 | Android-first launch unblocks store without Mac | COMMS + Apex | Both AABs buildable | In progress (Gate C) | — | 2026-07-20 |
| E002 | Apex RLS + race fixes enough for Jigsy pilot trust | Apex | Staging smoke pass | Code in repo; DB apply pending | — | 2026-07-20 |
| E003 | Custom agent OS / Work Center aids shipping | Platform | Weekly shipped value | Failed — overbuilt | **Kill** | 2026-07-19 |

## Detail

### E001 — Android-first packaging
- **Bet**: Skip iOS until Apple account; ship Play Store first.
- **Evidence needed**: Keystore + `flutter build appbundle` for both apps; privacy URLs live.
- **Status**: Prep done in [[Gate C — Android Packaging & Store Listings 2026-07-20]]; human execution open on [[NOW]].

### E002 — Apex security gate for pilot
- **Bet**: Authored RLS + claim/clock guards unblock real multi-user pilot.
- **Evidence needed**: Migration applied staging→prod; no cross-org leakage in smoke test.
- **Status**: Code-complete (`a66b039`); human apply remaining.

### E003 — Custom agent platforms (closed)
- **Bet**: wisense-os / local-agent-work-center improve agent throughput.
- **Result**: Theatre vs Claude CLI + Ollama. See [[Abandoned Projects — Lessons]], [[Working Stack — Claude CLI and Ollama]].
- **Decision**: Kill — do not revive.

## How to log a new experiment

1. Add a row (IDs sequential).
2. Write a short Detail section: bet, evidence, status.
3. When done: fill Result + Decision (**Keep** / **Kill** / **Pivot**).
4. Update [[NOW]] scorecard if it was the week’s bet.

Related: [[Business Hub]], [[Ideas Log]], [[Revenue Ideas — 12 Buildable B2B Plays 2026-07-20]], [[2026 Commercial B2B Software Ideas]], [[NOW]], [[DECISIONS]]
