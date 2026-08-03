---
title: COMMS LINK — Code Reference
tags: [app, reference, comms-link, launch-priority]
aliases: [wisense_decompression, comms-link-code]
---

# COMMS LINK — Code Reference

**Repo:** `C:\development\projects\wisense_decompression`
**Bundle ID:** `com.wisense.commslink`
**Stack:** Flutter · on-device Gemma 2B-IT (`flutter_gemma`) · local_auth · flutter_secure_storage · crypto
**Platforms:** iOS · Android
**No cloud, no accounts, no data retention.**

## File inventory

### lib/ (23 Dart files)

#### constants/
- `ai_persona.dart` — AI system prompt + banned phrases base rules
- `app_config.dart` — build config (privacy URL, feature flags)
- `crisis_support.dart` — crisis resource references

#### models/
- `message.dart` — chat message model

#### screens/ (8)
- `breathing_screen.dart` — breathing exercise UI
- `decompress_screen.dart` — main debrief chat screen (largest file, ~1100+ lines)
- `legal_gate_screen.dart` — legal/privacy acceptance gate
- `loading_screen.dart` — startup + model load
- `lock_screen.dart` — **UNTRACKED** — PIN/biometric lock UI
- `onboarding_screen.dart` — first-run intro
- `resources_drawer.dart` — crisis resources drawer
- `settings_screen.dart` — **UNTRACKED** — settings (lock, voice, metrics)
- `web_marketing_screen.dart` — web-only marketing landing

#### services/ (10)
- `ai_service.dart` — on-device Gemma inference
- `ai_service_web.dart` — web stub (no on-device model on web)
- `crisis_classifier.dart` — **UNTRACKED** — keyword/weapon crisis detection
- `debrief_orchestrator.dart` — **UNTRACKED** — 4-state debrief flow (intake → validate → commit → close)
- `lock_service.dart` — **UNTRACKED** — biometric + PIN lock (flutter_secure_storage + local_auth + crypto)
- `metrics_service.dart` — **UNTRACKED** — session/validator/crisis counters
- `notification_service.dart` — local notifications
- `notification_service_stub.dart` — web stub
- `response_validator.dart` — **UNTRACKED** — bans therapist-register phrases
- `voice_service.dart` — speech_to_text + flutter_tts

#### theme/
- `app_theme.dart` — **UNTRACKED** — app theme

#### widgets/ (6)
- `blinking_cursor.dart` — **UNTRACKED**
- `breathing_circle.dart` — **UNTRACKED** — animated breathing circle
- `decompress_log_entry.dart` — **UNTRACKED**
- `input_bar.dart` — **UNTRACKED**
- `mic_button.dart` — **UNTRACKED**
- `soundwave_animation.dart` — **UNTRACKED**

### test/ (4)
- `crisis_detection_test.dart`
- `debrief_orchestrator_test.dart` — **UNTRACKED**
- `decompress_widgets_test.dart` — **UNTRACKED**
- `response_validator_test.dart` — **UNTRACKED**

### assets/
- `scenarios.json` — **UNTRACKED**
- `icon/` — app icons

## Dependencies (pubspec.yaml)

| Package | Purpose |
|---------|---------|
| `flutter_gemma` / `flutter_gemma_mediapipe` | On-device Gemma 2B-IT inference |
| `provider` | State management |
| `shared_preferences` | Local prefs |
| `url_launcher` | Open crisis resources |
| `speech_to_text` | Voice input |
| `flutter_tts` | Voice output |
| `flutter_local_notifications` | Crisis notifications |
| `timezone` / `flutter_timezone` | Notification scheduling |
| `local_auth` | Biometric lock |
| `flutter_secure_storage` | Secure PIN storage |
| `crypto` | PIN hashing |

## Architecture

- **State management:** Provider
- **AI:** On-device Gemma via flutter_gemma (no network calls)
- **Voice:** speech_to_text → AI → flutter_tts
- **Crisis detection:** CrisisClassifier scans input for self-harm keywords → crisis_support resources surface
- **Response validation:** ResponseValidator bans therapist-register ("I understand how you feel") — keeps peer-support voice
- **Debrief flow:** DebriefOrchestrator runs 4 states: intake → validate (1-line AI confirm) → commit (1-line directive + TTS) → close (ritual)
- **App lock:** LockService + LockScreen — biometric or PIN, PIN hashed with crypto, stored in flutter_secure_storage
- **Metrics:** MetricsService tracks sessions started/completed, validator rejections, crisis taps, durations

## Git state

- **Branch:** `cursor/fable-5-fleet-cursorrules-447c` (NOT main)
- **Last commit:** 2 weeks ago — "Add Fable 5 fleet .cursorrules"
- **Uncommitted:** 35 files (20 modified + 15 untracked) — see UNTRACKED markers above
- **Working tree:** 87MB

## Launch blockers

- [ ] Commit 35 uncommitted files
- [ ] Merge cursor branch → main
- [ ] `flutter analyze` + `flutter test` green
- [ ] Privacy policy hosted at public URL
- [ ] Store screenshots + listings

Related: [[projects/dormant/COMMS LINK]], [[Code Reuse Analysis]]