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

1. **`git-vault-sync`**: Auto-commit and push vault Markdown notes to GitHub.
2. **`flutter-test-gate`**: Run `flutter analyze` and `flutter test` across Apex/Horizon projects.
3. **`wisense-engine-probe`**: Probe local Python backend daemon on port `5050`.

Related: [[agents]], [[00_AI_AGENT_MANIFEST]], [[Hermes 3 Agent Memory Architecture]]
