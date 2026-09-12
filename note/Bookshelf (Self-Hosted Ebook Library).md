# Bookshelf (Self-Hosted Ebook Library)

A self-hosted library for the ebooks you already own: a single Next.js app that catalogs EPUB and PDF files, serves a searchable shelf with covers, streams downloads, and reads books in the browser (epub.js and pdf.js), running either as a Cloudflare Worker over R2 or as a plain Node server over a directory on disk. What makes it worth reading as code isn't the ebook domain — it's the discipline of a hexagonal architecture done at small scale: the entire app is written against a narrow `Storage` interface and never against Cloudflare, so the same build serves a Worker and a VPS, and a read-only library is a first-class mode rather than an error.

---

## Architecture

A Turborepo monorepo (~30,600 lines of TypeScript across 95 files):

```
apps/bookshelf       the app: a Next.js Worker over object storage
packages/core        the library format, the ZIP reader, the provider contract
packages/provider-r2 Cloudflare R2, both halves of it (worker + node)
packages/provider-fs the library as a directory on disk
packages/sync        the CLI that builds a library and publishes it
packages/fixtures    generated books, for tests and the demo shelf
```

The core of the design is **ports and adapters**. `packages/core/src/provider.ts` defines the contract as two faces, because the two callers run in different places and want different things: `Storage` (what the app reads at request time — `head`, `read`, `readRange`, optional `write`/`remove`, and deliberately *no* `list`) and `StorageAdmin` (what the CLI manages with — `put` from a disk path, `remove`, optional `create`/`list`/`removeAll`). A provider package exports a manifest plus a factory for each face, so `@bookshelf/provider-r2/worker` and `@bookshelf/provider-fs/node` are imported by different runtimes and get the half that suits them.

`apps/bookshelf/src/services/container.ts` is the composition root. `createServices(storage, cache)` wires the five services (`CatalogService`, `BookContentService`, `ProfileService`, `ProgressService`, and a cache) and knows nothing about any provider; `getServices()` is the *only* place that names one, choosing between the filesystem and Cloudflare at startup. There is no `list` on `Storage` by design — the catalog is what enumerates the library, so a request cannot discover books by walking the bucket; enumeration lives on `StorageAdmin`, which never runs in the app.

The library format (`packages/core/src/catalog.ts`, `state.ts`) is the shared contract between the CLI and the app: `catalog.json` at the root, one folder per book holding its formats, cover, and `metadata.json`. App-written state — profiles and reading positions — lives under a reserved `.bookshelf/` prefix, dot-prefixed because book folders are slugified titles and can never begin with one, so the namespace is reserved *by construction* and a `--force` publish cannot delete everyone's bookmarks.

## Key techniques

- **A hand-rolled ZIP reader over ranged reads.** `packages/core/src/bytes.ts` defines `ByteSource { size(), read(offset, length) }`; the ZIP reader in `zip.ts` is written against it, so the same parser serves the CLI (a whole book in memory, `bytesSource`) and the app (one chapter pulled out of a 42 MB archive over ranged reads, `rangedSource`). Reading a chapter never transfers the archive. Decompression goes through `DecompressionStream("deflate-raw")` rather than `node:zlib`, which is what lets the one implementation run in both workerd and Node. Zip64 archives are *rejected* rather than mis-parsed.

- **Speculative single-read entry extraction.** `readZipEntry` makes one read that covers the local header plus a 512-byte allowance for its extra field plus the data, then parses the header back out — avoiding two sequential round trips per chapter. It trusts the local header over the central directory about data length, because a caller may hold a memoized directory that no longer describes a republished object, and a stale length that silently truncates is worse than failing.

- **A minimal PDF parser, written by hand.** `packages/core/src/pdf.ts` (1,350 lines) implements just enough of the PDF skeleton — the cross-reference table in both forms, object streams, Flate with PNG predictors, and enough of the object grammar to walk from the trailer to the info dictionary and XMP packet. Page content, fonts, and graphics are not read at all; decryption is deliberately absent, so a permissions-encrypted PDF returns empty metadata plus an `encrypted: true` flag instead of mojibake.

- **Metadata extraction as judgement over untrustworthy markup.** `packages/sync/src/lib/metadata.ts` reads Dublin Core out of an EPUB's package document and a PDF's XMP packet + info dictionary. The subtle parts: junk-title rejection that is deliberately conservative (it takes real evidence to reject a title, so `coyotiv-brochure-v1.3-web` is kept), author splitting that refuses to split on commas because "Richardson, Leonard" is one person written backwards, and ISBN extraction that *checksums* candidates rather than pattern-matching, so an internal catalogue number is never mislabeled as an ISBN.

- **Stale-read recovery via re-read-on-failure.** `BookContentService` memoizes each book's ZIP central directory (an LRU of 8) and caches it in the Workers Cache for an hour. When a read fails — the directory said the entry was there and it was not — it re-reads the directory and retries once, because republishing a book replaces the object at the same key. `CatalogService` does the same at the shelf level: it keeps an expired catalog past its 60-second TTL as the best available answer when a refresh fails (serving stale for 10s), because "a shelf a minute out of date beats an error page."

- **Read-only is a degradation, not a mode.** `writableStorage()` narrows a `Storage` by asking whether `write`/`remove` are actually functions; a provider that fronts a genuinely read-only destination and a `BOOKSHELF_READ_ONLY=1` deployment are the *same situation* to the app, so every service already knows how to behave. Reading positions move back into the browser's `localStorage` with newest-wins reconciliation; profiles stop offering to change anything. It is enforced where the writing happens, not by hiding forms — posting the action directly gets the same refusal.

- **The whole-file-rewrite hazard, handled honestly.** The progress file holds every book a profile has open and is rewritten whole, so `ProgressService.save()` returns `false` (keep the place locally, retry) after a failed read rather than replace N positions with one. Profiles *refuse* rather than degrade for a related reason: the answer decides which file a position is written to, and falling back to the implicit default would write one person's place into a file belonging to nobody.

## Design decisions

The authors optimized for **portability, cheap reads, and graceful degradation**, and the trade-offs are named rather than hidden. No authentication — explicitly listed under "Not done yet," with the honest warning that anyone who can reach the app can read the whole library and pick any profile. Two devices reading one profile is last-write-wins, made lossless in the common case by clients only sending positions they believe are newer. No multi-range HTTP support, because nothing that reads the library asks for more than one range at a time. No Zip64. Windows is unsupported because the sync tool finds its image tools with `which`.

The deepest design choice is **regenerability as the safety property**. Books are the source of truth; the published tree, covers, and catalog are all derived and rebuilt from scratch each publish, so a removed book cannot linger. The only non-derived data — who is reading, and how far they got — is precisely the data `--force` is prevented from touching. This mirrors the wiki's own raw/summary/topic split: the derived artifacts are disposable; only the sources are precious.

## Comparison notes

- **[[SmolForge]]** is the closest sibling — a full-stack GitHub clone on Cloudflare Workers + D1 + R2 — and the contrast is instructive. SmolForge bets the whole Cloudflare platform can host a code forge; Bookshelf is the disciplined small-app counterexample using the same primitives (Workers + R2, OpenNext), with the added property that the identical build also runs as a plain Node server over a filesystem, which SmolForge cannot. Two ends of the "how much Cloudflare do you actually need" spectrum.

- **[[Random Access Parquet (RAP)]]** collapses a chain of dependent reads into an index lookup plus ranged reads to serve point queries from a data lake. Bookshelf performs the same shape of trick at book scale: the ZIP central directory (or a PDF's cross-reference table) is the index that lets ranged reads pull one chapter out of a 40 MB archive without transferring it. RAP makes the index external and explicit; Bookshelf inlines it in the format's own structure.

- **[[Libraries over Frameworks]]** is the design philosophy Bookshelf instantiates. The app talks to interfaces, never to Cloudflare; provider packages are the extension point, and a package "published by anyone can be installed and named in the config." It's the library-style escape hatch (a lower, explicit level beneath the convenience) that Petricek argues frameworks must have.

- **[[Edge Computing for Web Developers]]** calls the hybrid edge/self-host architecture "the article nobody has written yet." Bookshelf is a working instance of it: one build, chosen at build time, serves either on the edge (Workers + R2) or on a VPS (Node + filesystem), with a read-only environment toggle for anything reachable by strangers. Edge and self-host are the same code with different composition roots.

---
*Sources: [[raw/bookshelf]], [[summary/bookshelf]]*
*Last updated: 2026-08-25*
