# Maybe Coding Agents Don't Need a Bigger Memory

**Santi (oldskultxo) argues that the coding-agent problem isn't memory size — it's continuity between sessions.** Every new session starts cold: re-read the README, re-discover the project structure, re-derive what was tried and what failed. Bigger context windows help while a session is alive, but the moment it ends, the operational thread snaps. The fix isn't remembering everything; it's preserving the *right* kind of information — compact, evidence-weighted, repo-local — so the next session can resume from what actually happened, not from vibes.

---

## The Core Distinction

> Context is what the agent has available *now*. Continuity is what lets the next execution continue from what actually happened before. Those are not the same thing.

Long context helps a live session reason over more material. But when the session ends, gets compacted, switches tools, or starts fresh tomorrow, the same cold-start questions return: What was active? What changed? What failed? What was validated? What was only assumed?

## Why "Remember Everything" Fails

The default instinct — dump everything into chat history → summary → next prompt — rots. Summaries become too broad. Old assumptions mix with verified facts. Failed approaches sit next to successful ones with identical visual weight. The agent retrieves something that sounds related, but nobody knows if it's current, useful, or a hallucination from three sessions ago.

> "More memory" can be worse than no memory because the agent needs operationally trustworthy information and not only related information.

The difference between a memory item and a continuity record:

- **Weak:** "We probably fixed the parser by changing the tokenizer."
- **Strong:** "Task: fix parser edge case. Files edited: `src/parser/tokenizer.py`, `tests/test_parser.py`. Command: `pytest tests/test_parser.py` — passed. Known failure: full test suite not executed. Next: run full parser test group. Evidence: partial."

## What Each Existing Approach Misses

**Instructions files** (CLAUDE.md, etc.) are mostly static — they explain *how* to work in the repo, not what happened ten minutes ago. They don't know a task is paused or that the last validation failed.

**Vector databases** retrieve semantically similar chunks, but coding-agent continuity isn't a semantic retrieval problem. The most important facts are small, boring, and operational: `npm test failed with TS2322`, `migration file was inspected but not edited`, `user said don't touch auth middleware`. A vector store may retrieve something related, but relation isn't provenance.

**Chat history** feels like continuity because it contains the conversation, but a conversation isn't execution state. It mixes useful facts with abandoned ideas, outdated plans, user corrections, and "we should do X" statements that never became real. And it's often bound to one provider, one tool, one session.

## The Repo Is the Natural Boundary

> The state that matters should live with the project. Continuity should not belong only to the conversation. It should belong to the repository.

Repo-local continuity artifacts are inspectable, correctable, and tool-agnostic. Another compatible agent can read them. The user can review them. The memory isn't trapped inside one chat session.

## What Agents Actually Need: A Compact Execution Surface

After iterating on a repo-local continuity runtime (AICTX) on a large production codebase, the useful pieces clarified:

**Resume payload** — active work state, relevant decisions, known failures, validation expectations, structural entry points, stale/unverified warnings, next action.

**Finalize payload** — files touched, commands executed, tests observed, failures learned/resolved, decisions made, unresolved risks, next handoff.

> The better the continuity layer became, the *less* I wanted it to return. The best resume payload is not the largest one. It is the one that gives the agent enough operational grounding to avoid starting cold.

## Provenance Over Volume

> If a continuity layer cannot say "this is stale", "this is unverified", "this was demoted", or "this lacks validation evidence", then it is too trusting and that is dangerous.

Evidence-weighted continuity: runtime observed, agent claimed, validation supports, user corrected, later work contradicted, still unknown. The goal shifts from "believe memory" to "use it with the right weight."

## The Two Turning Points

**Failure memory** was the first breakthrough. Agents waste enormous time repeating plausible mistakes — opening the wrong file because the name looks right, running the same command that failed before, following a path abandoned two sessions ago. A failure memory layer doesn't say "never do this" but "this failed before — here's the command, the error, the area, whether it was later resolved. Treat as context, not truth."

**Work State** handles unfinished work, which is different from a handoff summary or decision record. It preserves the live thread: task hypothesis, relevant files, current status, next action, risks, recommended validation, unverified gaps, branch context. "This is where continuity starts to feel operational rather than documentary."

## Execution Contracts and Guardrails

Execution contracts provide soft guidance for the next session — a route, not a law. If the route isn't followed, that becomes a signal. The system can compare expected vs. observed execution: "canonical validation was not observed", "edited outside expected scope", "first action was skipped."

Guardrails should be compact and appear only at **boundaries**: before first edit, before risky command, before final answer, before finalize, when scope changes, when switching agents. A guard should answer a compact question: "Is this action aligned with current continuity?" Possible outputs: allow, caution, re-ground, block. "Most of the time, the answer should be *allow*. If the guardrail becomes louder than the work, the system has failed."

## The Economics

Continuity has overhead — resume payloads cost tokens, finalize summaries cost tokens, guard calls cost attention. Not every task needs it:

| Scope | Value |
|---|---|
| 1–2 prompts | Probably not worth it |
| 3–7 prompts | Break-even zone |
| 7+ prompts | Increasingly useful |
| Multi-session | Strong use case |
| Cross-agent | Very strong use case |

Continuity pays for itself when it prevents repeated orientation, wrong-path exploration, and re-derivation of previous decisions. The real saving isn't "fewer tokens" — it's less wasted cognition and operational amnesia.

## The Hardest Part Is Pruning

> Capturing memory is easy. The hard part is deciding what deserves to survive.

A system that remembers everything will eventually force the agent to rediscover what matters *inside* the memory itself. That's just moving the cold start to another folder. Continuity should optimize for reuse, not accumulation.

---

## Key Themes

- #continuity — the missing primitive between agent sessions, distinct from memory
- #context-engineering — context is RAM, continuity is the filesystem
- #agent-architecture — repo-local artifacts, lifecycle design, evidence-weighted state
- #failure-memory — remembering what went wrong matters more than remembering what succeeded
- #work-state — preserving unfinished work as operational handoff, not documentary summary
- #provenance — was this observed, validated, assumed, or contradicted?
- #tool — AICTX, the author's open-source continuity runtime
- #pattern — resume → work → finalize lifecycle; execution contracts as soft guidance

## Critical Analysis

Santi has put his finger on something real that the "just make the context window bigger" crowd misses. The essay is strongest where it's most concrete: the distinction between "we probably fixed the parser" and a structured continuity record, the economics table for when continuity pays off, the observation that guardrails should mostly say "allow." These are the marks of someone who's actually built and used the thing, not just thought about it.

The architecture sketch is deliberately boring, which is the right call — this shouldn't be exotic infrastructure. But the essay is weaker on what happens when continuity records themselves accumulate noise over weeks and months. The pruning section acknowledges the problem but doesn't solve it. That's fair — pruning what matters is genuinely the hard part — but it means the system still depends on discipline, and discipline doesn't scale.

The MCP interface is smart but underspecified. Exposing continuity through local MCP tools makes it tool-agnostic, which is the right bet, but the essay doesn't address what happens when different agents write conflicting continuity records. In a multi-agent scenario, who owns the truth?

The essay's real contribution is reframing the problem. "Big memory vs. small memory" is a boring argument. "Context vs. continuity" is sharper and more actionable. It's a lens that makes you re-examine every memory system you've built: are you helping the agent *remember*, or helping it *continue*? Those are different design goals.

---

## Related

- [[Agent Memory and Context]] — Hub page: context management as the real engineering challenge, memory taxonomies
- [[Agent Coding Workflow]] — The practitioner's daily loop; continuity is what makes compound engineering possible
- [[napkin]] — The simplest continuity primitive: a markdown file per repo where the agent logs its mistakes
- [[Planning With Files]] — "Context Window = RAM; Filesystem = Disk" — the RAM/disk metaphor Santi extends
- [[How Hightouch Built Their Long-Running Agent Harness]] — Context compression, subagent isolation, the fanout pattern
- [[Memory Mechanism]] — Five-type memory taxonomy; instruction vs. learning memory distinction
- [[StrongDM Factory Techniques]] — Filesystem-as-memory pattern from the dark factory floor
- [[Slate]] — Thread-and-episode architecture for long-horizon agent tasks; context routing as core primitive
- [[Honey I Shrunk the Coding Agent]] — Empirical proof that the harness matters more than the model
- [[Components of a Coding Agent]] — The harness matters more than the model; six-component taxonomy
- [[Claude Code Mastery]] — CLAUDE.md as compounding infrastructure; the mental model flip
- [[Agent Identity]] — Memory is retrieval; identity is participation
- [[Guardrails and Feedback Loops]] — Deterministic enforcement at boundaries, not instructions everywhere
- [[Smart Models Dumb Pipes]] — The end-to-end principle applied to AI; smart models own decisions
- [[Elements of Agentic Systems Design]] — Memory as one of ten elements in agentic system design

---

*Source: [Maybe Coding Agents Don't Need a Bigger Memory. Maybe They Need Continuity.](https://oldskultxo.substack.com/p/maybe-coding-agents-dont-need-a-bigger) — Santi (oldskultxo), Substack, 2026-06-05. Also published on [dev.to](https://dev.to/oldskultxo/maybe-coding-agents-dont-need-a-bigger-memory-maybe-they-need-continuity-3327). Reference implementation: [AICTX](https://github.com/oldskultxo/aictx).*
