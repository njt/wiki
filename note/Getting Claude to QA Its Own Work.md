# Getting Claude to QA Its Own Work

Skyvern built an MCP server with 33 browser tools and two Claude Code skills (`/qa` and `/smoke-test`) that make Claude verify its own frontend changes by actually opening a browser, looking at the pixels, and running interactions. One-shot PR success rate jumped from ~30% to ~70%, and the QA loop was cut in half. The CI version reads the git diff, forms a hypothesis about what changed, and tests only the nearby flows — a narrow approach that avoids the usual flaky-E2E sprawl.

---

## Key Quotes

> "We had a consistent failure mode: code that looked right, passed type checks, but when you actually tested it, something was subtly off."

The entire motivation in one sentence. Type systems catch structural errors; they can't catch "the button renders but the click goes nowhere." This is why [[Harness Engineering]] distinguishes computational feedback (tests, type checking) from inferential feedback (human judgment) — and why browser-based QA fills a gap neither covers.

> "Read the diff, form a hypothesis about what changed, and test only the nearby flows."

The anti-flake strategy. Instead of running every test on every commit, `/smoke-test` stays surgically focused on the change's blast radius. This is the same narrow-scope discipline that makes [[Designing Agentic Loops]] work: YOLO-mode agents converge when the problem is bounded.

> "Buttons that render but don't fire, forms that submit the wrong thing, elements that are technically present but unusable."

The class of bugs that static analysis and unit tests will never catch. These are the exact failure mode that [[Teaching Claude to QA a Mobile App]] encountered on mobile — and the same category that [[AI PR Reviewer]]'s diff-level review catches at the logic level but misses at the visual level.

---

## Key Themes

#tool #qa #browser-automation #CI #feedback-loop #claude-code #MCP

**Browser automation as computational feedback.** The Skyvern MCP server is a [[Harness Engineering]] pattern made concrete: 33 browser tools that turn "does this page actually work?" from an inferential question (human squints at it) into a computational one (agent drives a browser, asserts behavior). This is the same architectural insight as [[surf-cli]] and [[Browser Use]], but wired directly into the development feedback loop rather than as standalone automation.

**Diff-driven test scoping.** The `/smoke-test` skill doesn't run a test suite — it reads the diff, classifies the change, and generates targeted tests. This is the inverse of traditional CI: instead of "run everything and hope," it's "read the change and test only what's nearby." Compare with [[Learn from PRs Skill]], which also mines the diff to close feedback loops, but does it for lint rules rather than browser tests.

**The 30%→70% success rate jump.** This number is the article's headline finding, but the real story is the remaining 30%. The acknowledged hard problems — keeping tests current, scoping mixed diffs, detecting shallow test plans — are all variations on "the agent doesn't know what it doesn't know." This is the [[Agent Coding Workflow]] maturity gap: verification catches errors the agent made, but doesn't prevent the agent from making them.

**CI as evidence archive.** Posting PASS/FAIL tables, screenshots, and failure reasons directly to the PR is underrated. It creates a searchable history of *why* a change failed that survives beyond the CI run. [[If AI Is Doing the Investigation, Version the Investigation]] makes the same argument for agent transcripts.

---

## Critical Analysis

The 30%→70% jump sounds impressive, but there's a selection effect at work: this number applies to frontend changes tested by the same model that wrote them. Same-model QA has a fundamental limitation — Claude checking Claude's work is vulnerable to the same blind spots that [[Fresh Eyes]] identifies. A more rigorous setup would route QA to a different model, or at minimum a different session with no shared context.

The real contribution here isn't the success-rate number — it's the **diff→hypothesis→test→report pipeline as a reusable pattern.** The `/qa` skill prompt is 700 lines of open-source YAML; the `/smoke-test` skill is 300. These are small enough to study and adapt. Anyone running Claude Code on a frontend project could replicate this approach today, and many should.

What's missing: a story for **keeping the test bank current.** They acknowledge this as an unsolved problem, and it's the same one that plagues every testing approach. If the agent generates tests from the diff, and the codebase evolves, who maintains the tests for flows that haven't changed recently? The `/smoke-test` approach cleverly sidesteps this by making tests ephemeral (generated fresh per PR), but that means no regression protection for unchanged code.

The narrow-scope CI strategy is smart and correct — full E2E suites on every commit would be both expensive and flaky — but it leaves a gap: **regressions at a distance.** A change to the auth flow might break the settings page, but if the diff classifier sees "auth" and scopes testing to auth-related flows, it'll miss it. This is the same blast-radius problem they acknowledge as unsolved.

Compared to [[Teaching Claude to QA a Mobile App]], this is the polished-product version: desktop web, CDP-native, zero coordinate-guessing. The mobile QA piece is the war story; this is the product launch. Both are correct in their context, and both converge on the same truth: **browser automation for agent-driven QA works today on Android and desktop, and barely works on iOS because Apple won't expose the protocol.**

---

## Cross-Links

- [[Teaching Claude to QA a Mobile App]] — same domain, same problem, mobile war-story version
- [[AI PR Reviewer]] — automated PR review at the diff level; Skyvern adds the pixel level
- [[Feedback Loop is All You Need]] — linters > prompts; browser QA is computational feedback made concrete
- [[Compound Engineering]] — adding a QA system to trust the output rather than reviewing every change manually
- [[Harness Engineering]] — the 33 browser tools are computational feedback; the `/qa` skill is inferential feedback
- [[Fresh Eyes]] — different-model review addresses same-model blind spots; Skyvern uses same-model QA
- [[Learn from PRs Skill]] — closing the feedback loop from reviews; diff-driven, like Skyvern
- [[Designing Agentic Loops]] — the QA loop as an agentic loop: tools, guardrails, success criteria
- [[surf-cli]] — browser automation for agents via CLI; same domain, desktop rather than CI
- [[Browser Use]] — AI browser automation with deterministic rerun; complements the QA story
- [[Don't Fear the Dark Factory]] — validation problem, not generation problem; Skyvern is the validation harness
- [[If AI Is Doing the Investigation, Version the Investigation]] — PR evidence as durable investigation artifact
- [[Agent Coding Workflow]] — verification over generation; the QA step in the practitioner's loop
- [[Guardrails and Feedback Loops]] — the self-tightening feedback loop with browser verification as a new layer

---
*Sources: [[summary/getting-claude-to-qa-its-own-work]]*
*Last updated: 2026-05-15*
