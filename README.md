# FocusGuard AI

A single Android app you install directly on your phone. No account, no cloud
backend, no internet server of ours involved — everything runs locally.

## What it does

- **Website blocking**: a local VPN (`VpnService`) inspects outgoing DNS
  requests on-device and refuses to resolve domains in blocked categories
  (adult, gambling, dating, streaming, social-media-in-browser). No traffic
  ever leaves your phone to any server run by this app — DNS queries for
  *allowed* domains are simply relayed to Cloudflare (1.1.1.1).
- **App blocking**: an `AccessibilityService` watches for a blocked app
  coming to the foreground (YouTube, Instagram, TikTok, Facebook, Reddit, X,
  Snapchat, Discord, Telegram, Netflix, Prime Video, Disney+, or any custom
  package you add) and immediately sends the device home and shows a
  "This app is blocked" screen.

## How to build

### Option A: On a computer with Android Studio
1. Open this folder in Android Studio (Koala or newer).
2. Let Gradle sync.
3. Run on a device or emulator with API 26+.

### Option B: From your phone, no computer needed
This project includes a GitHub Actions workflow (`.github/workflows/build.yml`)
that compiles the APK in the cloud.

1. On GitHub (in your phone's browser), create a free account if you don't
   have one, then create a new **empty** repository.
2. Use GitHub's "Add file → Upload files" web UI to upload everything from
   this unzipped folder into that repository (keep the folder structure —
   upload the whole tree, not a flattened list). Commit to the `main` branch.
3. Go to the repo's **Actions** tab. A workflow run should start
   automatically (it triggers on every push to `main`). If it doesn't, click
   **Build APK → Run workflow**.
4. Wait a few minutes for the run to finish (green checkmark).
5. Open the finished run, scroll to **Artifacts**, and download
   `FocusGuardAI-debug-apk`. That download is a `.zip` containing
   `app-debug.apk` — unzip it, then tap the `.apk` file on your phone to
   install it (Android will prompt you to allow installing from this source
   the first time).

## First-run setup (required, one-time, done by the user in-app)

1. Open the app, toggle **Protection** on → Android shows the standard
   "Connection request" VPN consent dialog → accept it. This grant is
   mandatory and cannot be skipped or automated; Android requires explicit
   user consent before any app can start a VPN tunnel.
2. Tap **Enable Accessibility permission** → in system settings, find
   "FocusGuard AI" and turn it on. This is also a manual, one-time step;
   Play Store policy and Android both prohibit auto-enabling accessibility
   services.
3. Toggle the categories and apps you want blocked.

## Honest limitations (please read)

- **Accessibility Service + Play Store policy**: Google restricts what
  accessibility services can be used for. An app-blocking use case like this
  is generally allowed for "parental control"/"digital wellbeing" apps, but
  if you plan to publish to the Play Store (rather than just installing the
  APK yourself/sideloading), you'll need to fill out Google's Accessibility
  usage declaration and may face review scrutiny. This is documented at
  https://support.google.com/googleplay/android-developer/answer/10964491
- **A determined user can defeat this on their own device.** Android gives
  the device owner full control: they can uninstall the app, revoke VPN/
  Accessibility permissions, or disable the app in Settings at any time
  (unless it's set up as a Device Owner/Device Admin via Android Enterprise,
  which requires MDM enrollment, not a simple install). This is a deliberate
  Android design constraint, not a bug — Android doesn't allow arbitrary
  apps to make themselves un-removable by the phone's owner. If your goal is
  parental control on a **child's** device that you manage, look at Android's
  official **Family Link**/**Android Enterprise "dedicated device"** APIs
  for tamper-resistant enforcement.
- **Only one VPN can be active at a time.** If the user has another VPN app
  running, FocusGuard's DNS filtering will conflict with it.
- **The DNS filter only inspects DNS (UDP/53).** Apps that hardcode DNS-over-
  HTTPS/TLS to their own resolvers (e.g. some browsers' "secure DNS") or that
  connect directly by IP can bypass DNS-based blocking. Closing this gap
  fully requires deeper packet inspection (SNI-based TLS filtering), which is
  a natural next module to add.
- **The starter domain lists are small** (a handful of well-known sites per
  category) so you can see the mechanism working immediately. Swap in a full
  hosts-file-style blocklist (e.g. the public StevenBlack "unified + porn"
  hosts list) in `DomainBlocklists.kt` for real coverage — it's just a set of
  strings, so this is a copy-paste change.
- **No AI classifier is included** in this build, since it requires a
  backend. If you want unknown domains auto-categorized, that needs a small
  server (even a free-tier Cloud Function) to call a classification API —
  happy to add that as a following module if useful.

## Project layout

```
app/src/main/java/com/focusguard/ai/
  MainActivity.kt                  – UI: protection switch, category & app toggles
  BlockedAppOverlayActivity.kt      – full-screen "blocked" message
  BootReceiver.kt                   – restarts protection after reboot
  data/BlockStore.kt                 – on-device settings (SharedPreferences)
  data/DomainBlocklists.kt           – domain lists per category
  vpn/FocusGuardVpnService.kt        – the VPN + DNS filtering engine
  vpn/DnsPacketParser.kt             – IPv4/UDP/DNS packet parsing & building
  accessibility/AppBlockerAccessibilityService.kt – app-level blocking
```
