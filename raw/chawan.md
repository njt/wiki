---
url: https://chawan.net/
title: Chawan — TUI Web Browser
author: bptato
date_fetched: 2026-07-25
date_published: unknown
---

# Chawan: TUI Web Browser

Chawan is a text-mode web browser and pager for Unix-like systems, developed from scratch in the memory-safe Nim programming language. Its UI draws inspiration from w3m and vi.

## Features

- **JavaScript Support**: Uses QuickJS with DOM manipulation and network APIs. Opt-in via configuration.
- **CSS Capabilities**: Flow layout (block, inline, float), table layout, flex layout, colors, and formatting.
- **Inline Terminal Images**: Supports Sixels and the Kitty graphics protocol. Input formats include PNG, JPEG, BMP, GIF, WebP, and SVG.
- **Multiple Protocols**: HTTP(S), SFTP (via libssh2), FTP, Gopher, Gemini, Finger, and Spartan. Users can extend these.
- **Viewer Formats**: Built-in viewers for HTML, plain text, Markdown, man pages, and directory listings. Users can add HTML converters to replace built-in viewers.
- **Sandboxing**: Websites load in separate processes with syscall filtering on FreeBSD, OpenBSD, and Linux.
- **Customizable Keybindings**: User-programmable via JavaScript.

## Subprojects

- **Chame**: HTML5 parser in pure Nim
- **Chagashi**: Character coding library in pure Nim
- **Monoucha**: QuickJS binding generator and runtime glue for Nim

## Current Release

Version 0.4.3 is the latest stable. Packages available via Alpine Linux, Arch Linux, Debian (testing), FreeBSD, Gentoo, Homebrew, NixOS, Slackware (SBo), and Void Linux. Unstable master-branch packages via AUR, AppImage, Gentoo's guru overlay, and Homebrew (`--HEAD`).

## License

Public domain, with permissively licensed components.

## Gallery

Site includes a gallery page showing websites rendered in Chawan.

Source: https://chawan.net/
