# Chawan

A text-mode web browser and pager built from scratch in memory-safe Nim, with vi-inspired UI, opt-in JavaScript via QuickJS, and support for protocols most browsers abandoned decades ago.

---

> "Chawan is a text-mode web browser and pager for Unix-like systems. It has been developed from scratch in the memory-safe Nim programming language."

The choice of Nim is the quiet thesis here. Writing a web browser in C or C++ is a memory-safety gamble at industrial scale — the attack surface of an HTML parser alone is staggering. Nim gives you C-like performance with memory safety guarantees, making Chawan one of the few browsers built on a foundation where buffer overflows aren't table stakes.

> "Chawan supports HTTP(S), SFTP (via libssh2), FTP, Gopher, Gemini, Finger, Spartan."

This protocol list is a deliberate design statement. It treats the web not as a singular platform but as one protocol among many, all equally valid targets for a browser. Gopher and Gemini in particular represent an alternate-history internet — one where hypertext stayed simple and the browser never became an operating system. Chawan renders that alternate history live.

> "JavaScript can be used for DOM manipulation and networking. It is opt-in via a configuration option."

Opt-in JavaScript is a UX decision masquerading as a technical one. Every modern browser runs JS by default and makes you work to disable it. Chawan inverts that: JS is off unless you choose it. This isn't just about performance or battery — it's about agency. The browser treats scripting as a capability you grant, not a default you suffer.

## Key Themes

- **#tool** — A working web browser for the terminal, not a toy or a demo
- **#concept** — Protocol agnosticism as browser philosophy: the web is one option, not the only option
- **#pattern** — Memory safety by construction: Nim as a deliberate alternative to C/C++ in systems programming. See also [[Zeroclaw]] (Rust agent runtime), [[Zellij]] (Rust terminal multiplexer), [[Grok Build]] (Rust TUI coding agent) for the broader trend of memory-safe systems tools
- **#pattern** — Sandboxing per site with syscall filtering — a security boundary more browsers should adopt

## Critical Analysis

Chawan is a browser built on taste. It doesn't try to be a terminal Chrome — it picks its battles and wins them cleanly. The vi keybindings, the protocol breadth, the JS-as-opt-in posture, the Nim foundation: every choice coheres.

The trade is real, though. This is not a daily driver for the modern web. QuickJS is fast but not V8 — complex SPAs will struggle or fail. The CSS support covers flow, table, and flex layout but not grid. You'll hit walls. But Chawan isn't arguing you should abandon Firefox; it's arguing that "browser" shouldn't mean one thing. A pager for man pages and Markdown that can also follow an HTTP link is a different category of tool, and Chawan defines that category well.

The sandboxing model deserves more attention than it gets. Loading each site in a separate process with syscall filtering is the right architecture — Chromium does this at enormous complexity, and Chawan gets it in a fraction of the code. The per-site boundary is the correct granularity for web isolation, and Chawan proves you don't need a Google-scale engineering org to achieve it.

The project's handling of subprojects — spinning Chame, Chagashi, and Monoucha out as standalone Nim libraries — is good open-source citizenship. Each is independently useful, and the separation makes the architecture legible.

What's missing: no mention of accessibility (screen reader support in a text-mode browser should be trivial but isn't stated), and no clear indicator of how complete the CSS engine is for real-world sites. The gallery exists but isn't linked from the landing page in a way that builds confidence.

---
*Sources: [[raw/chawan]]*
*Last updated: 2026-07-25*
