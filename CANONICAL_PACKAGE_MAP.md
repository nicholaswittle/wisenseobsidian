---
title: Canonical Package Map & Shared Code Sync
tags: [packages, wisense_ui, wisense_core, refactoring, sync]
aliases: [Canonical Packages, Shared Code Map]
date: 2026-07-20
---

# CANONICAL PACKAGE MAP & SHARED CODE SYNC

Specification for shared Dart packages (`wisense_core`, `wisense_ui`) across WiSense apps.

---

## Master Package Source

| Package | Canonical Path | Files | Description | Used By |
|---|---|---|---|---|
| **`wisense_core`** | `C:\development\packages\wisense_core` | 47 | Core models, Result type, Duffel client, affiliate deep links, unified travel providers, search intent, hydration, provider proxy | `wisense_new_horizon`, `wisense_horizon_v2` |
| **`wisense_ui`** | `C:\development\packages\wisense_ui` | 19 | Glassmorphism UI tokens, spacing, flight cards, affiliate redirect handler, loading overlay | `wisense_new_horizon`, `wisense_horizon_v2` |

All apps reference these via relative path dependencies in `pubspec.yaml`:
```yaml
dependencies:
  wisense_core:
    path: ../../packages/wisense_core
  wisense_ui:
    path: ../../packages/wisense_ui
```

No vendored copies exist. The fork reconciliation (2026-07-20) promoted New Horizon's vendored packages to canonical and removed the duplicates. See [[Fork Reconciliation]].

---

## Modification Rules

1. **Master Modification First**: Always make changes to `wisense_core` or `wisense_ui` inside `C:\development\packages\` first.
2. **No Vendored Copies**: Do not create vendored copies inside project repos. Use path dependencies only.
3. **Test After Changes**: Run `flutter test` in the canonical package and any affected app after modifications.

Related: [[00_AI_AGENT_MANIFEST]], [[Fork Reconciliation]], [[Code Reuse Analysis]]