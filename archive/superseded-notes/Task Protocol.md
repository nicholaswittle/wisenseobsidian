---
title: Task Protocol
tags: [governance, task, protocol, reference]
aliases: [Task Execution Protocol, Pre-Execution Validation]
---

# Task Protocol

Source: `C:\development\TASK_PROTOCOL.md`

Governs the execution of all tasks within the WiSense New Horizon pipeline (and referenced by other projects).

## Pre-Execution Validation

1. **Target Verification** — Ensure the task is correctly routed to you (Claude or Cursor) by checking the `Target` frontmatter.
2. **Tripartite Audit Validation** — Before executing any code changes or generating a final response, the proposed plan or result must be evaluated through the [[Tripartite Audit Chain]].
3. **Artifact Generation** — Any result must be documented according to the `Write-TaskResultArtifact` format, noting success or failure.

## Execution Rules

- Never bypass the audit chain.
- Ensure all paths are relative to `C:\development` unless specified otherwise.
- Document any failures clearly in the task result output.

Related: [[WiSense Governance — Rules and Protocols]], [[Tripartite Audit Chain]], [[Master Status]]