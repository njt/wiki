# The GUS Stack — Go, Unix, SQLite

Noah Zoschke's prescription for a technology stack that works with AI coding agents rather than against them: Go for the language, Unix for the OS layer, and SQLite for persistence. The thesis is that agentic coding has flipped the economics of stack selection — spinning up an app by chatting with an agent is now faster than writing planning docs, and a shared foundation of boring, stable, well-understood technologies saves rebuilding from scratch each time an agent starts a new session.

---

## Key Quotes

> "Chatting with an agent to spin up an app is now easier than drafting planning docs."

The economic inversion that makes the rest of the article matter. When the cost of building something drops to near-zero, the constraint shifts from "can we build it?" to "can we understand what we built?" Zoschke's implicit answer is: pick components that are individually legible and don't change. An agent that encounters Go code, a Unix environment, and a SQLite file can reason about all three from training data alone — no proprietary DSLs, no bespoke config formats, no framework magic that only your team understands. [[AI Slop Starts with the Codebase Itself]] makes the same argument from the opposite direction: proprietary patterns are an AI tax.

> "A simple and well-designed shared foundation saves rebuilding from scratch each time."

The counter-argument to "let the agent figure it out." Without a shared foundation, every agent session starts from zero — the agent picks whatever stack its training data weights highest this week, and you end up with a polyglot mess. Zoschke is arguing that the role of the human in agentic development shifts from writing code to curating constraints. The stack is the constraint that prevents chaos.

> "We diverge from the GUTS stack by preferring server-side rendering with minimal JS, using HTMX as much as possible."

The Go/TypeScript fork is the article's most opinionated architectural claim. exe.dev's GUTS (Go, Unix, TypeScript, SQLite) adds TypeScript for the frontend; Zoschke strips it back to server-rendered HTML with HTMX sprinkles. This isn't Luddism — it's a bet that the simplicity of a single-language server (Go + templ + HTMX) compound-wins over the flexibility of a JS frontend when agents are writing the code. [[Phoenix LiveView]] makes the same bet from the Elixir side: server-rendered HTML diffs over WebSocket eliminate an entire class of complexity that agents handle poorly. The difference is that Go + templ + HTMX has no real-time diff story — each interaction is a full page render or a partial swap — which keeps it simpler at the cost of interactivity.

> "Agentic coding works best with fast feedback. Verify the page layout with rodney."

The testing philosophy compressed to a command. Zoschke uses headless Chrome via DevTools Protocol (rod for tests, rodney for one-off validations) to give agents visual feedback on rendered output — write a test that clicks through the happy path, check DOM state, and ship. This is the deterministic verification layer that [[Guardrails and Feedback Loops]] calls for: the agent can see whether the button actually rendered, not just whether the HTML compiled. The contrast with [[A New Era for Software Testing]] is interesting: antirez argues for LLM-driven exploratory QA, while Zoschke argues for deterministic DOM-state assertions. They're complementary — the DOM tests catch regressions, the exploratory agent catches the things you didn't think to test.

## Key Themes

#stack #go #sqlite #unix #htmx #agentic-development #technology-choice #pattern

- **Boring technology as an agent accelerator**: The GUS components share a property that matters more for agents than for humans: they're stable, well-documented, and appear heavily in training data. Go's syntax hasn't changed since 2012. SQLite is the most deployed database engine in the world. Unix is Unix. An agent encounters these technologies in every corpus it trains on, which means it generates idiomatic code more often and hallucinates less. [[The Lindy Effect in Software]] provides the theoretical backing: survival implies fitness, and fitness means the agent knows the tool well.

- **Go's goroutines as a correctness multiplier for agents**: Bob Nystrom's [[What Color is Your Function]] argues that languages with lightweight threads (goroutines, coroutines, fibers) eliminate the "function coloring" problem — the async/sync divide that forces every function to declare a color and prevents free composition. Go's goroutines mean agents writing Go code don't have to reason about async propagation, `Task<T>` wrappers, or which color a higher-order function should be. This is a structural advantage for agent-generated code: one less category of bug that compounds with every function the agent writes.

- **Stack selection as constraint curation**: The human's job shifts from choosing technologies to choosing constraints. Picking Go over Python isn't about performance — it's about giving the agent a smaller, more coherent target. A batteries-included standard library means fewer dependency decisions for the agent to get wrong. Single-binary deployment means fewer deployment topologies for the agent to misconfigure. The stack is the spec.

- **SQLite as the "obvious" starting point**: Zoschke treats SQLite as so self-evidently correct that he barely argues for it — just notes it's "the most widely deployed database engine in the world" and moves on. This confidence is backed by [[SQLite Is All You Need]]'s benchmarked case (3,654 req/s, 50K users on one file) and [[Learning a Few Things About Running SQLite]]'s honest operational experience. The GUS Stack article doesn't add new SQLite arguments — it assumes the consensus and builds on it.

- **HTMX as the TypeScript substitute**: Where exe.dev keeps TypeScript in the stack, Zoschke replaces it with HTMX — a library small enough to fit in a single HTML attribute (`hx-get`, `hx-post`, etc.). This is the most debatable piece of the stack. HTMX handles simple interactivity cleanly but hits a complexity cliff when interactions go beyond request-response. Zoschke's implicit claim is that for agent-built apps, staying on the simple side of that cliff is the whole point.

- **Go library ecosystem as expressed taste**: The specific library choices (cockroachdb/errors, templ, fuego, sqlc, modernc.org/sqlite, goose, dbos) are a portrait of Go ecosystem design philosophy: type safety over dynamism, code generation over reflection, explicit over implicit. `sqlc` generates type-safe Go from SQL rather than building an ORM. `templ` is type-safe HTML templating rather than string interpolation. These aren't arbitrary preferences — they're libraries that catch errors at compile time, which is what you want when an agent is writing the code.

- **Visual verification as the agent feedback loop**: The rod + rodney setup is the article's most novel contribution. Giving an agent a way to see what it built — not just read compiler output — closes a feedback loop that LLMs are particularly bad at simulating. The agent can verify "the button is blue and in the top-right corner" by checking DOM state, not by reasoning about CSS. This is [[Guardrails and Feedback Loops]] in practice: deterministic verification over prompt pleading.

## Critical Analysis

**The article's real argument is about defaults, not innovation.** Nothing in the GUS Stack is new. Go is 14 years old. SQLite is 24. Unix is 55. HTMX is a reimplementation of ideas from intercooler.js (2013) and Rails' turbolinks (2013) and pjax (2012). The argument isn't "here's a new stack" — it's "here's the stack that makes agents produce working software on the first try." The innovation is the selection, not the components.

**The stack is a bet on training-data density.** Zoschke doesn't say this explicitly, but every component of GUS wins on training-data frequency. Go is one of the most-used languages on GitHub. SQLite documentation, forum posts, and source code are everywhere. Unix commands and conventions appear in every programming tutorial ever written. An LLM has seen more correct Go+SQLite+Unix code than it has seen correct Next.js+Prisma+PlanetScale code by at least an order of magnitude. The GUS Stack is a stack the models already know.

**HTMX is the weakest link, and Zoschke knows it.** The article's HTMX recommendation is qualified ("as much as possible") in a way the SQLite recommendation isn't. That qualification is doing real work. HTMX is brilliant for server-driven interactivity but falls apart when you need client-side state that survives navigation, optimistic UI updates, or interactions that don't map cleanly to request-response. The unstated assumption is that agent-built apps don't need those things — or more precisely, that the complexity cost of adding them outweighs the UX benefit. This is probably true for internal tools and early-stage products. It's probably not true for consumer-facing products with real users.

**The HTMX "as much as possible" ceiling has a concrete shape, and [[Progressively Enhanced Forms with HTMX]] maps it.** Rafa's field report catalogs where HTMX's abstraction leaks: the `hx-target` coherence problem (partial swaps create stale data in unswapped regions), transient state that disappears on reload (form values don't survive page refresh), and Enter-key behavior that forces form-structure decisions (splitting one logical form into two HTML forms). These aren't HTMX bugs — they're consequences of the request-response model that HTMX embraces. Zoschke's qualification is correct, and Rafa shows exactly where the qualification bites.

**The library recommendations are a snapshot of mid-2026 Go ecosystem taste.** `cockroachdb/errors` for stack traces is a reaction to Go's famously minimal error handling. `templ` is the current frontrunner in Go's type-safe templating renaissance. `fuego` for OpenAPI generation reflects the growing consensus that API specs should be generated from code, not written separately. `dbos` for durable workflows in SQLite is the newest and least proven recommendation. A year from now, some of these will look prescient and others will look dated. The stack itself will outlast the specific library choices.

**What's missing: the auth story.** Zoschke covers the database (SQLite), the language (Go), the deployment target (Unix), the frontend (HTMX + templ), and the testing strategy (rod + rodney). He doesn't mention authentication. For agent-built apps, auth is where the most consequential security decisions live, and it's the part agents are worst at getting right. Omitting it from the stack description is a significant gap — not because Zoschke doesn't have opinions, but because the article implicitly treats auth as something you bolt on later, which is exactly the attitude that produces vulnerable applications.

**The "chatting with an agent is easier than planning docs" claim is a description of the present, not a prediction about the future.** Right now, it's true: spinning up a prototype with Claude Code is faster than writing a design doc. But this is a local maximum, not an equilibrium. As [[The New Software Lifecycle]] documents, the bottleneck is migrating from implementation to verification — and planning docs are the artifact that makes verification possible. The GUS Stack article is a snapshot of the "build fast" phase; it doesn't address what happens when you need to maintain, extend, or hand off what you built.

**Rust as a GUS variant.** [[celld]] (Deno's self-hosted Durable Objects runtime) is a Rust counterpoint: it uses SQLite even more pervasively — one database *per Durable Object* rather than one per application — and relies on Rust's type system to distinguish definite from ambiguous failures at the CAS layer. Same "SQLite as foundation" thesis, different language and granularity.

**The deeper argument: the stack is the harness.** [[Components of a Coding Agent]] established that the harness matters more than the model. Zoschke extends this: the technology stack is part of the harness. Pick components the model already knows, and the harness does less work. Pick components with stable interfaces, and the harness doesn't need updating. Pick components that fail at compile time rather than runtime, and the harness catches errors before the agent moves on. The GUS Stack isn't a technology recommendation — it's a harness engineering recommendation disguised as a technology recommendation.

---

*Sources: [[raw/the-gus-stack-go-unix-sqlite]]*
*Last updated: 2026-07-18*
