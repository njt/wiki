# Standard Webhooks

An open-source specification and tooling initiative to make webhooks consistent, secure, and reliable across providers — tackling the fragmentation that makes every webhook integration a bespoke engineering effort.

---

## Key Quotes

> "every webhooks provider implements them differently and with varying quality. This makes it hard for providers who need to reinvent the wheel every time and repeat the same costly mistakes, and annoying for consumers who need to have a different implementation for each provider."

The ur-problem of webhooks in two sentences. The fragmentation isn't a minor inconvenience — it's an ecosystem-level tax. Providers burn cycles on solved problems (signature schemes, retry logic, SSRF protection) while consumers maintain N different webhook handlers for N SaaS integrations. The same dynamic played out with REST APIs before OpenAPI standardized description; the question is whether webhooks are similar enough to benefit from the same treatment.

> "It's also holding back the ecosystem as a whole, as these incompatibilities mean that no tools are being built to help senders send, consumers consume, and for everyone to innovate on top."

This is the standardization pitch in its strongest form: the value isn't just consistency, it's unlocking a new layer of the stack. Standardized webhooks make shared tooling viable — webhook debuggers, delivery monitors, testing harnesses, schema validators. The same argument won for OpenAPI: once every API described itself the same way, an entire ecosystem of tooling (Swagger UI, Postman, code generators) became possible.

> "Every webhook implementation needs to protect themselves and their users from SSRF, spoofing, and replay attacks."

Security as a first-class concern, not an afterthought bolted onto each implementation. The three named threats — SSRF (where an attacker tricks the sender into making requests to internal systems), spoofing (forged webhook payloads), and replay (captured-and-replayed legitimate payloads) — are the universal webhook threat model. Standardizing their defenses means each implementer doesn't have to independently discover and fix the same classes of vulnerability.

> "Standard webhooks is designed by experts with extensive experience building reliable and scalable webhook services."

The authority claim. The TSC model — community-driven with expert governance — is the same pattern used by OpenAPI, Kubernetes, and other successful infrastructure standards. Whether the expertise translates into adoption is the open question.

## Key Themes

#specification #tool #pattern #webhooks #api-design #standardization

**Standardization as ecosystem unlock.** The core thesis: webhooks are popular enough to justify shared infrastructure but fragmented enough to prevent it from being built. Standard Webhooks aims to be the OpenAPI of webhooks — a common format that makes the ecosystem bigger than any single provider's implementation.

**Security by default.** Rather than documenting threats and leaving mitigation to implementers, the spec bakes SSRF protection, signature verification, and replay prevention into the protocol. This is the difference between a security *guide* and a security *contract*.

**Open-source with TSC governance.** The community-driven model with a Technical Steering Committee mirrors successful infrastructure standards. It signals that this isn't a single-company play — it's an attempt at industry coordination.

**Reference libraries over reference docs.** Shipping working implementations alongside the spec is a practical concession: most teams won't read a specification, but they will import a library. The SDKs are the spec's adoption vector.

## Critical Analysis

**The page is a landing page, not a specification.** What we have here is a marketing surface for the project — value propositions, target audiences, high-level feature categories. The actual specification, library APIs, and tool documentation live elsewhere. This makes the page useful for understanding the project's ambitions but thin for evaluating its technical choices. A proper analysis would require reading the spec itself.

**The standardization argument is correct but the adoption question is everything.** Webhooks absolutely suffer from the fragmentation problem the page describes. Every Stripe webhook handler is different from every GitHub webhook handler, and the differences aren't expressive — they're accidental. But standardization only helps if it's adopted, and adoption requires either network effects (enough providers implement it that consumers demand it) or a killer app (a tool so useful that providers implement the standard to get access to it). The page doesn't address adoption strategy.

**The security framing is the strongest part.** SSRF, spoofing, and replay are the right three threats to name, and baking defenses into the spec rather than documenting best practices is the right call. The security track record of ad-hoc webhook implementations is poor — signature verification is often optional, replay protection is rare, and SSRF is frequently overlooked entirely. A spec that makes these non-negotiable would meaningfully raise the floor.

**Relationship to the Tricot thesis.** [[Event-Driven vs Polling Architectures]] argues that webhooks alone are a production trap — you need reconciliation backstops, structural idempotency keys, and per-source delivery contracts. Standard Webhooks doesn't contradict this; it operates one layer up. Tricot is saying "webhooks have inherent reliability limits regardless of implementation quality." Standard Webhooks is saying "given that you're going to use webhooks, at least implement them consistently and securely." Both can be true. A standardized webhook is still a webhook — it still loses events, arrives out of order, and needs a reconciliation backstop. But a *standardized* webhook is easier to build that backstop for, because every provider speaks the same protocol.

**The "reverse API" framing is worth pondering.** The page describes webhooks as "a sort of a reverse API" — where the client initiates API calls, the service initiates webhooks. This framing is common but undersells the asymmetry. APIs are synchronous request-response by default; webhooks are asynchronous fire-and-forget by default. The delivery contract is fundamentally different. Treating webhooks as "just APIs in the other direction" papers over the reliability gap that Tricot's piece is entirely about.

**What's conspicuously absent:** No mention of delivery guarantees (at-least-once? exactly-once?), retry semantics, ordering, or batching. These are the hard problems in webhook infrastructure, and the landing page doesn't hint at how the spec addresses them. The reference libraries presumably implement specific choices, but the marketing page is silent.

**The ecosystem play is the real bet.** Standard Webhooks is less interesting as a specification (most of the individual pieces — HMAC signatures, idempotency keys, structured payloads — are well-understood) than as a coordination mechanism. The value isn't in inventing new webhook primitives; it's in getting enough providers to agree on the same ones. This is a governance and adoption problem, not a technical one. The same was true of OpenAPI, and it took a decade.

## See Also

- [[Event-Driven vs Polling Architectures]] — Tricot's definitive argument that webhooks alone are a production trap; Standard Webhooks operates one layer up, standardizing the webhook itself
- [[Your Backend Is Full of Hidden Workflows]] — webhooks are one of the coordination mechanisms that accrete invisibly; standardization makes them visible and consistent
- [[All Your Agents Are Going Async]] — webhooks are the canonical async notification mechanism; Standard Webhooks tries to bring consistency to the transport
- [[HTTP API Design Guide (Heroku)]] — the ur-text of API design conventions; Standard Webhooks attempts the same role for the webhook side of the API surface
- [[Software Engineering Craft]] — hub page for API design, reliability, and the fundamentals that standardization efforts serve

---

*Sources: [[raw/standardwebhooks-com]], [[summary/standardwebhooks-com]]*
*Last updated: 2026-08-06*
