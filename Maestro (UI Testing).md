# Maestro (UI Testing)

End-to-end UI testing framework for mobile and web apps. Open-source CLI plus a paid cloud for parallel execution. Not to be confused with [[maestro]], the multi-agent orchestration tool.

---

## What It Is

Maestro makes mobile UI testing approachable by replacing fragile XPath selectors and flaky waits with a simple YAML-based test language. You describe flows ("tap on Login", "enter text", "assert visible"), and Maestro handles the platform plumbing. Tests can target iOS, Android, and web (beta), across React Native, Flutter, SwiftUI, Jetpack Compose, NativeScript, .NET MAUI, Capacitor, and Cordova.

The free tier includes a CLI and Maestro Studio, a desktop app with a visual element inspector that lets you build tests by clicking on UI elements. There's also a recording mode that converts interactions into test commands. MaestroGPT provides AI-assisted test generation.

## Cloud Offering

The paid cloud product runs tests in parallel on managed infrastructure with CI/CD integration (PR checks, nightly builds, pre-release gates). Enterprise logos include Microsoft, Meta, DoorDash, Uber, Amazon, Disney, and Stripe.

## Why It Matters

Mobile UI testing has been a graveyard of abandoned frameworks (Appium, Detox, XCUITest wrappers). Maestro's bet is that a purpose-built DSL with built-in wait logic and cross-platform support can make mobile e2e tests actually maintainable. The enterprise adoption suggests they've found real traction.

## Critical Analysis

This is a solid, unglamorous infrastructure play. The YAML DSL approach trades flexibility for reliability -- you can't express arbitrary test logic, but what you can express tends not to break on OS updates. The AI assistant (MaestroGPT) feels bolted on rather than foundational; the real value is the deterministic test runner, not the LLM wrapper.

The interesting question is whether Maestro's cloud becomes the default CI/CD testing layer for mobile, the way Playwright has become for web. The enterprise customer list suggests momentum, but mobile testing is a market where big names adopt and then quietly let licenses lapse. The open-source CLI is the moat -- if teams standardize on the YAML format, the cloud upsell follows naturally.

Not relevant to the agentic development space this wiki mostly covers, but useful context if agents start generating mobile apps (as [[Building low-level software with only coding agents]] and [[vibes-cli]] suggest they will).

---
*Sources: [[raw/maestro-ui-testing]]*
*Last updated: 2026-05-14*
