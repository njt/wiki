# The Lifecycle of a Swamp Issue

Paul Stack's field report on how the Swamp team dogfoods their own agent workflow framework. An issue isn't a GitHub thread — it's a state machine instance (`@swamp/issue-lifecycle`) that enforces five phases: triage, planning, adversarial review, iteration, and implementation. The agent physically cannot skip steps. No plan is auto-approved; no issue is auto-triaged. The model is the guardrail.

---

## Key Quotes

> "The real state lived in the swamp datastore and GitHub was just a reflection of it."

When the issue lifecycle grew to include structured plan versions, adversarial findings, and feedback rounds, GitHub issues became a sync problem. Swamp-club became the source of truth — itself a swamp model where each method call posts a structured lifecycle entry. This is [[Long Live Systems of Record]] in practice: "where does the truth live" is the only question that matters.

> "The agent physically cannot skip steps."

Methods like `approve`, `implement`, and `triage` are guarded by model-level checks. Approving a plan requires it to exist, to have gone through adversarial review, and to have no unresolved critical or high findings. Call a method in the wrong phase and it bounces. This is [[claude-ctrl]]'s "an instruction in context is not a constraint" applied to workflow state — the model is the enforcement mechanism, not a suggestion in a prompt.

> "If the tool can't handle its own process, it isn't ready to handle yours."

The issue lifecycle model runs on Swamp infrastructure. When the team wants to change triage behavior, they edit the `issue-lifecycle` extension, and the upgrade system carries old instances forward. Conventions change via `agent-constraints/` files. This is dogfooding as engineering discipline, not marketing.

> "The human reads it and either gives feedback... or says to proceed."

No auto-approval of plans. The agent generates, reviews, and presents — but a human who will live with the outcome must approve. Operator-driven triage serves double duty: it prevents prompt injection via issue bodies and ensures the human frames the context.

---

## Key Themes

#tool #workflow #state-machine #adversarial-review #dogfooding

- **State machine as guardrail** — The five-phase lifecycle (triage → planning → adversarial review → iteration → implementation) isn't a convention; it's enforced by the model. The agent bounces if it tries to skip phases. This is the same philosophy as [[Pre-Commit Lint Checks]] but applied to process state rather than code quality.

- **Adversarial review as a distinct phase** — Before any code is written, the plan is critiqued across repo-specific dimensions with severity-graded findings. This is [[Trycycle]]'s fresh-eyes principle baked into the workflow: review before implementation, not after. It also echoes [[Fresh Eyes]]'s cross-model review but with the added structure of severity tracking.

- **GitHub as reflection, not source of truth** — The team tried dual-wielding GitHub + Swamp but found sync problems inevitable. Moving the source of truth to Swamp is a specific instance of [[Long Live Systems of Record]]'s argument: agents don't kill systems of record, they raise the bar for what a good one looks like.

- **Human gates at the critical transitions** — Triage is operator-initiated (not automatic). Plan approval is human-gated (not automatic). The agent does the work but humans decide when it proceeds. This is the [[Agent Coding Workflow]] maturity sweet spot: Level 3-4, where the agent generates and the human supervises at checkpoints rather than line-by-line.

- **Repo-local customization via `agent-constraints/`** — Each repo can tailor the lifecycle without modifying the core model. This is filesystem-as-configuration, the same pattern as [[CLAUDE.md (Universal)]] and [[Writing a Good CLAUDE.md]], but for workflow constraints rather than agent behavior.

- **Resumability as a first-class property** — Any agent can join mid-process, inspect `swamp model get issue-N --json`, and see the next valid transition. This addresses the [[Agent Memory and Context]] problem from the workflow side: state lives in the model, not in the conversation.

---

## Critical Analysis

**The adversarial review phase is the most important idea here and the least discussed.** Most agent workflows have planning then implementation. Adding a dedicated adversarial review phase — where the plan is systematically attacked before any code is written — addresses the core failure mode of agentic development: the agent confidently executing a bad plan. This is cheaper than fixing bad code and catches problems at the level of intent rather than syntax. It's the same insight as [[Specifications as the Product]] (invest in the spec, not the code) but operationalized as a state machine phase.

**The GitHub divorce is honest about something most agent tools dance around.** Claude Code, Codex, Aider — they all integrate with GitHub Issues as the task source. Swamp argues that as soon as your issue has structured state (plan versions, findings, feedback rounds), GitHub becomes a liability, not an asset. The sync problem between a rich data model and GitHub's flat issue format is real. Whether Swamp's answer (replace GitHub entirely) or [[Kata]]'s answer (local-first SQLite issue tracker) is better depends on whether you need GitHub's collaboration surface.

**Operator-driven triage as prompt injection defense is clever but incomplete.** Stack mentions it almost in passing, but it's a genuine architectural insight: if an attacker can file an issue with a prompt injection payload, automatic triage means the agent processes attacker-controlled text without human oversight. Requiring a human to initiate triage closes this vector. But it's a speed bump, not a wall — the human might not notice the injection either, and the payload could be subtle enough to survive triage and affect planning or implementation.

**The "agent physically cannot skip steps" claim depends on the model being correct.** If the state machine model has a bug, the agent can skip steps. This is the same trust-the-compiler argument — the guard is only as strong as the guard's implementation. Stack acknowledges this implicitly by dogfooding: the issue lifecycle model is itself built and maintained through the issue lifecycle. That's recursive validation, which is stronger than testing alone but not immune to systemic failure.

**What's conspicuously absent:** error recovery. What happens when adversarial review finds a critical flaw in phase 3? Does the plan go back to phase 2 (planning) or phase 4 (iteration)? The state machine presumably encodes these transitions, but Stack doesn't describe them. Also missing: the implementation verification process, which is promised as a follow-up post. Without that piece, we see the gate but not the verification on the other side.

**The relationship to [[The Plan Is the Program]] is direct.** Swamp's lifecycle makes the plan the atomic unit of work: triage produces classification, planning produces the plan, adversarial review attacks the plan, iteration refines the plan, and only then does implementation render the plan into code. The plan is literally the thing that moves through the state machine. Code doesn't appear until phase 5.

---

## Cross-Links

- [[Swamp Club]] — the framework this lifecycle runs on: Zod-typed models, DAG execution, immutable versioned data
- [[Long Live Systems of Record]] — "where does the truth live" is why GitHub lost to Swamp's datastore
- [[The Plan Is the Program]] — the plan as the atomic unit of work, operationalized as a five-phase state machine
- [[Trycycle]] — adversarial review as a distinct phase before implementation
- [[Fresh Eyes]] — cross-model review; Swamp's adversarial review is the workflow-level version
- [[claude-ctrl]] — "an instruction in context is not a constraint"; Swamp enforces process via model-level guards
- [[Specifications as the Product]] — invest in the spec/plan, not the code; Swamp makes this a phase-gated requirement
- [[Agent Coding Workflow]] — the maturity spectrum; Swamp's lifecycle is Level 3-4 operationalized
- [[Guardrails and Feedback Loops]] — the enforcement hierarchy; Swamp's state machine is mechanical enforcement at the process level
- [[Kata]] — local-first issue tracker for AI-assisted work; alternative to Swamp's issue model
- [[Agent Orchestration]] — the planner/worker/judge pattern; Swamp embeds the judge in the lifecycle itself
- [[Pre-Commit Lint Checks]] — lint as production infrastructure; Swamp's state machine guards are the process equivalent

---

*Sources: [[summary/the-lifecycle-of-a-swamp-issue]]*
*Last updated: 2026-05-15*
