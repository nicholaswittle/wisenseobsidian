---
type: meta
title: "Hot Cache"
tags: [meta, hot-cache, context]
updated: 2026-07-20T20:40:00
---

# Recent Context

> ~500-word cache for fast boot. Any agent/session reads this FIRST, then [[NOW]], then [[index]]. Overwrite completely each update — cache, not journal.

## Last Updated
2026-07-21. Apex feature plan executed — Section 0 (iOS build) + Features B/C on an unpushed branch.

## Key Recent Facts
- **Vault = curated static reference** — no auto-wiki. Boot: [[hot]] → [[NOW]] → [[index]] → note.
- **Execution layer:** [[NOW]] (scorecard + human blockers). Customer truth: [[customers/_Index]]. Experiments: [[business/Experiment Log]]. Health: [[VAULT_LINT]].
- **Schema:** thin [[CLAUDE]]; protocol in [[agents]]; paths in [[00_AI_AGENT_MANIFEST]] (status always here / NOW).
- Git mirror: `github.com/nicholaswittle/wisenseobsidian` (origin/main).

## Active Project Status
- **COMMS LINK** (`wisense_decompression`) — ⏸️ **PARKED 2026-07-20 (deliberate):** ~1.5 GB model download is too heavy for a first launch + model quality unverified. Full state in [[COMMS LINK]]. Do not treat as a launch candidate. Previously: ⚠️ **NOT ready to ship.** The on-device model has **never actually run**. The hardcoded HF URL (`google/gemma-1.1-2b-it-gpu-int4`) is licence-gated and returns **401 to every user** — verified 2026-07-20. 59/59 tests stayed green because none of them touch Gemma. Fixed in `94cecea` + `750a275`: **default model switched to ungated Qwen2.5-1.5B** (Apache-2.0, verified HTTP 200 no token — no self-hosting/R2 needed), swappable `ModelPreset` (qwen15 / qwen05 521MB / gemma2b gated), CPU fallback, `integration_test/model_smoke_test.dart`. **On-device verification still pending** — needs iPhone 15 run (Mac + Xcode, which is FREE; $99 is only for distribution). Gate C BLOCKED until the smoke test passes. Also unverified: Qwen vs Gemma reply quality — the persona prompt was written for Gemma.
- **Apex Scheduler** (`apex\apex`) — pushed through RLS `a66b039`. **NEW 2026-07-21:** branch `feat/apex-plan-2026-07-21` (`0fabf68` + `b469b6c`) implements Section 0 + Features B/C of [[Apex — Feature Plan 2026-07-21]] — **pushed to origin, not merged, not device-verified, and NOT audited** (built outside the Tripartite Protocol; Completion Report is BLOCKED / MCA+MDT NOT RUN — see [[DECISIONS]] 2026-07-21). Conformance remediation in `594b4be`. **Judicial branch is currently unimplementable — no auditor credentials exist anywhere in the workspace.** iOS bundle ID is now `com.nicholaswittle.apex` (Android stays `com.wisense.apex`); **Sentry removed → no crash reporting**; both in [[DECISIONS]]. Feature A (photo import) not started. Full record: [[Apex — Feature Plan Implementation 2026-07-21]]. **Still the only human launch step:** apply `20260720000000_launch_blockers_rls.sql` on Supabase staging→prod, then Gate C. See [[output/Launch Readiness — COMMS LINK and Apex 2026-07-20]].
- **New Horizon** (`wisense_new_horizon`) — 117/117; fork reconciliation COMPLETE. Open: untracked agent files; may need push.
- **DELETED:** wisense-os, my_ai, local-agent-work-center, command_center.

## Active Threads
- **LAUNCH:** Both apps code-complete. Remaining = ops/human: (1) Apex RLS apply; (2) Gate C keystore/AAB/graphics/privacy/Play listing — [[output/Gate C — Android Packaging & Store Listings 2026-07-20]]; (3) iOS later via Codemagic.
- **⚡ CURRENT PLAN:** [[business/Plan of Attack — Build While Mac-Blocked 2026-07-20]] — **Mac gap blocks iOS only, NOT revenue.** Apex has a working Vercel web/PWA pipeline → ship to Jigsy's now (no store, no store cut; iPhone staff via Add to Home Screen). COMMS LINK native-only but Play needs no Mac. Wk1: RLS + deploy web + onboard Jigsy's + measure hours saved. Wk2: Stripe on (flat ~$99/location/mo) + landing page. Wk3–4: Shift Compliance Checker (Fair Workweek) w/ free scan as lead magnet.
- **REVENUE:** [[business/Revenue Ideas — 12 Buildable B2B Plays 2026-07-20]] — 12 researched plays ranked by time-to-first-dollar. Thesis: **distribution is the constraint, not ideas**; Jigsy's + working Apex code = the unfair advantage. Sequence: ship Apex to Jigsy's → Tier 0 services for cash → Apex multi-tenant + Stripe + Fair Workweek compliance add-on → only then a second front (COMMS LINK B2B). Kill criteria: no 2nd paying venue by ~wk 12 = stop building, go sell.
- **This week board:** [[NOW]]
- Working stack: Claude CLI + Ollama — [[Working Stack — Claude CLI and Ollama]]
