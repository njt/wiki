# Alphamaxxing (bobbytables)

A founder's field report on starting a project in the agent era: define the gRPC/protobuf API contract first, let Claude Code throw away two sacrificial prototypes against it (Go, then Rails), then hand-write the production implementation over three months — not out of nostalgia, but because a hand-built foundation is the context that makes agents fast afterwards. The sharpest claim: agents are now the predominant *users* of software, so the API is the product.

---

## Key Quotes

> "AI is not only building our software, **it's also becoming the predominant user of our software.**"

The thesis of the piece, and the frame that justifies everything else. If Claude is a persona in your addressable market, a bad programmatic interface shrinks your TAM even when the human-facing UI is fine. This is [[AI Product and Business]] reasoning smuggled into a workflow essay.

> "Farmers don't name their livestock. You should not treat any of your code as long-for-this-world, either."

The "sacrificial code" ethic, stated memorably. Two prototypes, two `rm -rf`'s, a few days to value — code as an experiment instrument, not an heirloom. Note the inversion of [[The Cost YAGNI Was Never About]]: Beck argues cheap generation amplifies over-building; here cheap generation is deliberately spent on throwaways that buy architectural information.

> "I wasn't building features so much as I was writing context."

The hinge of the whole essay. Three months of hand-writing packages, migrations, and Envoy config is re-framed as feeding future agent sessions: "My prompts for new features were rooted in examples, not half-baked 'now make it do this.'" This is the strongest first-person evidence yet that a codebase's *style* is agent infrastructure — the same intuition as [[The GUS Stack — Go, Unix, SQLite]] (boring, training-data-dense stacks) but argued from the founder's chair rather than the stack list.

> "I became Google Maps for coding agents in my codebase."

The steering metaphor: the human routes, the agent drives. Compare the lead-singer coda ("I'm the lead singer... they'll pick the tempo and volume. I can't get upset if it's not what I wanted") — an unusually honest admission that once you delegate tempo, you accept some drift.

## Key Themes

#concept — API-as-product in an agent-consumed world
#pattern — spec/contract first, disposable prototypes, then hand-rolled foundations
#tool — Claude Code, grpcurl, Protocol Buffers/gRPC, Zed

## Analysis

The essay's structure is its argument: contract → prototype → discard → prototype again → hand-build. That staging is what makes it more than vibe-coding nostalgia. The author isn't rejecting agents; he's locating the human contribution at exactly two points — the API contract (where taste lives) and the foundation (where context is manufactured) — and happily delegating everything between.

Worth noting what the essay doesn't say: three months of hand-writing is a luxury of a funded solo founder, and "I felt good about it" is doing real work as a motivation. The claim that hand-rolled foundations buy hours-not-days shipping later is plausible but unquantified — this is a testimonial, not a measurement like [[Code Cleanliness and Coding Agents]]. Still, as a counterpoint to both dark-factory maximalism and "coding is solved" triumphalism, it lands: the durable artifacts here are the contract and the exemplars, which is precisely the position of [[Specsmaxxing]] — if code is disposable, what humans write by hand is the thing agents can't invent for you.

## Relations

- [[Specsmaxxing]] — independently argues the same conclusion from the spec side: the human-written artifact worth maintaining is the one that encodes taste. Alphamaxxing is the API-contract variant of that thesis, with `rm -rf` as the proof of seriousness.
- [[The GUS Stack — Go, Unix, SQLite]] — Zoschke picks training-data-dense stacks so agents write idiomatic code; bobbytables hand-builds the exemplars so agents imitate *his* idioms. Same goal (agent-legible foundations), opposite mechanisms — which suggests the underlying variable is how much your taste deviates from the training distribution.
- [[The Cost YAGNI Was Never About]] — Beck worries cheap generation amplifies over-building; this essay shows the flip side, cheap generation as a deliberately disposable prototyping instrument that de-risks the expensive hand-written phase.
- [[SDD Case Study — 13 Apps in 70 Days]] — Fontoura's "correct the spec, not the code" matches the contract-first move here, but Alphamaxxing goes further: after the spec, it invests three months in hand-written context that pure spec-driven workflows don't account for.

---
*Sources: [[raw/alphamaxxing]], [[summary/alphamaxxing]]*
*Last updated: 2026-11-09*
