---
title: Hermes 3 Scratchpad Reasoning Protocol
tags: [hermes, scratchpad, reasoning, internal-monologue, memory]
aliases: [Scratchpad Protocol, Hermes Monologue]
---

# 🧠 Hermes 3 Scratchpad Reasoning Protocol

> Specification for persisting Hermes 3's `<scratch_pad>` internal monologue reasoning logs into `scratchpad/` for multi-session decision tracing.

---

## ⚙️ Monologue Logging Rules

1. **Tag Format**: Hermes 3 wraps internal monologue in `<scratch_pad>` XML tags:
   ```xml
   <scratch_pad>
   1. User requests X.
   2. Checking file Y for existing implementation.
   3. Hypothesis: Need to modify function Z to avoid breaking dependency W.
   </scratch_pad>
   ```
2. **Persistence Location**: Save long multi-step reasoning logs to `scratchpad/YYYY-MM-DD_[task_id].md`.
3. **Audit Cross-Linking**: Link the scratchpad note to the final output note in `wiki/` and record the action in `log.md`.

---

## 🔍 Benefits of Persistent Monologues

- **Multi-Session Continuity**: Agents resume complex multi-day tasks without re-evaluating past decisions.
- **Root Cause Analysis (RCA)**: If an agent makes a mistake, developers can inspect the `<scratch_pad>` note to see *why* the agent chose an invalid path.

Related: [[agents]], [[log]], [[Hermes 3 Agent Memory Architecture]]
