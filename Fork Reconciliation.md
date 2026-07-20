---
title: Fork Reconciliation
tags: [decision, architecture, new-horizon]
status: open
---

# Fork Reconciliation — New Horizon vs canonical wisense_core/wisense_ui

## The problem

New Horizon vendors its own copies of `wisense_core` and `wisense_ui` at `projects/wisense_new_horizon/packages/...` instead of referencing the canonical packages at `C:\development\packages\...` via path dependency. The two have diverged:

- New Horizon added ~20 files (unified travel offers, hydration, more affiliate providers) that canonical lacks
- Some shared files like `api_client.dart` drifted independently
- Switching New Horizon to the canonical path dependency would currently break its build

## Options

1. **Promote New Horizon's additions into canonical** — then switch New Horizon to path dep. One-way merge; canonical becomes the source of truth again.
2. **Formally accept the split** — New Horizon keeps its vendored fork; document it as intentional. Other apps keep using canonical.
3. **Defer** — ship New Horizon as-is, reconcile after launch. Risk: the split hardens over time.

## Status

Open decision. Needs [[Tripartite Protocol]]: Legislative design + Gemini review + Architect ratification before any merge work begins.

Related: [[New Horizon]]