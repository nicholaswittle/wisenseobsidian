---
title: Decisions Register
tags: [decisions, adr, rationale, governance, history]
aliases: [Decisions, ADR, Decision Log]
date: 2026-07-20
---

# ⚖️ Decisions Register

> Append-only record of **settled decisions** — the *why* behind the current state. Any agent (Hermes, Claude, Cursor, Codex, Gemini) checks here before re-opening a question or "helpfully" undoing something deliberate. Newest at top. Never edit a past entry; supersede it with a new one and mark the old `SUPERSEDED`.

Entry format: **date · decision · status · rationale · consequences**. Status ∈ ACTIVE / SUPERSEDED / REVERSED.

---

## 2026-07-26 · Restaurant SaaS Lead Offer = $0 Setup + $99/mo (Starter Tier) — `ACTIVE`
- **Decision**: WiSense adopts a 2-tiered commercial offer for the restaurant website + online ordering platform: Tier 1 (Lead Pitch) = **$0 Setup Fee + $99/month**, Tier 2 (Pro Value) = **$299 Setup Fee + $79/month**.
- **Rationale**: Market research across 9 competitors revealed that setup fees ($299+) create massive upfront sales friction for cash-strapped local restaurants. $99/mo eliminates setup risk while keeping price below the $100/mo psychological threshold.
- **Unit Economics**: A single client at $99/mo pays 100% of WiSense AI agent tool costs and infrastructure ($100/mo total overhead). Every subsequent client yields a 95%+ net profit margin because AI agent templates allow deployment in <30 minutes.
- **Consequences**: Outreach to the 408 local Enola prospects will lead with the $0 Setup / $99/mo offer. See [[business/Restaurant Website SaaS — Master Pitch Model and Strategy 2026-07-26]].

## 2026-07-23 · Jigsy's = free core + 99¢ per accepted online order; pay at pickup; no Stripe — `ACTIVE`
- **Decision**: The Jigsy-specific offer is a free core website/order demo plus a **$0.99 Online ordering fee on each accepted order**. Jigsy's collects the food, tax, and fee in person at pickup. WiSense does not process customer payments through Stripe or hold customer/restaurant funds.
- **Rationale**: Nicholas is not comfortable asking Jigsy's for the broader `$299 setup + $79/month` price. The 99-cent model keeps the relationship simple, charges only when the system produces an accepted order, and preserves Jigsy's existing cash/card-at-counter workflow.
- **Settlement**: The system records accepted orders and produces a monthly statement: `accepted orders × $0.99`. Jigsy's pays WiSense separately by check, cash, or bank transfer. Rejected/abandoned requests do not count.
- **Product consequences**: Staff workflow is intentionally **Accept & Print** only, plus pause/reopen ordering, prep time, sold-out controls, and ticket reprint. The ticket must disclose the fee and say payment is collected at the counter.
- **Boundary**: This decision is Jigsy-specific. The general WiSense `$299 + $79/month` website offer may remain for other clients. The current demo is browser-local and is not a live cross-device ordering system. See [[business/Jigsys Ordering Demo — Build Record 2026-07-23]].

## 2026-07-21 · AI in Apex = parsing, not scheduling optimization — `ACTIVE`
- **Decision**: LLMs in Apex are for turning **unstructured input into structure** (photo of a paper schedule → rows; later, free-text availability). They are **not** for building or optimizing schedules. Assignment logic stays deterministic in `SuggestionEngine` / `staff_ranker.dart`.
- **Rationale**: scheduling is a constraint problem — LLMs are non-deterministic and weak at it, while rules are testable and free. Extraction from messy real-world input is the opposite: exactly what vision models are best at and what no amount of Dart solves. Cost is **not** the constraint — ~1 schedule photo/week on Opus 4.8 (`$5/$25` per MTok, high-res vision at 2576px) is roughly **$0.08/photo ≈ $4/year**. The real constraints are privacy and engineering time.
- **Consequences**: `44adeac` shipped the deterministic half (staff ranking) first — no dep, no API, no privacy question. Feature A (photo import) remains the only planned LLM surface. **Privacy middle path preferred**: run on-device ML Kit OCR, then send only the *recognized text* — never the photo — to the LLM for structuring. Cheaper, and no image of a staff board leaves the device, which matters given WiSense's on-device/zero-retention positioning on [[COMMS LINK]].
- **Sequencing (agreed)**: (1) ship what's built — RLS → phone QA → Gate C; (2) staff ranking *(done, `44adeac`)*; (3) Feature A last. **Kill criterion**: if the OCR+verify flow doesn't cut a week's entry below ~5 minutes, drop the photo path and keep `copyPreviousWeek` + suggestions.
- **Related risk**: [[COMMS LINK]] was parked because an AI dependency made it unshippable. Do not repeat that on Apex before it has a paying user.

## 2026-07-21 · Labor cost excluded from staff ranking — `ACTIVE`
- **Decision**: `staff_ranker.dart` does **not** factor `hourly_rate` into its score, though the field is available and `LaborCostPanel` already displays it.
- **Rationale**: ranking people by how cheap they are is a judgment for the human, not a default the app makes silently. The admin can see cost in the existing panel and weigh it themselves.
- **Consequences**: one-line change to reverse if Nicholas decides margin should drive ranking. Flagged as an open question rather than assumed.

## 2026-07-21 · Apex vendors its own `wisense_ui` — fork NOT fully reconciled — `ACTIVE`
- **Finding** (not yet a decision — needs ratification): Apex's `pubspec.yaml` depends on `packages/wisense_ui` (v1.0.0, **2 files**: `spacing.dart`, `loading_indicator.dart`), not canonical `C:\development\packages\wisense_ui` (v0.1.0, **18 files** incl. `text_styles.dart`, `error_banner.dart`). This is a **second fork**, separate from New Horizon's.
- **Why it matters**: [[SYSTEM_ARCHITECT_DIRECTIVE]] §2 mandates `WiSenseTextStyles` as the text-scale base for all apps. `WiSenseTextStyles` **does not exist in Apex's dependency**, so §2 is unsatisfiable there and every Apex widget hardcodes `fontSize`. The 2026-07-20 entry below is marked SUPERSEDED because its consequence line ("the Known fork caveat is now historical") is false — it reconciled New Horizon only.
- **Not fixed deliberately**: canonical `WiSenseTextStyles` derives from `WiSenseThemeText` (travel-app theme), semantically wrong for Apex's brewpub palette. Switching Apex to the canonical package is a Phase 2 refactor, not a drive-by. **Open decision.**

## 2026-07-21 · Tripartite Protocol breached on the Apex feature branch — `ACTIVE`
- **What happened**: Section 0 + Features B/C were designed, implemented, committed **and pushed** without the protocol. Skipped: startup read of [[SYSTEM_ARCHITECT_DIRECTIVE]] + `global_status.md`; the Judicial audit (Claude self-reviewed with `flutter analyze`, which the protocol explicitly forbids); the Completion Report and Delivery Gate.
- **Root cause**: no reachable auditor. There is no audit tooling in `C:\development\scripts\`, no `secrets\` directory, and no Gemini/Groq/OpenRouter credential path — the Judicial branch is currently unimplementable by the agent, so it silently degrades to self-audit.
- **Consequences**: `feat/apex-plan-2026-07-21` is **unaudited and unmerged**; a Completion Report exists marked BLOCKED with MCA/MDT NOT RUN. Remediation commit `594b4be` fixed the Directive §2 conformance defects that self-review missed. **`main` must not take this branch until an external audit runs.** Also: `global_status.md` is stale (2026-07-03, describes deleted `my_ai`), so the Directive's hand-off chain is broken.

## 2026-07-21 · Apex iOS bundle ID = `com.nicholaswittle.apex` — `ACTIVE`
- **Decision**: iOS `PRODUCT_BUNDLE_IDENTIFIER` (6 spots) + the `Info.plist` Supabase auth redirect scheme become `com.nicholaswittle.apex`. Android `applicationId` **stays** `com.wisense.apex`.
- **Rationale**: `com.wisense.apex` was registered to another Apple team; the Mac was building with a throwaway `com.nicholaswittle.apex.local` under a free Personal Team. Android's ID is a separate namespace already registered with Firebase — changing it would break push for no gain.
- **Consequences**: iOS and Android bundle IDs legitimately differ; `docs/LAUNCH_CHECKLIST.md` records the split. Firebase iOS registration must use the new ID. **Supabase auth must allow `com.nicholaswittle.apex://` as a redirect or iOS login will not return to the app.** Xcode signing needs re-selecting after pulling. See [[Apex — Feature Plan Implementation 2026-07-21]].

## 2026-07-21 · Sentry dropped from Apex — `ACTIVE`
- **Decision**: Remove `sentry_flutter` entirely rather than upgrade to 9.x — out of `pubspec.yaml`, `sentryDsn` out of `app_config.dart`, `error_monitoring.dart` reduced to a plain `FlutterError.onError` handler.
- **Rationale**: It had no `SENTRY_DSN`, so it reported nothing in production, and 8.x does not build under current Xcode. Upgrading would have paid a migration cost for a feature that was inert.
- **Consequences**: **Apex currently has no crash reporting** — re-evaluate before store launch if crash visibility matters. Sentry also dropped out of the Linux/macOS/Windows generated plugin registrants. Pods must be regenerated on the Mac. See [[Apex — Feature Plan Implementation 2026-07-21]].

## 2026-07-20 · Vault AI performance layer — `ACTIVE`
- **Decision**: Add execution + founder-memory layer without reviving auto-wiki: [[NOW]], `customers/`, [[business/Experiment Log]], [[VAULT_LINT]]; thin [[CLAUDE]] to point at [[agents]]; single boot chain.
- **Rationale**: Research (Karpathy LLM Wiki + founder OS) showed status drift and missing tasks/customer truth hurt agents more than missing folders. See research canvas / [[log]] 2026-07-20.
- **Consequences**: Boot order is now [[hot]] → [[NOW]] → [[index]] → note. Live status only in hot/NOW (not duplicated in CLAUDE). Monthly lint via [[VAULT_LINT]]. Customer truth replaces retired `/crm`.

## 2026-07-20 · Vault is a curated static reference — `ACTIVE`
- **Decision**: Treat this vault as hand-written, cross-linked knowledge. Retire the MindStudio auto-synthesis pipeline; delete empty `wiki/`, `journal/`, `crm/` folders and the sync scripts/task.
- **Rationale**: The `raw→wiki` loop never ran in 6 months; all real value came from hand-written manifests. Empty scaffolding actively misled agents.
- **Consequences**: Codification is manual/on-request. Knowledge lives as **root notes**. Boot order extended by AI performance layer decision above. `WiSenseVaultAutoSync` + sync scripts removed. See [[log]] 2026-07-20.

## 2026-07-20 · Fork reconciliation complete — `SUPERSEDED` (see 2026-07-21 · Apex vendors its own wisense_ui)
- **Decision**: Promote New Horizon's vendored `wisense_core` (47 files) + `wisense_ui` (19 files) to canonical `C:\development\packages\`; New Horizon consumes canonical via path deps; delete the vendored copies.
- **Rationale**: Ends the long-standing package divergence documented in [[Fork Reconciliation]].
- **Consequences**: All green — core 69/69, UI 21/21, NH 117/117, HV2 7/7. `CLAUDE.md`'s "Known fork" caveat is now historical.

## 2026-07-19 · Deleted dead projects — `ACTIVE`
- **Decision**: Remove `wisense-os`, `my_ai`, `local-agent-work-center`, `command_center` from active status.
- **Rationale**: Abandoned/superseded; see [[Abandoned Projects — Lessons]].
- **Consequences**: Do NOT treat as live. `wisense-os` port 5050 daemon no longer exists (invalidated `wisense-engine-probe`).

## 2026-07-19 · Working stack = Claude CLI + Ollama — `ACTIVE`
- **Decision**: Standardize on Claude Code CLI for build + local Ollama (`11434`) for on-device/model work; abandon other agent platforms as primary.
- **Rationale**: See [[Working Stack — Claude CLI and Ollama]].
- **Consequences**: Community AI plugins route through local Ollama; no unencrypted cloud sync without consent.

## (undated) · Stripe deferred for Apex pilot — `ACTIVE`
- **Decision**: Do not integrate Stripe billing for the initial Apex Scheduler pilot.
- **Rationale**: Out of scope for pilot validation. See [[Stripe]].
- **Consequences**: Revisit post-pilot. *(Confirm/adjust date — carried over from vendor notes.)*

---

Related: [[Home]], [[index]], [[log]], [[hot]], [[WiSense Governance — Rules and Protocols]], [[Fork Reconciliation]], [[Abandoned Projects — Lessons]]
