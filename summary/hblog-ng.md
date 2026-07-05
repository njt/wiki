---
url: https://github.com/halit/hblog-ng
title: "hblog-ng: Obsidian Knowledge Graph Blog"
author: Halit Alptekin
date_fetched: 2026-07-03
date_published: 2024
---

## Repository Analysis

hblog-ng transforms an Obsidian vault of Markdown notes into a statically-generated site with an interactive, canvas-rendered knowledge graph of notes and their `[[wikilinks]]`. Built with Next.js (App Router, static export) and deployed to Cloudflare Pages. ~15,100 lines of TypeScript/TSX across ~120 source files.

### Architecture

The project is a **static site generator** with a **build-time data pipeline** that pre-processes an Obsidian vault into JSON data files, which then power a Next.js static export.

```
Obsidian Vault (external, not in repo)
  → scripts/parse-vault.ts (unified + remark pipeline)
    → data/vault.json (full: content + signatures, for SSG)
    → public/vault.json (lite: no content/signatures, for client graph/search)
    → public/search-index.json (MiniSearch index)
  → Next.js static export → Cloudflare Pages
```

The vault is resolved by `scripts/pipeline/vault-path.ts` in priority order: `VAULT_PATH` env var → bind-mounted `vault/` directory → bundled `example-vault/` fallback. Content is never committed to the repo.

### Data Pipeline (build-time)

Entry: `scripts/parse-vault.ts`

1. **Markdown parsing**: Uses `unified` + `remark-parse` with custom remark plugins (`scripts/pipeline/remark-plugins.ts`):
   - `remarkFrontmatterExtractor` — extracts YAML frontmatter, removes from AST
   - `remarkWikiLinks` — detects `[[links]]` for relationship mapping
   - `remarkHashtags` — extracts `#tags` from text nodes
   - `remarkAssets` — resolves and copies images/files/videos to public/
   - `remarkPreserveSyntax` — converts custom syntax (`[[...]]`, `$math$`, `[file:...]`, `[video:...]`) to raw HTML nodes so remark-stringify doesn't escape them

2. **Node construction**: Frontmatter fields map to `VaultNode` (`types/vault.ts`). The filename IS the title (no `title` frontmatter field — mirroring Obsidian's convention). Node IDs use format `type:normalized-title` (e.g., `blog:system-capabilities`).

3. **Relationship resolution** (`scripts/pipeline/graph.ts`): `processRelationships()` builds a forward-link map by matching `[[wikilinks]]` from each node's content against all other nodes' IDs/titles/aliases (via `lib/routing.ts`'s `linkMatchesNode()`), then makes all links bidirectional and injects `related_ids` into each node.

4. **Dual JSON output**: Full `data/vault.json` (with content and signatures) for server-side generation; lite `public/vault.json` (content and signatures deleted) for client-side graph and search. This keeps the full content out of the browser bundle.

5. **MiniSearch index**: Built from full nodes (excluding system nodes), indexed on `title`, `content`, `description`, `keywords`, `stack` with boosted field weights. Written to `public/search-index.json`.

The VaultProcessor class (`scripts/pipeline/processor.ts`) is a shared factory used by both `parse-vault` and `sign-posts` to guarantee byte-identical content output, so PGP signatures verify against published content.

### Markdown Extensions (runtime)

Custom `marked` TokenizerAndRendererExtensions in `lib/markdown/extensions.ts`:
- `wikilink`: `[[Page]]`, `[[Page|Label]]`, `[[Page#Section|Label]]`
- `embed`: `![[Embed]]`
- `mathBlock`/`mathInline`: `$$...$$` and `$...$` (KaTeX)
- `callout`: `> [!TYPE] Title` (Obsidian-style callouts)
- `referenceLink`: `[ref:key]` (BibTeX citations)
- `fileAttachment`: `[file:path|name]`
- `videoPlayer`: `[video:url|caption]`
- `asciinema`: `[asciinema:id|caption]`
- `hashtag`: `#tag`

Shared regexes live in `lib/markdown/syntax.ts` and are consumed by both the React renderer and the RSS markdown renderer (`lib/rss-markdown.ts`).

### Knowledge Graph (client-side)

The graph is **canvas-rendered with manual force physics**, not D3 SVG:

- `hooks/graph/useGraphData.ts` — transforms `VaultNode[]` into `GraphNode[]` and `GraphLink[]`. Adds keyword nodes as separate graph entities. Uses `related_ids` (pre-computed at build time) for wikilink edges, deduplicating undirected pairs. Implements DFS pathfinding (capped at 5 depth, 20 paths).
- `hooks/graph/useForceSimulation.ts` — manual force simulation in a `requestAnimationFrame` loop: center pull, N-body repulsion (O(n²) with distance-squared cutoff at 640000px²), friction damping, velocity-threshold sleep.
- `components/graph/GraphCanvas.tsx` — renders nodes and links on `<canvas>` with context menu support. Draws content nodes (160×70 cards with spectrum meter, icon, title) and keyword nodes (compact pill shape). Handles search highlight, path highlight, and zoom/pan transforms.

Node spectrum (offensive/defensive/misc) is calculated via `utils/metrics.ts` and displayed as a 12-block colored meter on each graph node card. Config in `config/graph.ts`: offense #ff0055, defense #00e5ff.

### Routing

`lib/routing.ts` provides:
- `extractSlugFromId()` — strips `type:` prefix from node IDs
- `getPathFromId()` — maps node type to URL path (`blog→/posts/`, `project→/projects/`, `research→/research/`)
- `findNodeBySlugOrId()` — lookups by slug, full ID, or alias
- `extractInternalLinks()` — regex extraction of `[[links]]` from markdown
- `linkMatchesNode()` — matches a wikilink label against a node's ID, title, or aliases (with normalization for spaces/hyphens)

### Page Factory Pattern

`lib/page-logic.tsx` exports `createCollectionPage()` and `createDetailPage()` factories that produce the `{metadata, Page}` or `{generateStaticParams, generateMetadata, Page}` triple for route files. Each route under `app/{posts,projects,research}/` just spreads the factory result. Adding a new content type requires: update `VaultNode.type`, add route, call the factory.

### SEO & Syndication

- `lib/metadata.ts` — generates Next.js Metadata with Open Graph, Twitter cards, JSON-LD structured data (TechArticle/ScholarlyArticle/SoftwareApplication + breadcrumbs)
- `lib/feed.ts` + `lib/feed-utils.ts` — RSS 2.0, Atom, and JSON Feed generation (all static routes)
- `lib/rss-markdown.ts` — separate markdown-to-HTML pipeline for feeds (string-based, not React components). Handles wikilinks, math, callouts, file attachments, references — converting custom syntax to plain HTML suitable for feed readers
- `scripts/generate-og-images.ts` — 1200×630 PNG per note, rendered from SVG template via Sharp

### PGP Signing

`scripts/sign-posts.ts` writes detached `.asc` signatures using the same `VaultProcessor.getSignableContent()` to ensure byte-identical content. OpenPGP.js verifies in-browser (`components/modals/SignatureModal.tsx`). Key material stays in `.env.local`.

### Environment Configuration

`config/env.ts` uses a `STATIC_ENV` map where every `NEXT_PUBLIC_*` var is accessed literally (for Next.js inlining), with a `pub()` helper that falls back to `window.__ENV__` runtime injection and then a default. Defaults live in the committed `.env` file, not inline in code.

### Dev Server

`scripts/dev-server.ts` spawns two child processes: the watcher (`watch-data.ts` — re-runs data pipeline on file changes) and `next dev`. Cross-platform compat via `npm.cmd`/`npx.cmd` on Windows.

### Key Dependencies

- **marked** (v17) — markdown parsing with custom tokenizer extensions
- **unified + remark** — build-time markdown pipeline (parse, frontmatter, GFM, preserve)
- **MiniSearch** (v7) — client-side fuzzy search with field boosting
- **D3-force** physics (reimplemented manually) — not actually using the d3-force npm package
- **Recharts** (v3) — chart blocks in markdown
- **KaTeX** — math rendering
- **Mermaid** — diagram rendering (lazy-loaded)
- **OpenPGP.js** — browser-side signature verification
- **Sharp** — image optimization (WebP conversion, resize to 1920×1920)
- **Next.js 16** — static export mode

### Design Decisions

1. **Canvas over SVG for graph**: The D3 force simulation math is kept but rendered on canvas instead of SVG. This avoids DOM node overhead for large graphs and enables smooth animations. Trade-off: no built-in event handling per node — all interaction (hover, click, drag) is computed from mouse coordinates against node bounding boxes manually.

2. **Dual-output data pipeline**: Full content stays server-side (`data/vault.json` for SSG), lite data goes to client (`public/vault.json` without content/signatures). This is a bandwidth optimization — the full vault could be hundreds of KB of markdown content.

3. **Bidirectional link injection at build time**: `processRelationships()` makes all wikilinks bidirectional before the site is built. This ensures graph completeness (if A links to B, B lists A as related) without runtime computation. Trade-off: this is O(n²) in the worst case at build time.

4. **Shared processor for byte-identical content**: The `VaultProcessor` class is a singleton-style factory used by both the data pipeline and the PGP signer. This guarantees that the content that gets signed is exactly the content that gets published. A subtle design choice that prevents signature verification failures from whitespace or escaping differences.

5. **Filename as title**: Mirroring Obsidian's convention, the markdown filename is the sole source of truth for the note title. There is no `title` frontmatter field. This simplifies content authoring but means renaming a file changes its URL and breaks existing links.

6. **Static export over SSR**: The entire site is static files with no server runtime. This keeps hosting costs near zero (Cloudflare Pages free tier) but means no dynamic content — everything must be pre-computed at build time.

7. **Factory pattern for routes**: `createDetailPage()` and `createCollectionPage()` eliminate boilerplate across content-type routes. Adding a new type (`intel`, `profile`) is mostly a matter of calling the factory in a new route file and updating types.

### Weaknesses / Trade-offs

- **O(n²) force simulation**: The N-body repulsion loops over all node pairs every frame. For very large vaults (500+ nodes), this could become a performance bottleneck on weaker devices.
- **No incremental builds**: The full data pipeline runs on every build. For large vaults with many images (optimization), this could be slow. There's no caching or change detection in the pipeline itself (the dev watcher re-runs on any file change).
- **DFS pathfinding is depth/breadth capped**: `MAX_DEPTH=5` and `MAX_PATHS=20` mean the shortest-path feature won't find connections in deeply connected but sparse graphs.
- **Bidirectional linking is lossy**: All wikilinks are made reciprocal at build time, but the directionality information is discarded. A→B and B→A look the same in `related_ids`.
- **RSS markdown is a separate implementation**: `lib/rss-markdown.ts` duplicates much of the markdown extension logic as string regex replacements rather than reusing the `marked` extensions. This is a deliberate trade-off (string rendering can't use React components) but creates a maintenance burden — custom syntax must be handled in two places.
- **No content in repo**: The vault-is-external design is clean but means the repo can't be cloned and run with real content. The `example-vault/` is a demo, not production content.
