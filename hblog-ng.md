# hblog-ng

An Obsidian vault-to-website static site generator with a canvas-rendered interactive knowledge graph. Written by Halit Alptekin as a personal blog engine, it ingests an Obsidian vault of Markdown notes, resolves `[[wikilinks]]` into bidirectional graph edges, and produces a fully static Next.js site deployable to Cloudflare Pages. ~15K lines of TypeScript.

The interesting thing isn't that it's a markdown blog — there are hundreds of those. It's the combination of (a) Obsidian-native wikilinks as the graph substrate, (b) canvas-based D3 force simulation rather than SVG for rendering the graph, and (c) a byte-identical content pipeline that makes PGP-signed posts verifiable in-browser.

---

## Architecture

The system has two halves: a **build-time data pipeline** that pre-processes the vault into JSON, and a **static Next.js site** that reads those JSON files to generate pages.

```
Obsidian Vault (external markdown, set via VAULT_PATH)
  │
  ├─ scripts/parse-vault.ts
  │   ├─ unified + remark pipeline (frontmatter extraction, wikilink detection,
  │   │   asset resolution, syntax preservation)
  │   ├─ VaultProcessor (shared factory for byte-identical content)
  │   ├─ processRelationships() → bidirectional link resolution
  │   │
  │   ├── data/vault.json       (full: content + signatures, for SSG)
  │   ├── public/vault.json     (lite: no content, for client graph/search)
  │   └── public/search-index.json  (MiniSearch)
  │
  ├─ scripts/optimize-images.ts (WebP via Sharp, 1920×1920)
  ├─ scripts/generate-og-images.ts (1200×630 per note)
  └─ scripts/sign-posts.ts (PGP detached signatures)
       │
       ▼
  Next.js static export → out/ → Cloudflare Pages
```

### Core data model

`types/vault.ts` defines `VaultNode`, the central type. Each note becomes a node with:
- `id`: `type:normalized-title` (e.g., `blog:system-capabilities`)
- `type`: `system | blog | profile | project | intel | research`
- `related_ids`: injected at build time by `processRelationships()` — the bidirectional wikilink edges
- `offensive`/`defensive`/`misc`: spectrum scores driving graph node colors
- `keywords`: extracted from frontmatter or `#hashtags` in body text

The filename is the title — there's no `title` frontmatter field, mirroring Obsidian's own convention (`scripts/parse-vault.ts:112`).

### Key modules

| Module | Role |
|---|---|
| `scripts/parse-vault.ts` | Entry point: walks vault, runs remark pipeline, builds nodes |
| `scripts/pipeline/processor.ts` | `VaultProcessor` class — shared factory for byte-identical content |
| `scripts/pipeline/remark-plugins.ts` | Custom remark plugins: frontmatter extraction, wikilinks, hashtags, asset resolution, syntax preservation |
| `scripts/pipeline/graph.ts` | `processRelationships()` — bidirectional link resolution |
| `lib/vault.ts` | Server-side data loading: `loadVaultData()`, `getNodesByType()`, `getRelatedNodes()` |
| `lib/routing.ts` | URL resolution: `getPathFromId()`, `findNodeBySlugOrId()`, `linkMatchesNode()` |
| `lib/page-logic.tsx` | Factory functions `createCollectionPage()` and `createDetailPage()` — DRY route definitions |
| `lib/markdown/extensions.ts` | Custom `marked` tokenizer extensions for Obsidian syntax |
| `lib/markdown/syntax.ts` | Shared regexes consumed by both React renderer and RSS renderer |
| `lib/rss-markdown.ts` | Separate string-based markdown pipeline for feed HTML |
| `lib/metadata.ts` | Open Graph, Twitter cards, JSON-LD structured data generation |
| `lib/feed.ts` + `lib/feed-utils.ts` | RSS 2.0, Atom, JSON Feed generation |
| `components/graph/NeuralGraph.tsx` | Graph container: wires data, simulation, interaction |
| `hooks/graph/useForceSimulation.ts` | Manual force physics in rAF loop (center pull, N-body repulsion, friction) |
| `hooks/graph/useGraphData.ts` | VaultNode→GraphNode transform, DFS pathfinding, filtering |
| `components/graph/GraphCanvas.tsx` | Canvas rendering: nodes (160×70 cards with spectrum meter), links, highlight states |

---

## Key Techniques

### Canvas force simulation, not SVG

Instead of using D3's SVG rendering (which creates a DOM node per graph element), hblog-ng implements the force physics manually and renders to `<canvas>`. The physics math is D3-style — center pull, N-body repulsion with O(n²) pairwise computation, friction damping, velocity-threshold sleep — but the rendering is a single `requestAnimationFrame` loop drawing onto a 2D canvas context (`hooks/graph/useForceSimulation.ts`).

The trade-off: smooth performance with hundreds of nodes, but all interaction (hover detection, click targeting, drag) must be computed from mouse coordinates against node bounding boxes. SVG would give free per-element event handling but chokes on DOM node count.

### Byte-identical content pipeline for PGP verification

The `VaultProcessor` class in `scripts/pipeline/processor.ts` is the single path through which markdown is transformed into published content. Both `parse-vault.ts` (the data pipeline) and `sign-posts.ts` (the PGP signer) use `VaultProcessor.forVault()` to create a processor with identical configuration. The signer calls `getSignableContent()` which returns the exact trimmed string that `parse-vault` stores as `node.content`.

This guarantees that PGP signatures created during the sign step verify against the published content — no whitespace drift, no escaping mismatch. It's a subtle but critical design choice that most static site generators don't have to think about because they don't sign their output.

### Dual JSON output for bandwidth control

`parse-vault.ts` writes two copies of the vault data:
- `data/vault.json` — full nodes with `content` and `signature` fields (for server-side generation via `lib/vault.ts`)
- `public/vault.json` — lite nodes with `content` and `signature` deleted (for client-side graph, search, and related nodes)

The lite file is fetched by the browser to power the graph and search. For a vault with long-form articles, this can save hundreds of KB in the initial page load. The search index (`public/search-index.json`) is built from the full nodes at build time so it can index content, but the raw content text isn't shipped to the client.

### `remarkPreserveSyntax` — passing custom syntax through the parser

The build pipeline uses `unified` + `remark-parse` + `remark-stringify` to normalize markdown. But remark-stringify would escape characters inside custom syntax like `[[wiki links]]` or `$math$`. The `remarkPreserveSyntax` plugin in `scripts/pipeline/remark-plugins.ts` solves this by:
1. Merging AST text nodes that were split by autolinks (e.g., `[video:https://...]` being parsed as `text([video:)` + `link(https://...)` + `text(])`)
2. Converting all custom syntax matches to raw `html` AST nodes, which remark-stringify passes through unescaped

This is the non-obvious trick that makes the pipeline work: the math `$e_1$` doesn't become `$e\_1$` (which KaTeX would render as a literal underscore rather than a subscript).

### Bidirectional link injection

`processRelationships()` in `scripts/pipeline/graph.ts` resolves every `[[wikilink]]` against all nodes (matching by ID, title, and aliases), then makes all connections reciprocal. If post A links to post B, post B gets A in its `related_ids`. This means the graph edges are complete without any runtime link resolution, and the "related posts" section always works — even if the author only created unidirectional links.

### Factory pattern for routes

Rather than duplicating `generateStaticParams`, `generateMetadata`, and page component logic across content types, `lib/page-logic.tsx` exports `createCollectionPage()` and `createDetailPage()` factories. A route file like `app/posts/[id]/page.tsx` is literally:
```typescript
const detail = createDetailPage({ type: 'blog', label: 'Posts', basePath: '/posts/' });
export const { generateStaticParams, generateMetadata, Page } = detail;
```

Adding a new content type (`intel`, `profile`) means updating `VaultNode.type`, adding the route file that calls the factory, and updating `lib/routing.ts`.

---

## Design Decisions

### Static export over server runtime

The entire site compiles to static files (`next.config.mjs` sets `output: 'export'`). No API routes, no serverless functions, no database. This means:
- **Zero hosting cost** on Cloudflare Pages free tier
- **No cold starts**, no runtime failures
- **Build time is the only moving part** — if it builds, it works

The cost: everything must be pre-computed. No dynamic content, no user-specific pages, no real-time data. For a personal blog, this is the right trade-off.

### Content lives outside the repo

The Obsidian vault is external — resolved via `VAULT_PATH` env var or bind-mount. The repo only ships `example-vault/` as a demo. This means the code and content have completely separate lifecycles, but also means cloning the repo doesn't give you real content.

### Filename as title

Mirroring Obsidian's convention, the markdown filename IS the note title. There is no `title` field in frontmatter. This simplifies authoring (no need to state the title twice) but means renaming a file changes its URL — the filename is both display name and URL slug.

### Canvas over SVG

Covered above — performance wins over developer convenience. The trade-off is real: all mouse interaction must be manually computed from coordinates.

### Triple feed format

RSS 2.0, Atom, and JSON Feed are all generated. The `lib/rss-markdown.ts` is a completely separate markdown-to-HTML pipeline from the React renderer — it uses string regex replacements rather than `marked` extensions because feed HTML can't use React components. This duplication is a maintenance burden (custom syntax must be handled in two places) but is necessary because the feed generation runs at build time outside React.

---

## Comparison Notes

**vs. other Obsidian publishers** (Obsidian Publish, Quartz, etc.): hblog-ng is a personal engine, not a general-purpose tool. It bakes in domain assumptions (offense/defense spectrum, cybersecurity content types) that general publishers don't. The PGP signing pipeline is unique — no other Obsidian publisher verifies post authenticity in-browser.

**vs. [[graphify]]**: Both build knowledge graphs from content, but graphify extracts knowledge graph data from codebases and produces multimodal graphs (code+PDFs+screenshots). hblog-ng's graph is built from explicit `[[wikilinks]]` in markdown — it doesn't extract entities from the content itself.

**vs. [[lat.md]]**: lat.md builds knowledge graphs from codebases in markdown with drift validation. hblog-ng's graph is a visual navigation layer over blog content, not a validated knowledge representation.

**vs. [[DeepWiki]]**: DeepWiki produces AI-generated wiki pages from codebases. hblog-ng produces a blog from hand-authored Obsidian notes. The similarity is in the "codebase → website" transformation, but one is AI-generated documentation, the other is a personal publishing pipeline.

**vs. D3-based graph visualizations**: Most projects reach for `d3-force` + SVG. hblog-ng implements the physics manually on canvas. This is more work but scales better — SVG node count is the limiting factor in large graphs, and canvas avoids that entirely.

**vs. incremental static regeneration (ISR)**: Next.js ISR would allow updating content without a full rebuild, but requires a server runtime. hblog-ng's full static export is simpler and cheaper, at the cost of rebuilding everything when content changes.

---

## Tags

#tool #project #static-site #obsidian #knowledge-graph #markdown #typescript #nextjs #pgp #canvas

*Sources: [[raw/hblog-ng]]*
*Last updated: 2026-07-03*
