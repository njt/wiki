# Slate

Random Labs' technical report on Slate, a terminal-based coding agent that replaces the ReAct loop with a thread-and-episode architecture. The core insight: the bottleneck in long-horizon agent tasks is context management, not model intelligence. Slate solves this by treating threads as OS processes and episodes as compressed return values, giving the orchestrator explicit control over what context flows where. The AlphaZero analogy -- tactics learned first, strategy later, in separate network layers -- is the most compelling framing I've seen for why agents need both planning and execution infrastructure.

---

## Key Quotes

> "The real bottleneck in long-horizon agentic tasks is context management, not model intelligence. Models are already capable enough to solve many more tasks than they currently succeed at due to the knowledge overhang. The gap is a systems problem, not a capability problem."

This is the central thesis, and it aligns with what [[How Hightouch Built Their Long-Running Agent Harness]] and [[Scaling Long-Running Agents]] both discovered independently. The claim is stronger here though: Random Labs argues there's a "knowledge overhang" -- knowledge the model has but can't access without architectural support like planning files or thread decomposition.

> "In several games AlphaZero sacrificed pieces for long-term strategic advantage, suggesting that it has a more fluid, context-dependent positional evaluation than the rule-based evaluations used by previous chess programs."

Borrowed from Silver et al., but the application is sharp: strategy and tactics in software engineering are analogous to value networks and policy networks in game-playing AI. Running a bash command is a tactic. Designing a backwards-compatible schema is strategy. Most AGENTS.md rules are tactical.

> "Episodic memory is the compressed representation of a completed episode: only the important results are retained, not the full tactical trace of every step taken to reach them."

This is what distinguishes Slate's threads from naive subagents. Claude Code and Codex subagents return a single response string. Slate's episodes carry structured context that can be composed -- one thread's episode becomes another thread's input.

> "We can describe this as knowledge overhang. The knowledge that a given model has access to theoretically, but can't access tactically without a trick like 'think step by step' or by planning in files."

A new term worth tracking. It reframes chain-of-thought and markdown planning not as prompting tricks but as retrieval mechanisms for the model's own latent knowledge.

> "Compaction is largely unsolved. Most is actually lossy compression (despite having gotten better)."

They list Claude Code's compaction as "notoriously bad," the Geoffrey Huntley Ralph Wiggum loop, and Amp handoffs. The honesty about compaction being unsolved is refreshing when most tools pretend it's a solved problem.

## Key Themes

### Architecture: Threads and Episodes #concept #pattern

Slate's core primitive is the **thread** -- an isolated execution context that the orchestrator can dispatch, monitor, and compose. Each thread's work produces an **episode** -- a compressed summary of what happened, stripped of the tactical trace. This maps to Karpathy's LLM OS framing: the orchestrator is the kernel, threads are processes, episodes are return values, and the context window is RAM.

The critical difference from subagents: threads share context explicitly. The orchestrator passes context *in*, and episodes come *back out*, composably. One thread's episode can initialize another thread. Naive subagents (Claude Code, Codex) only pass back a response string, losing the structured context.

### Strategy vs. Tactics #concept

The AlphaZero analogy is the report's intellectual backbone. McGrath et al. (PNAS 2022) showed that AlphaZero learns tactical concepts (material value) in the first 32k training steps, then strategic concepts (king safety, mobility) at 32k+, and sophisticated trade-offs only beyond 128k steps. They emerge in different layers of the network.

Random Labs maps this onto software engineering as a spectrum: running a bash command (pure tactic) through writing a test suite, planning a refactor, to designing a schema (pure strategy). When you ask a model to plan, you're actually triggering knowledge retrieval (strategy) before execution (tactics).

### Knowledge Overhang #concept

A new concept: the gap between what a model knows in principle and what it can access during agentic execution. Chain-of-thought, "think step by step," and markdown planning are all ways to close this gap -- they're retrieval mechanisms for the model's own knowledge. As models are trained on more planning-style outputs, this overhang should shrink, but Random Labs argues it will never fully close.

### Compaction is Unsolved #concept

The report is admirably honest that compaction (context compression) is lossy and unpredictable. Claude Code's compaction is "notoriously bad." The Ralph Wiggum loop (agents repeating themselves after compaction) is real. Amp's handoffs are the best current approach because they bootstrap fresh sessions rather than trying to compress. Slate sidesteps compaction entirely by making episodes the natural compression boundary.

### The Dumb Zone and Context Rot #concept

Dex Horthy's "Dumb Zone" term for the degraded portion of the context window, combined with Hong, Troynikov & Huber's [[Context Rot]] research showing all frontier models degrade non-uniformly as input length grows. Working memory = context window minus the Dumb Zone. This is the fundamental constraint that forces architectural solutions.

## Critical Analysis

**What's strong:** The AlphaZero-to-software-engineering mapping is genuinely insightful and well-supported by the McGrath et al. research. The thread/episode architecture is a clean answer to the subagent composability problem -- it's the first architecture I've seen that treats inter-agent context routing as a first-class design concern rather than an afterthought. The "knowledge overhang" concept names something real that the field has been working around without naming.

**What's missing:** No benchmarks. They explicitly punt on formal evaluation ("we leave a formal analysis and benchmarking of this routing behavior as future work"). For a technical report, this is a significant gap. The claim that "the models seem to understand how to route context" without explicit training is extraordinary and deserves evidence. The "interesting observations" section (parallel execution, cross-model composition) reads like marketing copy rather than engineering findings. No comparison to [[Cord]]'s spawn/fork/ask primitives or [[maestro]]'s queue-based coordination, which solve similar decomposition problems differently.

**How it connects:** This is the architectural theory behind the same problems [[Scaling Long-Running Agents]] found empirically (flat coordination fails, hierarchy works) and [[How Hightouch Built Their Long-Running Agent Harness]] solved pragmatically (subagent threads as "scratch paper"). The episode concept extends what [[Planning With Files]] does with markdown -- externalizing state -- but makes it a runtime primitive rather than a file convention. The knowledge overhang concept explains *why* [[Feedback Loop is All You Need]]'s "linters beat prompts" works: linters close the tactical gap so the model can focus on strategy. [[LongHorizon-Harness]] realizes a production version of the episode idea for computer use — fresh-context role episodes whose "return value" is an auditor report selectively re-injected, with filesystem-grounded read-only verification standing in for Slate's unbenchmarked context routing.

The thread/episode pattern is essentially what [[Process-Based Concurrency BEAM OTP]] describes at the language level -- BEAM processes with message passing, supervision trees, and isolated heaps. Random Labs rediscovered Erlang's actor model for LLMs, which is either validation or a sign they should read Joe Armstrong.

## Related Pages

- [[Context Rot]] -- the research Slate cites on context window degradation
- [[Agent Memory and Context]] -- synthesis page; Slate's episodes are a new entry in the memory taxonomy
- [[Scaling Long-Running Agents]] -- Cursor's empirical finding that hierarchy beats flat coordination
- [[How Hightouch Built Their Long-Running Agent Harness]] -- subagent threads as "scratch paper"
- [[Planning With Files]] -- the simpler predecessor pattern (context window = RAM, filesystem = disk)
- [[Memory Mechanism]] -- xAI's taxonomy; episodes are closest to "episodic memory" type
- [[Process-Based Concurrency BEAM OTP]] -- BEAM's actor model is what this architecture reinvents
- [[Cord]] -- dynamic task tree coordination with spawn/fork/ask primitives
- [[maestro]] -- multi-agent team with role-based queue
- [[Feedback Loop is All You Need]] -- linters close the tactical gap
- [[Agent Orchestration]] -- synthesis page on multi-agent coordination patterns
- [[Distributed Systems]] -- agent orchestration IS distributed systems

---
*Sources: [[summary/slate-moving-beyond-react-and-rlm]]*
*Last updated: 2026-05-14*
