# AI Code Migration with Claude Code

Anthropic's field report on running large-scale code migrations (Bun: Zig→Rust, 1M lines; internal tool: Python→TypeScript, 165K lines) using Claude Code multi-agent orchestrations. The core thesis: fix the process that produces the code, not the code itself.

---

## Key Quotes

> "You fix the process (loop) that produced the code" rather than fixing the code itself.

The article's thesis in one sentence. This inverts the normal debugging instinct — when an agent produces wrong output, don't fix the output; fix the rules, the review structure, or the verification gate that let it through. It's [[Lean Software Production]] applied to migration: engineer the system, not the artifact.

> "Make review adversarial and verification mechanical"

The design principle that separates this from vibes-based migration. Reviewers are prompted to *refute*, not confirm. Verification is a script, not a human judgment call. This is the same architecture as [[Cloudflare Security Audit Skill]] and [[VulnHunter]] — adversarial agents checking each other's work, with deterministic gates as the final word.

> "Don't use the largest model for everything"

Tiered model routing in practice: Sonnet for high-volume implementation fan-out, Opus for reviewers and rule-writers. This mirrors the [[The Advisor Strategy]] pattern and [[Thrifty (Tiered Delegation for Claude Code)]] — cheap models do the bulk work, expensive models do the judgment work.

> "Throw out any translated files. The goal is to refine the rules, not make incremental progress."

The hardest discipline in the whole piece. The shakedown cruise produces output you deliberately discard. This is [[SDD Case Study — 13 Apps in 70 Days]]'s "correct the spec, not the code" applied to migration rulebooks instead of product specs.

> "Done" means the output file exists on disk.

The mechanical, resumable work queue as the backbone of the whole operation. No task tracker, no Jira — the filesystem IS the kanban board. This is the [[StrongDM Factory Techniques]] "Filesystem-as-memory" pattern at production scale.

## Key Themes

- **#pattern — Rulebook as compile target**: The rulebook is a living document that agents write against and reviewers enforce. When systemic issues surface, you add one sentence to the rulebook and regenerate. The rulebook is to migration what a spec is to [[SDDW (Spec-Driven Development Workflow)]] — the durable artifact that outlives any individual agent session.

- **#pattern — Adversarial review with mechanical verification**: Two reviewers evaluate each translation; disagreements go to a third. But the real referee is the compiler and test suite — scripts that can't be argued with. This is [[Guardrails and Feedback Loops]] at its most literal: linters beat prompts, and here, `cargo build` beats code review.

- **#pattern — Tiered model economics**: At $165K in API costs for the Bun migration, model selection isn't academic — it's the dominant cost lever. The pattern of Sonnet-for-implementation / Opus-for-judgment is the same architecture as [[The Advisor Strategy]] and cuts costs by ~64% per [[Thrifty (Tiered Delegation for Claude Code)]].

- **#concept — The queue writes itself**: Compiler errors and test failures *are* the work queue. You don't plan the next batch of work; you run the build and feed the output back to fixer agents. This is the self-tightening feedback loop from [[Loop Engineering]] — the system generates its own next tasks.

- **#concept — Front-load the human hours**: The rulebook and stress-test phases are the most time-consuming human work, but they're also where human judgment has the highest leverage. This is [[Optimizing for Decision Points]] in practice — design the workflow so human attention lands on the taste-sensitive decisions, not the mechanical ones.

- **#tool — Claude Code as migration harness**: Not a special-purpose migration tool, but Claude Code configured with skills, subagents, and a rulebook. The migration harness IS the development harness — same primitives, different configuration.

## Critical Analysis

**What's genuinely new here isn't the technology — it's the process engineering.** Claude Code could do multi-agent work before this. What's novel is the *recognition that migration is a process-design problem, not a code-generation problem.* The article is essentially a case study in [[Loop Engineering]]: the meta-skill is designing the system that produces the code, not prompting the agent that writes it.

**The $165K number is a Rorschach test.** If you see "this is absurdly expensive," you're thinking about human-scale migrations. If you see "1M lines of production Rust in two weeks for the price of one senior engineer's annual salary," you're thinking about organizational-scale migration. Both are right. The real question is what kind of migration warrants this spend — and the answer is probably "whatever your org has been putting off for five years because the human cost was too high."

**The Bun migration's post-merge results are more impressive than the token count.** 19 regressions across 1M lines, all since fixed. 4% unsafe blocks (mostly FFI boundaries). Memory dropped 91%. Binary 19% smaller. 2–5% faster. This isn't "AI wrote some code that compiled" — it's "AI produced a better artifact than the hand-written original on multiple dimensions." That's the threshold that matters.

**The article is strategically silent on what failed.** We get the 19 regressions number but no taxonomy of what they were. We get the success stories (Bun, Python→TS) but no mention of migrations that were attempted and abandoned. The "Related Resources" point to a starter kit and a plugin, but there's no postmortem of the hard parts. This is a launch post, not an engineering postmortem — read it accordingly.

**The most important sentence might be the one about the rulebook.** "Add one sentence to the rulebook and regenerate the affected batch." This is the compounding loop: each failure discovered in review becomes a permanent rule that prevents that entire class of failure in every future batch. The rulebook doesn't just guide agents — it *learns*. This is [[Specifications as the Product]] applied to process: the rulebook is the durable asset; the generated code is disposable.

**The "judge" prerequisite is under-described and probably the hardest part.** Rewriting tests to be portable across both codebases, validating they fail on deliberately broken code — this is the verification infrastructure that makes everything else possible. It's also the step most teams will skip because it feels like overhead. Without it, you're doing vibe-based migration with no mechanical verification, and the whole "review adversarial, verification mechanical" principle collapses.

**The tiered-model economics have a dark side the article doesn't mention.** Sonnet-for-implementation / Opus-for-judgment works because Sonnet is good enough at translation and Opus is distinctly better at judgment. If the gap closes — or if a single model becomes good enough at both — the architecture collapses to a simpler form. The article's advice is tied to the current model landscape, which is the most transient thing in AI right now.

---

*Sources: [[raw/ai-code-migration]]*
*Last updated: 2026-07-25*
