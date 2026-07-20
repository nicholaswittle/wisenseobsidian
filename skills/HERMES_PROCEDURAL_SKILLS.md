---
title: Hermes 3 Procedural Skill Specification
tags: [hermes, skills, agentic, procedural-memory, nous]
aliases: [Hermes Skills, Procedural Memory]
---

# 🛠 Hermes 3 Procedural Skill Specification

> Specification for procedural skill documents (`agentskills.io` standard) used by **Hermes 3** and autonomous AI agents for dynamic multi-step task execution.

---

## 🏛 Skill Document Structure

Every procedural skill stored in `skills/` MUST follow this Markdown structure:

```markdown
---
name: Skill Name
description: High-level overview of when to invoke this skill
inputs: [input_param_1, input_param_2]
outputs: [output_artifact]
---

# Skill Title

## Step 1: Pre-flight Verification
- Run verification command or sanity check.

## Step 2: Core Execution
- Step-by-step instructions and tool calls (`<tool_call>`).

## Step 3: Acceptance Criteria & Audit
- Define test or validation check before concluding.
```

---

## 📚 Installed Hermes Skills

Each maps to a real, runnable command. Prefer the existing scripts in `C:\development\scripts\` over ad-hoc commands.

1. **`vault-commit`**: Stage + commit vault changes with an auditable message (`git add -A && git commit`). Manual/on-request — the old auto-sync task was retired 2026-07-20. Push only on explicit approval.
2. **`workspace-analyze`**: Run `.\scripts\analyze_workspace.ps1` — flutter/dart analyze across all projects.
3. **`flutter-test-gate`**: Run `.\scripts\run_tests.ps1` — all package tests must pass before commit (`--update-goldens` to regenerate baselines).
4. **`apex-diagnose`**: Run `.\scripts\diagnose_apex.ps1` — environment checks, analysis, and tests for Apex Scheduler.

> ⚠️ Removed 2026-07-20: `wisense-engine-probe` (targeted deleted `wisense-os` on port 5050 — service no longer exists).

Related: [[agents]], [[00_AI_AGENT_MANIFEST]], [[Hermes 3 Agent Memory Architecture]]
