---
title: Keystore Setup Instructions
tags: [launch, keystore, android, comms-link, apex]
date: 2026-07-20
status: needs-nicholas
---

# Android Keystore Setup — DO THIS FIRST

> You must run these commands yourself. They involve passwords an agent must never handle. Copy-paste ready.

## Step 1: COMMS LINK keystore

Open a terminal and run:

```bash
cd C:\development\projects\wisense_decompression

keytool -genkey -v -keystore android/upload-keystore.jks \
  -keyalg RSA -keysize 2048 -validity 10000 -alias upload
```

It will ask for:
- Keystore password (pick one, SAVE IT)
- Key password (can be same, SAVE IT)
- Your name: "WiSense LLC"
- That's it — the rest can be Enter through

Then create `android/key.properties`:

```properties
storePassword=YOUR_PASSWORD
keyPassword=YOUR_PASSWORD
keyAlias=upload
storeFile=../upload-keystore.jks
```

## Step 2: Apex keystore

```bash
cd C:\development\projects\apex\apex

keytool -genkey -v -keystore android/upload-keystore.jks \
  -keyalg RSA -keysize 2048 -validity 10000 -alias upload
```

Same process. Create `android/key.properties` with the Apex password.

## Step 3: Tell Hermes you're done

Just say "keystores done" and I'll:
1. Run `flutter build appbundle --release` for both apps
2. Verify the .aab files were created
3. Tell you where to find them for Play Console upload

## IMPORTANT

- `android/key.properties` and `android/upload-keystore.jks` are gitignored — they will NOT be committed
- If you lose the passwords, you can never update the app under the same Play Store listing
- Save them in your password manager NOW
- Use a DIFFERENT keystore per app (isolates the signing key)

Related: [[output/Gate C — Android Packaging & Store Listings 2026-07-20]], [[output/Launch Readiness — COMMS LINK and Apex 2026-07-20]], [[NOW]], [[Apex Scheduler]], [[COMMS LINK]]