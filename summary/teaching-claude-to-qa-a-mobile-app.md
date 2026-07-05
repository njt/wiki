---
url: https://christophermeiklejohn.com/ai/zabriskie/development/android/ios/2026/03/22/teaching-claude-to-qa-a-mobile-app.html
title: "Teaching Claude to QA a Mobile App"
author: Christopher Meiklejohn
date: 2026-03-22
fetched: 2026-05-14
---

# Teaching Claude to QA a Mobile App

**Author:** Christopher Meiklejohn
**Date:** March 22, 2026

## Overview

The post documents how the author taught Claude to automate quality assurance testing for Zabriskie, a cross-platform community app built with Capacitor (React web app wrapped in native shells for iOS and Android).

## The Challenge

Zabriskie uses a single codebase across three platforms via Capacitor, which positions it in "a testing no-man's-land." Web testing tools like Playwright cannot reach inside the native shell, while native frameworks like XCTest and Espresso cannot interact with WebView content. The author needed to test 25 app screens on mobile platforms without manual intervention.

## Android Solution (90 minutes)

**Connectivity Fix:**
```
adb reverse tcp:3000 tcp:3000
adb reverse tcp:8080 tcp:8080
```

**Key Discovery - Chrome DevTools Protocol Access:**
The WebView exposes a DevTools socket that can be forwarded to a local port:

```
WV_SOCKET=$(adb shell "cat /proc/net/unix" | \
  grep webview_devtools_remote | \
  grep -oE 'webview_devtools_remote_[0-9]+' | head -1)

adb forward tcp:9223 localabstract:$WV_SOCKET

curl http://localhost:9223/json
```

**Advantages:** Using CDP meant "authentication is one WebSocket message — inject a JWT into localStorage and navigate to the feed."

The Python script sweeps all 25 screens in about 90 seconds, analyzes screenshots for visual issues, and files formatted bug reports to the production forum when problems are found.

## iOS Solution (6+ hours)

iOS presented multiple obstacles:

### Problem 1: Email Input Typing
The `type="email"` input field rejected the `@` symbol due to keyboard shortcut interception. Solution: modify backend to accept usernames, change input type to "text," and create a test user with a simple password ("qatest").

### Problem 2: Native Dialog Dismissal
The native iOS notification permission dialog (rendered by UIKit, not WebView) could not be dismissed through standard means. Solution: write directly to the Simulator's TCC.db permissions database with proper timing:
- Uninstall app
- Write TCC permission
- Restart SpringBoard
- Reinstall app
- Launch
- Then login

### Problem 3: Navigation Coordinate Accuracy
Multiple approaches showed varying success rates:
- AppleScript `click at`: 42% accuracy (affected by window position, scaling mode, toolbar state)
- Facebook's `idb`: 57% accuracy
- **Winning approach:** Use the `ios-simulator-mcp` tool's `ui_describe_point` function to measure coordinates precisely, then execute taps with `idb ui tap`

```
ui_describe_point(365, 163)
→ AXLabel: "Currents", type: Link, frame: (342, 159, 40x40)
```

### Fundamental Limitation

The contrast is stark: Android provides "a WebSocket and says here's the browser, do whatever you want." iOS locks developers out—WKWebView doesn't expose Chrome DevTools Protocol, Safari Web Inspector uses proprietary binary protocol, and other debugging options have severe constraints.

## The Agent Discipline Problem

Between Android and iOS work, Claude made a critical error: operating in a git worktree (designed for isolated changes), it instead `cd`'d into the main repository and committed all dirty files together—mixing QA work with unrelated codebase changes. This led to:

- Duplicate variable declarations throughout test suite
- Breaking changes to E2E tests (form placeholder rename)
- Catalog tests failing against CI database
- Four follow-up commits across three PRs to resolve

The author notes: "Three rounds of push-and-pray before doing what should have been step one: run the tests, read the output, fix what's broken, verify, then push."

## Key Lessons

1. **Use protocol-level access over coordinate-based interaction** when available
2. **Query systems for information** rather than assuming coordinates or states
3. **Respect isolation boundaries** in version control
4. **Run tests before pushing**—enforcement through discipline, not hope

## Current State

Both Android and iOS now run automated QA sweeps every morning at 8:47 AM, analyzing 25 screens each and filing bug reports automatically. The iOS implementation is fragile compared to Android due to platform constraints, but functional.

The post concludes with a direct appeal: "Apple, if you're reading this: please expose CDP or WebDriver for Simulator WebViews."
