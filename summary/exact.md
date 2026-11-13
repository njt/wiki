---
url: https://github.com/expo/exact
title: "Exact — a cross-platform app runtime designed for agents"
author: Expo (Charlie Cheever and team)
date_fetched: 2026-11-13
date_published: 2026-10-16 (README revision; pre-1.0, changing daily)
topics:
  - coding-agents-and-frameworks
  - agent-coding-workflow
---

Exact is a cross-platform application runtime from the Expo team (repo `expo/exact`, internally "exact2", a rebuild of an earlier design). You write the UI in **Contract**, a small declarative language (no JavaScript in the view, no I/O, no escape hatches), and your data logic in TypeScript (on Hermes) or Rust behind a single typed "data seam." The Contract compiler bakes the interface into a compact *plan* of relational tables plus bytecode; on the web a ~20 KB ES module runtime drives the real DOM, and on macOS, iOS, and Linux a Rust runner executes the same plan with a Taffy-based CSS layout kernel and native presenters (AppKit, UIKit, or a GPU/CPU painter). No webview, no JS bridge, no app JavaScript before first pixel — cold start budget is 100 ms p50.

Its defining claim is being **agent-native**: `scripts/agent.mjs` exposes exactly ten operations (`tree · screenshot · tap · type · state · layout · logs · clock · prefer · perf`) identically on every host; the agent owns a *seekable clock* (say `clock +60000` instead of sleeping, so no timing flakes); tests are scripts of those operations (`app.test.contract`) that run unchanged on any host; and every tool emits machine-readable refusals with stable diagnostic ids. The README documents a fresh Claude Code agent building a todo app and passing tests on web, macOS, and iOS in about 57 minutes.

"Write the web, ship the platform": authors write CSS vocabulary and names; whatever is unsaid (controls, colour, type, motion, navigation) comes out as the platform's own. The browser is the parity oracle for native layout and rendering. Speed is enforced by rules (`rules/RULES.md`): six blocking checks in 65 s, 1,500-line file cap, boot path executes nothing, budgets regressions are P0s. Pre-1.0 by design — delete, don't deprecate.
