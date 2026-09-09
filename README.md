# Phishing Guard — Android SMS Blocking App (Flutter + Kotlin)

Blocks phishing SMS **before** they reach your inbox by registering as the
device's default SMS app, scoring every incoming message with a fast local
heuristic engine, and only forwarding safe/suspicious ones to the normal
Android SMS provider. Phishing-scored messages are quarantined in-app only.

## How blocking actually works (read this first)

- Android only lets an app intercept a message **before delivery** if that
  app is the **default SMS app**. This app requests that role on first run.
- Detection happens in two passes:
  1. **Native heuristic pass** (`DetectionEngine.kt`) — runs synchronously
     inside the `SMS_DELIVER` broadcast receiver, fully offline, <50ms.
     This is what makes the actual block/allow decision, because it has to
     finish inside Android's short broadcast execution window.
  2. **AI refinement pass** (Gemini, `ai_service.dart`) — runs afterwards,
     inside the app, to give a richer explanation and can upgrade a
     "suspicious" verdict, shown in the message detail screen.
- Messages scored `PHISHING` (≥55) are **never written to the system SMS
  provider** — they only exist in this app's own database until you review
  them. Everything else is delivered normally so the phone still works as
  a regular texting app.

## What this does NOT do (and can't, legitimately)

- It **cannot** block or hide messages inside other apps (WhatsApp,
  Telegram, Instagram DMs, etc.). There's no Android API for that — apps
  that claim to do this either don't work or abuse Accessibility
  permissions in ways Play Store bans as spyware behavior.
- Email blocking would need a separate Gmail API integration (OAuth +
  server-side filters) — not covered in this scaffold yet.
- Image/MMS OCR scanning is stubbed but not implemented yet (see Roadmap).

## Tunable detection (Settings → Detection Tuning)

Nothing in the scoring logic is hardcoded. Everything lives in a JSON config
(`android/app/src/main/assets/detection_config.json` on first install, then
copied to app-internal storage) and can be edited **live from inside the
app**, no rebuild needed:

- **Thresholds** — the score cutoffs for `PHISHING` (blocked) and
  `SUSPICIOUS` (flagged) verdicts.
- **Weights** — how many points each signal contributes (urgency language,
  credential-harvesting phrases, spoofed brand names, links, shortened
  links, numeric-sender-claiming-to-be-a-bank).
- **Keyword lists** — the actual word/phrase lists for urgency, credential,
  and brand detection, plus the known link-shortener domains — all editable
  as comma-separated text.

Flow: `DetectionEngine.kt` never hardcodes anything — it takes a
`DetectionConfig` object as a parameter, loaded fresh from disk by
`SmsDeliverReceiver` on every incoming SMS. `MainActivity.kt` exposes
`getConfig` / `updateConfig` / `resetConfig` over the MethodChannel so the
Flutter tuning screen (`lib/screens/tuning_screen.dart`) can read and write
it directly. Bad JSON is rejected before it's written (validated by
re-parsing), so you can't corrupt the live config from the UI.

If you want to change the *defaults* that ship with a fresh install, edit
`android/app/src/main/assets/detection_config.json` directly.

## Get an installable APK without installing anything locally (GitHub Actions)

This project includes `.github/workflows/build.yml`, which builds the
release APK automatically on GitHub's servers.

1. Go to https://github.com/new and create a new repository (any name,
   public or private, no need to add a README there).
2. On your computer, unzip this project, then in that folder run:
   ```
   git init
   git add .
   git commit -m "Phishing Guard"
   git branch -M main
   git remote add origin https://github.com/<your-username>/<repo-name>.git
   git push -u origin main
   ```
   (No git installed? On the new repo's GitHub page, use "uploading an
   existing file" and drag the whole unzipped folder in instead.)
3. On GitHub, open your repo → the **Actions** tab. A workflow run called
   "Build APK" starts automatically (takes ~5–8 minutes).
4. When it finishes (green check), click into that run → scroll to
   **Artifacts** → download **phishing-guard-apk** (a zip containing
   `app-release.apk`).
5. Transfer that APK to your Android phone (email it to yourself, Google
   Drive, USB, whatever) and tap it to install. You'll need to allow
   "install from unknown sources" the first time Android asks.
6. Open the app, tap **"Set as default SMS app"**, and you're live.

You can also trigger a rebuild anytime from the Actions tab (the
"Run workflow" button) without pushing a new commit.

## Setup

1. Install Flutter (3.x) and Android Studio / SDK (API 34).
2. `flutter pub get`
3. `flutter run` (needs a physical device or emulator with a SIM/SMS
   capability — most emulators support fake SMS via `adb emu sms send`).
4. On first launch, tap **"Set as default SMS app"** — Android will show
   its native role-request dialog.
5. Open **Settings** in the app and paste a Gemini API key
   (from https://aistudio.google.com/apikey) to enable the AI refinement
   pass. The app works without it — heuristic-only mode still blocks.
6. Test: `adb emu sms send +910000000 "URGENT: Your SBI account is
   suspended. Verify now at bit.ly/xyz123 or it will be blocked in 24
   hours."` — should get quarantined and trigger a "Message blocked"
   notification.

## Project structure

```
lib/
  main.dart                  # App entry, theme, Provider setup
  models/message_model.dart  # ScannedMessage
  services/
    sms_service.dart         # MethodChannel/EventChannel bridge to Kotlin
    ai_service.dart           # Gemini API call for second-pass analysis
    settings_service.dart     # Secure storage for API key
    messages_provider.dart    # App state, live refresh, AI enrichment loop
  screens/
    home_screen.dart          # Dashboard + default-SMS-app onboarding
    history_screen.dart       # Full message list
    message_detail_screen.dart# Explainability view
    settings_screen.dart      # API key entry

android/app/src/main/kotlin/com/phishingguard/app/
  DetectionEngine.kt   # Offline heuristic scorer (urgency/credential/brand/link checks)
  SmsDeliverReceiver.kt# THE core piece — intercepts SMS_DELIVER, decides block/allow
  LocalDb.kt           # SQLite: message history + quarantine
  MainActivity.kt      # Flutter <-> native bridge (MethodChannel/EventChannel)
  SmsEventBridge.kt    # Pushes live "new message" events to Flutter
  MmsReceiver.kt, HeadlessSmsSendService.kt, ComposeSmsActivity.kt
                       # Required stubs for default-SMS-app eligibility
```

## Roadmap (not yet built)

- [ ] MMS/image OCR (ML Kit Text Recognition) feeding the same
      `DetectionEngine` scorer.
- [ ] Gmail API integration for email quarantine (separate OAuth flow).
- [ ] `NotificationListenerService` to *flag* (not silently block — Play
      policy) risky content in notifications from other messaging apps,
      with explicit user consent screen.
- [ ] Per-contact allowlist so trusted senders skip AI calls.
- [ ] Background WorkManager sync for AI enrichment instead of
      app-foreground-only.

## Play Store note

Default-SMS-app apps face extra Play Store review scrutiny (Sensitive
Permissions declaration required). Have a clear privacy policy before
submitting — required for READ_SMS/RECEIVE_SMS approval.
