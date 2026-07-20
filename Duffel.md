---
title: Duffel
tags: [vendor, api, travel]
---

# Duffel

Flight booking API used by [[New Horizon|Horizon V2]] and [[New Horizon]].

## WiSense integration rules

Per the Travel Data Integrity MDT extension:
- All Duffel API interactions must route through `ToolRegistry` (New Horizon: `lib/core/ai/agent_bridge.dart`)
- No direct partner HTTP from UI or ViewModels
- Raw API envelopes mapped to clean domain types; no untyped `Map` leakage to view layers
- Authenticated proxy calls use session binding; unconfigured session fails early with `Result` error
- Destructive tools and quota-consuming searches require explicit Architect ratification

## Recent work (New Horizon)

- Duffel checkout flow with session-gated payment intents
- Order creation, search, provider fallback test coverage
- Vercel Hobby function-limit deploy failure fixed (consolidated air routes)