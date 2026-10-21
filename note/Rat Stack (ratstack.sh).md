# Rat Stack — Effect + Alchemy as an Agent-First Reference Stack

Rat Stack (rat-stack) is Joel Hooks' open reference application: a full app plus its cloud infrastructure as one typed TypeScript program, on Effect and Alchemy, deployed to Cloudflare. What makes it wiki-worthy is that it is written *for coding agents first* — every page is Markdown by default, an llms.txt agent guide, an MCP server, and session-scoped skills are the primary interface, and the architecture is chosen so an agent "can work longer without help."

---

The stack's four ideas are pieces (service = interface, Layer = construction recipe that may carry cloud resources), trust, reliability, and more useful features. Its distinctive move is what it calls the **fence**: "compiler checks, lint rules, and commit hooks. They reject prohibited code and shortcuts."

> "Make the easy path the right path. The codebase and the compiler stop mistakes that rules and style guides can only ask about. Remove a cloud resource connection and the code that uses it stops compiling."

This is the strongest formulation yet of the guardrails-as-compiler position: rather than *telling* the agent rules (which it may or may not follow), make the violation impossible. It links directly to the [[Fences, not Sandboxes]] argument and to deterministic spine over agentic leaves.

> "Search its rules and skills before you write code, follow its patterns, and tell me when my code breaks them."

The connection prompt is a fascinating inversion of skills management: skills are fetched per-session and explicitly *not* installed ("Do not install it or write it to any skills folder"), keeping the agent's environment clean and the source of truth remote and current — the opposite of the accumulate-and-rot skill folder.

## Key themes

- #tool — Effect, Alchemy, XState, Oxlint, lefthook, Cloudflare (the "CAFE" stack)
- #concept — the fence; contract → capability → projection; cartridges as removable package-per-job units
- #pattern — one capability, five interfaces (CLI, HTTP+OpenAPI, MCP, RPC, sandbox code mode); hexagonal ports and adapters
- #project — a curated, opinionated reference repo meant to be copied and trimmed ("keep or cut" removal checklists)

## Analysis

Rat Stack is the most thorough realized example of "boring, training-data-dense choices" ([[The GUS Stack — Go, Unix, SQLite]]) I've seen applied where the technology isn't boring at all: Effect is a niche, dense type-level library. The bet is that agent-legible *structure* (contracts, projections, Layers) matters more than popular APIs — the agent learns the local dialect from llms.txt rather than from priors. That partially contradicts the GUS Stack thesis and partially extends it: hooks argues you can take exotic-but-principled stacks if you invest in the teaching surface.

The fence idea also sharpens the guardrails debate: style guides *ask*, the compiler *stops*. But note the honest cost ledger — the site ships a debt.md "lint and type escape ledger," conceding the fence is "aspirational" purity. Real fences have gateways, and rat-stack tracks every one of them.

A quieter contribution is the per-session skill pattern. As skill libraries multiply ([[Malicious Agent Skills in the Wild]], [[Claude Code Skills System]]), rat-stack's "fetch and follow, never install" is a concrete supply-chain mitigation: the skill's provenance is one URL, its lifetime is one session.

## Related

- [[The GUS Stack — Go, Unix, SQLite]] — shares the "pick the stack agents already know" logic, but rat-stack answers the exotic-stack case with an agent teaching surface; the two are complementary halves of the same problem.
- [[Fences, not Sandboxes]] — rat-stack's fence is a working implementation of exactly this thesis, pushed all the way into type-level cloud resource checking.
- [[HATEOAS — Hypermedia as the Engine of Application State]] — rat-stack uses HATEOAS response links to guide the next action, a rare production pairing of Effect-style typed contracts with hypermedia agent guidance.
- [[Code Standards That Survive the Next Agent]] — Bertram's "standardize what every change must prove" maps onto the fence: commit hooks and deployment checks are the enforcement half of the same prescription.

---
*Sources: [[raw/ratstack-sh]], [[summary/ratstack-sh]]*
*Last updated: 2026-10-21*
