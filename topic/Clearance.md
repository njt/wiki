# Clearance

A native macOS Markdown viewer and editor from [[Prime Radiant (Company)]] (the company behind [[Serf]]). Swift-native, fully local at runtime, with first-class YAML frontmatter support. 178 stars, Apache-2.0. The repo is structured as a monorepo with a placeholder Tauri app for future cross-platform expansion.

---

## Key Quotes

> "A native macOS app for reading and editing Markdown files, with first-class support for YAML-frontmatter documents."

> "Fully local at runtime for normal editing and rendering. No CDN dependencies."

## Key Themes

#tool #markdown #editor #macos #swift #local-first

**Native and local-first.** Clearance is Swift on macOS, not Electron. No cloud, no CDN, no telemetry during normal use. This is the opposite of most modern editors. Network access is only triggered by user-initiated actions (opening web links, checking for updates via Sparkle).

**YAML frontmatter as a first-class citizen.** This matters for anyone working with agent-generated markdown, Obsidian vaults, static site generators, or any tool pipeline that treats frontmatter as structured metadata. Most Markdown viewers either ignore frontmatter or render it as ugly text. Clearance treats it as part of the document model.

**Monorepo with cross-platform ambitions.** The workspace layout (`apps/macos`, `apps/tauri`, `packages/assets`, `packages/demo-corpus`) signals intent to ship on Windows, Linux, and Android via Tauri. The Tauri directory is a placeholder today, but the shared asset and demo-corpus packages suggest the infrastructure is being laid now. Compare with [[tolaria]], which already ships cross-platform via Tauri 2.

**XcodeGen-based build.** The project uses `project.yml` + XcodeGen rather than committing `.xcodeproj` files. This is a strong signal of engineering discipline -- generated Xcode projects are reproducible and diff-friendly, which matters for agent-assisted development.

## Critical Analysis

Clearance is a focused, opinionated tool. It reads and edits Markdown. It doesn't try to be a knowledge management system ([[tolaria]]), a collaborative editor ([[Mist]]), or a document conversion pipeline ([[markitdown]]). That restraint is a feature.

The Prime Radiant connection is interesting. The same company ships [[Serf]] (a non-interactive coding agent). An agent that produces Markdown output + an editor purpose-built for reading Markdown with frontmatter is a coherent product thesis: agents write, Clearance reads. The shared demo corpus in `packages/demo-corpus` reinforces this -- it's test fixtures for how agent-generated Markdown should render.

At 178 stars and 18 releases through v1.3.3, this is a real shipping product with a release cadence, not a weekend project. The Apache-2.0 license is friendlier to commercial use than [[tolaria]]'s AGPL-3.0.

The weakness is platform lock-in. Today, this is macOS-only. The Tauri placeholder is a promise, not a product. If you need cross-platform Markdown editing now, [[MarkText]] (Electron, 56k stars) or [[tolaria]] (Tauri 2) are shipping alternatives. If you're on macOS and want something native and fast, Clearance fills a gap that Electron-based editors can't.

The real question is whether "native macOS Markdown viewer with frontmatter support" is a large enough niche to sustain a product, or whether this is primarily a dogfooding tool for Prime Radiant's agent ecosystem. Either way, the engineering is solid and the design philosophy is sound.

---
*Sources: [[summary/clearance]]*
*Last updated: 2026-05-14*
