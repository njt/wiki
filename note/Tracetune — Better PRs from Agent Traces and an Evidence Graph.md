# Tracetune: Better PRs from Agent Traces and an Evidence Graph

A comic-driven AI Engineer SF 2026 talk (Stephen Chin with Cassandra Chen, Tracetune's creator) arguing that most AI pull requests fail on **description quality, not code quality** — and that the fix is to stop letting the agent write the PR from its own compressed session log, and instead reconstruct what actually happened from the machine's session traces, pre-computed into a graph-backed "evidence packet" by a new embedded Rust graph database, Relate. The result: judge-scored PR quality up (74→82, 76→86) and token cost roughly halved.

---

## The story, briefly

The talk is staged as a cartoon about Alex, a token factory worker whose PRs keep failing review by Mira, with finance (and the "token billionaire program") looming over him. Under the whimsy sits a serious pipeline: Tracetune finds all traces for a branch, researches them, and loops until it has enough evidence, then has an independent writer and reviewer produce the PR — plus, famously, a comic.

## Key quotes

> "It just reads your session log to try to figure out what it should have done using its inflated context window. It actually often... compresses its context window so it gets more and more inaccurate as it goes along."

The diagnosis that carries the talk. The default "ask your agent to write the PR" flow has the agent narrate its own lossy memory of itself — the PR is a compression artifact. This connects directly to [[What a Useful AI Trace Should Actually Contain]]: the trace is the evidence; the session summary is the rumor.

> "The original PR claim was 'no later passing run is recorded.'... in the actual Trace, it shows it running a button test at the same time as the bun x tsc and you can't run those two at the same time."

The worked failure is the best part: even with trace-derived drafting, claims about *what the checks did* diverge from what the trace shows. Verification claims in PRs need to be checked against recorded evidence, not asserted — which is exactly the "review behavior, not diffs" stance of [[The End of Code Review]] taken one step earlier in the pipeline, to the PR text itself.

> "It's common that people always want to have the planner, the reviewer, the checker to be the highest quality model. But if you're doing high quality writing, a lower model, no matter how many iterations and how much feedback you give it, can't do it. It just turns things into a checklist and makes it very mechanical."

The counterintuitive staffing decision: Sonnet-class writer, Haiku-class reviewers. Reviewing against a structured evidence packet is mechanical; prose that survives human approval is not. This is a genuinely useful inversion of the usual "biggest model must be the reviewer" instinct, and it echoes the cost-tiering found in [[harnesseval — Harness vs Prompt for AI Code Review]] and [[RevEngine]] — though those sources put the premium model on *review*, the opposite allocation.

> "Agents are not that good at doing the data analysis, at doing the evaluation and the curation. So if you give them a help at the front end by creating a custom graph layer... suddenly your costs go down, your quality goes up."

The generalizable thesis: replace agent research loops with deterministic pre-computation. Relate ("Archie") is an in-memory Rust graph database doing multi-hop joins over why/code/checks/attributions at zero LLM cost — a retrieval-before-generation move that mirrors [[Context as Infrastructure (Knowledge Compilation for Agentic Retrieval)]]'s argument that context should be compiled, not improvised.

## Key themes

- #concept — PR descriptions as evidence documents, not diff narration; "neighborhood brief" connecting facts to sources
- #pattern — cheap reviewers + quality writer; pre-compute the evidence graph so agents don't research
- #tool — Tracetune (open source PR builder), Relate/Archie (embedded Rust graph DB), PRBench (side benchmark)
- #person — Cassandra Chen, Stephen Chin

## Opinionated take

The strongest idea here is unglamorous: **the PR is a data-reconstruction problem, not a writing problem**. Everyone iterating on "make the agent write a better PR description" is polishing the output of a lossy process; Tracetune attacks the input. The comic framing is a delivery vehicle, but the 50% token reduction from pre-computation is the real argument — it says most of the expensive agent loop was spent re-deriving facts a database could hold.

Two caveats. First, the numbers come from the authors' own PRBench with LLM judges, which is close to grading your own homework; the +4–8 point deltas are plausible but unverified externally, and the talk itself jokes about the benchmark saturating a gaming PC with parallel test runs. Second, the evidence-packet approach presumes rich, well-structured session traces exist in the first place — the "log the CWD before the CD runs" detail implies you must instrument your agents' sessions deliberately, which is a real adoption cost the talk waves past.

The writer-over-reviewer model inversion is the most transferable insight for anyone building multi-model pipelines, and it quietly contradicts several review-at-scale designs in this wiki that spend their best model on judging.

---
*Sources: [[raw/online-stephen-cassandra]], [[summary/online-stephen-cassandra]]*
*Last updated: 2026-11-26*
