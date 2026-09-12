# The New Software Lifecycle

Addy Osmani's definitive map of how AI is reshaping every phase of software development — not uniformly, but unevenly. Implementation compresses from weeks to hours while requirements, architecture, and verification remain stubbornly human because they involve judgment. The core argument: the harness (instructions, tools, context, observability) matters far more than the model, and context engineering is the financial lever most teams are ignoring.

---

## Key Quotes

> "An agent is a model plus a harness."

The cleanest framing in the piece. Osmani estimates the split at 10% model, 90% harness — and backs it with concrete benchmarks: a team moved from outside the top 30 to the top 5 on Terminal Bench 2.0 by changing only the harness. LangChain added 13.7 points by altering "just the system prompt, tools, and middleware around a fixed model." Neither touched the model. This is the inverse of what most teams obsess over.

> "Set the bar at the eval, not the demo."

A demo shows an agent *can* work once. An eval suite shows it *reliably* works. Osmani positions verification as the literal line between vibe coding and engineering — tests for deterministic parts, evals (both output and trajectory) for non-deterministic parts. "An answer that looks right but skipped its checks is more dangerous than one that's obviously broken."

> "AI turns implementation from writing into reviewing."

The honest assessment behind the 25–39% productivity numbers. A METR study found experienced developers going 19% *slower* on some tasks when counting check-and-fix time. The gain isn't writing faster — it's reviewing instead of writing. That's a different skill, a different cognitive load, and a different fatigue profile.

> "AI amplifies whatever engineering culture it lands in, the good parts and the bad parts both."

The closing line and the one that should be taped to every monitor. AI doesn't fix broken processes; it accelerates them. If your team ships sloppy code, AI makes it sloppier faster. If you invest in specs and verification, AI compounds that investment.

> "Architecture is the most stubbornly human phase."

Trade-offs like consistency versus availability depend on business context models can't fully grasp. The developer's role becomes making and documenting structural decisions the agent then implements. This is the phase where judgment has no substitute yet.

## Key Themes

- **#concept Harness over model:** The model is ~10% of what makes an agent work. Everything else — instructions, tools, sandboxes, observability, context management — is the harness. Configuration failures, not model failures, are the dominant failure mode. This is both the most actionable and most underinvested insight in agent engineering.
- **#pattern Static vs. dynamic context:** The boundary between context loaded every turn and context loaded on demand is an architectural decision that should be reviewed in PRs and versioned like code. Progressive disclosure — skills that load minimally at startup and pull heavy reference material only when matched — is the scaling pattern.
- **#pattern Conductor vs. orchestrator:** Two daily modes. Conductor: real-time, in-IDE, keystroke-by-keystroke, good for exploration. Orchestrator: async goal-handoff to agents, good for well-specified work like migrations. The shift from conductor to orchestrator is "a skills shift before it's a tooling one."
- **#tool Evals as infrastructure:** Verification moves to the center of the process. Tests cover deterministic parts; evals cover non-deterministic parts (output evaluation and trajectory evaluation). Both are necessary, and the eval suite — not the demo — is where you set the quality bar.
- **#concept The uneven compression:** AI compresses some phases dramatically (implementation) and leaves others untouched (architecture, requirements). Spec quality becomes the new bottleneck; verification moves from afterthought to centre. The 80% problem — agents get the first 80% fast but the last 20% needs context they don't have — remains the ceiling.
- **#concept Vibe coding vs. agentic engineering as an economic choice:** Vibe coding is cheap upfront (prompts + subscription) but expensive to run (token burn, maintenance tax, security cleanup). Agentic engineering flips this: invest upfront in schemas, tests, structured context; pay less per feature afterward. The crossover is illustrative (~3–10x), not measured — but the directional truth holds.
- **#pattern Model routing as cost control:** Route hard reasoning to large models, routine work (test generation, code review, CI checks) to small cheap ones. Quality holds and costs drop. This is financial engineering, not model engineering.

## Critical Analysis

Osmani is synthesising here, not discovering — and that's the point. The paper draws on a Google whitepaper and his own extensive writing on harness engineering and loop engineering. What's valuable isn't novelty but integration: he connects the dots between harness design, context architecture, verification economics, and lifecycle compression into one coherent map.

The 10/90 split (model/harness) is a rhetorical device as much as an empirical claim — he admits it "sounds high until you've spent a week debugging one." But the benchmarks backing it (Terminal Bench 2.0 moves, LangChain +13.7 points) are real and independently verifiable. The directional claim — harness matters more than model for most failures — is well-supported.

The "3x to 10x more per feature" for vibe coding vs. agentic engineering is the weakest link. It's illustrative, not measured, and he's careful to flag that. But the underlying economics are sound: unstructured context burns tokens linearly with conversation length, structured context doesn't; ad-hoc code without tests creates maintenance debt that compounds; fast-generated vulnerabilities that reach production are catastrophically expensive to clean up. You don't need a precise multiplier to know which direction the curve points.

What's missing: Osmani doesn't address the organizational pathologies that make the upfront investment (specs, tests, structured context) hard to secure. Saying "invest in evals" is correct and easy; getting an org that's addicted to demo-driven development to fund eval infrastructure is the actual problem. The friction isn't technical — it's cultural and budgetary.

The maintenance point is the most underrated in the piece and deserves more attention. Code "too risky to touch" is a massive category in most orgs — legacy systems where only the original authors understood the intent and they've all left. Agent-assisted refactoring and modernization of these systems could be the single largest ROI from AI coding, and it's the phase most discussions skip.

The conductor/orchestrator distinction is useful but incomplete. The real spectrum has at least four modes: conductor (realtime pair-programming), reviewer (agent writes, human checks), orchestrator (async handoff), and autonomous (agent runs unattended with guardrails). Osmani collapses this to two, which is cleaner but loses resolution.

## Cross-References

- [[Agentic Code Review]] — Osmani's companion piece on the review bottleneck; the verification half of this lifecycle map
- [[Loop Engineering]] — Osmani's name for the meta-skill of designing systems that prompt agents rather than prompting them yourself
- [[Agent Coding Workflow]] — Hub page for the maturity spectrum from vibes to compound engineering
- [[Specifications as the Product]] — Spec quality as the new bottleneck, directly reinforced by Osmani's lifecycle analysis
- [[Guardrails and Feedback Loops]] — The verification infrastructure Osmani argues should be at the centre
- [[Agent Memory and Context]] — Context engineering as the financial lever; static vs. dynamic context as architectural decision
- [[Lean Software Production]] — Matt Wynne's parallel argument: the product is still working software, but the work is engineering the system that produces it
- [[Components of a Coding Agent]] — Harness vs. model taxonomy; the empirical case that harness matters more
- [[Honey I Shrunk the Coding Agent]] — Empirical proof of the harness-over-model thesis: 9B model jumps from 19% to 46% on scaffold redesign
- [[Nicole Forsgren on AI and Developer Productivity]] — The bottleneck shift from inner loop to outer loop
- [[Five Studies That Are Changing How I Think About AI in Software Engineering]] — Brian Houck's synthesis converging on the same story: AI compressed upstream, everything downstream is breaking
- [[Thrifty (Tiered Delegation for Claude Code)]] — Model routing in practice: ~64% cheaper at equal quality
- [[Vibe Coding as a Team Sport]] — Jon Udell's constructive answer to vibe coding chaos with structured workflow
- [[Writing Code vs. Shipping Code]] — Demirer et al.: 180% AI gains at commit level attenuate to 30% at release
- [[Human-in-the-Loop is Tired]] — Laura Summers on the psychological cost of reviewing AI output that Osmani doesn't fully address
- [[The Enterprise Gap from Vibe Coding]] — A concrete field report of the demo-to-production gap Osmani maps economically: half-hardcoded data, zero auth, client-side DB calls — the exact maintenance tax his vibe-coding-vs-agentic-engineering cost curve predicts
- [[The AI-Native SDLC Playbook]] — Anthropic's official operating manual for this exact map: six stages each committing an artifact the next reads (`intent.md` → `spec.md` → `plan.md`), governance enforced by hooks rather than habits, and maintenance closing the loop autonomously
- [[Shipping AI Agents to Production]] — the enterprise restatement of the same map: "a verification problem, not a modeling problem," with the five-component production feedback loop and the "context is the moat" thesis as the harness-over-model argument aimed at CTOs

---
*Sources: [[raw/new-software-lifecycle]], [[raw/new-sdlc-vibe-coding]]*
*Last updated: 2026-07-18*
