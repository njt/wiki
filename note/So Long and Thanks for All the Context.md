# So Long and Thanks for All the Context

Andrew Stellman's definitive practitioner's guide to the U-shaped context problem — the finding that LLMs attend best to the beginning and end of their context window and worst to the middle. The fourth installment of his O'Reilly "context management trilogy" (yes, four articles), this one moves from diagnosing context loss to prescribing five concrete techniques. The core insight is that context loss and lost-in-the-middle are fundamentally the same problem: working-memory unreliability. The fix isn't bigger windows — it's treating context like RAM and the filesystem like disk.

---

## Key Quotes

> "Bigger windows have made simple single-fact retrieval much better. They have not made long-context agent work reliable."

The single most important sentence in the piece. It cleanly separates the metric everyone benchmarks (needle-in-a-haystack recall) from the thing everyone actually needs (sustained reliability across long sessions). This is the same distinction Santi makes in [[Maybe Coding Agents Don't Need a Bigger Memory]] between context and continuity — just approached from the architecture side rather than the state side.

> "A two-million-token window means a bigger middle to fall into."

Stellman's pithiest line. The mathematical finding that the U-shape exists at initialization — before any training — means this isn't a training artifact you can fix with more data. It's structural to transformer attention. Bigger windows don't shrink the middle; they create more of it.

> "Don't run one long session. Run many short ones, each reading fresh from disk."

The operational translation of "context is RAM, filesystem is disk." This maps directly to the [[Planning With Files]] pattern and the resume-work-finalize lifecycle in [[Maybe Coding Agents Don't Need a Bigger Memory]]. Stellman's contribution is the emphasis on *fresh restarts as a feature, not a workaround* — build the expectation of running out of context into the process design.

> "Do not paraphrase from memory."

Four words that fixed a persistent bug. The Quality Playbook was producing skeletal stub files because the agent was summarizing from its internal state rather than re-reading the source. The fix wasn't a better prompt or a bigger window — it was forcing a fresh read at the point of use. This is the context equivalent of [[Guardrails and Feedback Loops]]: deterministic enforcement beats instructions.

> "If your agent's ability to do its job depends on information, that information needs to live somewhere more durable than working memory."

The closing line that ties it all together. From 32KB core memory in the 1970s to 2M-token context windows in 2026 — the lesson is the same.

## Key Themes

- **#context-engineering** — The U-shape as a structural property of transformer attention, not a training artifact
- **#pattern** — Five techniques: curate don't accumulate, position at edges, short sessions, restate at point of use, test the middle
- **#concept** — Context brief: a separate document capturing everything a fresh session needs, serving as both working spec and reproducible audit trail
- **#pattern** — Short sessions over long ones: treating the agent as a pipe, not a database; state lives on disk
- **#concept** — Primacy bias and recency bias as the twin forces creating the U-shape; the middle as the dead zone
- **#tool** — Claude Code's `--append-system-prompt` as a practical implementation of edge-positioning

## The Five Techniques

1. **Curate, Don't Accumulate** — Clear context and reload with only what matters. Write a context brief. Start fresh sessions against the brief. This isn't just about token count; fresh context enforces stricter instruction adherence and creates a reproducible audit trail.

2. **Position Critical Information at the Edges** — Put load-bearing information at the beginning and end of context. Use `--append-system-prompt` to place it where the model attends most. The middle is for less important material. This is [[Steering Claude Code]] in practice: instruction placement as a first-class design decision.

3. **Short Sessions Over Long Ones** — "Don't run one long session. Run many short ones, each reading fresh from disk." Stellman built a system using Haiku 4.5 to summarize multi-tool chat history, with a cursor-based resume protocol. The breakthrough was accepting that the session *will* run out of context — build fresh restarts into the process.

4. **Restate Key Info Close to the Point of Use** — The fix for the stub-file bug: tell the agent to re-read the source file before writing. "Do not paraphrase from memory." Force a fresh read at the moment of writing. This is the context equivalent of a cache flush.

5. **Test the Middle** — Run deterministic checks comparing what the agent claims to know against what's on disk. When `progress.json` disagrees with the actual last line written, flag and stop — don't build on broken state.

## Critical Analysis

Stellman has done something unusually useful here: he's written the piece that bridges academic research (Liu's "Lost in the Middle," the ICML structural proof, Chowdhury's initialization finding) with the day-to-day reality of someone running an agent on real work. Most practitioner pieces skip the research; most research pieces skip the practice. This one does both, and the five techniques are field-tested rather than theorized.

The context brief technique is the most original contribution. It's not just "clear context and start fresh" — it's a designed artifact with three explicit properties (self-contained, enforcement, auditability) that make it more than a summary. It sits between [[Planning With Files]]'s markdown planning and [[napkin]]'s per-repo scratchpad, adding the audit trail dimension that neither emphasizes.

The weakness is the same one that haunts all context-management advice: discipline doesn't scale. Every technique requires a human to decide what matters, write a brief, position information at edges, restart sessions, and write verification checks. Stellman acknowledges this implicitly by describing systems he *built* to automate parts of it (the Haiku summarizer, the resume protocol), but the techniques themselves remain manual. The next step — which nobody has shipped — is automating the *discipline* itself: an agent that writes its own context briefs, decides when to restart, and positions information at edges without being told.

The "lost in the middle" finding also has an uncomfortable implication for multi-agent orchestration. If every agent in a pipeline has its own context window, and each one has a U-shaped attention profile, then information handed off between agents is always in someone's middle. The pipeline architecture that solves one problem ([[Agent Orchestration]]) may be creating another one that nobody is measuring.

A structurally different bet on the same problem: [[Recursive Language Models]] automate the decomposition that Stellman does manually. Instead of curating, positioning, restarting, and restating by hand, an RLM gives the model a REPL environment with the context as a variable and lets it decide at test time how to peek, grep, chunk, and delegate. The trade is designer discipline (Stellman's approach) for model capability (Zhang's approach) — and Zhang's early results (GPT-5-mini + RLM > GPT-5 alone) suggest the model-capability side of that trade is improving faster than anyone expected.

The connection to [[Engineering for Bounded Cognition]] is underappreciated. The U-shape is a machine-learning finding, but it's also a cognitive one: human working memory has primacy and recency effects too. The techniques Stellman prescribes (curate, position, restart, restate, verify) are essentially the same strategies humans use to compensate for bounded working memory. The difference is that humans have metacognition — we know when we've forgotten something — and LLMs don't. The agent doesn't know it's in the middle.

## Connections

The article's five techniques map cleanly onto existing wiki concepts:

- **Curate/context brief** → [[Planning With Files]] (context = RAM, filesystem = disk), [[Coding Agents Continuity Not Memory]] (repo-local state artifacts)
- **Position at edges** → [[Steering Claude Code]] (instruction placement as design decision), [[Matt Pocock — Grill Me, Then Go AFK]] (smart zone/dumb zone model)
- **Short sessions** → [[Maybe Coding Agents Don't Need a Bigger Memory]] (resume-work-finalize lifecycle), [[How Hightouch Built Their Long-Running Agent Harness]] (subagent isolation)
- **Restate at point of use** → [[Guardrails and Feedback Loops]] (deterministic verification over prompt-based instruction)
- **Test the middle** → [[Context Rot]] (outcome-based retrieval scoring), [[Giving Claude Agent Memory in 12 Steps]] (Dreaming's consolidation as ground-truth check)

---
*Sources: [[raw/so-long-and-thanks-for-all-the-context]]*
*Last updated: 2026-07-18*
