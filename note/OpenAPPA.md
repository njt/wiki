# OpenAPPA

OpenAPPA is a deterministic, MIT-licensed guardrail engine that sits outside the agent's prompt and execution loop and enforces information-flow security over every tool call: each agent trajectory carries a monotonically-restricting `audience × trust` label, and every decision is derived algebraically from that label rather than from pattern matching or an LLM judge. The pitch is 100% resistance to prompt-injection-driven data exfiltration, without breaking the agent — because refusals come with machine-readable remedy plans (sanitizers, authorities, disposable subagents).

---

## The three-way argument

The landing page is structured as a demolition of the two incumbent approaches before presenting its own:

1. **LLM judges can't see the data.** Auto-review modes that run a second model over tool calls structurally can't track data flow across calls, and because classifiers are prompt-injectable themselves, harnesses hide tool outputs from them. The judge is blind by design — and probabilistic besides (99.3% at best, which is "a lot of breaches" at millions of calls).
2. **Blacklists break or rot.** Block `rm -rf /` and the model writes a Python one-liner; block `curl` and it pushes to an external git remote. Regex rule sets offer no verifiable coverage: "nobody can verify that it closes every path, or that three mundane tools chained together don't leak."
3. **Flows, not patterns.** Instead of allowed/blocked tool lists, the configuration describes data sources, audiences, trust levels, and authorities. Labels only tighten (reading a private repo narrows the audience; reading an unvetted web page lowers trust), and the engine derives each decision from the label.

> "An injected prompt telling the agent to leak secrets is irrelevant: you cannot prompt-inject an algebra."

That line is the whole thesis: move the security boundary out of the model's reasoning entirely, into a system the model cannot see, negotiate with, or manipulate. Because the configuration is declarative, coverage of the whole tool graph is checkable in CI/CD — a property pattern lists can never have.

## Utility is the hard part, and the remedy plan is the answer

Strict enforcement usually kills the agent: a bare "forbidden" makes it stall, retry, and fail. OpenAPPA instead returns a **machine-readable remedy plan**:

- **Sanitizers** transform the payload (mask secrets, redact PII) so it can flow to a wider audience; stock ones ship in the box, custom ones plug in "with a clear blast radius."
- **Authorities** approve one specific action through a human or internal API — without lifting the session's restrictions for later calls.
- **Subagents** isolate an untrusted read in a disposable branch so the parent trajectory continues unpoisoned.

Claimed benchmark: task completion lifted from 37% to 90% versus strict enforcement. Whether one buys the 100% claim, the shape is what matters: the guardrail is a *negotiator of legal paths*, not a wall.

## Critical take

This is a vendor landing page, and the benchmarks are self-reported with no links to the paper or methodology visible here — "outperforms competitors on benchmarks" plus "NeurIPS-accepted" is marketing-shaped. But the architectural ideas are genuinely interesting and independently respectable:

- The `audience × trust` label is a **taint-tracking / lattice information-flow** model applied to agent trajectories, which is a much older and better-understood theory base than anything an LLM judge rests on. Monotonic label-tightening is exactly the property that makes static guarantees possible.
- Placing the engine **outside the prompt and execution loop** answers the judge-blindness problem correctly: any guardrail the model can perceive is a guardrail the model can reason around.
- The remedy-plan pattern is the most transferable idea in the piece. It reframes the security/utility trade-off: instead of choosing between refusal and permission, the engine offers *transformed* paths. That's a design pattern any harness could steal regardless of whether OpenAPPA is the implementation.

The open question the page doesn't answer: who writes the configuration, and how wrong can it be? "Declarative and auditable" is only as good as the humans encoding sources, audiences, and trust levels — a policy-authoring burden, not an eliminated one.

## Related pages

- [[CaMeL — Defeating Prompt Injections by Design]] — the closest intellectual neighbor: OpenAPPA strengthens CaMeL's thesis that dataflow control beats LLM-judging, arriving at a label-on-trajectory design where CaMeL uses a per-value capability graph; both are "prompt injections are not solved, so control the plumbing" answers.
- [[LLM01 Prompt Injection (OWASP)]] — OpenAPPA is a concrete, maximally-ambitious entry in exactly the mitigation categories OWASP lists (deterministic output validation, least privilege), complicating OWASP's admission that fool-proof prevention may not exist by claiming the 100% number.
- [[Ship Safe]] — the complementary failure direction: Ship Safe audits the agent-facing surface (MCP configs, instruction files, injection surfaces) statically, while OpenAPPA enforces runtime information flow; a complete posture probably wants both.
- [[Fences, not Sandboxes]] — the sharpest philosophical contrast in the wiki: Yegge argues governance must live inside the model's judgment once models are good enough; OpenAPPA bets the opposite, that security must live somewhere the model can never reach.

---
*Sources: [[raw/openappa-com]], [[summary/openappa-com]]*
*Last updated: 2026-10-11*
