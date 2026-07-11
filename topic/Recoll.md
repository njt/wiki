# Recoll

Full-text desktop search that indexes everything: files, email attachments, compressed archives, all transparently. Built on Xapian since the early 2000s, Recoll is the Unix answer to "where did I put that document?" — and at version 1.44 in 2026, it's still actively maintained.

---

## What It Does

Recoll is not an LLM-powered semantic search. It's a traditional inverted-index search engine that runs locally, indexing the actual contents of your documents. The party trick: it can find text inside "an MS-Word document stored as an attachment to an e-mail message inside a Thunderbird folder archived in a Zip file." That depth of nesting is Recoll's thesis — don't make the user think about where something lives, just index everything and let the query engine find it.

Supported on Linux (primary), Windows, macOS, and Android (via F-Droid). Feature surface: Qt GUI, web UI for remote use, command-line tools, Gnome Shell search provider, Mutt integration via Links browser. The Debian packaging splits into `recollcmd`, `recollgui`, and a meta-package — you can run headless on a server and query via the web UI.

## Key Quotes

> "Recoll can index an MS-Word document stored as an attachment to an e-mail message inside a Thunderbird folder archived in a Zip file."

The homepage's elevator pitch, and it works. Each layer is decompressed and parsed transparently. The text extraction pipeline is Recoll's real value — Xapian does the indexing, but Recoll does the hard work of turning arbitrary file formats into indexable text.

> "Recoll could not exist without a rich free software environment."

Not false modesty. Recoll sits on top of dozens of format-specific extractors (antiword, poppler, unzip, etc.) that each solve one piece of the problem. The project's value is integration, not novel extraction.

## Key Themes

#tool #search #desktop #local-first #open-source #document-indexing

## Architecture

**Xapian** is the indexing and retrieval engine — a mature C++ search library that handles inverted indexes, ranked retrieval, and query parsing. Recoll adds:

1. **Text extraction layer** — a pipeline that decomposes archives, decodes email formats, and runs format-specific extractors. Each file type gets the right tool; Recoll's job is knowing which tool to call and stitching the results together.
2. **Indexing modes** — default single-pass index, plus a newer "multiple temporary indexes" mode for faster incremental updates.
3. **Configuration** — `~/.recoll/recoll.conf` for indexing behaviour, `~/.config/Recoll.org/recoll.ini` for GUI preferences. Sensible Unix conventions.
4. **Language processing** — n-gram based for languages like Tibetan where word segmentation is hard; configurable non-alphanumeric character handling for CJK; stemming and stop-word lists for European languages.

The text extraction layer is where Recoll does the work that [[Kreuzberg]] and [[Dolphin]] now do as standalone libraries. Recoll predates both by ~20 years.

## Critical Analysis

**The good:** Recoll is a solved problem, and that's the point. It doesn't need a GPU, doesn't phone home, doesn't hallucinate. You index your files, you search them, you find them. The engineering is boring in the best way — it works, it's fast, it's correct. Version 1.44 in 2026 means the maintainer (Jean-Francois Dockes) has been at this for over two decades. That's rare and valuable.

**The honest tradeoff:** Recoll does keyword search with ranking, not semantic understanding. You can't ask "find the email where Bob talked about budget concerns" — you search for "budget" and "Bob" and use your brain to connect them. [[QMD]] does hybrid BM25+vector+LLM re-ranking, which is more sophisticated retrieval. But QMD requires a few GB of model downloads and a Node.js runtime. Recoll requires neither. Different tools for different philosophies.

**The text extraction problem never goes away.** Recoll, [[Kreuzberg]], [[markitdown]], and [[docmason]] all face the same fundamental challenge: turning arbitrary file formats into searchable text. Recoll solved this with a pipeline of format-specific tools in 2005. Kreuzberg solved it with a Rust core and 97+ formats in 2025. The problem is the same; the packaging is different.

**The platform story is uneven.** Linux is first-class. Windows works. macOS and Android exist but feel maintained by a smaller community. The Flatpak and AppImage are third-party contributions — they work, but you're one volunteer away from them breaking. For a tool this mature on Linux, the cross-platform story feels like an afterthought rather than a strategy.

**The web UI as escape hatch** is the most forward-looking feature. It means you can run Recoll on a headless server and search from any device on your network. This is the pattern that [[QMD]]'s MCP server integration and [[Kreuzberg]]'s REST API also reach for — index locally, query from anywhere. Recoll had this before "MCP server" was a term.

## Relationship to Modern Tools

Recoll is the desktop-search generation that came before the LLM-native tools. It's Xapian and grep and file-format extractors, not embeddings and re-rankers. But the *problem* it solves — "I know I have a document about X but I can't find it" — is the same problem that [[QMD]], [[docmason]], and every RAG pipeline are attacking. Recoll's answer is simpler, less shiny, and completely adequate for the use case.

If you want to find your tax return from 2019, Recoll is the tool. If you want to ask "what was my tax strategy in 2019 and how did it change?", you need something that understands language, not just indexes it. Both are valid.

---
*Sources: [[raw/recoll]]*
*Last updated: 2026-07-11*
