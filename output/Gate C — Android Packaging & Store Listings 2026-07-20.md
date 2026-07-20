---
title: Gate C — Android Packaging & Store Listings 2026-07-20
tags: [launch, gate-c, android, play-store, packaging, comms-link, apex, output]
aliases: [Gate C, Play Store Packaging, Store Listing Copy]
date: 2026-07-20
status: active
---

# 📦 Gate C — Android Packaging & Store Listings

> Everything needed to get **COMMS LINK** and **Apex** into Google Play internal testing. Both apps already have release signing wired in `android/app/build.gradle.kts` (`key.properties` → `signingConfigs.release`, debug fallback) and a `scripts/build_release.sh`. No code work remains — this is keystore + build + listing.

| App | Package ID | Store name | Version | Category |
|-----|-----------|-----------|---------|----------|
| COMMS LINK | `com.wisense.commslink` | Comms Link | 1.0.0+1 | Health & Fitness |
| Apex | `com.wisense.apex` | Apex Scheduler | 1.0.0+1 | Business |

⚠️ **You run the keystore + build steps** — they involve entering signing passwords, which an agent must never handle. Commands below are copy-paste ready.

---

## 1. One-time: create an upload keystore (per app)

Use a **separate keystore per app** (isolates the signing key if one is ever compromised). Run from each app's project root (`C:\development\projects\wisense_decompression` and `C:\development\projects\apex\apex`):

```bash
keytool -genkey -v -keystore android/upload-keystore.jks \
  -keyalg RSA -keysize 2048 -validity 10000 -alias upload
```

It prompts for a store password, a key password, and a name/org (name/org can be anything — e.g. "WiSense LLC"). **Save both passwords in your password manager** — losing them means you can never update the app under the same listing.

Then create `android/key.properties` (copy from `android/key.properties.example`) — **do not commit it** (already gitignored):

```properties
storePassword=<the store password you just set>
keyPassword=<the key password you just set>
keyAlias=upload
storeFile=../upload-keystore.jks
```

---

## 2. Build the release bundle

```bash
flutter pub get
flutter build appbundle --release
```

Output: `build/app/outputs/bundle/release/app-release.aab` → this is what you upload.

> COMMS LINK is fully on-device, so no `--dart-define` needed. **Apex** needs its Supabase keys at build time — use its script or pass them:
> ```bash
> flutter build appbundle --release \
>   --dart-define=SUPABASE_URL=... --dart-define=SUPABASE_ANON_KEY=...
> ```
> (`scripts/build_release.sh android` wraps this.)

**Apex prerequisite:** apply the RLS migration (`supabase/migrations/20260720000000_launch_blockers_rls.sql`) to prod **before** onboarding real staff, or tenant isolation isn't enforced on the live DB.

---

## 3. Play Console (per app)

1. Create app → default language English (US), app (not game), free.
2. **Internal testing** track → create release → upload the `.aab` → add tester emails → roll out.
3. Complete the required declarations: **Store listing**, **Content rating** (questionnaire), **Data safety**, **Privacy policy URL**, **Target audience**.

Account: Google Play Console, **$25 one-time**, no Mac required.

---

## 4. Store listing copy

### COMMS LINK — "Comms Link"
- **Short description (≤80):** `A private, on-device AI space to decompress after a hard call.`
- **Full description:**
```
Comms Link is a private space to decompress. Talk through a hard shift, a
tough call, or a heavy moment with an AI companion that runs entirely on your
device — nothing you say is sent to the cloud or stored on a server.

• On-device AI — the model runs locally; zero cloud retention.
• Guided debrief — a calm, structured way to talk it out and wind down.
• Breathing and grounding built in.
• Locked behind your fingerprint, face, or PIN — your reflections stay yours.

Comms Link is a wellbeing and decompression tool, not a medical device or a
substitute for professional care. If you are in crisis, contact your local
emergency services or a crisis line.
```
- **Data safety:** *No data collected, no data shared.* (Everything is on-device.) This is COMMS LINK's strongest selling point — say it plainly.
- **Content rating:** likely Everyone / PEGI 3, but the debrief content mentions crisis — answer the questionnaire honestly re: sensitive themes.
- **Privacy policy:** host `docs/PRIVACY_POLICY` content at a public URL (GitHub Pages or wisense site).

### Apex — "Apex Scheduler"
- **Short description (≤80):** `Shift scheduling, swaps, availability, and time clock for small teams.`
- **Full description:**
```
Apex Scheduler is staff scheduling built for small hospitality teams. Managers
build the week, staff see their shifts, and everyone stays in sync.

• Shift calendar — week and month views, published by managers.
• Open shifts and swaps — claim open shifts and request covers with approval.
• Availability and time-off requests with live status updates.
• Time clock — clock in and out against scheduled shifts.
• Sidework and role assignments per shift.
• Push notifications for schedule changes and approvals.

Built for Jigsy's Brewpub and small hospitality crews.
```
- **Data safety:** *Collects* — email, name, work-schedule data, optional push token. *Shared with* — Supabase (auth/database), Firebase (push). Data is encrypted in transit; user can request deletion. Not sold.
- **Content rating:** Everyone / business app.
- **Privacy policy:** host `docs/PRIVACY_POLICY.md` at a public URL.

---

## 5. Required graphic assets (both apps)

You must supply these in Play Console (I can't generate binaries):

| Asset | Spec |
|-------|------|
| App icon | 512×512 PNG, 32-bit |
| Feature graphic | 1024×500 PNG/JPG |
| Phone screenshots | 2–8, min 320px, 16:9 or 9:16 |
| (optional) 7"/10" tablet shots | if targeting tablets |

Confirm the launcher icon in the app is final first (`flutter_launcher_icons` if not yet set).

---

## 6. iOS (later, no Mac)
Both have `.github/workflows/build-ios.yml`; **Codemagic** is the easiest path once the Apple Developer account ($99/yr) exists. See [[output/Launch Readiness — COMMS LINK and Apex 2026-07-20]] and `docs/LAUNCH_WITHOUT_MAC.md`.

---

Related: [[output/Launch Readiness — COMMS LINK and Apex 2026-07-20]], [[COMMS LINK]], [[Apex Scheduler]], [[Stripe]], [[business/Launch Playbook]], [[index]], [[Home]]
