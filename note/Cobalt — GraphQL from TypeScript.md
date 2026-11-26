# Cobalt — GraphQL from TypeScript, in the Age of AI

Cobalt (cobalt27.dev) is a TypeScript framework that infers a GraphQL schema from your code and generates a typed client SDK, pitched explicitly as a framework designed for the age of AI coding agents: dense, collocated code that fits in a context window.

---

The pitch walks a triangle: REST has endpoint fatigue and over-fetching; pure GraphQL has the "Double Declaration Problem" (TS interface + GraphQL schema + resolver, three artifacts that drift and crash your app); tRPC has the "Tight Coupling Trap" (backend types leak into the client bundle, making later microservices or public APIs a rewrite). Cobalt claims all three virtues at once: "The speed of tRPC, the flexibility of GraphQL, and the simplicity of REST," with "magnetic type safety" — define logic once, schema and SDK are inferred.

> "Your TypeScript code IS your schema. No client-side codegen steps or schema sync required."

The eliminations are the point: no controllers, no manual serialization, no routers, no adapters, no codegen watchers. Every deleted glue layer is both less code for humans and fewer files an agent must load.

> "LLMs struggle with large, scattered codebases. Providing context across 50 files degrades performance and increases cost. … Dense, collocated code. Smaller context window footprint means smarter AI assistance and lower token costs."

This is the section that earns Cobalt a place in the wiki. It's a framework whose architecture is explicitly optimized for the machine reading the codebase, not just the human — an inversion of the usual ordering. It rhymes with [[The GUS Stack — Go, Unix, SQLite]], which prescribes boring, training-data-dense technologies so agents produce idiomatic code on the first try; Cobalt adds a second axis: not just *what* the code says but *how much of it an agent must hold at once*. It also connects to the token-economics findings in [[The Economic Benefit of Refactoring]] (Fowler's 83% input-token reduction from semantic decomposition) and [[Code Cleanliness and Coding Agents]] (SonarSource's 7–8% token savings) — Cobalt is those results turned into a product thesis rather than measured after the fact.

## Key themes

- #tool — a TypeScript API framework
- #concept — "magnetic type safety": single source of truth spanning the network boundary
- #pattern — architecture-for-context: collocation as a token-cost strategy
- #comparison — the REST / GraphQL / tRPC triangle, each criticized on its weakest axis

## Take

Credible direction, unproven product. The diagnosis is right and increasingly conventional wisdom: scattering a feature across controllers, serializers, and schema files taxes both humans and agents. And the argument against tRPC — that leaking server types into the client couples you to a monolith — is a real architectural objection, not marketing noise. But the page is all assertion and no evidence: no benchmark on token costs, no link to a repo, no numbers on what "dense" code actually saves. The swagger of "The last framework that actually makes sense" from an anonymous team with no visible adoption should be read as a manifesto, not a result. The genuinely interesting claim is also the riskiest: designs optimized for current context windows are optimizations against a moving target — the model that makes Cobalt's collocation free may ship next quarter. Frameworks that bet their architecture on today's model limits have a habit of becoming yesterday's cleverness (see the long history of "APIs designed for the network of 2010"). Still, as a data point it's valuable: here is a framework saying the quiet part out loud — the primary consumer of your codebase architecture is now an LLM.

---

*Sources: [[raw/cobalt27-dev]], [[summary/cobalt27-dev]]*
*Last updated: 2026-11-26*
