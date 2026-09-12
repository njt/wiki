# Grimmory

A self-hosted digital library server — a community fork of Booklore — that turns a folder of eBooks, PDFs, comics, and audiobooks into a multi-user, browser-based reading platform with one genuinely unusual property: it speaks many client protocols at once. The same catalog is exposed as OPDS feeds, a Komga-compatible API, a native Kobo sync endpoint, and a KOReader sync API, so a Kobo, an OPDS app, and a comic reader all sync against one library instead of their vendor clouds. Architecturally it's a Spring Boot 4 / Java 25 monolith (~91K lines Java, ~122K lines Angular) tuned with unusual care for the self-hosted scale it actually runs at.

---

## Architecture

**One monolith, two tiers, many plugins.** The backend is a Spring Boot app in package `org.booklore` — the fork heritage is baked into the package name. The frontend (Angular 21, Tailwind 4, TanStack Query) is a separate pnpm workspace that compiles into the Spring Boot jar's static resources (`build.gradle.kts` copies `frontend/dist` into `static/`).

**Per-format strategy pattern for file ingestion.** `BookFileProcessorRegistry` discovers all `BookFileProcessor` beans via Spring's `List` injection and maps them into an `EnumMap<BookFileType, BookFileProcessor>`. Each format (EPUB, PDF, MOBI, AZW3, FB2, CBX, audiobook) has its own processor, metadata extractor (`metadata/extractor/*`), and metadata writer (`metadata/writer/*`). Adding a format means implementing one interface and registering types.

**Dynamic querying as composed Specifications.** Browse is not a hand-written SQL layer — every filter is a JPA `Specification`. `BookFacetRegistry` maps 27 facet names (author, genre, mood, content_rating, per-source ratings, comic metadata) to Specifications; `BookSortRegistry` registers pluggable sort keys including `random` and a composite `readingProgress` that folds five per-reader progress fields together. `FacetLogic` (AND/OR/NOT) is applied uniformly to every facet.

**Rules compiled to queries.** "Smart shelves" (`MagicShelfService`, `BookRuleEvaluatorService`) let users define shelves with rules; those rules (serialized as JSON DTOs `GroupRule`/`Rule`/`RuleField`/`RuleOperator`) are compiled into Criteria-API Specifications at query time, including composite fields like series gaps and status that need subqueries. A shelf can be referenced as a facet (`magic:<id>`), so shelves compose with other filters.

**Events over WebSocket.** Book and file changes broadcast via STOMP (`Topic.BOOK_ADD`, `BOOK_UPDATE`, `BOOKS_REMOVE`, `TASK_PROGRESS`) to permission-scoped subscribers; the frontend uses `@stomp/rx-stomp` plus SSE (`ngx-sse-client`) for task progress.

**One app, three auth surfaces, plus per-protocol filters.** Local JWT (BCrypt), OIDC, and remote-header proxy auth coexist; then separate `SecurityFilterChain` beans for OPDS and Komga (Basic auth), plus `KoboAuthFilter`, `KoreaderAuthFilter`, and `QueryParameterJwtFilter` for the device protocols. `BookAccessAspect` / `LibraryAccessAspect` enforce per-library access via `@CheckBookAccess` / `@CheckLibraryAccess` annotations.

## Key Techniques

**Hand-rolled embeddings, brute-force recommendations.** `BookVectorService.generateEmbedding` builds a 128-dimension vector using the hashing trick: each feature (title words, author, category, series, publisher, description words) is hashed with `hashCode() % 128` and added at a hand-picked weight (authors 5.0, series 6.0, categories 4.0, title 3.0, description 1.0). The vector is L2-normalized, serialized as JSON into a `metadata.embedding_vector` column, and similarity is a raw dot product. `BookRecommendationUpdaterTask` batch-recomputes in memory (batches of 500) and writes back via `compareAndSaveEmbeddings`. No vector DB, no ANN index, no ML library — this is [[Just Brute Force Your Embeddings]] shipped as a feature. A fallback path (`findSimilarBooksViaEntities`) loads full entities and computes weighted Jaccard/cosine on the fly when a vector is missing. A `MAX_BOOKS_PER_AUTHOR = 3` cap diversifies results, and same-series books are excluded.

**Sparse MD5 content fingerprints.** `FileFingerprint.generateHash` reads 1KB blocks at exponentially spaced offsets (256 bytes out to ~1GB, stepping 4× each time, breaking at EOF) and hashes them — a fixed handful of reads regardless of file size, for duplicate detection on multi-GB audiobooks. It's a deliberate approximation: content beyond the sampled offsets is invisible to the hash. Folder audiobooks hash the first audio file plus the file count.

**A debounced, hash-keyed deletion pool.** `PendingDeletionPool` solves the "rename looks like delete-plus-add" race. A vanished file isn't deleted immediately — it's parked with a timer and indexed by content hash. If a new file with the same hash appears (a move/rename), `matchByHash` recovers the book's metadata and ID instead of recreating it. Only when the timer expires is the book soft-deleted. Folder moves are matched by ≥50% hash overlap. Two `ConcurrentHashMap`s (path→pending, hash→path) keep it lock-light.

**Sidecar metadata that travels with the file.** `metadata/sidecar/*` writes a JSON sidecar (and optional cover) next to each book, with bidirectional import/export and bulk operations — the catalog is portable with the files, not locked in the database. It only runs in `LOCAL` disk mode; `NETWORK` mode disables destructive file ops (delete/move/rename) to avoid wrecking shared mounts.

**SSRF-guarded OIDC discovery.** `application.yaml` lists ~30 RFC 1918 / reserved CIDR ranges under `outbound.restricted-ranges`, blocked for outbound OIDC calls by default, with an explicit `allow-unsafe-hosts` escape hatch — the rare self-hosted app that treats its own metadata fetching as an SSRF surface.

**A second, app-level migration framework.** Beyond Flyway's 144 schema migrations, `migration/migrations/*` runs Java-based data backfills (populate hashes, embeddings, search text, covers, metadata scores) behind a `Migration` interface — schema via Flyway, data via code.

## Design Decisions

**Optimized for one household, not a fleet.** Virtual threads are on, Tomcat caps platform threads at 10, and the Hikari pool is 5 — with comments in `application.yaml` explaining that "virtual threads handle concurrency" and "5 is enough for a self-hosted app." Hibernate is tuned the same way: `default_batch_fetch_size: 16`, `in_clause_parameter_padding`, `plan_cache_max_size: 128`, and `fail_on_pagination_over_collection_fetch: true` (fail rather than silently paginate in memory). This is the [[SQLite Is All You Need]] ethos applied to a client-server stack: shrink every component to the traffic you actually have.

**Continuity with Booklore over clean-slate naming.** The fork keeps the `org.booklore` package, `booklore` service names, and a documented drop-in migration (keep your compose service name, container name, database, ports, and volumes; swap only the `image:` line). Easy migration is prioritized over a tidy rename.

**Brute force over index infrastructure.** The recommender and the fingerprinter both choose O(n)/approximate approaches over building index infrastructure — correct for a few thousand books and zero operational surface. The cost is explicit: similarity is recomputed as a batch task, not kept continuously fresh.

**Protocol breadth over protocol depth.** Emulating Kobo, Komga, and OPDS at once means three partial reimplementations rather than one deep native experience. The Kobo path even proxies through to kobo.com (`KoboServerProxy`) where needed. The bet is that "works with the device you already own" beats "best-in-class single client."

## Comparison Notes

**vs. Calibre** — Calibre is a desktop, single-user, local-first library manager with a famously deep ebook-conversion engine. Grimmory is a multi-user server: the browser is the primary UI, and device sync is native rather than USB-driven.

**vs. Kavita / Komga** — Self-hosted reading servers with narrower scope (comics/ebooks, one client). Grimmory doesn't just compete — it *implements* the Komga API (`mapper/komga`, `KomgaService`) so existing Komga clients point at Grimmory unchanged, then extends past it to audiobooks and Kobo/OPDS/KOReader sync.

**vs. [[Just Brute Force Your Embeddings]]** — Grimmory is a production confirmation of Turnbull's thesis: recommendations via hashed 128-dim vectors and brute-force cosine stored in MariaDB, no vector DB. It also shows the pattern's shape at self-hosted scale — a few thousand books with batch recomputation, not millions of documents at QPS.

**vs. [[SQLite Is All You Need]]** — The same "engineer for the traffic you actually have" discipline, applied to a MariaDB monolith instead of a single file: tiny pools, virtual threads, batch brute-force compute, all tuned for one household.

**vs. [[celld]]** — The same operational-sovereignty argument one layer up the stack: where celld reclaims serverless compute state, Grimmory reclaims reading data — the catalog lives in sidecar files next to the books, not a vendor account.

---

*Tags: #tool #project #self-hosted #database*

---

*Sources: [[raw/grimmory]], [[summary/grimmory]]*
*Last updated: 2026-08-25*
