<div align="center">
  <img src="logo.png" alt="Triora Logo" width="200" height="200">

  # Triora
  ### DNS Privacy Filter

  **Blocks trackers at the DNS layer. Nothing else.**
  <br>
  *Built for people. We build for users — not advertisers.*
</div>

---

## What Triora is

Triora is an Android app that filters **DNS**, so tracker and telemetry domains never resolve. It runs
on your phone, under your control: no account, no remote server, and no analytics of its own.

It runs a local `VpnService` that intercepts DNS lookups. If a name is on the shipped blocklist it is
answered locally — `127.0.0.1` for `A`, an empty answer for everything else — so the connection never
leaves the phone. If it is not, the lookup goes to Quad9. Nothing is proxied, decrypted or stored
anywhere but the device.

**In one sentence:** a DNS filter for Android that stops tracker and telemetry domains — and reports
only what it actually saw.

## What Triora is not

This matters more than the feature list, so it is stated plainly:

- **Not a VPN service.** It uses Android's `VpnService` only to reach DNS. Your traffic is not
  tunnelled through a server, and there is no subscription, account or exit node.
- **Not a firewall.** It does not block apps from the internet, and there is no per-app on/off
  switch. Android already does that in system settings, and Triora points you there.
- **Not a malware or phishing scanner.** The app ships one tracker/ad list. Quad9, the upstream
  resolver, filters known-malware domains — that protection comes from Quad9, not from Triora.
- **Not an ad blocker.** It blocks many ad domains because tracker lists include them, but it will
  not remove every ad in every app (YouTube, for example). It is a DNS-level tracker filter, not a
  content blocker.
- **Not a content filter.** No categories, no website blocking, no schedule.
- **Not a "privacy score".** Every number on screen is a count of something that actually happened:
  lookups that were answered locally instead of upstream.

## What it does

- **Blocks tracker and telemetry domains** with OISD Basic (≈59k domains) plus ~45 curated mobile-SDK
  domains that OISD leaves out — Amplitude, Segment, Braze, Kochava, Singular and similar. Many ad
  domains are on the same lists, so some ads disappear too.
- **Never breaks push, time or connectivity.** A small allowlist (Firebase Cloud Messaging,
  Firebase Installations, network time, the connectivity check) is applied *before* the blocklist,
  so a list update cannot silently kill notifications.
- **Tells you which app did it.** Each DNS query is attributed to the app that sent it, at query
  time, using `getConnectionOwnerUid`.
- **Shows blocked activity over time.** Day / Week / Month / Year, with today vs your own average,
  a seven-day strip on the dashboard, and per-period comparisons. History is stored as daily
  aggregates, so it survives the 30-day raw-log retention.
- **Starts again after a restart**, if you turn that on (Settings → Start on restart). Default off.
- **One notification, only when it matters.** Settings → Notifications has a single switch:
  *Remind me when Triora is off*. Off by default. It fires only for drops you did not cause —
  Android or another VPN closing the tunnel, the tunnel failing to open, or a restart after which
  Triora could not reconnect. It is silent, and swiping it away silences that outage until Triora
  has reconnected and dropped again.
- **A status notification while connected — and a way to hide it.** Android requires one while the
  VPN runs. Triora posts one quiet entry, never re-posts it, and on Android 13+ it stays gone once
  you swipe it. If you want the shade completely clean, the Notifications screen has a one-tap
  **Hide status notification** link to the channel switch in Android settings; protection keeps
  running with it hidden.
- **Two themes:** Deep Ocean (aqua) and Forest Night (green).

## How to use it

1. Install the APK (or build it from source — see below).
2. Open Triora and tap **Connect**. Accept Android's VPN consent dialog once.
3. Optional: **Settings → Notifications → Remind me when Triora is off**, if you want to know when
   protection drops unexpectedly.
4. Optional: **Settings → Start on restart**, if you want Triora to reconnect after a reboot or an
   app update. Android must still hold the VPN consent, and no app can ask for it from a receiver.

## Screens

| Screen | What it shows |
| --- | --- |
| Dashboard | Protection state, today's blocked count, 7-day trend, recent activity |
| Activity | One row per app → the companies it contacted → the exact domains blocked |
| Apps | Installed apps with per-app blocked counts (30-day raw log) |
| Blocked (from the trend card) | Period picker, chart, today vs average, 7-day average, comparisons |
| Summary (View all) | Daily, weekly, monthly and annual cards, plus lifetime total |
| Settings | Protection list, start on restart, notifications, backup/restore, theme, data deletion |
| Notifications | The one reminder switch, and the link that hides the status notification |

## Build

```bash
./gradlew :app:testDebugUnitTest :app:assembleDebug
```

Kotlin · Jetpack Compose (Material 3) · MVVM · Hilt · Room · minSdk 29 / targetSdk 34.

A debug APK lands in `app/build/outputs/apk/debug/`.

## Where the details live

- `PLAN_V5.6_ENGINE.md` — what was measured, what was changed, and the evidence for each change.
  This is the honest record: it also lists what is *not* verified.
- `Engine.md` — how a DNS query travels through the app.
- `walkthrough.md` — the same journey, step by step, for a new contributor.
- `Vision.md` — product principles, including what is deliberately out of scope.
- `GRAPHIFY.md` — project knowledge base and code-graph notes.

## Status

V5.6 BETA. The engine is verified on a Vivo V2514 (Android 16) and a Realme X2 Pro (Android 10, our
`minSdk`): real blocks recorded, tracker answers no longer leak to the network on either version,
statistics correct across all four periods, and the tunnel survives app updates when "Start on
restart" is on.

Android 11–15 were exercised on emulators (API 30–35): blocked and allowed answers and the resolver
route were correct on every level, but that is emulator evidence, not hardware evidence.

Android 10 caught a bug worth knowing about: the blocked answer used to be `0.0.0.0`, which Android 10
treats as "no address" and silently re-asks the network's own resolver. It is now `127.0.0.1`, and the
network's resolvers are routed into the tunnel.

The connection reminder is implemented and verified on API 30 and API 34 emulators: silent, one per
outage, removed on reconnect, and a swipe silences that outage for good. A final hardware pass on
Android 16 is still pending.

Not finished yet: a real-device pass on Android 11–15, and the reminder build on physical hardware.
Battery is measured on both ends: under 1 mAh across 2.5 idle hours, and 0.42 % of the phone's drain
during 20 minutes of active use.
