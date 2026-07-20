---
title: COMMS LINK — Commit Plan
tags: [plan, comms-link, launch-priority]
status: pending-ratification
---

# COMMS LINK — Commit Plan for 35 Uncommitted Files

## Verification (done)

- `flutter analyze` — No issues found
- `flutter test` — 59/59 passed
- Working tree is on `cursor/fable-5-fleet-cursorrules-447c`, 2 weeks since last commit

## Proposed commit grouping (9 atomic commits) — REVISED after Judicial audit

### Commit 1: Platform config for biometric lock + Java 17
**Files:**
- `.gitignore` (adds `.vercel`)
- `android/app/src/main/AndroidManifest.xml` (biometric + fingerprint permissions)
- `android/app/src/main/kotlin/com/wisense/commslink/MainActivity.kt` (FlutterFragmentActivity for biometric)
- `android/build.gradle.kts` (Java/Kotlin 17 target)
- `ios/Runner/Info.plist` (NSFaceIDUsageDescription)
- `linux/flutter/generated_plugin_registrant.cc` + `.cmake`
- `macos/Flutter/GeneratedPluginRegistrant.swift`
- `windows/flutter/generated_plugin_registrant.cc` + `.cmake`

**Message:** `feat: platform config for biometric lock (Face ID, fingerprint, Java 17)`

### Commit 2: Dependencies (pubspec only — no asset file)
**Files:**
- `pubspec.yaml` (adds local_auth, flutter_secure_storage, crypto, speech_to_text bump)
- `pubspec.lock`

**Message:** `feat: add local_auth, flutter_secure_storage, crypto deps`

### Commit 3: Crisis detection system
**Files:**
- `lib/constants/crisis_support.dart` (expanded crisis keyword list)
- `lib/services/crisis_classifier.dart` (NEW — keyword/weapon/semantic hopelessness classifier)
- `test/crisis_detection_test.dart` (new crisis classifier tests + expanded support tests)

**Message:** `feat: CrisisClassifier with semantic hopelessness + weapon-context detection`

### Commit 4: Debrief domain logic (response validator + orchestrator + FSM prompts)
**Files:**
- `lib/constants/ai_persona.dart` (buildFsmPrompt for validate/commit states)
- `lib/services/response_validator.dart` (NEW — bans therapist-register phrases)
- `lib/services/debrief_orchestrator.dart` (NEW — 4-state debrief FSM)
- `test/debrief_orchestrator_test.dart` (NEW)
- `test/response_validator_test.dart` (NEW)

**Message:** `feat: debrief orchestrator FSM + response validator domain logic`

### Commit 5: Service integration (ai_service + voice_service)
**Files:**
- `lib/services/ai_service.dart` (modified — FSM prompt integration)
- `lib/services/voice_service.dart` (modified — TTS state gating)

**Message:** `feat: integrate debrief FSM into ai_service + voice_service`

### Commit 6: UI orchestration (decompress_screen + loading_screen + main)
**Files:**
- `lib/screens/decompress_screen.dart` (modified — debrief flow integration, 1217 line diff)
- `lib/screens/loading_screen.dart` (modified — lock check on startup)
- `lib/main.dart` (modified — DebriefOrchestrator provider + settings route)
- `test/decompress_widgets_test.dart` (NEW)

**Message:** `feat: debrief flow UI integration in decompress_screen + app routing`

### Commit 7: App lock (service + screen)
**Files:**
- `lib/services/lock_service.dart` (NEW — biometric + PIN)
- `lib/screens/lock_screen.dart` (NEW — lock UI)

**Message:** `feat: app lock with biometric + PIN authentication`

### Commit 8: Settings + metrics
**Files:**
- `lib/services/metrics_service.dart` (NEW — session/validator/crisis counters)
- `lib/screens/settings_screen.dart` (NEW — settings UI)

**Message:** `feat: settings screen + session metrics tracking`

### Commit 9: Theming + widgets + docs + scenarios
**Files:**
- `lib/theme/app_theme.dart` (NEW)
- `lib/widgets/` (NEW — blinking_cursor, breathing_circle, decompress_log_entry, input_bar, mic_button, soundwave_animation)
- `CLAUDE.md` (NEW — project context)
- `assets/scenarios.json` (NEW — scenario data — only place this file is committed)

**Message:** `feat: UI theme + widgets; docs: project context + scenarios`

## After commits: merge to main

Once all 9 commits land on the cursor branch, merge to main:
```
git checkout main
git merge cursor/fable-5-fleet-cursorrules-447c
```

## What is NOT included
- `android/build/` — untracked build output, should NOT be committed. Will add to .gitignore.

## Risk assessment

- **Risk level:** Medium (touches shared logic, multiple files, 1217-line diff on decompress_screen)
- **Destructive?** No — these are additive commits only, no deletions
- **Atomic testability:** Each commit should independently pass `flutter analyze` — will verify after each commit

Related: [[COMMS LINK]], [[COMMS LINK — Code Reference]]