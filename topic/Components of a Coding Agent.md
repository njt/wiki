# Components of a Coding Agent

Sebastian Raschka's anatomy of what makes coding agents work. The key insight: "a lot of apparent 'model quality' is really context quality." The harness -- the software scaffold around the LLM -- matters more than the model, especially now that "vanilla versions of LLMs nowadays have very similar capabilities." He identifies six components and backs them with a reference implementation (mini-coding-agent on GitHub).

---

## The Taxonomy

Raschka draws careful distinctions:

- **LLM:** the core next-token model
- **Reasoning model:** an LLM trained to spend more inference-time compute on intermediate reasoning
- **Agent:** a control loop around the model that decides what to inspect, which tools to call, and when to stop
- **Agent harness:** the scaffold managing context, tools, prompts, state, and control flow
- **Coding harness:** a harness specialized for software engineering

This taxonomy matters because it explains why model benchmarks don't predict real-world coding performance: the harness contributes at least as much as the model.

## Key Quotes

> "A good coding harness can make a reasoning and a non-reasoning model feel much stronger than it does in a plain chat box."

The harness is an amplifier. A good one elevates any model; a bad one wastes even the best model.

> "A lot of apparent 'model quality' is really context quality."

This is the article's thesis in seven words. If your context is bloated with stale file reads, your model will look dumber than it is. Context management is the underrated engineering discipline.

> "Coding work is only partly about next-token generation. A lot of it is about repo navigation, search, function lookup, diff application, test execution, error inspection."

The model generates tokens. Everything else is the harness's job. The more of these non-generation tasks the harness handles well, the better the overall system performs.

> "The tricky design problem is not just how to spawn a subagent but also how to bind one."

Subagents are easy to create and hard to constrain. Binding -- how much context they inherit, what they're allowed to do, how deep they can recurse -- is where the engineering judgment lives.

> "The harness can often be the distinguishing factor that makes one LLM work better than another."

And if you dropped a capable open-weight LLM "into a similar harness, it could likely perform on par with" leading models. The harness, not the model weights, is the moat.

## The Six Components

1. **Live repo context** -- gather workspace state upfront (git status, AGENTS.md, README) so the agent doesn't start blind
2. **Prompt shape and cache reuse** -- stable prefix stays stable; only the user request and recent transcript change each turn
3. **Structured tools, validation, permissions** -- pre-defined tools with clear boundaries; the harness validates before execution. "Giving the model less freedom, but it also improves usability"
4. **Context reduction** -- clipping overlong outputs, summarizing older transcript segments, deduplicating repeated file reads. "One of the underrated, boring parts of good coding-agent design"
5. **Structured session memory** -- two layers: full transcript (JSON, resumable) plus working memory (distilled task state)
6. **Bounded subagents** -- delegate with inherited context but tighter constraints than the main agent

## Connections

This article is the reference taxonomy for much of this wiki's agent architecture section. [[Harness Engineering]] provides the theoretical framework (feedforward vs. feedback, computational vs. inferential) behind what Raschka describes empirically. [[Honey I Shrunk the Coding Agent]] proves the thesis quantitatively: redesign the scaffold around a 9B model's behavioral profile and it jumps from 19% to 46% on Aider Polyglot.

[[Designing Agentic Loops]] names the meta-skill Raschka implies: choosing tools, guardrails, and success criteria so agents converge. [[Scaling Long-Running Agents]] confirms that flat self-coordination fails while planner/worker/judge works -- the subagent binding problem at scale.

The context reduction strategies connect to [[Three Tier Memory]]'s hot/warm/cold architecture and [[Agent Memory and Context]]'s context-as-RAM metaphor. The tool validation component is the mechanism behind [[Guardrails and Feedback Loops]] and [[claude-ctrl]]'s principle that "an instruction in context is not a constraint."

Raschka's mini-coding-agent on GitHub is the instructional complement to the conceptual taxonomy -- a [[Reference Implementation]] worth studying alongside [[What I learned building an opinionated and minimal coding agent]].

## Critical Analysis

Raschka's strength is clarity: he names things precisely and draws distinctions that many practitioners blur. The LLM/reasoning-model/agent/harness taxonomy alone is worth the read -- it gives you vocabulary for design conversations that otherwise collapse into "the model is good" or "the model is bad."

What's missing from the article is failure modes. Raschka explains what each component does when it works. He doesn't cover what happens when context reduction clips something critical, when working memory drifts from the actual task state, or when a subagent's constraints are too tight to be useful. [[Cognitive Debt]] and [[Slowing the Fuck Down]] fill that gap from the practitioner side.

The claim that vanilla LLMs "have very similar capabilities" is provocative and probably true-ish at the frontier but undersells the gap between frontier and commodity models. A Llama-3-8B dropped into Claude Code's harness won't perform like Opus 4.7. The harness amplifies; it doesn't equalize. [[Honey I Shrunk the Coding Agent]] is more honest here: the harness redesign helped, but it was tuned to the specific model's behavioral profile.

Still: this is the single best conceptual map of what a coding agent actually is. Essential reading.

---
*Sources: [[summary/components-of-a-coding-agent]]*
*Last updated: 2026-05-15*
