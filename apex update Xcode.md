---
title: 2026-07-21 Daily Log
tags: [daily-log, log, apex, ios, mac, flutter, setup]
date: 2026-07-21
---

# 2026-07-21

## Apex iOS build — Mac setup (CONTINUE HERE TOMORROW)

Got the **Mac (M1 Pro)** building and running **Apex** on a physical iPhone for the first time. Claude Code drove the
whole setup. The Mac is the twomodel machine (see [[Working Stack — Claude CLI and Ollama]]); the vault normally lives
on the desktop — this note documents Mac-side work so it syncs back.

**Goal was hit:** Apex now BUILDS, SIGNS, INSTALLS, and LAUNCHES on the iPhone in debug mode. Only remaining step
needs my Supabase keys (not Claude).

### Installed on the Mac (persists)
- **Xcode 26.6** + license accepted (`sudo xcodebuild -runFirstLaunch` / `-license accept`)
- **Homebrew** at `/opt/homebrew/bin/brew` (not on non-login shell PATH — use the full path)
- **Flutter 3.44.7** (`brew install --cask flutter`)
- **CocoaPods 1.17.0 via brew** — `sudo gem install cocoapods` FAILED because the system Ruby is 2.6.10 and the `ffi`
  gem needs Ruby ≥3.0. Fix = `brew install cocoapods` (bundles modern Ruby). The `sudo gem` route in [[xcode_notes]] is
  a dead end on this Mac.
- Apex cloned to `~/Developer/apex` (Flutter project is at the **repo root**, NOT nested `apex/apex` — [[xcode_notes]]
  is stale on that point)

### Device + signing
- iPhone **"Nick Nasty"** (iPhone 15, iOS 26.5.2, UDID `00008120-000179CA3AE1A01E`) — paired, **Developer Mode
  enabled** (Settings → Privacy & Security; the menu item only appears after Xcode recognizes the device)
- Signed by a **free Personal Team** (I started but never paid the $99 enrollment). Team `GWJ2K5U9T6`. Free-team
  installs re-sign every 7 days.
- Signing red-errors worked through: bundle ID conflict → no devices → Developer Mode. All resolved.

### Local mods to `~/Developer/apex` (all revertable via `git checkout .` — fresh clone)
- Bundle ID `com.wisense.apex` → `com.nicholaswittle.apex.local` in `ios/Runner.xcodeproj/project.pbxproj` (original was registered to another team; free Personal Team couldn't claim it). Info.plist URL scheme
  left as `com.wisense.apex` for Supabase auth redirect. Backup at `project.pbxproj.bak`.
- iOS deployment target `13.0` → `15.0` (firebase-core / firebase-messaging require iOS 15).
- **Dropped `sentry_flutter`** (pubspec + `lib/core/error_monitoring.dart`): `sentry_flutter 8.14.2` is incompatible with the `sentry-cocoa 8.58.4` SPM resolves under Xcode 26 (`SentryBinaryImageCache.image`
  removed). Sentry was unused locally (no `SENTRY_DSN`). `error_monitoring.dart` now just sets `FlutterError.onError` + runs `appRunner`. Re-enable by upgrading Sentry to 9.x once we choose to.

### Build state
- App compiles + signs + installs + launches via `flutter run -d 00008120-000179CA3AE1A01E`.
- Firebase init is try/caught (no `GoogleService-Info.plist` → "Firebase init skipped", benign; app does NOT crash).
- First-launch "Dart VM Service not discovered" cleared on the second `flutter run` attempt (common on free Personal Team first launch).

### 🔴 Continue here tomorrow
App shows its built-in **"Supabase not configured"** screen. Needs:
1. `SUPABASE_URL` + `SUPABASE_ANON_KEY` (anon key is public/client-side — safe). Get from Supabase dashboard → project → Settings → API, or from the Vercel env vars.
2. Create `~/Developer/apex/.env.local` from `.env.local.example` with those two values.
3. Run `scripts/run_dev.sh` — it sources `.env.local` and runs `flutter run --dart-define=SUPABASE_URL=… --dart-define=SUPABASE_ANON_KEY=…` (compile-time, so it REBUILDS with creds baked in).
4. App should then render real data instead of the config-missing screen.

Optional later: release/profile build for home-screen icon launch (debug builds only start from the Flutter tool); Supabase auth-redirect URL config if OAuth/email-link login is used.

### Notes
- `wisense-decompression` repo is **private** (404 unauthenticated) — can't clone from the Mac without credentials. Not blocking; Apex first, [[COMMS LINK]] parked per [[Plan of Attack — Build While Mac-Blocked
  2026-07-20]].
- This daily note was written from the Mac, which has **no GitHub auth** (no `gh`, no SSH key, no credential helper) — it could not be pushed from the Mac. It needs to be added to the vault from a machine with
  access (desktop Obsidian-Git, or GitHub web UI), or the Mac needs a PAT/SSH key set up.

Related: [[xcode_notes]], [[Apex Scheduler]], [[Apex Scheduler — Code Reference]], [[Plan of Attack — Build While Mac-Blocked 2026-07-20]], [[Supabase]], [[Working Stack — Claude CLI and Ollama]], [[NOW]],
[[log]]