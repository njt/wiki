# Manifest-Driven Development

CL Kao's proposal for the next paradigm in software development: ask coding agents to **assume functionality already exists, then gradually materialize it.** Not vibe coding, not spec-driven development — something fundamentally different that emerges when the system itself can "fake it until it works."

> "we can ask coding agents to assume some functionality exists already, and gradually materialize the deterministic parts and fix the quirks along the way."

The worked example: Kao asked Pi coding agent to pretend Spacedock already worked natively on Pi, filed the 16 things that broke, then launched a sprint to make them real. The agent first spiked to test Pi's underlying capability, then built the missing functionality and added CI. What's wild is that the "pretend" session *revealed the spec* through actual usage rather than upfront design.

> "Test-driven and spec-driven are means to the end of describing what we want at different levels of detail. Manifest-driven development is like spec'ing out in a few iterations of actual usage."

This inverts the traditional sequence. Instead of spec → build → test, you build → discover gaps → materialize. The discovery happens through friction, not foresight.

> "The most similar realization I had was when git chose its content-addressable storage backend. 'Oh, this means we have a way to point to all the code that can potentially be written (and maybe it already is, in a parallel universe, and just needs someone in this universe to manifest it).'"

Kao identifies three missing pieces needed to make this a real paradigm: a **durable spec format** so agents know what to assume, an **agentic-native workflow engine** for encoding user intents and autonomy levels, and a **rich UI** fast enough for just-in-time "fake it until it works" — malleable interfaces for human/agent interaction.

> "This is no longer no-code or low-code. This is mostly no-code, and just-in-time code when needed."

## Key themes

- **#paradigm** — Manifest-driven as the successor to test-driven and spec-driven, enabled by agents that can operationalize assumptions
- **#tool** — [[Pi Coding Agent]], [[PiClaw]] — the agent platforms where this pattern emerged
- **#pattern** — "Fake it until it works" as a legitimate engineering strategy when the system can self-repair
- **#concept** — Durable spec format, agentic-native workflow engines, just-in-time code — the infrastructure this paradigm needs

## Critical analysis

This might be the most important blog post about AI-native development that nobody has read yet. The idea is genuinely new — not a refinement of existing paradigms, but a phase change. Kao stumbled onto it because he's building an agent orchestration platform (Spacedock) that forces him to work *with* the system rather than *on* it.

The "manifest" framing is provocative and worth taking seriously. It's not "describe what you want" (spec-driven) or "explore until it works" (vibe coding). It's "act as if it already works and let the gaps declare themselves." This is how experienced engineers actually think — they hold the desired system in their head and let the implementation catch up. The difference is that now the implementation *can* catch up, autonomously.

However: the "mostly no-code, just-in-time code when needed" claim is doing a lot of work. Kao's example required a working agent platform (Spacedock + Pi), deep knowledge of both systems, and the ability to spike and debug. This is a **hyper-multiplier for the experienced**, as he admits — not a democratization play. The durable spec format and workflow engine he calls for don't exist yet. Manifest-driven development is a vision, not a practice you can adopt today.

The comparison to git's content-addressable storage is the essay's most interesting move. Git made it possible to point to code by its hash before it existed on your machine — "this exists somewhere in the space of all possible hashes; go get it." Manifest-driven development extends that metaphor to functionality: "this feature exists somewhere in the space of all possible agent behaviors; let's inhabit it."

That's either profound or woo. The difference depends entirely on whether the three missing pieces get built.

## See also

- [[Specifications as the Product]] — specs as durable artifacts; manifest-driven is the next step: specs as emergent from usage
- [[SDDW (Spec-Driven Development Workflow)]] — the most mature spec-driven pipeline; what manifest-driven would replace
- [[Agent Coding Workflow]] — the meta-practice this paradigm extends
- [[The Oracle Is the Asset]] — Sam Ruby's inversion; manifest-driven inverts further: the *session* is the spec
- [[Intent Is the Interface]] — user intents as the design primitive
- [[Vibe Coding as a Team Sport]] — another attempt to name what comes after vibe coding
- [[Building When It Feels Like There's Nothing Left to Build]] — the existential question manifest-driven answers: build to discover what to build
- [[The Dark Factory is a DOT File]] — the pipeline artifact as durable; contrast with manifest-driven where the session IS the artifact
- [[Automating Myself Out of Development]] — the human connector role that drove Spacedock's creation
- [[Pi Coding Agent]] — the agent used in the worked example
- [[Smart Models Dumb Pipes]] — the end-to-end principle underlying agentic-native design
- [[Agent-Native Architectures (Every)]] — the design principles for the platforms this needs
- [[Guardrails and Feedback Loops]] — the "fix the quirks along the way" infrastructure

---
*Source: [Manifest-Driven Development](https://spacedock.md/blog/manifest-driven-development/) by CL Kao, spacedock.md, 2026-06-25. Fetched 2026-07-05 via surf (browser automation).*
