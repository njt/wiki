---
source_url: https://github.com/prime-radiant-inc/clearance
fetched: 2026-05-14
type: github-repo
title: "Clearance — A Markdown viewer for macOS"
repo: prime-radiant-inc/clearance
stars: 178
language: Swift (95.3%), CSS (2.2%), Shell (2.1%), HTML (0.4%)
license: Apache-2.0
latest_release: "1.3.3 (2026-04-14)"
---

# Clearance

**Repository:** prime-radiant-inc/clearance
**Description:** A Markdown viewer for macOS
**Stars:** 178 | **Forks:** 26 | **License:** Apache-2.0
**Latest Release:** Clearance 1.3.3 (April 14, 2026)
**Total Releases:** 18

## Workspace README

This repository is the Clearance product workspace. It currently ships a native macOS app and now has the monorepo structure needed for alternate implementations to grow alongside it.

### Workspace Layout

- `apps/macos`: the current Swift/Xcode app, release notes, packaging scripts, and macOS-specific docs
- `apps/tauri`: placeholder home for the future shared Tauri app targeting Windows, Linux, and Android
- `packages/assets`: shared branding and static assets
- `packages/demo-corpus`: shared markdown fixtures that current and future implementations can reuse
- `docs`: workspace-level specs, plans, and engineering notes

### Current App

The shipping app lives in apps/macos/README.md. For macOS build, test, release, and CI details, see apps/macos/docs/DEVELOPMENT.md.

### Workspace Scripts

- `npm run macos:generate`: generate the macOS Xcode project from `apps/macos/project.yml`
- `npm run macos:build`: build the macOS app from `apps/macos`
- `npm run macos:test`: run the macOS test suite from `apps/macos`

### Contributing

- Put app-specific changes in the owning app directory, not at the workspace root.
- Put genuinely shared assets and fixtures in `packages/*`.
- Keep root docs and workflows workspace-oriented rather than macOS-only unless there is no shared concern.

## macOS App README

Clearance is a native macOS app for reading and editing Markdown files, with first-class support for YAML-frontmatter documents.

### Core Features

- Opens `.md` and `.txt` files
- Maintains a sidebar displaying recently accessed files organized by recency
- Toggles between View (rendered) and Edit (syntax-highlighted) modes
- Uses a right-side outline for document heading navigation
- Opens files in new windows
- Follows both local and web links
- Auto-saves during editing

### Keyboard Shortcuts

- `⌘O` for opening files
- `⌘1`/`⌘2` for mode switching
- `⌘F` for searching
- `⌘Z`/`⇧⌘Z` for undo/redo

### Privacy

Fully local at runtime for normal editing and rendering. No CDN dependencies. Network use is optional, limited to user-initiated actions like opening web links.

### Updates

Managed through Sparkle framework, accessible via the menu option to check for updates.

## About

Copyright 2026 Prime Radiant
https://primeradiant.com
