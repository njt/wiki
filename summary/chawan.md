---
url: https://chawan.net/
title: "Chawan — TUI Web Browser"
author: bptato
date_fetched: 2026-07-25
topics:
  - developer-tools
---

Chawan is a text-mode web browser and pager for Unix-like systems, written from
scratch in Nim. Its UI is inspired by w3m and vi.

It supports JavaScript (QuickJS, opt-in), a substantial subset of CSS (flow,
table, flex layout, colors), and inline terminal images via Sixels and the Kitty
graphics protocol. It handles HTTP(S), SFTP, FTP, Gopher, Gemini, Finger, and
Spartan protocols, with built-in viewers for HTML, plain text, Markdown, man
pages, and directory listings. Websites are sandboxed in separate processes with
syscall filtering on FreeBSD, OpenBSD, and Linux. Keybindings are user-customizable
via JavaScript.

Three subprojects support it: Chame (HTML5 parser), Chagashi (character
encoding), and Monoucha (QuickJS bindings for Nim). The current stable release is
v0.4.3, packaged for most major Linux distributions, BSDs, and Homebrew.

The project is public domain with permissively licensed components.
