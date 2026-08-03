---
title: Working Stack — Claude CLI and Ollama
tags: [workflow, ollama, claude-cli, decision]
aliases: [Claude CLI, Ollama workflow, daily stack]
status: active
---

# Working Stack — Claude CLI + Ollama

**Decision (2026-07-19):** Stop building custom agent platforms. Daily AI work runs through **Claude CLI** backed by **Ollama** models. Simpler, matches how Nicholas actually works.

## Do this

- Use Claude CLI for planning, coding, and audits against real project folders
- Use Ollama for local/cloud-routed models (`127.0.0.1:11434`) when a local or Ollama-cloud model is the right tool
- Keep product apps as the only long-lived codebases: [[COMMS LINK]], [[Apex Scheduler]], [[New Horizon]]
- Put durable decisions and audit summaries in this vault (`C:\Users\nikwi\Notes`)

## Do not build

- Custom "AI OS" / Companion shells that wrap the same work Claude CLI already does
- Local Agent Work Center–style watchers and handoff theatre
- Invented local builders / Autopilot when hardware and models are not ready
- Freeform chat as a write gate (if anything ever writes code to disk, digest/approve remains the rule)

## Why this won

Custom agent apps (WiSense OS, my_ai, Work Center) taught useful lessons but did not match the envisioned daily loop. Claude CLI + Ollama is enough surface; product apps get the engineering focus.

## Related lessons

See [[Abandoned Projects — Lessons]] for what was killed and what to keep from those experiments.

Related: [[Home]], [[meta/ENVIRONMENT_MAP]], [[projects/dormant/Apex Scheduler]], [[COMMS LINK]], [[projects/dormant/New Horizon]]
