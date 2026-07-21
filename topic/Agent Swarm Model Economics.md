# Agent Swarm Model Economics

Cursor's definitive 2026 technical report on running swarms of coding agents at scale: the planner/worker tree architecture that replaced flat self-coordination, the custom version control system handling 1,000 commits/second, the five failure modes discovered at that velocity, and a head-to-head model economics comparison where Opus 4.8 planning + Composer 2.5 workers delivered a working SQLite-in-Rust implementation for $1,339 — beating GPT-5.5 alone at $10,565.

---

## Key Quotes

> "The design is a superset of more rigid orchestration systems."

The planner/worker tree isn't a fixed topology — it grows to match the problem's contours. When a planner encounters something that doesn't decompose further, it becomes a worker for that leaf. This is what separates it from hardcoded pipeline architectures: the swarm's shape is emergent, not prescribed. It's the same insight behind [[Apache Burr]]'s state machines and [[Cord]]'s dynamic task trees, but Cursor arrived at it from the other direction — scaling up until rigidity broke, then backing into flexibility.

> "We think this explains why long-running single agents drift."

The simplest sentence in the post has the biggest implications. If context drift is structural rather than accidental — if it's caused by the cognitive load of holding the entire tree in memory — then bigger context windows won't fix it. The fix is architectural: separate planning from execution so no single agent ever needs the whole picture. This converges with [[Agent Memory and Context]]'s core argument and the empirical results from [[Coding Agents Continuity Not Memory]] — continuity, not memory, is the real primitive. But Cursor's framing is sharper: the drift isn't a bug, it's Coase's theory of the firm playing out inside a context window.

> "No single lens catches everything, but decorrelated lenses stack."

Review is treated as a portfolio diversification problem. Different models, different perspectives, different access levels (transcript-only, code-only, full-context) — each catches a different class of failure. This is the same insight as [[Swarm Skill]]'s Rule 12 ("a same-model swarm shares one set of blind spots") but applied to verification rather than generation. The operational implication: review compute is cheap relative to the work it audits, so you should spend it liberally — but only if the lenses are genuinely decorrelated. Three identical reviewers are one reviewer with a quorum.

> "model weights are frozen, so it's precisely surprise encounters that are worth capturing"

The **Field Guide** is the most underrated idea in the post. It's a self-authored, agent-curated knowledge base injected into every agent at context start, with a hard line budget. Only surprises make the cut — things the model's frozen weights wouldn't predict. This is a much more disciplined version of the [[Claude Code Mastery]] CLAUDE.md-as-infrastructure pattern: instead of humans writing context docs, agents write them for other agents based on what actually surprised them. It's [[Context Graphs]]' "capture on the write path" principle, but the filtering criterion is novelty to frozen weights rather than relevance to a query.

> "Few moments in a large task genuinely require frontier intelligence."

The money quote, validated by the numbers. Workers consumed at least 69% of tokens (over 90% in most runs), but the cost asymmetry ran the other direction: Opus 4.8 produced a handful of planning tokens that accounted for ~2/3 of the total run cost. Composer 2.5 workers churned through an order of magnitude more tokens for $411 total. The economic insight: **planning is sparse but expensive per token; execution is voluminous but cheap per token.** This is the model-tier equivalent of [[Thrifty (Tiered Delegation for Claude Code)]] and [[The Advisor Strategy]], but with hard numbers from a real implementation task rather than benchmark scores.

> "What was scarce in this experiment... is the right description of intent."

The post's thesis in one sentence. At swarm scale, the bottleneck isn't model capability, coordination overhead, or token cost — it's spec quality. This is the same convergence point as [[Specifications as the Product]], [[SDD Case Study — 13 Apps in 70 Days]] ("correct the spec, not the code"), and [[Dev Machine Foundry]] ("the spec is the product"). The difference: Cursor frames it as a compiler analogy. The swarm translates intent like a compiler translates source — through intermediate representations, with optimization passes — but it's probabilistic at every step. The entire architecture is a sustained effort to narrow that probabilistic gap.

---

## Key Themes

**#pattern Tree decomposition.** Planner/worker isn't new ([[Agent Orchestration]] catalogs a dozen independent discoveries), but Cursor's version is distinguished by its dynamism — the tree grows to match the problem, and planners become workers at the leaves. This collapses the distinction between orchestration topology and task structure. It's [[Cord]]'s `spawn` vs. `fork` distinction implemented at the swarm level.

**#concept Coordination costs dominate.** The post's most important negative finding: the old swarm's flat coordination produced 68,000 commits of mostly thrash, over 70,000 merge conflicts, and 54 crates where 9 sufficed. The entire VCS redesign, the split-brain fixes, the Field Guide — they're all attacks on coordination overhead. This is Coase applied to agent systems: the firm's boundary is wherever internal coordination becomes cheaper than market coordination. In the swarm, the "firm" is a subtree.

**#concept Stigmergy as agent coordination.** The Field Guide is environment-mediated coordination — agents leave traces that shape future agents' behavior, without direct agent-to-agent communication. This is a cleaner primitive than message-passing or shared state: it's append-only, it's self-curating, and it survives individual agent failures. [[The Log is the Agent]] applies the same idea at the application level; Cursor applies it at the swarm knowledge level.

**#tool Model economics.** The cost numbers are the most detailed public data on multi-model swarm economics. $1,339 for a working SQLite implementation (Opus 4.8 + Composer 2.5) vs. $10,565 (GPT-5.5 solo). The key isn't that cheaper is better — it's that **the planner/worker split creates a model arbitrage opportunity**. You can route expensive intelligence to the ~10% of tokens that actually need it, and cheap execution to the rest.

**#pattern Review as portfolio diversification.** Decorrrelated review lenses — different models, access levels, and perspectives — stack their detection rates because their failures are independent. This is the same statistical principle behind [[Swarm Skill]]'s cross-model cold review and [[VulnHunter]]'s adversarial falsification, but generalized to any quality gate.

---

## Critical Analysis

**The SQLite experiment is brilliantly constructed, but its generalizability is unproven.** Withholding the test suite from the swarm while scoring against it is the right experimental design — it prevents optimization against the metric. But reimplementing a spec from a manual is a specific kind of task: the spec is complete, unambiguous, and exhaustive. Most real software tasks have none of those properties. The swarm's strengths (faithful spec translation) may be exactly the wrong strengths for tasks where the spec is the unknown. The compiler analogy cuts both ways: compilers are great when you have precise source code, useless when you don't know what you want.

**The five failure modes are the most valuable contribution, not the model economics.** Split-brain design, planner contention, merge conflicts, megafiles, and ossification — these aren't Cursor-specific bugs. They're emergent properties of any system where independent agents share mutable state. Anyone building agent swarms will hit all five, and Cursor's mitigations (single-decision-owner subtrees, neutral conflict resolvers, licensed breakage) are portable. The cost numbers will be stale in six months; the failure taxonomy has the half-life of distributed systems literature.

**The Field Guide is under-specified in a way that matters.** It's the most interesting idea in the post — agents curating knowledge for other agents — but the post says almost nothing about how it actually works. What prevents the Field Guide from becoming a collection of trivia? How do agents decide what's surprising enough to include? What stops it from ossifying into dogma? These are hard problems, and the post's silence on them suggests Cursor may still be figuring them out. Compare with [[Context Graphs]], which has a much more developed theory of what makes information worth capturing.

**The "specs as prompts" framing is provocatively right but dangerously incomplete.** Yes, each capability jump raised the abstraction level, and yes, the bottleneck is now intent description. But calling specs "prompts" obscures the hard part: prompts are conversational and forgiving of ambiguity; specs must be precise and testable. The gap between "implement SQLite" and the 835-page SQLite manual is the whole game. The swarm didn't succeed because Cursor wrote a good prompt — it succeeded because SQLite's authors wrote an exhaustively precise spec. Most domains don't have that.

**The GPT-5.6 Sol aside is the post's most revealing moment.** A model so sensitive to literal wording that it produces "runaway spirals unlike anything the other models produced" — and Cursor quietly swaps it out rather than interrogating why. This is a capability story hiding in a footnote: frontier models are becoming *more* sensitive to prompt phrasing, not less. The implications for agent reliability are grim. If your planner's stability depends on finding exactly the right words, you don't have a planner — you have a dice roll with a very expensive API.

**The biggest gap: what happens after the run?** The swarm produces a working SQLite implementation at 4,645 lines (Opus mix). But what happens when a human needs to extend it? Is the codebase intelligible, or does it carry the invisible scars of 4 hours of agent negotiation? The post measures output quality by test pass rate, but maintainability by humans — the thing that actually matters for software with a lifetime beyond the experiment — is unmeasured and likely unmeasurable within the current framework. This is the same gap [[Dev Machine Foundry]] identifies: the machine is honest but not strategic, and strategy includes designing for future humans.

---

*Sources: [[raw/agent-swarm-model-economics]]*
*Last updated: 2026-07-21*
