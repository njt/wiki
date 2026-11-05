---
url: https://github.com/wailsapp/wails
title: "Wails"
author: Wails team (leaanthony and contributors)
date_fetched: 2026-11-06
date_published: 2026 (v3.0.0 stable)
topics:
  - developer-tools
  - mcp-and-tool-protocols
---

Wails is a Go framework for building desktop applications for Windows, macOS and Linux using web technologies for the frontend — the Go counterpart to Electron/Tauri, shipping v3.0.0 as the stable desktop release, with experimental Android and iOS support explicitly outside the desktop compatibility contract.

The v3 architecture, read from the code: a single Go process owns the webview and exposes a Go backend to JavaScript over a message-processor/IPC layer. `pkg/application` (~92k lines of Go) contains per-platform drivers, including a purego variant (`application_darwin_purego.go`) that binds ObjC/Cocoa via ebitengine/purego instead of cgo. Bound Go services are reflected into typed bindings (`bindings.go`) with stable numeric method IDs so calls survive name obfuscation; the frontend runtime (`internal/runtime/desktop/@wailsio/runtime`) talks a compact integer-dispatched protocol (`RuntimeRequest`: object id + method id) handled by `messageprocessor*.go` across 13 subsystems (call, events, dialogs, clipboard, window, screens, streams…).

Notable engineering choices: an asset server that rides the webview's custom URL scheme rather than a listening port; "Streams" (`stream.go`) implementing a WebSocket-like Go→JS channel over held long-polls to avoid opening any local TCP port; an opt-in MCP server (`mcp_enabled.go`, build tag `mcp`) that lets AI agents drive the running app's JS context over HTTP with token auth; and a static-analyser binding generator (`internal/generator`) that parses Go source with golang.org/x/tools to emit TypeScript definitions rather than relying purely on runtime reflection.

The README itself is multilingual scaffolding around a short stable-release announcement: platforms, experimental mobile status, links to docs and Discord.
