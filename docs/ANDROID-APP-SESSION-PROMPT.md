# Reusable Prompt — Scrcpy Network Device Scanner for the Android App Repo

> **Handoff document.** Capture of everything meaningful from the earlier session
> with the *desktop* scrcpy repo, rewritten so it can be pasted into a fresh
> session on the **real Android app repo**. Update/open tasks are marked `[ ]`.

---

## 1. Context & Goal

We are enhancing an Android app that mirrors/controls Android devices over the
network (an "scrcpy-like" Android app — Kotlin/Java with a GUI). The app is
distinct from the *desktop* scrcpy client (C + SDL) that was mistakenly examined
first; **this real codebase is an Android app with a launcher activity, Compose/XML
UI, etc.** Treat it as a regular Android app, not a native desktop program.

### Primary feature request
**Scan all devices connected on the same network** — both:
- devices on **Wi-Fi**, and
- devices **tethered using the phone/device's own hotspot/network**

…list their **IP addresses and ports (adb: 5555 folder)**, and present them in a
**selectable dropdown** so the user can pick one to connect/access.

### UX decision (user-confirmed)
- Picker should be an **Android GUI dropdown** (the app already has an interface),
  **not** a terminal/CLI picker.
- **Android/Linux first** — get it working fully on Android; other platforms later.
- **Simple, useful features first**, then extend later.

---

## 2. Technical notes (reuse what's already known)

### Device-selection basics in scrcpy-land
- `adb` only lists devices it already knows (USB-plugged or previously
  `adb connect`-ed). It does **not** actively scan the LAN.
- adb TCP-IP devices appear with serial `IP:PORT` (the `:` in the serial
  distinguishes TCP-IP from USB; see `sc_adb_device_get_type` upstream).
- Default adb listen port is **5555**.
- Connect flow for wireless: `adb tcpip 5555` (once, over USB) then
  `adb connect IP:5555`, then run scrcpy against that serial/IP.

### How the (desktop) prototype planned it — reuse the ideas
1. **Enumerate local IPv4 interfaces** (Wi-Fi + tether appear as separate
   interfaces) → each yields a subnet + netmask; skip loopback and
   "down"/inactive interfaces.
2. **Probe candidate hosts** on port **5555** with fast, **parallel** TCP
   connects; keep only reachable `IP:5555`; label which interface each came from.
3. **Combine with `adb devices`** output (deduplicate) so the picker shows both
   already-connected devices and newly discovered LAN candidates.
4. **On selection**, `adb connect IP:5555` then connect scrcpy to that serial/IP.
5. Backend probing must run off the UI thread (coroutines); return results to UI.

### Key constraints for the Android app
- Network probing + `adb connect` are blocking I/O → must be on a background
  dispatcher (coroutine / executor).
- Use the app's existing HTTP/socket stack; Android needs the **INTERNET** and
  (for debug build) **ACCESS_WIFI_STATE** permissions. If mirroring uses adb,
  also check **USB/network debugging** prerequisites on the target device.
- Keep the dropdown simple in v1: group by interface, show `IP:PORT` +
  friendly name (device model if resolvable). Good first-run/empty states matter.

---

## 3. Workflow / discipline to follow in the new session

- **Phase 1 Understand**: inspect the Android repo — build system (AGP/Kotlin/SDK),
  architecture (MVVM/MVI?), UI layer (Compose or XML), DI (Hilt/Koin?), navigation,
  permissions, manifest, existing networking/scanning code, and how devices are
  currently discovered/connected. Run the `android-fork-analysis` checklist if it's
  a fork.
- **Phase 2 Plan**: propose design (scan module, picker UI, connection flow)
  and get approval before coding. Min. risk, no unrelated refactors.
- **Phase 3 Implement**: minimal, behavior-preserving changes following existing
  conventions. Coroutines for background probing.
- **Phase 4 Verify**: `./gradlew :app:assembleDebug` (or app module), run unit +
  instrumented tests, run lint. Confirm behavior on a real device on the network.

**Iron law**: root-cause investigation before any fix. **Commit + push** when a
verified feature lands, update `TODO.md` in the repo root throughout.

---

## 4. Feature roadmap (deferred / later)

- [ ] Bookmark / remember previously-used devices
- [ ] Config-file or DataStore persistence of recent devices
- [ ] Port-range probing beyond 5555
- [ ] Auto-detect target device OS/API level before connecting
- [ ] Fancier picker: signal strength / latency per candidate, device model name
- [ ] Reflect newly scanned device back into an "available devices" list

---

## 5. Open questions for the next session

- [ ] Confirm the app's package name / applicationId and whether it's a fork
- [ ] Confirm target SDK + whether adb mirroring is via adb server or WebView/RTSP
- [ ] Target Android API level (affects wireless-debugging pairing flow: Android 11+)
- [ ] Confirm permission strategy (wireless debugging requires pairing QR/pairing
      code on Android 11+ in some setups)
