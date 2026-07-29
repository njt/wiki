# Dynamic Workflows in Claude Code

Anthropic's announcement of dynamic workflows — a feature that lets Claude Code author its own JavaScript-based orchestration harness on the fly, spawning and coordinating subagents for tasks that break the single-context-window model. Written by Thariq Shihipar and Sid Bidasaria, the engineers building Claude Code. This is the most detailed public documentation of how Claude Code's multi-agent substrate works, and it doubles as a pattern language for agent orchestration: six composable workflow patterns, nine use-case categories, and an honest accounting of when *not* to use the feature.

---

## Key Quotes

> "The default harness can break down over long-running, massively parallel, highly structured and/or adversarial tasks."

The article's diagnosis of *why* dynamic workflows exist is sharper than its prescription for what to do with them. Three named failure modes — **agentic laziness** (declaring partial progress complete), **self-preferential bias** (the model grading its own homework), and **goal drift** (lossy compaction eroding the original objective) — are the real contribution. These aren't Claude Code bugs; they're structural properties of long-context-window agents that no amount of prompt engineering fixes. The solution is architectural: fresh context windows with focused goals.

> "Claude is now intelligent enough to write a custom harness tailor-made for your use case."

The most important claim in the post, and the one that separates dynamic workflows from the static workflow systems that came before (Agent SDK, `claude -p`). Static workflows had to be generic enough to handle all edge cases. Dynamic workflows are generated at runtime for the specific task. This is the same shift that happened in programming languages — from compiled generic binaries to JIT-compiled specialized code. But it also means the harness itself is LLM-generated, which introduces a meta-verification problem the article doesn't address: who watches the watchmaker?

> "Parallelism and specialization have to earn their coordination cost."

The article's rare moment of restraint. After nine use cases and six patterns, this sentence is the guardrail: dynamic workflows are not free. Token costs multiply with agent count, and the coordination overhead of merging N agents' output is non-trivial. This is the same economic logic Adam Jacob applies in [[Reducing Token Spend with Deterministic Workflows]] — the LLM should build the program, not be the runtime.

> "Each summarization step is lossy."

Four words that explain why goal drift happens. Compaction isn't a neutral operation — it's lossy compression of context, and the loss compounds across compactions. This is why [[Loop Engineering]] treats short, focused agent sessions as the atomic unit rather than long-running conversations. Dynamic workflows are essentially a formalization of that insight: instead of one long context window that repeatedly compacts, spin off N short ones that never need to.

---

## Key Themes

#tool — Dynamic workflows are a Claude Code feature, not a general architectural pattern. They're specific to the Claude Code runtime and its JavaScript orchestration layer. The patterns (fan-out, tournament, adversarial verify) are portable; the implementation is not.

#pattern — The six workflow patterns (classify-and-act, fan-out-and-synthesize, adversarial verification, generate-and-filter, tournament, loop-until-done) are the article's most reusable contribution. They're a composable pattern language for agent orchestration that applies beyond Claude Code. Several already appear independently in other systems: [[Cloudflare Security Audit Skill]] uses fan-out + adversarial verification, [[Agent Swarm Model Economics]] uses planner/worker trees, [[Swarm Skill]] institutionalizes adversarial review.

#concept — **Agentic laziness**, **self-preferential bias**, and **goal drift** name three failure modes that every multi-agent system designer should know. They're the "eight fallacies of distributed computing" for the agent era — things that seem like bugs in your implementation but are actually structural properties of the architecture.

#concept — **Worktree isolation** as a primitive. The ability to spin off subagents into isolated git worktrees means parallel agents can mutate files without conflict. This is how the Bun migration (Zig → Rust) was done: each fix happened in its own worktree, adversarially reviewed, then merged. The worktree is the agent-era equivalent of a database transaction.

#pattern — **Quarantine** is the article's most understated contribution. Agents that read untrusted public content (web searches, external documents) should be barred from high-privilege actions (file writes, deploys). It's a security boundary enforced by workflow architecture, not by prompt instructions — which makes it real in a way that "please don't do bad things" prompts never are.

---

## Critical Analysis

The article is **excellent as documentation and incomplete as guidance**. It tells you what dynamic workflows can do with vivid examples, but it's strategically silent on what goes wrong. A workflow that spawns 50 agents against an ambiguous prompt will burn tokens at an alarming rate and produce 50 confidently wrong answers. The "when not to use" section is one paragraph at the end of a long post — honest but buried. The real failure mode of dynamic workflows isn't discussed: Claude authors a subtly wrong harness that then produces subtly wrong results at scale, and the user has no way to detect the error because the harness itself is opaque.

The **JavaScript-only constraint** is both a feature and a limitation. It's a feature because it means workflows are deterministic, version-controllable, and shareable — unlike conversation history. It's a limitation because it means the orchestration language is general-purpose code, which introduces all the failure modes of general-purpose code (bugs, edge cases, maintenance burden). The article frames this as "dynamic" vs. "static" workflows, but the real question is whether LLM-generated JavaScript is more reliable than LLM-generated conversation — and the answer is probably yes for structure but no for correctness.

The **failure mode diagnosis is stronger than the cure**. Agentic laziness, self-preferential bias, and goal drift are real and well-described. But the solution — "spin off more subagents" — has its own failure modes: coordination collapses, divergent interpretations of the task, and the meta-problem of verifying the verifiers. The article gestures at adversarial verification as the answer, but adversarial verification works best when the adversary has a *different* perspective — running the same model with a slightly different prompt doesn't create genuine independence. This is the territory [[Swarm Skill]] covers with its cross-model cold review requirement.

The **token economics are conspicuously absent**. A workflow that spawns 5 Opus agents with adversarial verification might cost $5–50 in API tokens. For a one-off task that saves an hour of human time, that's a bargain. For a workflow that runs on a `/loop` every 10 minutes, it's a mortgage payment. The article mentions token budgets in the tips section but never quantifies the real costs of the example workflows it describes. Compare with [[Reducing Token Spend with Deterministic Workflows]], which is built entirely around the cost question.

The **relationship with `/loop` and `/goal` is underdeveloped**. The article suggests combining dynamic workflows with these features but doesn't explain how they interact. A workflow on a loop that spawns subagents every N minutes, each of which can also spawn subagents — this is a recursive token furnace if not carefully bounded. The `/goal` mechanism as a hard completion requirement is promising but the interaction surface with dynamic workflows is uncharted.

Despite these gaps, the post is a **major architectural statement** from the team building the most widely-used coding agent. It formalizes patterns that were previously folk knowledge, names failure modes that every practitioner has hit, and signals Anthropic's bet that multi-agent orchestration — not bigger context windows — is the path to reliable agentic behavior. The closing note that "we're still discovering new ones [patterns]" is honest about where we are: this is a starting point, not a finished system.

---
*Sources: [[raw/dynamic-workflows-claude-code]]*
*Last updated: 2026-07-29*
