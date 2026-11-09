# ContextLayer Slates — A Platform for Bespoke Software

ContextLayer's "slates" are an attempt to build the missing platform layer for vibe-coded personal software: agents write all the code, the stack is fixed, deployment is `git push`, and every piece of I/O and side-effect flows through MCP so the blast radius of untrusted, unreviewed agent-written code stays observable and small.

---

## The argument

Two premises: AI is now spectacular at writing working code, and non-engineers are using it to build tools for their own lives. From these, two design decisions follow almost syllogistically. First, the platform must be **agent-first** — not "agents manage things for you" but "agents are the only ones writing code." Second, since LLMs don't care what language the code is in, the platform can be radically opinionated about the stack: TypeScript, React, React Router, Tailwind, Kysely, "gigatons of Cloudflare," and nothing else. Removing human choice converts it into packaging and composability.

Security is framed plainly rather than bureaucratically: *I feel more comfortable with vibe-coding when I know that the blast radius is small.* Slates run on Cloudflare Workers for Platforms; all egress except authorized MCPs is blocked; browser code is CSP-locked; code inside the slate is untrusted and never reviewed. Platform services (git, auth, migrations, Composio's 1,400+ integrations) absorb everything a bespoke tool shouldn't rebuild.

The wackiest and most interesting idea is the identity inversion: slates never learn who the logged-in user is. Resources are injected at runtime and declared as belonging to the *viewer* or the *owner*, so one email slate reads your inbox when you open it and mine when I do. Future bets: loaders/actions auto-exposed as MCP tools, forking as the default consumption mode, and agents attached to the live React state of every open browser session.

---

## Key quotes

> "If agents really is going to delete all of my files, there should be only one way for it to do so, because if there's only one way, I can observe this way. I can monitor it, constrain it, and keep it under control."

The best articulation I've seen of why *narrow* is a security feature rather than a limitation. Observability-of-the-one-path is the whole argument: capability by chokepoint, not by policy.

> "Code inside the slate is considered to be untrusted. I want to treat it like a black box. I don't want to have to review code."

This quietly accepts the review-bottleneck thesis — that human review of agent output doesn't scale — and answers it with containment instead of review. Most of the wiki's verification literature tries to make review cheaper; slates try to make review unnecessary.

> "We can be super opinionated about the stack... If agents are the only ones that are writing code, then it doesn't matter what language or framework the code is written in. LLMs don't care."

The same logic as the [[The GUS Stack — Go, Unix, SQLite]] prescription, taken to its endpoint: training-data-dense, boring choices, enforced by the platform rather than chosen per project.

> "We're putting identity, data, apps, and agents into the same environment... Give us a little bit of time and we're well on our way to rethinking operating systems entirely."

Grandiose, but structurally fair: viewer-owned runtime-injected resources genuinely dissolve the multi-user app pattern, and per-user resources inside one codebase is an OS-shaped claim.

## Key themes

#concept #tool #pattern — blast-radius minimization as a security architecture; MCP as the single I/O boundary; stack monoculture as a consequence of agent-only authorship; viewer/owner resource injection replacing identity.

## Critical take

The strongest part is the honest asymmetry the author names himself: ContextLayer is supremely engineered, but the software built on it should be "supremely valuable" instead. That's the correct inversion for personal tools and matches the home-cooked-meals lineage. The weak points: "agents are the only ones writing code" is aspirational (someone still debugs the platform's own SDK at 2am), and the MCP-only egress boundary is only as trustworthy as the MCP servers themselves — a compromised or over-broad integration is the new blast radius. Also unaddressed: what "untrusted" means for *data* — the viewer/owner injection model is elegant for isolation but leaves the classic confused-deputy question open when resources span viewers. Still, as a design essay this is one of the clearest statements of what a platform for agent-authored personal software should absorb versus expose.

## Relations

- [[Ask for What You Want (Robin Sloan)]] — the same home-cooked-meals ethos ("supremely valuable" over "supremely engineered"), but Sloan never looks at the code while slates assume nobody reads it either; ContextLayer is the platform-side answer to Sloan's platform-less program.
- [[A Deep Dive on Agent Sandboxes]] — strengthens it with the concrete claim that sandboxing-by-egress-chokepoint (MCP-only) is an instance of the containment patterns that survey catalogs.
- [[Authenticating MCPs]] — complicates it: making MCP the *only* boundary concentrates all trust in MCP auth, exactly the problem that page's identity/delegation machinery exists to solve.
- [[Mainbrella Build — Lovable-Style App Builder on Cloudflare Inference]] — a sibling project betting on the same Cloudflare substrate for agent-built apps, but aimed at product builders rather than personal tool owners.

---
*Sources: [[raw/a-platform-for-bespoke-software]], [[summary/a-platform-for-bespoke-software]]*
*Last updated: 2026-11-09*
