# Socio-technical Systems in the AI Era (Hila Fox, DDD Europe 2026)

Hila Fox — a principal-engineer-turned-PM at Kodo, an AI code review company —
argues that AI agents are not tools but new socio-technical actors, and that
the dominant failure mode of enterprise agent adoption is "mirroring":
automating existing (often dysfunctional) processes rather than redesigning
them. Context becomes the new system building block, and poorly designed
context boundaries will give us "context hell" the way data lakes did.

---

## Key claims

1. **Agents are socio-technical actors, not tools.** They are artifact
   *and* reasoning — they make decisions instead of humans, so they don't
   fit the old human/non-human split of socio-technical systems theory.
2. **We are automating our organizational illnesses.** The path of least
   resistance is to build agents for the processes you already have —
   including the toxic ones. Result: agent silos, competing agents from
   non-collaborating VPs, power dynamics ("my agent is better than yours
   because I outrank you"), and Tom's approval loop hard-coded into the
   agent's instructions.
3. **Context is queen, the new building block.** The old stack (data / APIs
   / orchestration) maps onto the new one (context / MCP / agent-to-agent /
   workflows). Coupling analysis transfers directly: shared data is tight
   coupling, APIs are loose, complex shared workflows are the worst kind.
4. **Poorly designed context boundaries lead to context hell.** Temporal vs
   semantic context maps to session / personal / team / project /
   organizational layers — mostly tribal knowledge with no clean exposure
   path. Large contexts bring lost-in-the-middle, drift, rot, and
   needle-in-haystack failures, which users flatten into "hallucinations."
5. **Everything is changing; be flexible.** Command→intent,
   sequential→parallel, micromanagement→oversight. Dark factories (fully
   autonomous software teams) are emerging; oversight becomes the craft.
   Her closing formula: **defend** (old-world architecture discipline),
   **evolve** (skills, human/agent role boundaries), **redesign** (org
   structure around context coupling).

## Key quotes

> "We are automating our organizational illnesses into our agents... it's
> not the agent's fault in the end, it's our fault because we told them to
> go to Tom and ask Tom to give an approval before they can move on."

The sharpest line in the talk, and the one most missing from agent-architecture
discourse. Agents inherit Conway's Law through their instructions and context;
blaming model quality for what is really process debt is category error.

> "95% automation isn't good enough, because if it's 95% automation it's
> not an automation" (quoting Kat Wu, head of product at Claude Code).

The last 5% is precisely the human sensitivities — Karen and David blocking
work, Tom's ego — that can't be encoded without writing down things companies
won't admit in writing.

> "Agents properly built will uncover the strong couplings between the
> teams."

A genuinely optimistic inversion of claim 2: the hard work of curating
context forces the organization to see its own real communication structure.
Agents as an org-diagnosis instrument, not just automation.

> "Context, when you think about them in the context of workflows, they are
> both organizational memory, but also their decision rationale."

Because the agent holds the if/else, the context you give it *is* the spec —
connects directly to specifications-as-the-product thinking.

## Themes

- **#concept** — Socio-technical actors: the artifact/reasoning hybrid that
  breaks the human-vs-tool taxonomy.
- **#concept** — Mirroring: agents as photocopies of existing process, org
  illnesses included.
- **#pattern** — Context as building block: temporal vs semantic context
  layered session→team→org; MCP and A2A as decoupling protocols.
- **#person** — Hila Fox: DDD background bridging into the AI era; unique
  position of building AI review tooling while using it internally.

## Critical take

The talk's strength is that it says out loud what enterprise agent pitch
decks suppress: most agent deployments are process debt with a model in
front of it. The Conway's-law framing is honest and the Kodo war stories
(duplicate IDE + CLI agents, then a re-architecture into context-collection /
issue-finding / compliance / judge agents) ground the abstraction.

The weakness is that it stays at the level of "take a step back and ask if
the process is right" — the too-wide, too-company-specific problem she
herself flags and then defers. There is no method offered for *deciding*
which organizational illness to fix versus encode, and the "be flexible"
close is motivational rather than operational. The dark-factory section
presents it as an uncomfortable novelty without engaging the strongest
counterargument in the literature (verification cost and comprehension debt).

## Related pages

- [[Agentic Software Engineering (Hassan)]] — strengthens this source's
  actor-not-tool reframe from the book-length angle; Fox gives it an
  organizational mirror (org illnesses as agent pathologies).
- [[The Mythical Agent-Month]] — complements Fox's "we are automating
  organizational illnesses" with the cost/estimation side of putting agents
  into human-shaped organizations.
- [[Architecture Is Designing Knowledge Flow]] — nuance and complication:
  Fox's context-as-building-block claim is that claim applied at org scale,
  with coupling analysis transplanted from services to context boundaries.
- [[Agent Memory and Context]] — Fox's temporal/semantic split and
  session→team→org layering slot directly into that topic's memory
  taxonomies, with lost-in-the-middle and context drift as failure modes.

---
*Sources: [[raw/socio-technical-systems-in-the-ai-era-by-hila-fox-ddd-europe-2026]], [[summary/socio-technical-systems-in-the-ai-era-by-hila-fox-ddd-europe-2026]]*
*Last updated: 2026-10-02*
