# Music Decoy

A dead-simple macOS utility that stops Apple Music from auto-launching by impersonating it at the bundle ID level. Zero CPU, zero background work — it wins by just existing.

---

## What it solves

Press ▶ Play with no audio playing? Music.app launches. Connect Bluetooth headphones? Music.app launches. End a phone call? Music.app launches. This is the `rcd` daemon's doing, and it's been annoying macOS users for years.

Music Decoy fixes it with a single elegant trick: it registers `com.apple.Music` as its bundle identifier. When `rcd` looks for a running Music app to forward the play event to, it finds Decoy instead — and since Decoy is already "running," `rcd` never tries to launch the real Music app.

The app literally does nothing. It's a process that sits there, burning 0% CPU, and that's the whole feature.

## The rcd daemon explained

> "The Play event is caught by the rcd daemon… The daemon checks if there is any app playing something and forwards the event to that app. If there is no such app, rcd checks if any running app has com.apple.Music as its bundle identifier."

This is a nice bit of reverse-engineering. The Remote Control Daemon (`rcd`) handles media keys system-wide. If nothing is playing, it launches Music as a fallback. Decoy exploits the fallback check — it doesn't intercept the key event, it just passes the "already running" test that prevents the fallback launch.

**Key insight:** Decoy doesn't fight `rcd`. It satisfies `rcd`'s precondition so the fallback never triggers. This is the difference between blocking something and making it unnecessary.

## Configuration: redirect to Spotify

Since v1.1, you can redirect play events to another app:

```
defaults write com.lowtechguys.MusicDecoy mediaAppPath /Applications/Spotify.app
```

This turns Decoy from a pure blocker into a router — ▶ Play now launches Spotify instead of doing nothing. Useful if you've switched streaming services but your keyboard habits haven't.

## Alternatives compared

| Approach | Mechanism | Cost |
|----------|-----------|------|
| **Music Decoy** | Bundle ID impersonation | 0% CPU |
| **noTunes** | Kill Music.app on launch | Tiny CPU (app monitoring) |
| **disable rcd** | `launchctl unload rcd.plist` | Kills ALL media key functionality |

The `launchctl` approach is the nuclear option — it disables the Play button entirely, which means you can't use it with Spotify or anything else. noTunes is reactive (spot → kill), while Decoy is preemptive (make the spot look occupied so nothing lands there).

## Why this is a good tool

It's a perfect example of **minimum viable intervention.** The author reverse-engineered the actual mechanism (rcd's bundle ID check), found the simplest possible intervention (register the same ID), and shipped it. No event monitoring, no daemon management, no configuration unless you want it.

This is the kind of software that respects the user's system instead of fighting it. The "do nothing" implementation is a feature, not a shortcut.

The trade-off — you can't launch Music while Decoy is running — is honest and documented. There's even a macOS Shortcut workaround.

---

## Key themes

- #tool — macOS utility
- #pattern — impersonation as intervention: satisfy the precondition instead of blocking the action
- #concept — minimum viable intervention: find the exact mechanism and touch nothing else

---

*Sources: [[summary/musicdecoy]], https://github.com/FuzzyIdeas/MusicDecoy*
*Last updated: 2026-06-09*
