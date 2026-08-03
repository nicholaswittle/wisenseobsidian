---
title: xcode_notes
tags: [setup, mac, xcode, flutter, ios, howto]
aliases: [Xcode Notes, Mac Setup, iOS Setup]
date: 2026-07-20
---

# 🍎 xcode_notes — Mac + Xcode + Flutter setup

> Step-by-step for getting the Mac building Flutter apps on a physical iPhone. Written to be read **on the Mac** while doing it.
>
> **Xcode is free.** The $99/yr Apple Developer Program is only needed to *distribute* (TestFlight / App Store). Running on your own iPhone needs only a normal Apple ID.

## Where do I type this?

**Terminal on the Mac.** Press `Cmd + Space`, type `terminal`, hit Enter.

Type one command, press Enter, wait for it to finish, then the next. Don't paste them all at once.

- Commands with **`sudo`** ask for your Mac login password. **The cursor does not move while you type it** — no dots, no stars. That's normal. Type it and press Enter.
- The only command that opens a GUI is `open ios/Runner.xcworkspace` (step 5). Everything after that is clicking in Xcode.

---

## 0. Xcode components

When Xcode asks which platforms to install, choose **iOS only**.

| Component | Install? |
|---|---|
| **iOS** | ✅ Required |
| macOS | Included automatically |
| watchOS | ❌ Skip |
| tvOS | ❌ Skip |
| visionOS | ❌ Skip |

Each extra platform is several GB of simulator runtimes that will never get used. They can be added later via **Xcode → Settings → Components**.

**Do NOT use Xcode's "Clone Git Repository" welcome option.** Flutter projects aren't opened in Xcode — Xcode is only the build toolchain underneath. Close that window.

---

## 1. Point Flutter at Xcode, accept the licence

```bash
sudo xcode-select -s /Applications/Xcode.app/Contents/Developer
sudo xcodebuild -runFirstLaunch
sudo xcodebuild -license accept
```

## 2. Install Homebrew (skip if `brew --version` works)

If a later command fails with `command not found: brew`, install it from <https://brew.sh> — it gives one line to paste into the same Terminal.

## 3. Install Flutter on the Mac

The Windows Flutter SDK does not carry over; the Mac needs its own.

```bash
brew install --cask flutter
```

Takes a while and prints a lot of text. Normal.

## 4. Install CocoaPods

Required for Flutter iOS plugins (both apps use several).

```bash
sudo gem install cocoapods
```

## 5. Check the setup

```bash
flutter doctor
```

Prints a checklist of ✓ / ✗. Aim for a green check on the **Xcode** line. This is the single command that shows where things stand — if something's wrong, this says what.

---

## 6. Clone the repos

Both are already pushed to GitHub.

```bash
mkdir -p ~/Developer && cd ~/Developer

git clone https://github.com/nicholaswittle/apex.git
git clone https://github.com/nicholaswittle/wisense-decompression.git
```

**Start with Apex** — it works end to end, so a successful build proves the whole toolchain. (COMMS LINK is parked; see [[COMMS LINK]].)

```bash
cd ~/Developer/apex/apex
flutter pub get
```

> Apex's Flutter project is in the **nested** `apex/apex` folder.

## 7. Set up signing (the one time Xcode's GUI is used)

```bash
open ios/Runner.xcworkspace
```

⚠️ **`.xcworkspace`, not `.xcodeproj`.** Opening the `.xcodeproj` gives cryptic build errors because Flutter uses CocoaPods.

In Xcode:
1. Select **Runner** in the left sidebar.
2. Open the **Signing & Capabilities** tab.
3. Tick **Automatically manage signing**.
4. **Team** → **Add an Account…** → sign in with your normal Apple ID (free).

## 8. Run on the iPhone

Plug in the iPhone 15, unlock it, and tap **Trust** if asked.

```bash
flutter devices     # confirm the iPhone appears
flutter run         # builds and launches on the phone
```

**First launch, the phone will refuse to open it.** On the iPhone go to
**Settings → General → VPN & Device Management** → tap your developer certificate → **Trust**.

---

## Gotchas

| Symptom | Cause / fix |
|---|---|
| First `flutter run` takes many minutes | Normal — CocoaPods resolving + Xcode indexing. Not a hang. |
| `command not found: brew` | Homebrew not installed — see step 2. |
| Cryptic iOS build errors | Opened `.xcodeproj` instead of `.xcworkspace`. |
| App won't open on phone | Trust the developer cert (step 8). |
| Password seems not to type | `sudo` hides input by design. Keep typing. |
| Free signing "expires" after 7 days | Normal for free Apple IDs — just `flutter run` again to re-sign. |

## Free Apple ID vs $99 Developer Program

| Task | Needs $99? |
|---|---|
| Build + run on your own iPhone | ❌ No |
| Keep using it past 7 days | ❌ No (re-run to re-sign) |
| TestFlight / App Store | ✅ Yes |

---

## Notes for these specific apps

- **Apex** — works end to end. Also deploys to **web via Vercel with no Mac at all** (`vercel.json`, `scripts/build_web.sh`), and the `web/manifest.json` makes it installable on any phone via *Add to Home Screen*. See [[Apex Scheduler]].
- **COMMS LINK** — ⏸️ parked (~1.5 GB model download, quality unverified). Don't start here. See [[COMMS LINK]].

Related: [[Apex Scheduler]], [[COMMS LINK]], [[Gate C — Android Packaging & Store Listings 2026-07-20]], [[meta/ENVIRONMENT_MAP]], [[Home]], [[index]]
