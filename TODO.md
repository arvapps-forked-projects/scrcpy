# scrcpy enhancement session — tracking

Status legend: [x] done · [>] in progress · [ ] pending · [!] blocked

## Session outcome
- [x] Recon: this repo is the DESKTOP scrcpy client (C + SDL + Java server),
      NOT a user-facing Android app. It was the wrong starting point for the
      user's device-scanning/dropdown feature.
- [x] Clarified with user: the REAL target is a separate Android app repo
      (Kotlin/Java with a GUI) that the user will share in a NEW session.
- [x] Captured all session knowledge into a reusable handoff prompt:
      `docs/ANDROID-APP-SESSION-PROMPT.md`
- [x] User will paste that prompt into the Android app repo session to proceed.

## Where implementation happens (NOT in this repo)
- [ ] Real feature work (LAN scan + IP:port listing + dropdown selection)
      happens in the Android app repo, guided by the handoff prompt above.
      Per user: Android/Linux first, simple useful features first, later others.
- [ ] UX: Android GUI dropdown (not terminal picker).
