---
title: Fork Reconciliation
tags: [decision, architecture, new-horizon]
status: COMPLETE
date: 2026-07-20
---

# Fork Reconciliation — New Horizon vs canonical wisense_core/wisense_ui

## The problem

New Horizon vendored its own copies of `wisense_core` and `wisense_ui` at `projects/wisense_new_horizon/packages/...` instead of referencing the canonical packages at `C:\development\packages\...` via path dependency. The two diverged:

- New Horizon added 29 files (unified travel offers, hydration, Duffel client, affiliate deep links, search intent) that canonical lacked
- 7 shared files drifted independently (api_client, flight_result, affiliate providers, vault_repository, barrel export)
- 1 file removed in fork (affiliate_query.dart — replaced by FlightQuery-based interface)

## Resolution

**Option 1 chosen: Promote.** Vendored packages promoted to canonical on 2026-07-20.

- wisense_core: 19 -> 47 files (full travel engine)
- wisense_ui: 17 -> 19 files (affiliate redirect handler, loading overlay)
- New Horizon pubspec.yaml switched to `../../packages/wisense_core` and `../../packages/wisense_ui`
- Vendored `packages/` directory removed entirely
- 4 pre-existing test failures fixed (EXP provider disabled in 0e37770 but tests never updated)
- Golden test baselines regenerated

## Test Results

- wisense_core: 69/69 pass
- wisense_ui: 21/21 pass
- New Horizon: 117/117 pass
- Horizon V2: 7/7 pass

## Audit

MCA: PASS, MDT: PASS, Travel Data Integrity: PASS (audited by external subagent, 36 API calls)

Related: [[New Horizon]], [[CANONICAL_PACKAGE_MAP]], [[Code Reuse Analysis]]