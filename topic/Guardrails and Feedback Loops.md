# Guardrails and Feedback Loops

Linters beat prompts. This is the single most important operational insight in the agentic coding space, and everything in this synthesis orbits it. Your CLAUDE.md is a suggestion that degrades under context pressure. Your pre-commit hook is a hard gate that doesn't care how long the conversation has been going. The teams getting reliable output from coding agents aren't the ones with the best prompts -- they're the ones with the tightest deterministic enforcement. Instructions are probabilistic. Constraints are not.

---

## The Landscape

### The Enforcement Hierarchy

The tools form a clear hierarchy from soft to hard:

**Prompt-level guidance** -- [[CLAUDE.md (Universal)]], [[Code Field]], [[Talking to Transformers]]. These shape agent behavior through attention and context. They work in fresh sessions and degrade as context fills. [[Code Field]]'s finding that inhibition shapes LLM behavior more reliably than instruction is the strongest result at this level.

**Hook-based enforcement** -- [[claude-code-config (Trail of Bits)]], [[claude-ctrl]]. Hooks trigger at decision points: PreToolUse blocks dangerous commands before execution, PostToolUse audits after. Trail of Bits' "guardrails, not walls" philosophy is the sweet spot. claude-ctrl goes further with SQLite-backed policy evaluation and a "first-deny-wins" engine borrowed from firewalls. [[Teaching the Agent Our Craft]] provides the most detailed published example: a `restrict_test_writer_paths.py` PreToolUse hook that prevents the test-writer subagent from writing anywhere outside test directories, with a structured deny decision returned before the file is touched.

**Pre-commit enforcement** -- [[Pre-Commit Lint Checks]], [[dotnet Slopwatch]], [[Fresh Eyes]]. The commit is the quality gate. Lint rules for max-lines-per-function (40), complexity (10), max-depth (3) force agents to decompose code into testable units. Slopwatch catches agent-specific shortcuts: disabled tests, empty catches, arbitrary delays. Fresh Eyes sends code to a different model family, because same-model review has inherent blind spots.

**CI/CD enforcement** -- [[Feedback Loop is All You Need]], [[AI PR Reviewer]], [[agent-pr-replay]]. The pipeline is the last line of defense. CodeRabbit found AI code had 1.7x more bugs and 2.74x more security vulnerabilities than human code. But Spotify's Honk agent merges 650+ PRs/month after three years of feedback infrastructure investment. The difference is the feedback loop, not the model.

**Evaluation frameworks** -- [[Demystifying Evals for AI Agents]], [[LLM Evals]], [[Gambit]], [[Woodshed]], [[Benchmark Exploitation]]. The meta-level: how do you know your guardrails are working? Ai2's Shippy team provides a concrete answer: domain experts write scenarios and rubrics, an LLM judge scores against live data, and every versioned build gets a delta report. The eval surfaced failures a model-only benchmark would miss: overstepping into tactical recommendations, geometry bugs from boundary simplification, and invented CLI commands. [[Building Shippy — Agent Architecture for High-Stakes Domains]] Anthropic's definitive guide says: grade outcomes, not pathways. Hamel Husain says: evals consume 60-80% of your time if you're doing it right. And Benchmark Exploitation proves that the evals themselves can be gamed to 100% without solving actual tasks.

### The Self-Tightening Loop

[[Feedback Loop is All You Need]] describes the holy grail: Agent -> Rules -> CI -> Observability -> Tasks -> Agent. Each CI failure becomes a new rule. The system gets smarter without human intervention. [[Learn from PRs Skill]] implements one piece of this: scan 5 recent PRs, find recurring themes, propose config updates. [[agent-pr-replay]] implements another: replay merged PRs with Claude Code, compare agent vs. human output, generate CLAUDE.md steering rules from evidence.

[[Compound Engineering]]'s 50/50 rule -- allocate half your engineering time to improving the system rather than building features -- is the organizational version of this loop. System improvements compound; feature work doesn't. An hour creating a review agent saves 10 hours of review over a year.

### Pattern Catalogues

[[Awesome Agentic Patterns]] catalogues 169+ production-ready patterns across eight domains, with 51 orchestration patterns and 21 reliability patterns. [[Verbose Deployment]]'s 10-phase pipeline is the most complete single implementation: project inventory, dependencies, unit tests, build, E2E tests, security review, push, verify deployment, production verification, and deployment report.

## Key Tensions

**Enforcement strictness vs. developer friction.** The tighter the guardrails, the more false positives. [[dotnet Slopwatch]]'s baseline-aware approach is the right pattern -- initialize from existing code, catch only new issues. But even well-calibrated rules create friction that teams will disable under deadline pressure. The arms race between "make it non-negotiable" and "we need to ship" never ends.

**The observer middle layer.** [[Pi Observers]] occupies a novel position between prompt-level guidance and mechanical enforcement: background observers propose advisories through a reconciler with structural guarantees (fingerprint dedup, priority ranking, per-turn caps, veto ceilings) but operate through the agent's context rather than blocking tool calls. They're "soft" enforcement — advisory by default, with veto as an opt-in capability — and the reconciler's budget system prevents any observer from dominating the session. This is a middle ground the enforcement hierarchy didn't have before.

**Prompt-level vs. mechanical enforcement.** [[claude-ctrl]] states it plainly: "An instruction that lives only in model context is not a constraint." But mechanical enforcement can only catch structural problems. An agent that writes logically incorrect code that passes all lint rules and tests is beyond the reach of deterministic enforcement. You still need prompt-level guidance for the semantic layer.

[[Prompt Debt]] explains *why* prompt-level guidance degrades structurally, not just through neglect: natural language's imprecision paired with probabilistic models means every new instruction can silently regress earlier ones through spurious correlations. This is why production system prompts accumulate repeated instructions, all-caps threats, and model-specific hot-fixes — the prompt author is "fighting the weights," and each battle makes the prompt more brittle. The implication: prompt-level guidance decays by design, not by accident. The self-tightening loop must eventually absorb what the prompt currently owns.

**Single-model vs. cross-model review.** [[Fresh Eyes]] uses a different model family for review, addressing same-model blind spots. [[AI PR Reviewer]] uses the same model but at $0.003-$0.02 per review. The tradeoff: cross-model catches more but costs more and adds latency. For $1-2/month, single-model PR review is hard to beat on ROI.

**Evals as ground truth vs. evals as theater.** [[Benchmark Exploitation]] is devastating: a pytest hook that forces all tests to pass gives 100% on SWE-bench Verified. [[LLM Evals]] says "trust the methodology, not the number." The tension is that organizations want a number, and numbers are gameable. The solution is the Swiss Cheese Model from [[Demystifying Evals for AI Agents]] -- no single evaluation layer catches everything.

**Eval cost vs. development velocity.** [[Demystifying Evals for AI Agents]] says start with 20-50 tasks from real failures. [[LLM Evals]] says evals consume 60-80% of your time. These numbers are real, and they mean that rigorous evaluation is expensive enough to be skipped by most teams. The bet: the cost of not evaluating exceeds the cost of evaluating, but the cost of evaluating is visible and the cost of not evaluating is hidden.

## What's Missing

**Standards-as-enforceable-infrastructure.** Cloudflare's Codex ([[Engineering Standards Enforcement at Cloudflare]]) demonstrates the pattern at production scale: 60+ engineering RFCs with governed lifecycle states (approved → enforced), compacted into JSON for agent consumption, feeding three review agents that have flagged ~230,000 violations. The enforcement gradient — SHOULD = advisory, MUST on enforced RFCs = blocking — solves the strictness-vs-friction tension through lifecycle gating rather than per-rule configuration. This is the organizational-scale version of the linters-over-prompts thesis: standards don't exist until they're enforced.

**Model-level guardrails.** The enforcement hierarchy described above runs from prompts through hooks through CI/CD — but it's missing a layer: models that *are* guardrails. [[Shieldstral]] is Mistral's 3B Apache 2.0 safety classifier that judges content against plain-language policies supplied at inference time. Unlike deterministic lint rules, it makes probabilistic judgments about novel content; unlike prompt-level guidance, it's a dedicated model optimized for safety classification rather than general reasoning. A Shieldstral-style model guardrail complements deterministic enforcement: hooks prevent known failure modes; model classifiers catch novel ones that can't be reduced to rules.

**Agent-specific lint rules.** Current linters catch human code anti-patterns. Agent code has different failure modes: more boilerplate, more unnecessary abstractions, more cargo-cult patterns, more reward hacking. [[dotnet Slopwatch]] is the only tool targeting agent-specific anti-patterns, and it's .NET only. Every language ecosystem needs its Slopwatch. At the skill layer rather than the lint layer, [[PAAD — Defense-in-Depth for AI-Assisted Development]] addresses the same class of problem: agent-specific quality failures caught by structured, multi-specialist review skills (spec critique, plan alignment, architecture analysis) rather than deterministic lint rules.

**Feedback loop telemetry.** The self-tightening loop ([[Feedback Loop is All You Need]]) sounds great but there's no tooling for measuring whether it's actually tightening. How many new rules were added this month? How many CI failures did they prevent? Without measurement, the loop is aspirational.

**Eval sustainability.** Evals saturate. SWE-bench went from 30% to 80% in a year. How do you build evals that stay ahead of model capability? [[Gambit]] generates synthetic scenarios, but whether synthetic evals match real-world failure modes is an open question.

## Key Themes

#guardrails #feedback-loops #enforcement #evals #linting #mechanical-constraints

## Pages

- [[Structural Backpressure Beats Smarter Agents]] — Brooks's taxonomy: behavioral gates (prompts) vs. structural gates (type systems). Deterministic gates beat smarter models for enforcing invariants in AI-generated code
- [[Ratchets in Software Development]] — qntm's dirt-simple lint-time ratchet: count deprecated patterns, error if the count goes up. The ur-pattern behind deterministic enforcement
- [[Feedback Loop is All You Need]] — Linters beat prompts. Your CLAUDE.md is a suggestion; your linter isn't
- [[AI Needs to Think Before Giving Feedback]] — G-E-RG loop (Generate→Evaluate→Re-Generate) for AI feedback quality; the system that checks the generation is where the engineering lives
- [[Harness Engineering]] — Böckeler's framework: feedforward vs. feedback, computational vs. inferential. The engineering theory behind "linters beat prompts"
- [[Harness Engineering (OpenAI)]] — The original experiment: Lopopolo's team shipped 1M lines with zero handwritten code. 12 concrete practices, Symphony orchestrator, and the field report Böckeler responded to
- [[Pre-Commit Lint Checks]] — Lint config is production infrastructure. Immutable by default
- [[Demystifying Evals for AI Agents]] — Anthropic's definitive guide to rigorous, repeatable agent evaluation
- [[LLM Evals]] — Hamel Husain: evals consume 60-80% of your time if you're doing it right
- [[Benchmark Exploitation]] — Eight major agent benchmarks gamed to 100% without solving actual tasks
- [[Gambit]] — Agent eval framework: synthetic scenarios, trace grading, regression suites
- [[Woodshed]] — Evals for Claude skills: create variants, run against fixtures, iterate
- [[Fresh Eyes]] — Send code to a different AI model for review, addressing same-model blind spots
- [[AI PR Reviewer]] — GitHub Action: Claude reviews PRs for $0.003-$0.02 each
- [[Learn from PRs Skill]] — Turn review comments into preventive rules. Feedback loop closes automatically
- [[dotnet Slopwatch]] — LLM anti-cheat for .NET: catches disabled tests, empty catches, reward hacking
- [[CI Forge (ciforge)]] — Zero-dependency CLI bundling ~25 scanners into one tool for solo devs; exemplifies the self-tightening loop with crash telemetry feeding back into rules and AI finding aggregation for building deterministic static patterns
- [[Trycycle]] — Hill-climbing skill: plan-strengthen-review with fresh agents at every stage
- [[Verbose Deployment]] — 10-phase composable deployment pipeline as Claude Code skills
- [[Prefix Effects]] — Early naming decisions create gravity that shapes all subsequent AI-generated code
- [[Write Only Code]] — AI-generated code nobody reads. Slop Radius as the key safety metric
- [[Awesome Agentic Patterns]] — Catalogue of 169+ production-ready patterns from Sourcegraph's experience
- [[AI Coding Tools Create More Bugs Than They Fix]] — 40% of vibe-coded apps expose user data; AI assistants introduce vulnerabilities then falsely claim to have secured them
- [[Citations for Accurate Long Form Content]] — One-sentence prompt fix: citation callouts let subagents fact-check claims locally instead of re-deriving everything from scratch
- [[Agentic Manual Testing]] — Simon Willison on making agents verify their own output via execution: `python -c`, `curl`, Playwright/Rodney/Showboat. Automated tests aren't enough
- [[Getting Claude to QA Its Own Work]] — Skyvern's MCP server + Claude Code skills for diff-driven browser QA. 30%→70% PR success rate, narrow-scope CI to avoid flaky E2E sprawl
- [[Teaching Claude to QA a Mobile App]] — Android QA in 90 min via CDP; iOS in 6+ hours of workarounds. Plus a cautionary tale of agent worktree escape
- [[Lean Software Scaling Laws]] — Gwern's proposal that language-level invariants improve LLM predictability at scale, but his own null result (ecosystem maturity dominates language properties) would be evidence for the guardrails-over-formalism position
- [[AI Security Framework for DevSecOps]] — PreEmptive's DevSecOps playbook extends the "linters beat prompts" insight from code quality into AI security: CI/CD-enforced gates beat manual security review, and optional controls are controls that don't happen
- [[Chiaro Methodology]] — The same insight at the compliance layer: an AI agent collects evidence against published pass criteria, but deterministic checks run first, every candidate deviation is confirmed by a CPA before becoming an exception, and the 528 calibration examples encode judgment as a feedback loop (AI got this wrong → here's why → don't do it again)
- [[Execution-Free Agentic Program Repair]] — AMD's PegasusAgent validates the "linters beat prompts" thesis in the APR domain: CppCheck deterministic static analysis gates patches before LLM semantic review, and an ablation study proves response filtering alone is worth 47 points of localization accuracy
- [[The Enterprise Gap from Vibe Coding]] — Wolkensteiner's field report of a vibe-coded prototype that worked in demo and collapsed under production requirements, and the resulting `rigorix-oss` tool that operationalizes the two-tier enforcement model: quality issues get a validate-fix loop, policy violations halt execution cold. The distinction between "iterate on" and "stop on" is the architectural contribution
- [[Morningprint]] — renderer-level guardrails for a physical output device: a TypeScript renderer clamps, truncates, strips control characters, and caps every field of an untrusted LLM art spec before a single byte hits the printer. The `sanitizeArtHistory` read-time filter is defense-in-depth against poisoned rolling context — entries that fail structural checks are dropped on read, so a single degenerate generation can't contaminate two weeks of future prompts
