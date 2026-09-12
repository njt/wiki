# grok-mermaid — Terminal Mermaid Renderer via WebAssembly

Simon Willison extracted a self-contained Rust Mermaid renderer from xAI's newly open-sourced Grok CLI codebase, compiled it unmodified to a 163 KB WebAssembly module, and wrapped it in a browser tool that converts Mermaid diagrams into terminal-style Unicode box-drawing art. Five diagram types are supported (flowcharts, sequence, state, class, ER); others fall back to a framed source listing. Output width is adjustable, and diagrams are shareable via copyable links.

---

## Key Quotes

> I learned about the underlying Rust renderer when I was exploring the codebase for the newly open-sourced Grok CLI coding agent.

The accidental discovery pattern — Willison wasn't looking for a Mermaid renderer; he was reading the Grok source to understand it, found a reusable component, and shipped it. This is the same impulse that produced [[Open Source AI Gap Map (Willison)]] and his Datasette ecosystem: explore, extract, repurpose. The Grok CLI being open-sourced (Apache 2.0) is what made this possible — open source as the substrate for recombinant innovation.

> I figured it would be fun to try that out in a browser via WebAssembly, so I did.

The Willison move in one sentence. No roadmap, no product plan — just curiosity followed by a Claude Code prompt. The PR he links shows the prompt he gave Fable 5 to build it. This is tool-building as a form of understanding: the byproduct of exploration, not the goal.

> There is no JavaScript reimplementation of any layout logic.

The architectural principle: compile the real thing, don't rewrite it. The 163 KB WASM module contains the actual Rust layout engine from the Grok CLI, unchanged except for two import lines. This is the anti-port: instead of reimplementing Mermaid layout in JavaScript (thousands of lines of layout logic), ship the original code to the browser. WASM makes this trivial now — a compile target, not a rewrite.

---

## Key Themes

- **#tool — Single-purpose browser utility**: Follows Willison's established pattern of small, zero-backend browser tools (see [[HTML Table Extractor]], [[CORS Fetch Tester]], [[SQL to ER Diagram]]). No signup, no server, no tracking. Tools as public goods.
- **#pattern — WASM as deployment target, not rewrite target**: Don't port layout engines to JavaScript. Compile the original Rust to WebAssembly and ship it. The 163 KB module size makes this viable for any utility. See [[Component Model 1.0]] for the evolving WASM ecosystem.
- **#pattern — Tool-building as understanding**: Willison didn't set out to build this. He was exploring the Grok CLI source to understand it, found a reusable component, and shipped the extraction as a tool. The tool is a side effect of comprehension. See [[Understand to Participate]] for Geoffrey Litt's argument that understanding is the prerequisite for remaining an active collaborator with coding agents.
- **#concept — Open source as recombinant substrate**: xAI open-sourced their coding agent (Apache 2.0), and within days someone had extracted and repurposed a component for an entirely different use case. This is how open source actually compounds — not through forks of the whole thing, but through surgical extraction of useful pieces.

---

## Critical Analysis

**The meta-story matters more than the tool.** The tool itself is charming — Unicode box-drawing art from Mermaid syntax, rendered in-browser — but it's a niche utility. What's important is what it demonstrates: open-sourcing a coding agent means open-sourcing its components, and those components have lives independent of the agent they were built for. xAI open-sourced Grok CLI. Willison extracted the Mermaid renderer. Next week someone extracts the markdown parser, or the diff formatter, or the sandbox harness. The agent becomes a library of parts, not a monolith.

**Willison is building a corpus, not a product.** His tools site (tools.simonwillison.net) now hosts [[HTML Table Extractor]], [[CORS Fetch Tester]], and grok-mermaid — each a single-purpose, zero-backend browser tool built with Claude Code. Together they form a demonstration that "build a tool" is now a sentence you say to an agent, not a project you plan. The corpus is the argument: small tools are back, build-something-in-an-afternoon is back, and the only gate is whether you bother to ask.

**The WASM story is undersold.** "Compiled unmodified apart from two import lines" buries the lede. A Rust library written for a terminal CLI now runs in the browser, handling all the layout logic, at 163 KB. This is the promise of WebAssembly finally arriving at the utility-tool level — not just Figma and Photoshop, but the kind of tool a solo developer builds in an afternoon. The WASM compile target turns "can I use this Rust library?" into a question with one answer: yes.

**The Grok CLI connection is a footnote that should be a headline.** xAI open-sourced their coding agent. That's a significant event — a major AI lab releasing their internal agent infrastructure as open source. Willison's tool is the first visible extraction, but it won't be the last. The Grok CLI codebase represents years of xAI's internal engineering; every file is a candidate for this treatment. The Mermaid renderer is the canary.

---

*Sources: [[raw/grok-mermaid]]*
*Last updated: 2026-07-18*
