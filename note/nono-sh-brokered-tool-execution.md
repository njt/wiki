# nono.sh — Brokered Tool Execution for Agent Security

nono is a commercial agent-security runtime that brokers every tool call an agent makes: a supervisor verifies the executable, checks argv policy, and spins up a per-invocation micro-sandbox with only the capabilities that single call needs — read-only workspace, no host secrets, network only through a proxy that injects credentials at the boundary. Human approval gates sit between policy layers, and every decision lands in a Merkle-sealed audit trail.

---

This is a product landing page, not an essay, but the architecture it describes is one of the most concrete articulations of the "the agent never touches the real credential" pattern in the wiki. The phantom-token design — the sandbox sees a `GH_TOKEN` the proxy recognises and swaps at the boundary — is the least-privilege principle carried all the way down to per-invocation granularity.

Four scenarios show where enforcement lives, and the layering is the interesting part:

> "Argv denied — `gh issue comment` is refused at argv authorization; no sandbox or outbound request is ever created."

Denial before sandbox creation means the cheap checks run first; expensive isolation is only provisioned for calls that passed cheap ones. And the L7 case shows why argv policy alone isn't enough: `gh api` passes a broad argv rule but the forbidden POST is caught at the proxy. Two enforcement planes for one tool.

> "The sandbox receives a phantom GH_TOKEN; the real credential stays with the supervisor and never enters the child."

This is the strongest single idea on the page. Compare it to the wiki's other sandboxing sources: most sandbox the *filesystem* or *network* around a process that still holds real secrets. nono makes credential exfiltration structurally impossible rather than merely hard, which answers a different threat model — one where the prompt-injected agent is actively trying to leak.

> "If an action falls outside the provided policy, the policy engine can route it to a human approval decision to approve, deny, or time out."

The human-in-the-loop gate is policy-triggered, not blanket — `gh pr merge` pauses, `gh issue view` doesn't. That's the same "human when cheaper, deterministic when possible" split argued across the wiki, implemented as a product feature.

## Critical take

The page is marketing, and the claims that matter are unfalsifiable from it: "zero latency", "zero setup", and "engineers at some of the world's largest tech companies run nono" are assertions, not evidence. The audit trail (hash-chained, Merkle-sealed) is the most verifiable claim in principle, but the page never shows the tooling. Still, the architecture is coherent and unusually specific for a landing page — it reads like a design doc with a signup button.

The notable tension: nono is emphatically a sandbox, and the strongest sandboxing yet ingested here — while [[Fences, not Sandboxes]] argues the opposite thesis, that permission fences beat containment. nono quietly reconciles the two: the fence (argv policy, proxy path policy, human approval) decides *whether* a call happens, and the sandbox is what the fence lets through runs inside. Fence and sandbox as layers, not rivals.

**Key themes:** #tool #pattern #concept

**Related:**

- [[Fences, not Sandboxes]] — nono complicates this page's fence-over-sandbox thesis: it ships both, with policy fences deciding what runs and ephemeral micro-sandboxes containing what runs, suggesting the dichotomy dissolves at per-invocation granularity.
- [[How We Contain Claude]] — same problem (containing a coding agent's tool calls) solved at the opposite scale: Anthropic's in-house containment vs. nono's commercial brokered runtime; nono's phantom-token credential injection is the piece most containment setups lack.
- [[AI Agents Need Their Own Identity and Least-Privilege Access]] — nono operationalises this page's least-privilege argument down to a single tool invocation, with credentials that exist only at the supervisor/proxy boundary.
- [[The Agent Access Model]] — nono's supervisor/proxy split is a concrete implementation of access mediation: the agent holds only phantom credentials and every capability decision is auditable at one choke point.

---
*Sources: [[raw/nono-sh]], [[summary/nono-sh]]*
*Last updated: 2026-10-18*
