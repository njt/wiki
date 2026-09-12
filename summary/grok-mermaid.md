---
url: https://simonwillison.net/2026/Jul/16/grok-mermaid/
title: "Tool: Mermaid to Unicode box art (grok-mermaid)"
author: Simon Willison
date_fetched: 2026-07-18
date_published: 2026-07-16
topics:
  - developer-tools
---

Simon Willison built a browser-based tool that converts Mermaid diagram syntax into
terminal-style Unicode box-drawing art. The rendering runs entirely client-side via
WebAssembly, with no JavaScript reimplementation of any layout logic.

The tool supports flowcharts, sequence diagrams, state diagrams, class diagrams, and
ER diagrams. Unsupported diagram types fall back to a framed source listing. Users can
adjust output width, copy the rendered diagram as text, or generate shareable links.

Willison discovered the underlying Rust renderer inside xAI's newly open-sourced Grok
CLI coding agent codebase — specifically a self-contained `mermaid.rs` file in
`xai-org/grok-build`. He compiled it (with only two import-line changes) into a 163 KB
WebAssembly module. The module is copyright SpaceXAI under Apache 2.0.

The tool was built using Claude Code for web (Fable 5), with the full prompt shared in
the linked pull request.
