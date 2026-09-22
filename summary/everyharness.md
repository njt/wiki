---
url: https://github.com/prime-radiant-inc/everyharness
title: "everyharness"
author: prime-radiant-inc (Jesse Vincent)
date_fetched: 2026-09-22
date_published: 2026-09-17
topics:
  - coding-agents-and-frameworks
  - claude-code
---

everyharness is a TypeScript CLI that turns one `everyharness.yaml` file into native plugin artifacts for twelve coding-agent harnesses through eleven adapters: Claude Code, Codex, Gemini CLI, Cursor, Copilot CLI, OpenCode, Pi, Kimi Code, Hermes, Devin CLI, Factory Droid, and Grok Build CLI (with Antigravity on the roadmap). Each harness has its own plugin format — `.claude-plugin/plugin.json`, `.codex-plugin/plugin.json`, `gemini-extension.json`, OpenCode JS plugin modules, Pi TypeScript extensions, Hermes YAML-plus-Python — and each format drifts on its own schedule. everyharness treats the harness as a compilation target: the config is source, the generated manifests, bootstrap wiring, install docs, and support matrix are committed build output, and `everyharness validate` catches hand-edits in CI via sha256 drift detection against a generation manifest.

The tool was extracted from the superpowers plugin, which carried nine hand-maintained manifests and four distinct bootstrap mechanisms; a dogfood test regenerates eight of superpowers' manifests from one config and compares them semantically (JSON key order explicitly not compared). Its deepest contribution is the **bootstrap** abstraction: a tagged config value (`none`, `generate`, or `{ skill: <name> }`) that compiles down to each harness's native session-start context-injection mechanism — Claude Code/Cursor/Muse shell hooks with a bash/cmd polyglot wrapper, OpenCode message-transform plugins, Pi lifecycle-flagged context handlers, Hermes `pre_llm_call` registration, Gemini `@`-imports, or nothing at all for harnesses without one.

Verification is a first-class concern rather than an afterthought: `everyharness test` runs two offline tiers inside a shared ~15GB container image — parse every generated manifest, then actually install the plugin into each harness CLI and assert the CLI enumerates the plugin's skills. The codebase is dense with empirically-verified harness behavior recorded as dated comments (Claude Code's hook double-fire dedup, Copilot reading only Claude's marketplace descriptor, Hermes' `register_skill` requiring a `Path`), making it as much a field survey of harness quirks as a build tool. v1.0.0, MIT, not yet on npm (clone-and-build).
