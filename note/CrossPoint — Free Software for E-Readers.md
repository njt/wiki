# CrossPoint — Open Source E-Reader Software

CrossPoint is free, open-source software for compact e-readers, positioned as "the software behind the screen" — a community-built replacement firmware-adjacent reading layer for devices like BOOX Picco, with an unusually deep feature set for typography, library management, and extensibility.

---

## What it is

The landing page is a feature tour of a mature reader. The typography pitch leads: "Read EPUBs with justified text, language-aware hyphenation, and working footnotes," plus drop caps, small caps, and illustrations — the details mass-market readers quietly drop. Users control typeface, size, margins, and line/word/letter spacing, and can convert fonts they own with a built-in font builder or load TrueType fonts directly on readers with external RAM.

## Getting books in and reading them

Ingest is deliberately multi-path: drop a book in a browser, send from Calibre, browse OPDS and OPDS 2.0 catalogs on the device, or use the reader's own Wi-Fi hotspot when away from home. The reading layer includes highlights, named bookmarks, clippings exportable as Markdown or images, Quick Resume, KOReader sync for position across devices, offline StarDict dictionaries, and a "Focus Reading" guided pace. Hardware integration goes as far as Bluetooth page turners with remappable short and long presses, and tilt-based page turns on supported hardware.

## Extensibility without reflashing

The most interesting architectural claim: "Add book catalogs and download sources with SD card plugins, without reflashing firmware." Web tools manage books from a browser and shrink EPUB files with an optimizer. Updates ship over Wi-Fi free on supported devices (USB for BOOX Picco), with a public roadmap and nightly builds.

## Community and sync outward

Development is funded through shop links and grown through "bug reports, translations, code, and conversations." Sync runs both ways: the companion app shows stats and current books, while highlights export to Readwise, progress updates to Hardcover, and position syncs with BookFusion and Audiobookshelf.

---

## Analysis

This is a landing page, not an essay, but the design choices speak. CrossPoint sits squarely in the local-first tradition: books on your SD card, catalogs via open protocols (OPDS), plugins that don't require reflashing, dictionaries that work offline. The feature list reads like a checklist of everything locked-down commercial readers omit — real hyphenation, working footnotes, font ownership, clippings as Markdown — which is exactly what you get when the maintainers are the users. The ecommerce coda ("Buying through our shop links helps fund development") is the classic sustainable-open-source compromise: free software, affiliate-funded.

Worth noting for this wiki: the page contains no AI or agent angle at all — a genuinely rare property here. It matters as a data point for what user-owned, agent-free tooling looks like when it's done well.

## Related pages

- [[How HTML Changes in ePub]] — CrossPoint's headline feature is faithful EPUB rendering (justified text, hyphenation, footnotes), which makes it a real-world consumer of exactly the EPUB/HTML semantics that page dissects.
- [[The Golden Age of Open Source Applications]] — CrossPoint is a live example of that thesis: free software built by its users, funding itself through shop links rather than a subscription.
- [[Introduction to Obsidian]] — shares the local-first, files-on-your-disk philosophy (SD-card library, Markdown clippings, no account required), applied to reading instead of note-taking.

---
*Sources: [[raw/crosspointreader-com]], [[summary/crosspointreader-com]]*
*Last updated: 2026-11-20*
