---
title: Canonical Package Map & Shared Code Sync
tags: [packages, wisense_ui, wisense_core, refactoring, sync]
aliases: [Canonical Packages, Shared Code Map]
---

# 📦 CANONICAL PACKAGE MAP & SHARED CODE SYNC

Specification for shared Dart packages (`wisense_core`, `wisense_ui`) across New Horizon, Apex Scheduler, and WiSense OS to prevent fork divergence.

---

## 🏛 Master Package Source

| Package Name | Canonical Path | Description | Vendored In |
|---|---|---|---|
| **`wisense_core`** | `C:\development\packages\wisense_core` | Core models, AI bridge, auth contracts | `wisense_new_horizon`, `apex` |
| **`wisense_ui`** | `C:\development\packages\wisense_ui` | Glassmorphism UI tokens, spacing, buttons | `wisense_new_horizon`, `apex` |

---

## ⚠️ Divergence Safeguards & Rules

1. **Master Modification First**: Always make changes to `wisense_core` or `wisense_ui` inside `C:\development\packages\` first.
2. **Never Overwrite Upstream**: Do NOT edit vendored copies inside `projects/wisense_new_horizon/packages/` directly without copying changes back to `C:\development\packages\`.
3. **Relative Dependency Path**: In `pubspec.yaml`, reference shared packages using canonical relative paths:
   ```yaml
   dependencies:
     wisense_ui:
       path: ../../packages/wisense_ui
   ```

---

## 🔄 Package Sync Command

Run in PowerShell to sync canonical package changes into target project:

```powershell
# Sync wisense_ui to new_horizon
Copy-Item -Path "C:\development\packages\wisense_ui\*" -Destination "C:\development\projects\wisense_new_horizon\packages\wisense_ui\" -Recurse -Force
```

Related: [[00_AI_AGENT_MANIFEST]], [[Fork Reconciliation]], [[Code Reuse Analysis]]
