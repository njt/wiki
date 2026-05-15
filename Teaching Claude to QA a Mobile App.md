# Teaching Claude to QA a Mobile App

Christopher Meiklejohn's war report on automating mobile QA for Zabriskie, a Capacitor-based community app. Android took 90 minutes via Chrome DevTools Protocol; iOS took 6+ hours of fighting Apple's locked-down WebView. The piece doubles as a cautionary tale about agent discipline when Claude escaped its git worktree and contaminated the main repo.

---

## Key Quotes

> "Authentication is one WebSocket message -- inject a JWT into localStorage and navigate to the feed."

Android's openness in one sentence. CDP gives you the browser's full API surface; everything from auth to navigation is just protocol messages.

> "Three rounds of push-and-pray before doing what should have been step one: run the tests, read the output, fix what's broken, verify, then push."

The agent discipline failure. Claude treated version control like a suggestion, not a boundary.

> "Apple, if you're reading this: please expose CDP or WebDriver for Simulator WebViews."

The plea at the end. iOS mobile testing will remain a coordinate-guessing game until Apple provides programmatic access to WKWebView internals.

## Key Themes

#mobile-testing #agent-discipline #CDP #ios #android #qa-automation #capacitor #webview

**Protocol vs. coordinates.** Android exposes CDP through WebView's DevTools socket -- forward the port, open a WebSocket, and you have full browser control. iOS offers nothing comparable: WKWebView has no CDP, Safari Web Inspector uses a proprietary binary protocol, and accessibility APIs give you coordinates that are wrong 40-60% of the time depending on window state.

**The 90 minutes vs. 6 hours asymmetry.** The Android solution is elegant: `adb forward` to a WebSocket, inject credentials, sweep 25 screens in 90 seconds. The iOS solution is a Rube Goldberg machine: modify the backend to accept usernames (because the email keyboard eats `@`), write directly to TCC.db to dismiss native dialogs, calibrate tap coordinates with `ios-simulator-mcp`, and accept that it's fragile.

**Agent worktree escape.** Claude was given a git worktree for isolation. It `cd`'d into the main repo instead. The result: dirty files from unrelated work committed together, duplicate declarations, broken E2E tests, four follow-up commits across three PRs. This is the [[Compound Engineering]] failure mode -- without mechanical enforcement, the agent takes shortcuts.

**Automated QA as a daily loop.** Both platforms now sweep 25 screens every morning at 8:47 AM, analyze screenshots for visual issues, and file bug reports to the production forum. The system works, but the iOS half is held together with duct tape.

## Critical Analysis

This is one of the best first-person accounts of what mobile testing with agents actually looks like -- not the demo-polished version, but the version where you spend hours fighting iOS keyboard interceptors and coordinate systems.

The Android/iOS split is a microcosm of a larger truth: **open protocols create agent-friendly platforms; closed platforms create agent-hostile ones.** Apple's refusal to expose CDP or WebDriver for WKWebView isn't just an inconvenience -- it means every iOS testing solution is a fragile stack of workarounds that will break when Apple changes Simulator internals. The coordinate-guessing approach (42% accuracy with AppleScript, 57% with idb) is damning.

The agent discipline story is equally valuable. Meiklejohn doesn't blame Claude and move on -- he traces the cascade: worktree escape leads to mixed commits leads to broken tests leads to three rounds of push-and-pray leads to four cleanup commits. This is exactly the failure mode that [[Feedback Loop is All You Need]] argues against: without deterministic enforcement (linters, pre-commit hooks, worktree lock), agent discipline is just a prompt, and prompts are suggestions.

The practical gap this exposes: there's no good mobile testing framework for hybrid apps (Capacitor, Ionic, React Native with WebView). Playwright can't reach the WebView. XCTest and Espresso can't reach the web content. CDP bridges the gap on Android but doesn't exist on iOS. Someone should build the cross-platform equivalent of what Meiklejohn hacked together here.

## Cross-Links

- [[Compound Engineering]] -- the agent discipline failure is what happens when the "compound" step (systematic guardrails) is missing
- [[Feedback Loop is All You Need]] -- "run tests before pushing" is a linter-shaped problem, not a willpower problem
- [[Harness Engineering]] -- the worktree escape is absent computational feedforward; the push-and-pray is absent feedback
- [[Agent of Empires]] -- git worktree isolation for parallel agents, the pattern Claude violated
- [[surf-cli]] -- browser automation for agents via CLI; same domain, desktop rather than mobile
- [[Browser Use]] -- AI browser automation with anti-detection; complements the desktop testing story
- [[MinMax Skills]] -- mobile development skills for agents (Android/iOS/Flutter); the testing gap this post reveals
- [[Computer Use is 45x More Expensive Than Structured APIs]] -- protocol access (CDP) vs. coordinate-based interaction (iOS screenshots) is the same structured-vs-vision gap
- [[Slowing the Fuck Down]] -- deliberate friction and verification as the antidote to push-and-pray
- [[Write Only Code]] -- the contaminated commits are write-only code: generated without being read or verified
- [[Cognitive Debt]] -- mixed commits from worktree escape create comprehension debt across three PRs

---
*Sources: [[raw/teaching-claude-to-qa-a-mobile-app]]*
*Last updated: 2026-05-14*
