---
title: Launch Playbook
tags: [business, launch, playbook, app-store]
date: 2026-07-20
---

# Launch Playbook — App Store Launch Checklist

Step-by-step for getting WiSense apps from local build to store listing.

## Phase 1: Code Gate

- [ ] `flutter analyze` — 0 issues
- [ ] `flutter test` — 100% pass
- [ ] Git: push `main` to `origin/main`
- [ ] No untracked governance files (AGENTS.md, CLAUDE.md — .gitignore or commit)
- [ ] No dead branches (prune before launch)

## Phase 2: Android Signing

- [ ] Generate upload keystore: `keytool -genkey -v -keystore upload-keystore.jks -keyalg RSA -keysize 2048 -validity 10000 -alias upload`
- [ ] Create `android/key.properties`:
  ```
  storePassword=***
  keyPassword=***
  keyAlias=upload
  storeFile=../upload-keystore.jks
  ```
- [ ] Configure `android/app/build.gradle` signingConfig
- [ ] Add keystore to `.gitignore` (NEVER commit)

## Phase 3: App Store Assets

- [ ] App icon (512x512 for Play Store, 1024x1024 for App Store)
- [ ] Feature graphic (1024x500 for Play Store)
- [ ] 2–8 screenshots (portrait + landscape)
- [ ] Short description (80 chars)
- [ ] Full description (4000 chars)
- [ ] Privacy policy URL (host publicly)
- [ ] App category + content rating questionnaire

## Phase 4: Build & Submit

### Google Play (no Mac needed)
- [ ] `flutter build appbundle --release`
- [ ] Create Play Console account ($25 one-time)
- [ ] Create app listing, upload `.aab`
- [ ] Internal testing track first
- [ ] Add test users
- [ ] Review → closed testing → open testing → production

### Apple App Store (needs Mac or cloud CI)
- [ ] Apple Developer account ($99/yr)
- [ ] `flutter build ipa --release` (on Mac or Codemagic)
- [ ] App Store Connect: create listing
- [ ] Upload via Transporter or `xcrun altool`
- [ ] TestFlight beta → production

## Phase 5: Post-Launch

- [ ] Monitor crash reports (Firebase Crashlytics or Sentry)
- [ ] Respond to first reviews within 24h
- [ ] Track activation + retention metrics
- [ ] Plan first update (2-week cadence)

## WiSense-Specific Notes

- COMMS LINK: easiest — no backend, no accounts, privacy story writes itself
- Apex: must fix RLS + security before listing (see [[output/Apex Security Audit 2026-07-19]])
- New Horizon: web-first (Vercel), app store later when affiliate revenue justifies it
- See [[output/Launch Readiness — COMMS LINK and Apex 2026-07-20]] for current status

Related: [[business/Business Model Canvas]], [[business/Go-to-Market]], [[business/Pricing Strategy]]