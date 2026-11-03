# Nine Harness Patterns (deepsense.ai)

deepsense.ai's practical playbook for reliable AI-assisted development, distilled from their Codex-hackathon-winning HyperCode sidecar and internal experiments with Codex and Claude Code. The thesis: "code is cheap. Human time and model context are not" — so stop tuning prompts and start engineering the system around the agent: repo-as-knowledge-base, just-in-time context, discoverable validation, bounded feedback loops, and a garbage collector that turns recurring agent mistakes into durable rules. #concept #pattern

---

## Key Quotes

> **"Code is cheap. Human time and model context are not."**

The cleanest one-line economics of the harness discipline. Everything that follows follows from it: if output is nearly free, the scarce resources are the reviewer's attention and the context window, so the harness should be designed to economize both.

> "Your coding assistant can read your code. What it cannot infer on its own is *why* the code is structured the way it is."

The argument for AGENTS.md as a table of contents, not an encyclopedia — architecture overview, key constraints, out-of-bounds areas, and pointers to deeper docs. Notably, the *why* store is split across three already-versioned artifacts: AGENTS.md for structure, commit message bodies for decision rationale, and regression-test docstrings for invariants that have bitten in production. Every agent-generated commit then becomes "a documentation artifact... readable by the next agent."

> "In practice, for a living codebase, it tends to fail quietly — the retrieved chunks are stale, semantically similar but task-irrelevant, or missing the one file that actually matters for *this* diff."

Their anti-RAG argument is refreshingly specific: the failure mode isn't retrieval quality, it's *who* decides relevance and *when* the context is assembled. JIT context assembles inside the loop, from the live repo, once the task is known. HyperCode (their hackathon project) implements this as a sidecar that builds a hypergraph from structural links, semantic summaries, and *Codex session memory* — capturing decisions, tradeoffs, and rejected approaches that normally evaporate when the context window closes. Cut prompt-to-plan time by over 50% in their benchmarks.

> "Discovering a command and choosing to use it are separate behaviors."

A quietly important empirical finding from their validation-menu pattern: an agent told to inspect the Makefile still constructed its own `ruff` invocation, missing environment details encoded in the repo target. A discovery pointer alone is insufficient — you must *state the preference explicitly* in AGENTS.md. Same lesson recurs in pattern 9: their agent recorded nothing ambiently despite instructions in AGENTS.md *and* a skill, then produced excellent notes reconstructed from memory when finally asked. Their fix is structural: capture must be a harness checkpoint (stop-hook, PR gate), not a request. Passive instruction loses to the primary objective.

> "The orchestrator should never ingest verbose review essays, raw logs, or every detail of the fixing process... The orchestrator's context holds loop state only."

Their bounded review/fix loop is a tidy piece of agent architecture: read-only reviewer returning JSON findings, a fixer scoped to selected findings, and an orchestrator that owns only the retry cap (max 3 iterations, then escalate). The failure mode is designed to be useful — an escalation report listing what was fixed, what wasn't, and why it stopped. This is the cleanest working example of [[Orchestrator - Worker Skill]]-style delegation keeping the main loop cheap and debuggable.

> "A NEVER/ALWAYS rule is a *pre-decision*. You decided the outcome of every future situation before seeing any of them."

The most transferable insight in the piece, from their "old prompts backfire on new models" pattern. The example: "NEVER push a PR unless the local build is green" — a proxy (green build) standing in for an intent (don't ship broken changes). A better model optimizes the proxy harder, blocking pushes when the failure is pre-existing or flaky. The rewrite formula — intent + decision procedure + escape hatch, reserving literal NEVER for true invariants like secrets and destructive ops — is a crisp repair technique for any accumulated rule library. It dovetails with Superpowers-creator Jesse Vincent's finding in [[Superpowers for Humans (O'Reilly)]] that prohibitions create perverse incentives: both locate the bug in the *sentence*, not the model.

> "If the plan is incomplete, over-specific, or based on a wrong assumption, you did not reduce risk — you promoted the mistake into the task contract."

Their plan-mode critique: accepting a plan is an agent handoff that discards the exploratory context that produced it. They cite HN reports of Superpowers-style frameworks burning ~10× more tokens with no benefit, and propose the alternative loop: short prompt, cheap throwaway run, inspect the diff — then promote whatever the failure exposed into the harness (an AGENTS.md pointer, a design note, a Make target). Planning effort should be proportional to the cost of discarded implementation attempts.

## Key Themes

- **Harness over prompts** (#concept) — the system around the model is where reliability lives; the workflow and validation contract should stay stable while models churn.
- **The why store** (#pattern) — AGENTS.md + commit bodies + test docstrings as three versioned layers of rationale the agent can read.
- **JIT context / sidecar** (#pattern #tool) — task-aware assembly over a live repo beats static embedding indexes for living codebases; session memory is the highest-leverage capture.
- **Bounded loops and enforced capture** (#concept) — feedback loops need retry caps; ambient self-reporting is unreliable and must be forced by harness checkpoints.

## Opinionated Take

This is the rare harness writeup that reports what *didn't* work — compressed screenshots made model judgment worse; passive capture instructions were ignored; a discovery pointer didn't change behavior — which makes the recommendations trustworthy in a way most pattern lists aren't. Its weaknesses are the usual consultancy-playbook ones: the HyperCode benchmarks are self-reported hackathon numbers, the "9 patterns" framing is really ~3 meta-patterns (make the why legible, assemble context per task, close bounded loops) stretched to a listicle, and the article leans heavily on the repo-already-has-it assumption that most teams' commit messages are nowhere near good enough to serve as a why store. Still, the piece earns its place in the canon: the "discovery ≠ preference" and "old prompts backfire on new models" findings are genuinely novel, falsifiable, and cheap to act on this week — which is more than most harness essays can claim.

## Relations

- Strengthens [[Harness Engineering (OpenAI)]]: OpenAI's field report argued for short AGENTS.md and encoded judgment; deepsense.ai independently arrives at the same repo-as-knowledge-base conclusion and adds the commit-body/test-docstring layering plus concrete failed experiments.
- Complicates [[Superpowers for Humans (O'Reilly)]]: where O'Reilly's piece presents Vincent's heavyweight planning workflow sympathetically, deepsense.ai reports the cost side — 10× token burn, plan handoffs promoting mistakes into the task contract — while still endorsing Vincent's narrow skills (TDD, systematic debugging, verification) borrowed opt-in.
- Nuances [[Your AI Agent Is Only as Good as the Harness Around It]]: that piece treats context assembly and traces as product decisions in the abstract; this one gives the concrete sidecar recipe and the empirical result that ambient self-reporting must be a checkpoint, not an instruction.
- Reinforces [[Agentic Verification]]: both make validation the bottleneck and both advocate agent-run feedback loops — deepsense.ai contributes the orchestrator/reviewer/fixer topology with an explicit retry cap as the worked example.

---
*Sources: [[raw/harness-engineering-for-coding-agents-9-patterns-for-more-reliable-ai-assisted-development]], [[summary/harness-engineering-for-coding-agents-9-patterns-for-more-reliable-ai-assisted-development]]*
*Last updated: 2026-11-03*
