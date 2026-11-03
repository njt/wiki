# CAVE — Compressed Atomic Verb Expressions

CAVE is Mirek Rusin's plain-text knowledge store: claims are one line each (`subject VERB object`, plus source, confidence, comment), kept append-only in a local SQLite file, queried across chains and inverses, extended by rules, and written by LLM agents through an MCP server. The site is landing page, tutorial, and documentation in one, with every command's output from a real run.

---

CAVE's model rests on five ideas: one line is one claim; belief only grows (append-only, retract by appending `@ 0%`); sources keep their own voice (contradictions coexist until `--resolve`); time is part of the value (`--as-of` for when the store learned something, `--at` for when in the world it holds); and rules derive what nobody wrote, stamping conclusions with `BECAUSE`/`VIA` lineage so losing a premise retracts the conclusion.

The agent story is not an add-on. `cave ingest` hands documents to a headless Claude Code and records what it writes, linted and digested so re-runs only read what changed — with line-level `@src:` spans and honest hedges like a 60% on a draft regulation. Actions make writes governed: preconditions checked against current belief, exposed to agents as named `act_<name>` MCP tools, so the agent gets a vocabulary rather than free-form appends. Automations close the loop: news arrives, rules derive pressure, an overweight position needs review, and the morning brief cites every claim it asserts.

---

## Key quotes

> "A contradiction is two claims with different sources, not an error."

This is the line that separates CAVE from every "single source of truth" system. Knowledge about the world is contested by design; the store keeps every voice and ranks them only when asked.

> "When alice leaves, you do not edit anything: you append the same claim at 0%."

Retraction-as-append is the temporal database move applied to belief. It buys `--as-of` queries — what did I believe on July 1st — at the cost of never being able to delete a mistake. The spec is honest about this: "there is no claim-level delete, so do not ingest secrets."

> "An agent gets a vocabulary of allowed writes instead of free-form appends."

Actions as governed writes is the most agent-native idea here. Precondition-checked named operations become MCP tools; `flag-review` literally cannot be talked into firing when the news did not actually pressure the company down. It is permissions expressed in the domain language rather than the tool layer.

> "The conclusion is stamped @src:rule/… with BECAUSE edges to its premises; lose a premise and it is retracted."

Derived claims are auditable all the way down — the review request traces to the pressure, the pressure to a sentence in an article, the sentence to a file and line. Reports render those citations as footnotes.

## Key themes

#concept #tool #pattern — plain-text append-only belief store; #agent — LLMs as ingest and write paths; #local-first — one SQLite file, browser playground, CC0.

## Analysis

CAVE is the rare tool whose marketing and its spec agree. The append-only belief model is genuinely well-executed: two clocks (when learned, when true), confidence multiplication through rules, contradiction coexistence with a resolution policy. The tutorial's honesty ("LLM output varies run to run; this is one real run") is the right register for a tool whose pitch is epistemics.

What's most interesting to this wiki is the governed-write pattern. Most agent-integration stories hand the model a generic write tool and hope; CAVE instead pre-declares the only ways a decision may be recorded, with preconditions evaluated against current belief. That's guardrails at the ontology level, not the prompt level — and it composes with the citation machinery so any agent-written fact can be interrogated.

The weakness is the usual one for claim languages: the world is not triples. `DRIVES ... #sign:-1` for market exposure works in the demo; whether ordinary practitioners will model their messy domains in uppercase verbs is the open question. The tutorial's own monorepo example is the strongest case — that one feels like something a team could actually maintain. The "no delete means no secrets" caveat also quietly concedes the model's cost.

---

This source strengthens [[Long Live Systems of Record]] with a concrete, working design for an append-only local system of record whose every fact is cited — the same thesis, but with retraction, confidence, and lineage spelled out. It complicates [[Your Knowledge Graph Is Making Your Agent Dumber]] by showing a graph that is deliberately dumb about winners (contradictions coexist by default) and derives only what rules conclude, with provenance attached. It nuances [[Lemmalog — LLM Memory as Program Analysis]]'s memory-as-lineage take with a shippable implementation where LLM-written memory carries sentence-level source spans and confidence. And it gives [[KTX Context Layer for Data Agents]] a sibling pattern: a self-describing MCP server over a local store, with agent writes mediated by named, precondition-checked actions rather than raw appends.

---
*Sources: [[raw/cave]], [[summary/cave]]*
*Last updated: 2026-11-03*
