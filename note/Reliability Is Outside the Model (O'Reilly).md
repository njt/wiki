# Reliability Is Outside the Model (O'Reilly)

This essay argues that agent reliability in production is achieved by architecture, not by better models: put every consequential decision in a layer that can enforce it. The model proposes; a policy service decides; the execution layer acts within those limits — and alongside that, context is treated as untrusted input, authority is granted as narrow expiring capabilities, stopping is a runtime behavior, recovery uses checkpoints and idempotency keys, evidence is kept in queryable logs and context graphs, and all of it is kept portable by enforcing authorization at the MCP gateway.

---

The strongest thing here is the opening example, because it kills the most common instinct in the field at a stroke. An agent misreads a ticket, updates the wrong account, and the CRM reports success — the API "successfully completed the authorized request, but because it had no way to know which account the customer approved versus what was intended, it saw no error." The diagnosis is precise:

> The model did exactly what the surrounding system allowed because the system treated the agent's decision as the final authority. Prompt tuning and stronger models can improve the agent's responses, but can't provide guarantees for actions.

That last clause is the whole thesis. Teams reach for prompt engineering or a frontier upgrade because it's cheaper than redesigning the authorization path, but "guarantee" and "model" don't belong in the same sentence. The fix is a division of labor: "The model interprets the request, while deterministic software enforces the conditions that must always hold." This is the same model-as-untrusted-component stance as [[Deterministic When Possible, Probabilistic When Necessary, Human When Cheaper]] and the deterministic-hooks half of [[The AI-Native SDLC Playbook]] — here applied to runtime authorization rather than to the dev loop.

On context, the essay refuses the comforting middle ground: stored instructions, skills, and memory written by *other agents* are all attack surface, and warnings alone are weak. Citing Shub Argha's "Trustworthy Context Is Untrusted By Default":

> a trust preamble reduced one class of context contamination from 88.8 percent to 33.3 percent. The remaining failures show why a warning should sit alongside technical checks.

A two-thirds reduction sounds impressive until you notice the residual third is exactly the tail that matters in production. Provenance, version-pinning, and risk labels on context — "who wrote the context and when" — is the concrete prescription, and it complements the injection-craft literature like [[Advanced Prompt Injection Techniques]] by attacking the trust model rather than the payload.

On authority, the essay's most useful contribution is the narrow capability as a *product* of approval: not a broad OAuth write scope, but "one update to the shipping address on account 123," expiring after use. This shrinks prompt-injection blast radius ("the agent may still request the wrong action, but the runtime can reject anything outside the capability") *and* makes audits legible, since the record contains the exact authority granted. That connects directly to the capability and authorization work in [[Enterprise-Managed MCP Authorization]] and [[Authenticating MCPs]], and to the human-side boundary question in [[You Shall Not Pass — Where Developers Draw the Line on AI Autonomy]]: that survey says the median developer is at L3 (AI produces, human approves) — this essay is the runtime machinery that makes L3 enforceable rather than aspirational.

The remaining patterns are quieter but load-bearing. Stopping as normal operation (confidence-threshold routing to humans, human replies canceling pending agent output, kill switches that work mid-plan) is the least-discussed pattern here and arguably the most neglected in practice. Recovery gets textbook-but-correct treatment — confirmed-step checkpoints, idempotency keys, sandboxes that hibernate and wake correctly — which pairs naturally with [[Idempotency Is Easy Until the Second Request Is Different]]. And the evidence section's context graphs (tool call → governing policy → source context → outcome) double as memory for later agents, echoing [[Agentic Context Management]].

## Take

Opinionatedly: this is a checklist disguised as an essay, and that's a compliment. Its weakness is that every pattern is described in one idealized paragraph with no migration story — retrofitting narrow capabilities and policy services onto an existing agent product is the actual hard problem, and the essay doesn't touch it. It's also striking how much of this is *old* distributed-systems wisdom (idempotency, audit trails, least privilege) wearing agent clothes; the genuinely new claim is that stored agent context needs the same provenance discipline as untrusted network input. Where the essay is thin is on who pays the latency and engineering cost of a policy service in front of every tool call. Still, as a counterweight to the "stronger model fixes it" reflex, this is one of the cleaner statements of the position.

## Key themes

- #concept — model proposes, deterministic layers decide and enforce
- #pattern — narrow expiring capabilities; provenance-labelled context; stopping as runtime behavior
- #tool — MCP gateways as the portable enforcement point

---
*Sources: [[raw/how-to-build-reliable-ai-agent-systems-for-production]], [[summary/how-to-build-reliable-ai-agent-systems-for-production]]*
*Last updated: 2026-10-11*
