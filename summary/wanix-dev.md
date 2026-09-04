---
url: https://wanix.dev/
title: Wanix — Wasm-Native Unix Sandboxing for the Web
author: unknown (wanix.dev project)
site: wanix.dev
date_fetched: 2026-09-04
date_published: unknown
---

# Wanix — Wasm-Native Unix Sandboxing for the Web

Wanix is a browser-native framework for running real Wasm and x86 programs, entirely sandboxed, with no server. Its one-line pitch: "Run and interact with real Wasm and x86 programs entirely sandboxed in the browser. No server. Inspired by Plan 9." You drop a `<script type="module">` tag pointing at the jsDelivr CDN (`wanix@0.4.0-rc2`) into any page and compose a Unix environment from a handful of custom HTML elements.

The system is built from a small set of web components. `<wanix-task>` runs an executable in a namespace (with built-in JS and Wasm drivers); `<wanix-term>` attaches an xterm.js terminal to a task or VM; `<wanix-vm>` boots a headless Linux VM powered by v86; `<wanix-namespace>` creates an explicit namespace container; and `<wanix-bind>` is the bind-mount primitive — it can fetch or inline a file (`type="file"`), unpack a `.tar`/`.tgz` archive into a directory tree and layer multiple archives into a recursive union (`type="archive"`), or import a remote namespace over 9P via WebSocket or an embedded iframe (`type="import"`). The extended component, `<wanix-workbench>`, embeds a full VS Code workbench backed by the namespace as editor, file explorer, or app shell.

The conceptual core is Plan 9 transplanted to the web: **per-process namespaces** and **everything-is-a-file**. Namespaces can be in-memory (`#ramfs`), backed by browser persistent storage (`#web/opfs`), or a VM's exported guest filesystem (`#vm/1/guest` via `export="ttyS0"`). Give a namespace an `id` and `allow-origins` and another page can import it with a bind — namespaces compose across origins the way Plan 9's 9P protocol let processes share file trees. The site's closing thesis, in its own words: "a research OS from the 90s turns out to be the right model for the local-first web."
