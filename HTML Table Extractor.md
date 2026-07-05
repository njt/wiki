# HTML Table Extractor

Simon Willison's browser-based paste-conversion tool that extracts HTML tables from rich text, previews them, and exports to five formats — plus a Wikipedia search integration that auto-fetches pages via the open CORS API.

---

## What It Is

A single-purpose browser tool at `tools.simonwillison.net/html-table-extractor`. Paste any rich text containing HTML tables (from a web page, a spreadsheet, a document) and it automatically detects each embedded table. You get a preview, then can export in any of five formats: HTML, Markdown, CSV, TSV, or JSON. A "Copy" button sits alongside each format tab.

Willison positions it as "yet another in my growing collection of paste-conversion tools" — a family that includes his recently-rebuilt [[Rich Text to Markdown]] converter. The tools share architecture and UI conventions.

## Wikipedia Integration

The significant update in the post is a Wikipedia CORS discovery: Wikipedia exposes an open CORS API at `en.wikipedia.org/w/api.php?action=parse` that returns rendered HTML for any page. This means a browser-side tool can search Wikipedia and pull in tables without a server-side proxy.

> "I had Codex add the ability to search Wikipedia for a page and then automatically import and display any tables from that page."

Willison wired this into the tool via a separate [cors-fetch](https://tools.simonwillison.net/cors-fetch) utility. The demo link pre-loads the Wikipedia API call for "List of cities and towns in the San Francisco Bay Area" — the resulting tables auto-populate the extractor.

## Key Themes

- **#pattern — Paste-conversion as a tool category**: Willison is building a family of single-purpose browser tools that all share the same core interaction: paste rich text → transform → export. The HTML Table Extractor, [[Rich Text to Markdown]], and presumably others form a coherent suite. This is the "do one thing well" Unix philosophy applied to browser-side utilities.
- **#pattern — CORS as an enabler for client-side tools**: The Wikipedia CORS API discovery is the sharpest technical insight here. Tools that run entirely in the browser (no server, no API key) are limited by what cross-origin APIs exist. Wikipedia's CORS support turns what would be a server-side proxy problem into a client-side fetch. Most developers don't know Wikipedia offers this.
- **#tool — Agent-built tooling**: Willison used Codex (OpenAI's coding agent) to add the Wikipedia search and import feature. The gist link shows the prompt and result. This is a working example of agent-assisted tool development for a single, well-scoped feature — not a whole app, just one capability bolted onto an existing tool.
- **#concept — Five-format export as a design pattern**: HTML, Markdown, CSV, TSV, JSON. The five formats cover the major use cases: web reuse, documentation, spreadsheet import, data analysis, and programmatic consumption. Most table tools pick one or two. Covering all five makes the tool a universal adapter.

## Critical Analysis

**The tool is unremarkable on its face but interesting as a pattern.** A table extractor isn't novel — browser devtools have had "copy as CSV" plugins for years. What's worth noting is Willison's *method*: build a family of browser-side paste-conversion utilities that share UI conventions and don't require servers, accounts, or API keys. Each tool does exactly one transformation well. The sum is more valuable than any individual tool.

**The Wikipedia CORS discovery is the real news here**, not the table extractor itself. Wikipedia's parse API with `origin=*` means any browser-side tool can retrieve structured content from Wikipedia without authentication or a server proxy. This is a significant piece of open-web infrastructure that most developers building browser tools don't know exists. Willison's contribution is less the code than the *connective tissue* — wiring up an obscure API to a practical use case.

**The agent-built feature is notable for what it isn't.** Willison didn't have Codex build the whole tool — he had it add one feature to an existing tool he already understood. This is the correct granularity for agent-assisted development in mid-2026: agents as feature-level subcontractors, not system-level architects. The gist is worth reading as a case study in how to scope agent tasks.

**The missing piece is persistence.** These tools are ephemeral — paste, export, done. There's no history, no saved transforms, no way to chain tools together. That's a deliberate choice (simplicity over features), but it limits the suite's power. A local-first pipeline where table extraction feeds into CSV-to-SQL conversion, for instance, would require the user to manually shuttle data between tools.

## Connections

- [[The solution might be cancelling my AI subscription (Willison)]] — Willison's own reflection on maintenance burden from AI-generated code. He's clearly resolved this tension by using agents for *features within existing tools*, not for greenfield projects.
- [[2025 in LLMs]] — Willison's annual LLM survey; this tool post is characteristic of his ongoing developer-tools practice alongside his LLM commentary.
- [[Moltbook]] — Willison's analysis of the AI-only social network; another example of his pattern of building small tools to understand a space.
- [[Rich Text to Markdown]] — The sibling tool in Willison's paste-conversion suite, recently rebuilt to add table support.
- [[Intent Is the Interface]] — The table extractor is a counterexample: unlike intent-driven interfaces, it's a conventional tool with explicit, discoverable controls. Sometimes the old way works fine.

---

*Sources: [[raw/html-table-extractor]]*
*Last updated: 2026-07-05*
