---
url: https://lowtechguys.com/musicdecoy/
title: "Music Decoy - Stop launching the Music app whenever you press ▶ Play"
author: lowtechguys (Alex Panaitiu)
date_fetched: 2026-06-09
date_published: unknown
topics:
  - developer-tools
---

# Music Decoy

A macOS utility that prevents the Music app from launching when you press the ▶ Play key, connect a Bluetooth headset, or end a call. It works by having the same bundle identifier (`com.apple.Music`) as the system Music app, tricking the `rcd` daemon into thinking Music is already running — while consuming zero CPU.

## Source: lowtechguys.com/musicdecoy/

### Core premise

Stop the Music app from launching. As long as Music Decoy is running, the system Music app won't launch when you accidentally press ▶ Play. The app does absolutely no work in the background — it works by simply existing as a running process.

### How it works

By having the bundle identifier `com.apple.Music`, the app makes the system think that the Music app is already running.

The technical mechanism:

1. The Play event is caught by the `rcd` daemon (`/System/Library/CoreServices/rcd.app/Contents/MacOS/rcd`)
2. The daemon checks if there is any app playing something and forwards the event to that app
3. If there is no such app, `rcd` checks if any running app has `com.apple.Music` as its bundle identifier
   - Without a running `com.apple.Music` app, `rcd` launches the system Music app
   - But if there is such an app, the event is forwarded to it instead

### When does Music auto-launch?

- When you press the ▶ Play key on your keyboard and there is no other app playing audio
- When a Bluetooth headset connects and sends a play command
- When ending a call, which causes the Bluetooth headset to switch from call mode to music mode

### Configuration (v1.1+)

Can be configured to launch another app (e.g., Spotify) when ▶ Play is pressed:

```sh
defaults write com.lowtechguys.MusicDecoy mediaAppPath /Applications/Spotify.app
```

Reset with: `defaults delete com.lowtechguys.MusicDecoy mediaAppPath`

### How to quit

No Dock icon, no menubar icon. Quit via Activity Monitor or `killall 'Music Decoy'`.

### Alternatives

- `launchctl unload -w /System/Library/LaunchAgents/com.apple.rcd.plist` — disables the Play button completely
- [noTunes](https://github.com/tombonez/noTunes) — listens for launched apps and kills Music. Uses a tiny bit of CPU

### Known issues

- VLC may crash because it uses ScriptingBridge to communicate with the Music app. Fix: configure VLC's setting for Music Decoy
- Can't launch Music while Decoy is running. Workaround: use the provided macOS Shortcut that quits Decoy, waits, then launches Music

### Installation

- Download from https://files.lowtechguys.com/MusicDecoy.zip
- `brew install --cask music-decoy`

### Source

GitHub: https://github.com/FuzzyIdeas/MusicDecoy
