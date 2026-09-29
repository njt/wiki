# Coding Is Not Solved — Alex Ewerlöf

Alex Ewerlöf, a veteran engineer who builds and uses LLM systems, delivers one of the most combative rebuttals to the "coding is solved" narrative: LLMs are good at *creation*, but software's real cost is maintenance, reliability, and accountability — none of which AI can carry. His ownership framework (Knowledge, Mandate, Accountability) and his "Nordic Gold" economics of AI-generated software are the load-bearing ideas.

---

Ewerlöf opens with a disclaimer that he is not anti-AI — he built his own harness and teaches the topics — but is challenging the "brain-dead narrative" that coding is solved and engineering is now about taste. His central distinction: cheap creation vs. expensive NFRs.

> Sure, *creation* is much cheaper, but anyone who has run software in production at scale knows that maintenance, reliability, security, scalability, etc. is the majority of the cost.

His taxonomy of what AI *is* good for is the essay's most useful contribution: personal software, POCs, throwaway automation — and "weaponized AI" as the fourth category, where the inherent risk is the point. The commonality is risk tolerance. Professional software in healthcare, finance, aviation "requires accountability," and:

> AI cannot be held accountable. It cannot suffer any consequences... You cannot punish AI, therefore it can never be held accountable.

Why coding specifically resists LLMs: it's logic, and LLMs are stochastic. He's honest about the mechanism — LLMs "succeed" at code only because harnesses feed syntax and runtime errors back until errors are solved or hidden. This is a sharper version of the argument in [[Loop Engineering]] and [[Poor Man's Loop Engineering]]: the loop is doing the logical work, not the model. His "jagged trust" line — a human can be wrong *consistently*, a model can be wrong about something it was right about yesterday — is the crispest statement of why verification never lapses into one-time trust.

The fallacy-by-fallacy section is where the essay earns its aggression. On specs: "Anyone with a few years of industry experience knows that it's impossible to spec the software meaningfully ahead of time" — a direct hit on [[Spec-Driven Development]]'s more utopian claims, though the working SDD practitioners in this wiki ([[SDD Case Study — 13 Apps in 70 Days]]) would say the spec corpus is a living artifact, not an upfront contract, which is exactly Ewerlöf's own point. On "English is the new programming language": natural language is vague and conflicting, which is *why we built compilers*. On "I've stopped reading code": "you are confessing to being redundant."

The ownership triad is the piece that will outlast the flame war:

1. **Knowledge** — you know what problem you're solving and how the solution works
2. **Mandate** — you're trusted to take decisions
3. **Accountability** — if it breaks, you're the one on-call

"Take away any of these three elements and you're dealing with broken ownership." This strengthens [[Own the Outer Loop]] (Osmani's accountability essay) with a cleaner articulation of *why* the outer loop can't be delegated: the model can't hold pillar three. It also complicates [[The End of Code Review]] (Monperrus): if review's job is transferring knowledge and establishing ownership, "agent-in-the-loop verification" solves throughput but not ownership.

On economics, his "Nordic Gold" metaphor (cheap, technically advanced, damn too realistic) frames the SaaS squeeze: vendors "charging human rates while paying for AI output prices" face either a price crash or a quality justification. This dovetails with [[Something Is Changing in the Unit Economics of Software]] and [[Managing AI Coding Costs at Scale]] — both arriving at the same frontier from the vendor side rather than the customer side. His prediction that SaaS becomes an SLA business is the most falsifiable claim in the essay.

**Critical take.** The essay is a polemic and shows it: the Anthropic/DHH sparring, the "Claude users" tribalism, and the sycophantic-AI theory of marketing weaken what are otherwise sound arguments. The AI Dunning-Kruger claim ("the less people know about the complexities of a task, the more likely they are to trust AI output") is asserted more than demonstrated, and the four future engineer roles are sketchy. But the core structure survives the tone: the risk-tolerance taxonomy, the accountability gap, and "slow is fast" are the right frame, and the essay is unusually willing to concede where AI genuinely wins (POCs, personal software, map-reduce on human language) — which is what separates a position from a vibe. The "old trick" section (make PRs massive so nobody reviews them; volume as trust hack) is pre-AI craft knowledge applied to agents, and worth quoting at anyone shipping 500-line agent diffs.

#concept #comparison

---

*Sources: [[raw/coding-is-not-solved]], [[summary/coding-is-not-solved]]*
*Last updated: 2026-09-29*
