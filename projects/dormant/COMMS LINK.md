---
title: COMMS LINK
tags: [app, launch-priority, comms-link]
aliases: [wisense_decompression]
---

# COMMS LINK

> ⏸️ **PARKED 2026-07-20 — deliberately, not abandoned.** Too heavy for a first launch: **~1.5 GB model download before first use**, plus unresolved model-quality risk. Ship something smaller first; return to this once a product is live and there are real users. Nothing is broken — see *Where it stands*. **Revisit trigger:** a much smaller/quantized model, or a first product already shipped.

On-device AI decompression assistant. Voice or text debrief with a local LLM. No cloud, no accounts, no data retention.

| | |
|---|---|
| **Repo** | `C:\development\projects\wisense_decompression` |
| **Bundle ID** | `com.wisense.commslink` |
| **Stack** | Flutter · on-device Gemma 2B-IT · local_auth · flutter_secure_storage |
| **Platforms** | iOS · Android |

## Launch blockers

- [x] Commit 35 uncommitted files (done — 9 atomic commits, merged to main)
- [x] Merge cursor branch → main (done)
- [x] `flutter analyze` + `flutter test` green (59/59 pass)
- [ ] Push local `main` branch to `origin/main` (ahead by 10 commits)
- [ ] Privacy policy hosted at public URL
- [ ] Store screenshots + listings

## Where it stands (2026-07-20, end of session)

**Code:** analyze clean, 59/59 unit/widget tests, **pushed — `origin/main` in sync**.

**Fixed this session:**
- The original build could **never** download its model. Hardcoded `google/gemma-1.1-2b-it-gpu-int4` is licence-gated and returns **401 anonymously** — every user would have failed at first launch. The 59 tests never caught it because none exercise the model.
- `ModelConfig` + `ModelPreset` (`94cecea`, `750a275`): default is now **Qwen2.5-1.5B (Apache-2.0, ungated, verified HTTP 200 — no self-hosting or token needed)**; also `qwen05` (521 MB) and `gemma2b` (gated). `MODEL_URL`/`MODEL_TOKEN` override; a token is never hardcoded.
- GPU→CPU fallback in `ai_service` (was GPU-only, died on hardware without a delegate).
- `integration_test/model_smoke_test.dart` — installs, generates, asserts non-empty reply.

**Unverified — and the reason it's parked:**
- ⚠️ **The model has never actually run.** Needs a physical iPhone 15 run (Mac + Xcode — **Xcode is free**; the $99 is only for distribution). Simulators can't do MediaPipe GPU.
- ⚠️ **Qwen vs Gemma reply quality unknown.** The persona in `ai_persona.dart` was few-shot tuned for Gemma. Small instruct models default to exactly the therapeutic register `ResponseValidator` bans ("I understand how you feel", "you're not alone") — so the likely failure is **frequent fallback to canned replies**: not unsafe, just lifeless. Measure validator pass-rate before trusting any model here.

**Good news for whenever it resumes:** safety is **not** model-dependent. `CrisisClassifier` is deterministic and runs *before* the model; `ResponseValidator` gates output with retry + hard fallback. Swapping models cannot break crisis handling.

**Resume order:** smoke test on device → measure validator pass-rate per role → pick the model → *then* package. Don't do store assets before that.

Related: [[Home|Priority launches]], [[Audit Findings Loop]]