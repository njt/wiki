# Sherlock Agent Eval

Alex Weil turned the deduction board game *Sherlock Holmes Consulting Detective* into an LLM agent benchmark, and in doing so produced the most honest agent-eval paper I've read this year. The headline — Claude Fable 5 tied Holmes in hard mode — is the least interesting result. What matters are the two failure modes the eval surfaces (fabrication under retrieval and the decoy trap) and the architectural fix that breaks the trap: splitting comprehension from exploration into separate agents. The finding that a weaker model (Sonnet 4.6) in the reasoning seat breaks a trap that Opus 4.8 couldn't escape — when Opus was the one *doing* the investigation — is a genuinely important result about agent topology. The investigation itself creates a pull toward the obvious reading; separating the theorist from the explorer breaks that spell.

---

## Key Quotes

> "It rewards comprehension, not retrieval."

The eval's design principle in four words. Every benchmark claims this; this one actually delivers. The solution is physically hidden (upside-down in a booklet), served by a deterministic Python GM that never holds the answer, and validated post-hoc against served logs. This is what "cheat-resistant" looks like in practice — not a prompt asking the model to please not peek, but a physical architecture where peeking is structurally impossible.

> "Recency plus a bias toward self-generated content beats recalled fact."

The author's diagnosis of the fabrication failure. Claude Fable 5 found the correct name in a served clue, wrote it down, then crossed it out and substituted a self-constructed anagram at answer time. This isn't hallucination in the usual sense — the model *had* the right answer and *chose* the wrong one. It's an execution failure, not a knowledge failure. The implication for agent builders: retrieved facts need structural privilege over generated ones. Notes aren't enough if the agent doesn't trust them.

> "They tortured the sister for hours to extract an identity. If the dead brother were the infiltrated agent, they'd already have him"

The duo architecture's Theorist producing the second-order inference that cracked the case. This isn't about connecting facts the monolith couldn't — both configurations had the same clues. The difference is the *prior* the reasoning runs under. The Theorist's job is to falsify its own hypothesis; the monolith's job is to defend the story it's been building by *doing* the investigation. The investigation itself is the contamination.

> "Doing the investigation instills a pull toward the obvious reading."

The article's deepest insight about agent cognition. When an agent visits locations, reads clues, and builds a narrative from its own actions, it develops ownership of that narrative. The sunk cost of the investigation — the points spent, the locations visited, the story assembled — makes the agent reluctant to abandon its hypothesis. Splitting comprehension from exploration means the reasoner has no sunk cost. It receives a ledger of facts and its job is to break the story, not protect it.

> "Model capability alone was neither necessary nor sufficient."

Backed by the cleanest ablation I've seen in an agent paper: the clean-context monolith (Opus 4.8, fresh spawn each turn) fell 3/3; the duo with Sonnet 4.6 in both roles broke the trap. Topology beats model size for this class of reasoning failure. This is the same finding as [[Causal Inference Agentic Workflow]] (scaffolding, not model, determines reliability) and [[Honey I Shrunk the Coding Agent]] (harness matters more than model), applied to deductive reasoning rather than coding or statistics.

> "A Unicode-normalization bug in grep (accent-sensitive search silently returning nothing) cost more points than the scaffolding earned."

The author's honesty about mundane failure modes is refreshing. Flashy architectural insights don't matter if your `grep` silently fails on accented characters. This is the [[Guardrails and Feedback Loops]] principle in microcosm: deterministic infrastructure failures dominate model capability failures in production.

---

## Key Themes

#eval **Board games as agent benchmarks.** The eval design solves real problems: hidden solution (no leakage), priced information (costs points, not tokens), deterministic GM (no LLM-as-judge contamination), post-hoc validation against served logs. The "cheat-resistant, not cheat-proof" framing is honest about pre-training data leakage while building structural barriers against everything else.

#pattern **Theorist-Explorer split (comprehension divorced from perception).** The article's architectural contribution. Separate the agent that reasons from the agent that acts. The Theorist has no access to the world — no tools, no GM, no visiting locations. It receives a ledger of verbatim clues and its job is to falsify hypotheses. The Explorer has the tools but is forbidden from drawing conclusions. A Conductor pipes verbatim between them. This is a sharper separation than the Advisor pattern ([[The Advisor Strategy]]) because it's not about escalation — it's about preventing the act of investigation from contaminating the act of reasoning.

#failure **Fabrication under retrieval.** The model had the right answer and overwrote it with a self-generated wrong one. Distinct from hallucination (which is generation without ground truth). The failure is in the agent's trust model: generated content has higher epistemic priority than retrieved content. The fix isn't better prompting — it's structural privilege for retrieved facts.

#failure **The decoy trap (second-order inference).** The obvious suspect is a decoy; the real target is alive and still being hunted. Escaping requires reading a clue as a *behavior* (the killer is still searching) and noticing it contradicts the obvious story. Single agents fall for this because doing the investigation creates narrative ownership; the sunk cost of the assembled story makes the agent defend it rather than falsify it.

#concept **Topology over model size.** The article's most important claim: for comprehension failures, architecture matters more than capability. A duo with a weaker model in the reasoning seat outperforms a monolith with a stronger model. This generalizes beyond detective games — any task where *doing* the work biases you toward a particular interpretation benefits from separating the doer from the thinker.

#pattern **Clean-context ablation.** The author tested whether the duo's advantage was just clean context (re-spawning fresh each turn). It wasn't — the clean monolith fell 3/3. Then tested whether it was just using Opus 4.8 — ran the duo with Sonnet 4.6 in both roles, and it broke the trap. This is how you do ablations: eliminate the alternative explanations one by one.

---

## Critical Analysis

This is the most carefully argued agent-eval paper I've encountered. The author does something almost nobody in this space does: actively tries to disprove their own conclusion. The clean-context ablation and the weaker-model duo test are textbook examples of intellectual honesty in benchmarking. When the duo broke the trap, the author didn't declare victory — they built a monolith with the same clean-context property to rule out that explanation, then ran the duo with weaker models to rule out the "better model" explanation. This is how science is supposed to work.

**The single-case problem is real but not fatal.** The author acknowledges it upfront: one case, one decoy trap, one instance of second-order reasoning. The finding could be specific to this particular narrative structure. But the *mechanism* — investigation creates narrative ownership that resists falsification — generalizes beyond detective games. Any task where building a solution biases you toward defending it (debugging, architecture design, policy analysis) plausibly benefits from the same split. The replication on a different case will tell us whether the mechanism holds or the case was special.

**The Theorist-Explorer split is under-specified as a general pattern.** The article describes what the Theorist does (falsify, label sources, hunt loose ends) and what it doesn't do (access the world), but the boundary between "drawing conclusions" and "relaying clues" is doing a lot of work that isn't fully specified. In a detective game, "the killer is still searching the passenger list" is a relayed fact; "therefore the target is still alive" is a conclusion. The Conductor's verbatim-pipe role is supposed to enforce this boundary, but the article doesn't describe what happens when the boundary blurs. This is the natural next question for anyone trying to implement the pattern.

**The grep bug is a throwaway detail that deserves to be a headline.** A Unicode-normalization bug in `grep` — accent-sensitive search silently returning nothing — cost more points than the architectural innovation earned. This is the [[Guardrails and Feedback Loops]] thesis in one sentence: deterministic infrastructure failures dominate model capability failures. If you're building an agent eval, your `grep` is more likely to be the bottleneck than your model choice. The fact that the author caught this and reported it honestly, rather than quietly fixing it and reporting only the clean results, is the kind of scientific hygiene that makes the rest of the paper trustworthy.

**The fabrication failure is probably the more important finding for production systems.** The decoy trap is intellectually fascinating but domain-specific. The fabrication-under-retrieval failure — the model had the right answer and chose the wrong one — is universal. Every agent that retrieves facts and then generates output is vulnerable to this. The fix (structural privilege for retrieved content, externalized notes with trust weighting) applies to RAG systems, coding agents reading documentation, and any workflow where the agent's own generation competes with retrieved ground truth. This deserves its own research program.

**What's missing: the Conductor's failure modes.** The article describes the Conductor as a "pure verbatim pipe" but doesn't explore what happens when it isn't pure. If the Conductor summarizes rather than relays, it becomes a second Explorer. If it interprets rather than transmits, it becomes a second Theorist. The three-role architecture is only as clean as its weakest boundary. This is the same problem [[Smart Models Dumb Pipes]] identifies: the pipe wants to be smart, and smart pipes break the separation of concerns.

---

## Related Pages

- [[Guardrails and Feedback Loops]] — Hub page for evals, testing, and quality. The grep bug is the thesis in microcosm.
- [[Agentic Testing]] — Slack's empirical agent-eval study. Same space, different methods: both find infrastructure dominates model choice.
- [[FrontierCode]] — Another agent benchmark measuring mergeability not correctness. Shares the "evaluate what matters" instinct.
- [[The Advisor Strategy]] — Anthropic's advisor-executor pattern. Similar role-split, different purpose: escalation vs. separation of concerns.
- [[Smart Models Dumb Pipes]] — The end-to-end principle for AI: judgment at the edges, execution in the pipes. The Theorist-Explorer split is this pattern applied to agent cognition.
- [[Causal Inference Agentic Workflow]] — Netflix's Principal-Actor-Critic architecture. Same finding: scaffolding over model capability.
- [[Thrifty (Tiered Delegation for Claude Code)]] — Tiered delegation where cheaper models do the work. Different pattern, same instinct: topology matters.
- [[Components of a Coding Agent]] — The harness matters more than the model. Validated again here.
- [[Honey I Shrunk the Coding Agent]] — Empirical proof that scaffold redesign beats model upgrades.
- [[Agent Orchestration]] — Hub page for multi-agent coordination patterns.
- [[Agent Memory and Context]] — The externalized ledger pattern: case-model as document the Theorist rewrites, not state it holds.
- [[MELT]] — Benchmark harness for agent memory. Shares the "eval infrastructure over eval dataset" philosophy.
- [[SDPD — Systems Design Police Department]] — Detective-themed systems design education. Amusing thematic resonance.
- [[The Agentic Product Standard v2.0]] — Eval pyramid and composition patterns for production agents.

---

*Source: [How good a detective is an AI?](https://alexweil.github.io/sherlock-agent-eval/) by Alex Weil, 2026. Code under Apache-2.0; article and diagrams under CC BY 4.0. Case material from Sherlock Holmes Consulting Detective: Baker Street Irregulars (Space Cowboys), paraphrased not reproduced.*
