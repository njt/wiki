# Extensible Software in the Age of LLMs

Jeremy Morrell's argument that LLMs flip the economics of software extensibility: the long tail of per-user needs was always too costly to serve, but "Software for One" is now cheap to author, and the web — the best software distribution system ever built — is where the next generation of user-extensible apps should live. The bottleneck is no longer writing extensions but *safely running them*, and the capability model (narrow references, not ambient I/O) combined with modern sandbox primitives (V8 isolates, microVMs, WASM) is what finally makes a "battle-tested core, endlessly extensible" web app feasible — with Salesforce's two-decade-old Apex platform as proof the model already works at immense scale.

---

## Key Quotes

> "In the past year your users have suddenly acquired the ability to speak code into existence."

The one-line statement of the entire essay. Extensibility used to mean a plugin SDK, a developer account, and a build pipeline. Now it means a user describing a feature to an LLM. Morrell's term for the result — **LLM-native software** — is precise: a core that is battle-tested and accountable, plus extensions that are born from conversation. Pi is his example, but the claim is general.

> "The web is the most successful software distribution system in the world. It shouldn't be left behind."

This is the pivot that distinguishes the essay from a roundup of local extensibility. IDEs, game mods, Blender add-ons, CAD extensions — all the mature extensibility examples are local, professional, high-barrier tools. Morrell's bet is that the *same* extensibility, moved to the web where distribution and multi-tenant data already live, is a category opportunity rather than an incremental feature. The [[Stevey's Google Platforms Rant]] accessibility thesis is the unstated ancestor here.

> "Ask most technologists what Salesforce is and you'll either get a blank stare... it's more accurate to describe Salesforce as a massive multi-tenant programmable platform."

The strongest evidence in the essay, and the reason the idea can't be dismissed as speculative. Salesforce has been safely running customer-authored code (Apex) in response to app events, within transactions, since 2007 — a year after S3 and EC2. Morrell's point is that Salesforce *had* to build a compiler, type system, runtime, and debugger because nothing off-the-shelf existed two decades ago. In 2026 those pieces are commodity. The hard part is no longer the language; it's the boundary.

> "A better way is to hand the untrusted code a narrow capability... If we remove ambient I/O, the code can only take actions via the references it has been passed."

The essay's sharpest technical contribution. The progression is devastating: raw `fetch` with an API key leaks and DoSes; a proxy that swaps opaque tokens is "strictly better" but collapses under the burden of anticipating every request a user might make; the correct shape is to pass the untrusted code a reference to exactly the action it's allowed — `getApprovedEmail()`, not `fetch` plus a token. His IFTTT example is the clean summary: it doesn't give you a Twitter API key, it gives you `twitter.post_new_tweet()`. This is [[Security and Sandboxing]]'s capability principle stated at the application-extension level rather than the agent level.

> "This is essentially the same problem: how can you run logic on behalf of a user that you cannot trust."

The bridge that connects extensible software to agent execution platforms — and justifies the essay's second half. The properties Morrell lists for his primitive (~$0 idle, millisecond cold starts, hard limits, solid isolation, capability-scoped action) are the same checklist anyone running agent code faces. Extensible web software and agent sandboxing are one problem wearing two hats.

## Key Themes

- **#concept Software for One / Small Software** — the long tail of unmet needs, reachable only because LLMs collapsed the cost of authoring a one-person app. Pete Koomen's YC "Small Software" bet is name-dropped as validation. The corollary, tucked in a footnote: participation inequality survives — a small percentage will author most extensions no matter how easy it gets.
- **#pattern LLM-native software** — a stable core plus extension hooks, where the ecosystem (not the vendor) absorbs the long tail. Pi's TypeScript extensions and shareable packages are the model.
- **#pattern Capability-based extensibility** — narrow, explicit references handed to untrusted code instead of ambient I/O and credentials. Object-capability thinking (Cap'n Web) as the general mechanism.
- **#tool Cloudflare Dynamic Workers** — Morrell's pick for the closest production-ready primitive: V8 isolates plus observability, Durable Objects/R2 storage, durable workflows, built-in source control, and hosted LLMs. He discloses working at Cloudflare, which should discount but not dismiss the conclusion.
- **#security sandboxing primitives** — interpreters (Lua/QuickJS), V8 isolates, microVMs (Firecracker, `libkrun`), and WASM+WASI, with the honest note that none are mutually exclusive. [[Wanix — Wasm-Native Unix Sandboxing for the Web]] is a live instance of the WASM pole: a browser-native Unix composed from HTML elements, where Plan 9 namespaces and 9P-over-WebSocket imports do the capability-passing Morrell argues for, expressed as file semantics instead of function references.

## Critical Analysis

**The thesis is right, and mostly not original — but the framing is useful.** "Let users extend your product" is [[Stevey's Google Platforms Rant]] and Salesforce's business model and every plugin ecosystem ever. What Morrell adds is the specific claim that LLMs move the *authoring* cost to ~zero and sandbox primitives move the *deployment* cost to ~zero, and that the two together cross a threshold where extensibility stops being a professional-tool luxury and becomes a consumer-web default. That's a genuinely useful way to read the moment.

**The Salesforce example cuts both ways, and Morrell knows it.** Salesforce could only justify building a whole language because the business problems were valuable enough to hire specialists — which is precisely the opposite of the "anyone can extend it" future he's predicting. He handles this by arguing we no longer *need* the compiler-and-ecosystem build-out, but he doesn't fully confront the second half of Salesforce's economics: the *distribution* of who writes extensions and who just consumes them. A read-it-later app that "anyone" can extend still needs someone to author the `arxiv` summarizer.

**The capability model is the load-bearing insight, and it's under-argued.** The essay's real contribution is that the safe-extensibility problem is best solved by *narrow capabilities*, not by stronger sandboxes or smarter proxies. This aligns with [[The Agent Access Model]] and Cloudflare OS's Gatekeepers, but Morrell's version is more useful because it's product-shaped: it's about what API surface you *hand to your users' extensions*, not what you hand to your own agent. The IFTTT analogy is worth a thousand words.

**The Cloudflare section is the weakest, and he flags it.** A disclosure doesn't neutralize a conflict, and the Dynamic Workers section reads like the essay was written to land there. But the checklist it builds — observability, multi-tenant storage, durable execution, source control, hosted LLMs — is a genuinely good scorecard for *any* extensibility primitive, and it's the thing a non-Cloudflare reader should steal even if they ignore the conclusion.

**What's missing: the client side.** Morrell footnotes that sandboxing custom *UI* (where extensions touch sensitive data in the browser) "deserves its own post" and points at Cloudflare OS. That's the essay's real blind spot — the server-side isolation story is getting solved, but extension points in the UI are where most users will actually meet "extensible software," and that threat model is still wide open.

## Related Pages

- [[Cloudflare OS]] — the internal-corporate-platform use case Morrell explicitly labels "basically Cloudflare OS"; Dynamic Workers are the shared substrate
- [[Security and Sandboxing]] — the capability-vs-raw-fetch argument restates this hub's core principle at the extension layer
- [[Nango — Running Untrusted Customer Code at Scale]] — the same "run untrusted customer code" problem, with a different isolation answer (Firecracker microVMs vs. V8 isolates) for a different workload
- [[Stevey's Google Platforms Rant]] — the platforms-over-products thesis that Morrell's "extensible web software" modernizes for the LLM era
- [[Personal Agents]] — Pi and the local agent ecosystem are Morrell's proof that the extensibility pattern already works, just not yet on the web
- [[celld]] — Deno's self-hosted Durable Objects runtime, named in the essay as one of the non-Cloudflare V8-isolate options

---
*Sources: [[raw/extensible-software-in-the-age-of-llms]], [[summary/extensible-software-in-the-age-of-llms]]*
*Last updated: 2026-08-25*
