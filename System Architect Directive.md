---
title: System Architect Directive
tags: [governance, architecture, shared-packages, reference]
aliases: [System Architect, Shared Package Enforcement]
---

# WiSense System Architect Directive

Source: `C:\development\SYSTEM_ARCHITECT_DIRECTIVE.md`

## Startup Protocol

Upon starting any session in this workspace, read `global_status.md` first to load current architecture state, in-progress work, and any pending audit hand-offs.

## Core Mandates

### 1. Shared Package Enforcement
Before creating any new utility, HTTP wrapper, error type, or UI widget — check:
- `packages/wisense_core/` — `Result<T>`, `ApiException`, `WiSenseApiClient`
- `packages/wisense_ui/` — `WiSenseLoadingIndicator`, `WiSenseErrorBanner`, `WiSenseSpacing`, `WiSenseTextStyles`

New shared logic goes into these packages, not into individual apps. Every app references them via path dependency.

**Known fork:** New Horizon vendors its own copies — see [[Fork Reconciliation]].

### 2. Design System Consistency
All spacing values must use `WiSenseSpacing` constants. All text scale must use `WiSenseTextStyles` as the base. App-specific brand colors stay in each app's own theme file — they do not belong in the shared packages.

### 3. Automated Testing Gate
Every modification to `wisense_core` or `wisense_ui` must pass `.\scripts\run_tests.ps1` before delivery. Golden baselines regenerated with `--update-goldens` only when visual output intentionally changes. No code ships with failing tests.

### 4. Audit Hand-off Chain
See [[Tripartite Audit Chain]]. Before each hand-off, write the reason and full task context to `global_status.md`.

### 5. Delivery Gate
No output reaches Nicholas Wittle without a Completion Report ratified under the Tripartite Protocol. All Judicial audits run MCA + MDT before the report is written.

## Workspace Scripts

```
.\scripts\analyze_workspace.ps1          — static analysis across all projects
.\scripts\run_tests.ps1                  — all package tests
.\scripts\run_tests.ps1 --update-goldens — regenerate golden baselines
.\scripts\correlate_changes.ps1          — match issues to recent git changes
.\scripts\generate_docs.ps1              — dart doc output to docs/
```

Related: [[WiSense Governance — Rules and Protocols]], [[Workspace Architecture]], [[Fork Reconciliation]]