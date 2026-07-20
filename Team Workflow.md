---
title: Team Workflow
tags: [governance, workflow, session, reference]
aliases: [Session Structure, 8-Step Workflow]
---

# WiSense Team Workflow

Source: `C:\development\projects\wisense_governance_hub\meta_prompts\team_workflow.md`

## Gemini Collaboration Mandate

Gemini is a **mandatory partner** on all WiSense builds — not an optional reviewer.

- Claude must consult Gemini for **research** (package selection, API usage, platform behavior) before writing implementation code.
- Claude must send all **proposed implementations** to Gemini for review before presenting them to the Architect.
- Claude must **not deliver code** without Gemini sign-off.
- This mandate applies to every task regardless of perceived complexity.

## Session Structure (8 steps)

Every working session follows this sequence:

1. **Boot** — AI reads all `meta_prompts/` files and summarizes current project state and objective.
2. **Plan** — Builder proposes a plan with a clear diff. No files are touched yet.
3. **Gemini Research** — Builder consults Gemini on any package APIs, platform specifics, or design questions relevant to the plan.
4. **Build Draft** — Builder applies the plan.
5. **Gemini Review** — Builder sends the full implementation to Gemini for audit. Gemini returns a numbered finding list with severity (low / medium / high).
6. **Resolve** — Builder resolves all Gemini findings. Cycle repeats until no high-severity findings remain.
7. **Ratify** — Architect gives explicit approval.
8. **Report** — Completion Report is presented to Architect.

## Communication Standards

- Builder speaks first only with a plan, never with applied changes.
- Gemini findings are delivered as a numbered list of flagged items with severity (low / medium / high).
- Head of Team blocks delivery on any unresolved high-severity finding.
- Architect's approval is required at steps 7 and 8.

## Escalation

If the Builder and Reviewer cannot reach agreement after two resolution cycles, the Head of Team escalates directly to the Architect with a written summary of the conflict and a recommended resolution.

Related: [[WiSense Governance — Rules and Protocols]], [[Head of Team Directive]], [[Governance Protocol]]