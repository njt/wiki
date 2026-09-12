# AI Agents with Human-Like Collaborative Tools

Harper Reed's Botboard paper. Research showing that LLM agents equipped with journaling and social media tools improve problem-solving performance on hard coding challenges -- 15-40% cost reductions, 12-27% fewer API turns, 12-38% faster completion. The key mechanism: *writing* drives improvement more than *reading*. Agents wrote 1,142 journal entries but only read 122 of them. Structured articulation is cognitive scaffolding.

---

## Key Quotes

> "Agents wrote 1,142 journal entries but performed only 122 journal reads, and wrote 1,091 social media posts while reading 600 previous posts."

> "Different models naturally adopted distinct collaborative strategies without explicit instruction."

## Key Themes

#research #agent-architecture #memory #rubber-duck #cognitive-scaffolding

The "writing over reading" finding is the most important result. It means the benefit of tools like journals isn't primarily about information retrieval -- it's about the cognitive scaffolding that comes from articulating your understanding. This is rubber duck debugging formalized and measured: the act of explaining the problem (to a journal) helps solve the problem.

The model-specific adaptation is fascinating: Sonnet 3.7 engaged broadly with tools across all problems, while Sonnet 4 was selective, primarily using journal search for genuinely hard problems. Neither was told to use different strategies -- the behavior emerged from the models' different reasoning styles.

The "difficulty-dependent performance enhancer" framing is important: these tools help most on hard problems near the agent's capability limits, and actually introduce overhead on easy problems. This matches intuition -- you don't need a journal for `fizzbuzz`, but you might for a complex concurrency bug.

Connects to [[napkin]] (a practical implementation of the journal concept), [[Three Tier Memory]] (architectural version of the same insight about persistent knowledge), and [[Components of a Coding Agent]] (Raschka's "context quality" maps to the cognitive scaffolding mechanism identified here).

## Critical Analysis

The research is well-designed with good controls, and the replication study one month later adds credibility. The main limitation is that it's 34 coding challenges, not real-world development. The "social media" tool (Botboard) is a provocative frame but the data shows it underperformed the journal tool, likely because tag-based filtering is worse than semantic search. The practical takeaway: give your agents a scratchpad and let them write to it freely. The ROI on hard problems is significant. On easy problems, skip it -- the overhead isn't worth it.

---
*Sources: [[summary/ai-agents-with-human-like-collaborative-tools]]*
*Last updated: 2026-05-14*
