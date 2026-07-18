# Devin Fusion

Cognition's multi-model agent harness that routes coding work between a frontier "main agent" and a cheaper "sidekick" model, delivering frontier-level performance at 35% lower cost on the FrontierCode benchmark. The architecture keeps both models in separate persistent cached contexts, switches between them during context compaction (making the switch free), and preserves the frontier model for planning, ambiguity, and final review while the sidekick handles bulk execution. The bet: multi-model routing is the end of single-model agent architectures.

---

## Key Quotes

> "The age of using one model for all of your work is coming to an end."

This is the thesis, and it's stronger than it sounds. Cognition isn't just saying "cheaper models exist" — they're arguing the architectural default should be *multi-model by design*, with routing as a first-class feature of the harness. The Lamborghini-for-groceries analogy is memorable because it's true: you don't need Opus to rename variables. But the architectural implication — that your agent harness should own model selection rather than you — is the quiet radicalism here.

> "When the judgment is the deliverable, delegating it backfires."

From the React/Redux team selector example, where costs dropped 28% but the score halved from 54 to 27. This is the most honest admission in the post, and the most instructive. The Fusion architecture delegates *execution* but reserves *judgment* for the frontier model. The React example shows what happens when you get that boundary wrong: the sidekick model can do the typing but not the thinking, and on tasks where the thinking *is* the work, you just paid two models to produce a worse result.

> "We switch models during context compaction — which would trigger a cache miss anyway — making model switching effectively free."

This is the engineering insight that makes the architecture work. Model switching has a natural tax: losing the KV cache on the model you're switching away from. By only switching when compaction would have flushed the cache regardless, Fusion eliminates the switching penalty. It's the kind of detail that separates production architectures from whitepapers — finding the seam in the system where the expensive operation is happening anyway and piggybacking on it.

> "88% of merged PRs from internal Cognition users were driven entirely by the automated Fusion router."

The internal dogfooding metric. Not a lab benchmark — actual developer PRs merged into production code. The fact that they're not claiming 100% is credibility-enhancing; the 12% where humans overrode the router is arguably more interesting than the 88% where they didn't.

## Key Themes

- #pattern **Main agent + sidekick architecture** — A distinct pattern from both the Advisor Strategy (executor calls advisor) and Thrifty (planner delegates to worker): here the frontier model retains authority while the sidekick runs in parallel with its own cached context. Not escalation, not delegation — *co-residence*.
- #concept **Cache-aligned model switching** — The insight that model switching should happen at natural cache boundaries (compaction events) rather than arbitrary decision points. This is a genuinely novel systems idea that generalizes beyond Fusion.
- #pattern **Judgment/execution boundary as the routing heuristic** — What makes a task suitable for the sidekick isn't complexity but *judgment density*. Mechanical work delegates well; tasks where the deliverable is a series of judgment calls don't. This is a sharper heuristic than "easy tasks go to cheap models."
- #tool **FrontierCode as the proving ground** — Cognition uses their own benchmark ([[FrontierCode]]) to validate their own product, which is circular but also transparent: the numbers are public, the methodology is documented, and the adversarial QC (using Devin to try to hack the rubric) adds credibility.
- #concept **The end of single-model architectures** — This post is part of a broader convergence. [[The Advisor Strategy]] inverts the pattern (cheap model drives, expensive model advises). [[Thrifty (Tiered Delegation for Claude Code)]] tiers by task granularity. [[Step 3.7 Flash]] tiers by model capability. Fusion tiers by judgment density within a single session. All four are attacking the same problem from different angles, and all four agree: one model per task is already obsolete.

## Critical Analysis

**The convergence is real, and Fusion's architecture is genuinely distinct.** The Advisor Strategy has the cheap model call the expensive one; Thrifty has the expensive one plan and the cheap one build; Fusion has both running in parallel with cached contexts and a dynamic midpoint switch. Each pattern has different failure modes and different sweet spots, and Fusion's is the best fit for long-running coding sessions where you want the frontier model "present" but not doing the typing.

**The React failure is more instructive than the successes.** The team selector example — 28% cheaper but score cut in half — reveals the architecture's sharp edge. When judgment and execution are interleaved (as they are in complex UI work where every component placement is a micro-decision), the sidekick can't help because there's nothing to delegate *to*. The model is doing judgment work continuously, and judgment is what you're paying the frontier model for. This isn't a bug in Fusion; it's a fundamental constraint on any multi-model architecture. Some tasks are judgment-saturated, and on those tasks, you either pay for the frontier model or you get a worse result.

**The 35% cost claim needs unpacking.** The benchmark table shows Fusion (without Fable 5) at $2.38/task vs. Opus 4.8 at $3.24 and GPT-5.5 at $3.64. That's 27% cheaper than Opus, 35% cheaper than GPT-5.5 — but Fusion also scores slightly lower than Opus (47.9 vs. 48.8). The claim "frontier performance at 35% lower cost" is approximate, not exact; you're trading ~0.9 points of FrontierCode score for ~$0.86 per task. Whether that trade is worth it depends on your tolerance for the quality gap, and the post is honest enough to provide the data that lets you decide.

**The 88% internal adoption stat is the strongest signal, and the least scrutinizable.** It's self-reported, unaudited, and covers an unknown number of PRs over an unknown time period. It's directionally credible — if your own engineers won't use it, you have a problem — but it's marketing, not evidence.

**Cache-aligned switching is the most exportable idea.** The observation that model switching should piggyback on compaction events is elegant and generalizable. Any agent harness that does context compaction (which is all of them) could adopt this pattern. It turns a cost (cache miss on compaction) into an opportunity (free model switch). This idea deserves to escape the Fusion blog post and become common infrastructure.

**The missing piece: what happens when the sidekick screws up.** The post shows examples where delegation works well and one where it fails catastrophically, but doesn't describe the recovery mechanism. Does the main agent detect sidekick failures and re-do the work? Is there a verification step? The architecture diagram implies the main agent reviews the sidekick's output, but the mechanics aren't described. In a production system, the recovery path matters more than the happy path.

## Related Pages

- [[The Advisor Strategy]] — Anthropic's inverted pattern: cheap executor calls expensive advisor on demand. Fusion is the mirror image: expensive main delegates to cheap sidekick by default.
- [[Thrifty (Tiered Delegation for Claude Code)]] — 2389 Research's tiered-delegation plugin: Sonnet plans, Haiku builds, Sonnet verifies on failure. Same cost-saving goal, different granularity (sprints vs. continuous session).
- [[FrontierCode]] — Cognition's mergeability benchmark used to validate Fusion. The benchmark that asks "would a tech lead merge this?" rather than "does it pass tests?"
- [[Step 3.7 Flash]] — StepFun's cost-optimized agentic model. Shares Fusion's thesis that the new frontier is efficiency, not raw capability.
- [[Smart Models Dumb Pipes]] — The judgment/execution separation that Fusion operationalizes. Models own decisions; infrastructure owns execution.
- [[DeepWiki]] — Cognition's other product: instant codebase wikis. Part of the same company's bet that understanding code is the bottleneck, not writing it.
- [[Loop Engineering]] — Addy Osmani's meta-skill of designing systems that route between agents and models. Fusion is loop engineering in production.
- [[Components of a Coding Agent]] — The thesis that the harness matters more than the model. Fusion is the harness evolving to manage multiple models, not just one.

---
*Sources: [[raw/devin-fusion]]*
*Last updated: 2026-07-18*
