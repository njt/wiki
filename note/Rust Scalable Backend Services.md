# Rust Scalable Backend Services

Sylvain Kerkour's field guide to building medium-sized backend services (10K+ lines, ~100 endpoints) in Rust on top of PostgreSQL: the layered HTTP → service → repository architecture, the crate stack that makes it work (tokio, hyper, axum, sqlx, moka, tracing), and the tricks that let a single Postgres database absorb the queue, the cron scheduler, and the cache invalidation that other stacks hand to separate infrastructure.

---

## Key Quotes

> "Rust's rich type system, compiler-enforced correctness and zero-cost abstraction make it a great choice nonetheless, especially for medium-sized services (10K+ lines of code and 100 or so endpoints) if you want to spend less time fixing business logic bugs or need high performance."

Kerkour is careful to concede the productivity gap up front — "Rust is not as productive as Go and its legendary standard library" — before making the case. This isn't evangelism; it's a scoped bet: the type system buys you fewer business-logic bugs, and you pay for it in slower iteration. The "medium-sized" qualifier is doing real work — he's explicitly *not* claiming Rust for the 20-endpoint CRUD app.

> "No more null pointer dereference or fields forgotten when creating a struct, and of course, there is no going back once you have tasted enums."

The emotional core of the piece, and the one engineers who've moved to Rust almost universally echo. Enums — not ownership, not the borrow checker — are the gateway drug. They make illegal states unrepresentable in a way `null` + a comment never can, which is the same argument [[Parse Don't Validate]] makes about parsing over validating.

> "Each layer can only communicate with the layer directly above or below e.g. the HTTP layer can't call a method on a repository."

The single most load-bearing rule in the article. The three layers — HTTP/scheduler/worker, service, repository — are only as valuable as the discipline that enforces their boundaries. Kerkour admits the architecture "has no official and shiny name," but claims it's scaled past tens of thousands of lines across Rust, Go, and Node.

> "Caching is done exclusively at the service layer. Remember, the repository layer must stay dumb, so no caching here."

A sharp division of responsibility: repositories are *just* SQL wrappers, and everything business-flavored — auth, validation, invariants, *and* caching — lives in the service layer. The justification is that cache TTLs are business rules ("safe to cache this entity for only 1 hour"), not data-access concerns. It's a subtle point most layered-architecture advice skips.

> "serving web applications on the same domain as the API lets you avoid the 'CORS tax'."

Kerkour serves the SPA and static assets from the same Rust binary via `rust-embed` rather than a CDN. His stated reasons — simpler deployment, no CORS — invert the conventional "put static on a CDN" wisdom. "Who could have guessed that simple systems are actually faster?" is a genuinely good line about latency hiding inside complexity.

> "Once again, Postgres got our back covered" — on electing a cron leader with advisory locks.

The article's quiet thesis is Postgres-as-platform: the queue is Postgres, the cron scheduler's leader election is a Postgres advisory lock, and the repository layer speaks `sqlx`. This is the practitioner's echo of [[Postgres Transactions Are a Distributed Systems Superpower]]'s "just use Postgres" convergence.

## Key Themes

- #pattern — **Layered architecture without a name**: HTTP/scheduler/worker → service → repository, with each layer only calling its neighbors. Change a dependency or a requirement and the blast radius stays local. It's the same instinct as [[Your Backend Is Full of Hidden Workflows]], but as a *preventative* structure rather than a *diagnostic*.
- #tool — **The Rust HTTP stack**: tokio (runtime) → rustls (TLS, Rust-native so statically linked) → hyper (HTTP protocol) → axum (ergonomics). `rustls` over `openssl` is a deployment argument: no matching OpenSSL version in the VM/Docker image, cleaner cross-compilation.
- #pattern — **Repository-as-SQL-wrapper**: repositories encapsulate queries, never "use both MongoDB and Postgres," and stay dumb — no caching, no business logic. Reuse queries and keep table changes in one file.
- #concept — **Postgres as the everything-database**: job queue, cron leader election via advisory locks, caching at the service layer. No Redis, no Celery, no separate scheduler service.
- #tool — **Converged observability via `tracing`**: the community converged on one structured-logging/tracing crate, maintained by the tokio team.
- #concept — **The CORS tax**: same-domain SPA serving removes an entire category of configuration and debugging.

## Critical Analysis

**The value is the specificity, not the novelty.** Nothing here is new — layered architecture is a 1990s idea, the repository pattern is Fowler's, "Postgres does everything" is a 2024–2026 convergence. What Kerkour adds is a *concrete Rust implementation*: the exact `service_fn` + `TlsAcceptor` wiring, the `Queryer` trait that lets `create_user` accept either a `&Pool` or a `&mut PgConnection`, the `#[instrument(skip_all)]` on the repository. Most architecture articles stay abstract; this one compiles.

**The "medium-sized" scoping is the honest part, and it's under-defended.** Kerkour says 10K+ lines and ~100 endpoints is the sweet spot, but never says what breaks on either side. Below it, Rust's ceremony isn't worth it — he admits Go wins. Above it, a single Postgres instance doing queue + cron + cache + storage starts to strain, which is exactly the ceiling [[SQLite Is All You Need]] and [[Postgres Transactions Are a Distributed Systems Superpower]] both acknowledge from opposite directions. The article is strongest in the middle band it claims and silent about the exits.

**The Postgres-as-platform bet is the most opinionated, least-argued claim.** The queue and the cron leader election are waved at via links to other posts ("nothing has changed since then," "take a look at my article"). That's fine for a pattern roundup, but it hides the real risk: advisory-lock leader election and a Postgres job queue are the *exact* infrastructure the GUS/SQLite camp argues you don't need until you've already won. Kerkour is building a different class of service than [[The GUS Stack — Go, Unix, SQLite]]'s weekend app, and never quite says so.

**The SPA-same-domain advice is quietly contrarian and mostly right for this audience.** Serving React from the Rust binary trades CDN edge-caching for deployment simplicity and zero CORS. For a medium-sized internal or B2B service it's a good trade; for a consumer product with global users it's a regression. The article doesn't mark that boundary.

**The strongest exportable idea is the dumb repository.** "No caching in the repository, because cache TTLs are business rules" is a clean, memorable principle that survives translation to any language or stack. It's the kind of constraint that prevents the accretion [[Your Backend Is Full of Hidden Workflows]] warns about — caching logic, once allowed into the data layer, is exactly the invisible coordination that compounds.

**Where it fits the wiki:** this is the Rust+Postgres counterpoint to the SQLite/GUS position. [[SQLite Is All You Need]] argues the database server is premature for most products; Kerkour assumes Postgres as the substrate and shows what you can build on it once you've committed. [[Unnecessary Optimization in Rust]] supplies the compiler-trust half of the Rust case. [[Microservices for the Benefits, Not the Hustle]] is the adjacent architecture argument — a modular monolith over microservices — which Kerkour's single-binary, single-database service embodies without naming it.

---

*Sources: [[raw/rust-scalable-backend-services]], [[summary/rust-scalable-backend-services]]*
*Last updated: 2026-08-14*
