---
title: "Bookshelf"
url: https://github.com/murerkinn/bookshelf
author: Murat Erkin Cicek
date_fetched: 2026-08-25
date_published: 2026-08-24
topics:
  - developer-tools
---

# Bookshelf: A Self-Hosted Ebook Library Over Object Storage

Bookshelf is a self-hosted library for the ebooks you already own. One server-rendered page lists your EPUB and PDF books, filters them with a search box, serves downloads, and reads them in the browser — each format with its own reader — either as a Cloudflare Worker over R2, or as a plain Node server over a directory on disk. It is ~30,600 lines of TypeScript across 95 files, in a Turborepo monorepo.

## How it works

A sync CLI reads the EPUB/PDF files you drop in a `books/` folder, extracts metadata from inside each book, and builds a "library" tree — one folder per book holding every format of it, a cover, and a `metadata.json`, plus a single `catalog.json` at the root. That tree is then published to a destination: R2 via `npm run deploy`, or a local directory. The app serves that published library.

## The one idea worth knowing

The whole app is written against a narrow `Storage` interface — ranged reads, no enumeration — and never against Cloudflare. Porting it to a new backend means writing a provider package (two faces: a `Storage` for the app, a `StorageAdmin` for the CLI) and one composition root. The same build runs on a Worker or on a VPS, and a read-only destination is a first-class mode, not an error case.

## Three things that stand out

- **Ranged reads, everywhere.** A hand-written ZIP reader (and a minimal PDF parser) are built over a `ByteSource` abstraction so the app pulls one chapter out of a 40 MB archive without transferring the archive, while the CLI reads the same book whole from memory.
- **The library is regenerable.** Books are the source of truth; the published tree, covers, and catalog are all derived and rebuilt from scratch. The only non-derived data — profiles and reading positions — lives under a reserved `.bookshelf/` prefix that `--force` never touches.
- **Degradation over failure.** The app distinguishes "absent" from "unreachable" as separate states, so an empty shelf, a missing profile, and an outage all render sensibly — and a public instance is made read-only by *removing the write path*, not by hiding buttons.
