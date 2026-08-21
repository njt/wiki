# Poor Man's Loop Engineering

The minimum viable agent loop, stripped down to two ingredients: give the agent a way to know it's wrong (a safe integration/E2E test environment), and have someone other than itself review the work (two fresh, cross-model agents prompted to find reasons *not* to merge). A practitioner's "poor man's" counterpoint to Addy Osmani's full [[Loop Engineering]] taxonomy — notable for what it leaves out, and for its closing warning that agents lack taste.

---

## Key Quotes

> "The poor man's version requires two things: your agent needs to be able to test its own work, and it needs someone other than itself to review that work. This is the foundation of loop engineering."

The whole article in one sentence. The rest is commentary on these two pillars — and it's a genuinely useful compression. Where Osmani's [[Loop Engineering]] taxonomizes five components plus state, this argues you can start with two.

> "The 'loop' isn't that hard to get started; making it truly autonomous is. The first step is... give the agent a way to know it's wrong."

The distinction between a loop and an *autonomous* loop does real work. The barrier to entry is low; the barrier to lights-out is high — and this article is honest that "poor man's" means you're not going lights-out.

> "Spend time making the system accessible to an agent, be it solidifying Make commands to reliably spin up the Docker stack or hooking up a UI sandbox for the agent to click around and gather screenshots."

The concrete, stealable detail. His example Makefile bottoms out in a single `verify: test check e2e` target — one command that runs unit tests, lint/type checks, and E2E, so the agent doesn't have to know *which* check to run, only that `verify` is the gate. This is [[Guardrails and Feedback Loops]] in its most mundane form: a deterministic entry point, not a prompt asking the agent to be careful.

> "The reviewer should actively try to find reasons the change should not be merged."

The instruction that separates adversarial review from rubber-stamping. A reviewer asked to *confirm* will; a reviewer asked to *refute* finds things. Same architecture as the "make review adversarial" principle in [[AI Code Migration with Claude Code]].

> "We use two agents (and, for best coverage, cross model validation) because agents aren't deterministic, and we want to cover as many possible paths, thought processes, and high-level failure points as possible."

Cross-model validation is the hedge against correlated blind spots — the same intuition Osmani lands on in [[Agentic Code Review]] ("heterogeneity is the whole point"). Two agents from the same model can share the same wrong mental model; two *models* are structurally less likely to.

> "Stepping inside the loop is a critical part of the process. We are shifting our work towards setting up automation, but we are still required to be the engineer and own the system at the end of the day. And be warned, the agents lack taste."

The closing warning, and the article's most important sentence. It echoes Osmani's "build it like someone who intends to stay the engineer" almost verbatim — the two pieces converge on the same terminal point from opposite directions (full taxonomy vs. bare minimum).

---

## Key Themes

#pattern #concept #tool

- **#pattern — The two-ingredient loop**: verification environment + independent review. Everything else (worktrees, connectors, automations, state) is optimization on top of these two. The "poor man's" frame is a floor, not a ceiling.
- **#pattern — Adversarial review with cross-model validation**: fresh isolated contexts, prompted to refute, doubled up and cross-model because agents aren't deterministic. The review-as-refutation pattern shared with [[AI Code Migration with Claude Code]] and [[Cloudflare Security Audit Skill]].
- **#tool — The `verify` command as the agent's gate**: a single Make target that runs tests, checks, and E2E. The interface between the agent and "am I done?" is a deterministic script, not judgment.
- **#concept — Taste is the human residual**: the agent can test and review but can't judge whether the task *should* be done at all. "The agents lack taste" is where the engineer stays in the loop — the same boundary [[Optimizing for Decision Points]] draws.

---

## Critical Analysis

**The value here is compression, not novelty.** Everything the author recommends — testing environments for agents, adversarial review, cross-model validation — is already in [[Loop Engineering]], [[AI Code Migration with Claude Code]], and a dozen other pages in this wiki. What's genuinely useful is the *minimum*: two ingredients, not five components plus state. For a solo dev staring at the gap between "Claude wrote some code" and "the agent did the task," that floor matters more than the ceiling. The article is honest that "poor man's" is a starting point, not the destination.

**"Someone other than itself" is the load-bearing phrase.** The entire case collapses into one principle: verification must be independent of generation. It's the same line Osmani draws in [[Loop Engineering]] ("the model that wrote the code is way too nice grading its own homework") and the foundational claim of [[Guardrails and Feedback Loops]]. The author's contribution is stating it as a *requirement* — not a nice-to-have — and making it the second of only two things.

**The unexamined assumption is the "goal you set for the session."** Reviewers compare the work against the original task or spec — but the article never asks where that spec comes from or whether it's *right*. A clean loop faithfully produces the wrong thing when the goal is wrong. That's the gap [[Specifications as the Product]] and [[Load-Bearing Assumptions]] exist to fill: verification is only as strong as the spec it checks against. This article inherits the blind spot of nearly every loop-engineering piece — it's all about the *inner* loop (write-test-fix-review), silent on the outer loop that decides what's worth building.

**The "poor man's" framing undercuts its own autonomy pitch, correctly.** The title invites the cheap version, but the body repeatedly refuses the conclusion the title suggests: the loop isn't autonomous, agents aren't deterministic, and taste doesn't transfer. The reader comment at the end — "the goal isn't to engineer ourselves out of the loop, just out of the most tedious parts of it" — is the honest bottom line, and it's better than the article's own closing.

**Where it sits in the wiki:** this is the *lower bound* counterpart to [[Loop Engineering]]'s taxonomy and the *single-agent-plus-review* counterpart to [[Designing Agentic Loops]]'s "tests as the force multiplier." Willison argues tests make loops converge; this article argues that convergence still needs an independent reviewer, because a passing test suite can't catch a wrong goal or a missing requirement. The two are complementary, not competing.

---

*Sources: [[raw/poor-mans-loop-engineering]], [[summary/poor-mans-loop-engineering]]*
*Last updated: 2026-08-21*
