# Benchmarking AGENTS.md Changes

Stet (of stet.sh) ran Codex through 8 iterations of improving its own AGENTS.md, measuring each version against real PRs. The single best candidate regressed on a clean holdout — better at local craft, worse at boundary judgment. The post argues that AGENTS.md files are tunable harness components that need empirical validation, not vibe-based editing, because instruction changes produce **AGENTS.md inversion**: average improvement that masks specific-task degradation.

---

## Key Quotes

> "AGENTS.md files are part of the runtime behavior of your coding system."

The thesis in one sentence. Your instruction file isn't documentation — it's a configuration parameter injected into every session. Changing it changes behavior. The question is whether that change is measurable.

> "The failure mode isn't simply 'everything gets worse' — it's 'enough gets better that you miss the damage.'"

This is the core danger Stet identifies. An instruction change improved most tasks while degrading a measurable subset. The average went up. Without a holdout, the author would have shipped it. This is the empirical version of [[Writing a Good CLAUDE.md]]'s warning that "a bad line in CLAUDE.md cascades across plans, research, and code generation."

> "The new instructions helped when the right boundary was obvious, and hurt when the task required judgment about how wide the boundary should be."

The regression was systematic, not noisy. The candidate AGENTS.md made the agent better at coherent local implementation (clearer names, explicit status fields, structured logs) but worse at boundary judgment (narrowing broad requests, documenting broader than implemented, adding parallel contracts instead of extending existing ones).

> "More process did not mean more discipline — sometimes it was just more ceremony."

One candidate required exact owner file/function and a validation command before editing. Tests stayed green. Code review correctness dropped by –0.40, coherence by –0.38, simplicity by –0.10. Tighter rules don't automatically produce better behavior.

> "I don't think anyone can claim to know model behavior well enough to one-shot a perfect AGENTS.md."

The closing takeaway: instruction engineering is engineering — iterate, measure, holdout-validate. Vibes are not enough.

## Key Themes

**#concept AGENTS.md Inversion** — An instruction change improves most tasks while degrading a measurable subset. The average goes up; specific task types regress. This makes shared instruction files dangerous to edit by feel, especially on multi-developer codebases where the person making the change sees improvement but downstream colleagues see degradation without knowing instructions changed.

**#pattern Obligation Ledger** — Before editing, the agent identifies named behavior, compatibility constraints, docs, tests, and non-goals, then marks each as met, missed, or not checked. This was the most promising rule in the experiment: it produced the strongest gains in review, correctness, maintainability, simplicity, and coherence. It recovered a previously missed task on the training slice.

**#pattern Bounded Instruction Loop** — Six steps: write hypothesis → test on real work (n=5) → inspect failures → revise rule → run holdout (n=10) → validate claim. Don't ship on mixed evidence. This is the scientific method applied to instruction engineering.

**#tool Stet.sh** — Local eval tool that measures coding agent behavior against historical repo tasks. Allows the agent to propose AGENTS.md changes while Stet evaluates them. Runs locally using the user's own LLM subscriptions. Disclosure: the author is building it.

**#person Stet** — Creator of stet.sh, building local eval tooling for coding agent instruction changes.

## Critical Analysis

**This is the first empirical treatment of AGENTS.md/CLAUDE.md authoring I've seen.** Every other source in the wiki — [[Writing a Good CLAUDE.md]], [[CLAUDE.md (Universal)]], [[Intent Layer]], [[How Claude Code Works in Large Codebases]] — offers principles and heuristics. Stet offers measurements. The claim that "better instructions make the agent cheaper" failed the holdout: token usage went *up*, not down. This is the kind of counterintuitive finding you only get from measurement.

**The methodology is honest about its limits.** One repo, one model (gpt-5.5), n=10 holdout, directional not statistically significant. Stet doesn't oversell. The post reads more like a field note than a paper, which is exactly what this space needs — more practitioners running experiments and sharing the raw results, fewer people pontificating about what "should" work.

**AGENTS.md inversion is the concept that matters.** It's the same failure mode as A/B testing without holdouts, applied to instruction engineering. The mechanism is intuitive once you see it: an instruction that helps with "add a feature" tasks may hurt with "debug this" tasks. Without measuring task-type splits, you optimize for the tasks you happened to test. This connects to [[Harness Engineering]]'s Ashby's Law observation — the harness must be at least as complex as the system it regulates, and your eval suite is a harness for your instruction file.

**The obligation ledger pattern is promising but incomplete.** It produced the best results, then regressed on scope discipline. Stet's diagnosis — the agent needed a rule about proving breadth is required before expanding — is the right next step. This is what iterative instruction engineering looks like: each rule exposes the next rule you need. Compare with [[Compound Engineering]]'s self-tightening loop: observe failure, add control, test again.

**The post is slightly self-serving.** Stet is building the tool used in the experiment, and the post is a showcase. That doesn't invalidate the findings — if anything, dogfooding your own tool on your own codebase is the most honest demo possible — but it means the post is part product announcement, not pure research. The line between "here's what I learned" and "here's what my tool does" blurs in places.

**The biggest gap: no comparison to human-edited AGENTS.md.** Codex iterated on its own AGENTS.md. It found improvements, regressions, and limits. But we don't know whether a skilled human would have done better, worse, or the same. The experiment compares Codex-iteration-7 to Codex-iteration-0, not to a human-crafted alternative. This matters because [[Writing a Good CLAUDE.md]] argues forcefully that AGENTS.md should be hand-crafted, not auto-generated.

**What this means for shared codebases is the most important unstated implication.** The author mentions it in passing — "the person making the change sees improvement, downstream colleagues see degradation" — but this deserves its own post. On a team of 10 engineers, a CLAUDE.md change that improves one person's tasks by 20% and degrades three others' tasks by 10% is net negative, but the committer won't see the degradation. Without per-task-type measurement, the damage is invisible. This is the strongest argument for version-controlling instruction files and measuring changes before merging, not after.

**The takeaway I'd add to Stet's five questions:** Does each instruction change get its own commit, or are you bundling five vibe changes into one "updated AGENTS.md" commit? Because if it's the latter, you can't attribute the regressions to any specific change. Atomic instruction commits are the missing practice.

## Cross-References

- [[Writing a Good CLAUDE.md]] — Kyle's instruction-budget argument and the case for hand-crafted brevity. Stet provides the empirical complement: measure, don't just assert
- [[CLAUDE.md (Universal)]] — The minimal six-rule template. The question Stet raises: have these six rules ever been empirically validated, or are they just the ones that "feel right"?
- [[Intent Layer]] — Hierarchical AGENTS.md at folder boundaries. Stet's finding that instruction changes have task-type-specific effects suggests Intent Layer's folder-scoping is a risk mitigator: bad instructions in one folder only affect tasks in that folder
- [[Harness Engineering]] — Böckeler's feedforward/feedback framework. AGENTS.md is pure feedforward; Stet is making the case that feedforward needs empirical validation loops too
- [[Feedback Loop is All You Need]] — The self-tightening loop: Agent → Rules → CI → Observability. Stet's bounded loop is the same pattern applied to the instruction file itself
- [[Demystifying Evals for AI Agents]] — Anthropic's guide to rigorous agent evaluation. Stet's methodology is a concrete instantiation: task-based, holdout-validated, multi-dimensional
- [[LLM Evals]] — Hamel Husain: evals consume 60-80% of your time. Stet's 8 iterations + holdout is what that looks like in practice for instruction engineering
- [[Compound Engineering]] — The 50/50 rule: half your time on system improvement. Applying the rule to your instruction files means measuring them
- [[How Claude Code Works in Large Codebases]] — Anthropic's claim that "setup matters more than the model." Stet's experiment tests exactly this: how much does changing the setup (AGENTS.md) change behavior?
- [[Designing Agentic Loops]] — Willison on the meta-skill of choosing guardrails and success criteria. Instruction engineering is part of that meta-skill
- [[Slowing the Fuck Down]] — Deliberate friction. Stet's bounded loop inserts friction between "I had an idea for AGENTS.md" and "I shipped it"
- [[claude-ctrl]] — "An instruction in context is not a constraint." Stet shows that even empirically validated instructions aren't perfectly constraining — the holdout regression proves context is leaky
- [[Guardrails and Feedback Loops]] — Synthesis page. Stet adds a new dimension: guardrails for the guardrail files themselves
- [[Agent Coding Workflow]] — The practitioner's daily loop. Stet treats AGENTS.md as part of that loop's configuration, not its static background
- [[Codex-maxxing]] — Jason Liu's Codex field report. Complementary perspective: what Codex can do beyond coding
- [[Simon Willison — Engineering Practices That Make Coding Agents Work]] — Willison on TDD as agent unlock. Stet extends the argument: eval-driven development for instruction files
- [[Specsmaxxing]] — Acceptance criteria with stable IDs. The same "stable measurement across changes" concept, applied to instruction files instead of code

---
*Sources: [[raw/codex-iterates-agents-md]]*
*Last updated: 2026-05-31*
