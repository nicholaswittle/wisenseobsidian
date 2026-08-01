---
title: Travel Data Integrity
tags: [governance, audit, MDT, travel, reference]
aliases: [Travel MDT Extension, Travel Data Integrity MDT]
---

# Travel Data Integrity (MDT extension)

Source: `C:\Users\nikwi\CLAUDE.md`, `C:\Users\nikwi\AGENTS.md`

When code involves flight, hotel, or travel booking logic, run this additional triage before issuing PASS:

| Check | Pass condition |
|---|---|
| ToolRegistry routing | All travel API interactions routed through `ToolRegistry` (New Horizon: `lib/core/ai/agent_bridge.dart`); no direct partner HTTP from UI or ViewModels |
| Typed transforms | Raw API envelopes mapped to clean domain types; no untyped `Map` leakage to view layers |
| Session binding | Authenticated proxy calls use session binding; unconfigured session fails early with `Result` error |
| Human Veto gates | Destructive tools and quota-consuming searches require explicit Architect ratification |
| Data efficiency | No redundant fields, duplicate parsing, or nested JSON passed through view layers |
| MCA conformance | Travel data structures pass Minimalist Conformance Audit — clean, minimal, no speculative abstractions |

## What this means in practice

- **No direct Duffel/Viator/Expedia HTTP calls from UI widgets or view models.** Everything goes through `agent_bridge.dart` → `ToolRegistry`.
- **Raw API responses never reach the view layer.** They must be mapped to typed domain models (e.g., `UnifiedTravelOffer`, `FlightResult`) first.
- **Unconfigured sessions fail early** with a `Result.error`, not a silent fallback.
- **Destructive actions** (booking, payment) and **quota-consuming searches** need explicit Architect ratification — the Human Veto gate.

Related: [[WiSense Governance — Rules and Protocols]], [[MCA and MDT]], [[Duffel]], [[New Horizon — Code Reference]]