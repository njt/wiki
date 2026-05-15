# Context Rot

Roampal's blog post names and diagnoses a problem every agent builder hits: retrieval quality degrades over time because RAG systems optimize for similarity, not for whether the retrieved context actually *helped*. Their solution -- Wilson scoring plus dynamic weighting that shifts from embedding similarity to outcome-based learning as memories prove themselves -- is one of the more rigorous approaches to agent memory I've seen.

---

## Key Quotes

> "They can't learn from outcomes. They optimize for similarity, not success."

## Key Themes

#memory #RAG #agents #retrieval #context-engineering

The five memory banks (working, history, patterns, memory_bank, books) map well to the taxonomy in [[Memory Mechanism]], which identifies session, project, semantic, episodic, and procedural memory types. The key innovation here is the *feedback loop*: rather than treating retrieval as a pure similarity problem, Roampal tracks whether retrieved memories led to successful outcomes and promotes or demotes them accordingly.

The Wilson scoring approach is borrowed from recommendation systems -- it solves the cold-start problem where a memory with 1/1 success looks better than one with 90/100. This is the kind of statistical thinking that's mostly absent from agent memory systems.

The three knowledge graphs (routing, content, action) map to real questions agents face constantly: which collection has the answer, how are concepts related, and which tools work when.

## Critical Analysis

The benchmarks are compelling (0% to 60% on adversarial semantic traps, 1% to 67% on retrieval accuracy), but they're self-reported by the company building the product. The frictionless scoring approach -- inferring success from user responses -- is clever but fragile. "Thanks, that worked!" is easy to detect; ambiguous responses are harder.

The deeper insight is that context windows are a red herring. Bigger windows don't help if you're stuffing them with irrelevant context. This connects to the Hightouch approach in [[How Hightouch Built Their Long-Running Agent Harness]], where they use file buffering and subagent delegation to manage context rather than relying on window size.

Missing: how this handles conflicting memories, and whether the feedback loop creates reinforcement cycles where popular memories crowd out correct-but-rarely-used ones.

---
*Sources: [[raw/context-rot]]*
*Last updated: 2026-05-14*
