# Immaculate Knowledge Graph

Harper Reed's field report on bootstrapping a personal knowledge graph from ~600 meeting transcripts using Claude Code, a 2389 Research summarization skill, and Obsidian. A practical, lazy-first recipe for turning accumulated conversation data into a browsable graph of people and concepts — and a striking example of how little upfront organization you actually need.

---

## Key Quotes

> "Laziness is key. Just make it work for you."

Reed cites Steph Ango's post on Obsidian usage to argue against over-optimizing organization. The knowledge graph emerged from dumping transcripts into a pipeline and letting it churn — not from designing a taxonomy upfront. This is the opposite of [[jibrain Knowledge Architecture]]'s three-tier curation pipeline with strict containment boundaries.

> "I checked my Obsidian vault and realized that my last note was from 2022 with only one word: hungry."

The reality check that kicks off the whole project. Two years of meeting transcripts existed in Granola but zero notes in Obsidian. The pipeline didn't replace a note-taking habit — it created one retroactively from exhaust data.

> "This could work with any transcript source, not just Granola."

The most important practical insight. The architecture (extract → parse → wiki-link → observe the graph) is source-agnostic. Meeting transcripts are just the input Reed had available; images, videos, or any text-producing source would work.

---

## Key Themes

#knowledge-graph #obsidian #claude-code #person #batch-processing

**The lazy-first knowledge pipeline.** Reed's 5-step process: stop optimizing, use conversational interfaces, get data local, parse into Obsidian format, let it churn. The entire thing is built on existing tools (Granola for capture, muesli CLI for extraction, 2389's summarization skill for parsing, Obsidian for display). No novel infrastructure.

**Meeting exhaust as knowledge feedstock.** The insight that your meeting transcripts already contain a latent knowledge graph — who you talk to, what concepts recur, which people cluster around which topics. Extracting it is a batch processing problem, not a note-taking discipline problem.

**Conversational interfaces over file browsers.** Reed's argument that interacting with your knowledge base through Claude Code (or similar) beats navigating a file tree. This is the same interface thesis behind [[Wuphf — Karpathy-Style Agent Wiki]]'s agent-as-reader pattern.

---

## Critical Analysis

This is a *recipe*, not a system. Reed's post has the right energy — just start, don't over-design, use what you have — but it skates past the hard problems that any lasting knowledge graph needs to solve.

**What's missing:** Deduplication (the same person mentioned across 50 meetings will appear as 50 slightly different name variants), factuality (co-occurrence is not truth — two people in the same meeting doesn't mean they're meaningfully connected), and staleness (the graph ages but there's no mechanism for decay or update). [[NornicDB]]'s built-in memory decay and [[Context Rot]]'s dynamic weighting would address the last problem; nothing in Reed's pipeline addresses the first two.

**The muesli gap.** Reed built a custom Rust CLI to extract Granola transcripts. That's exactly the kind of bespoke integration work that makes or breaks these pipelines — and it's the step most readers can't replicate. The generalizable insight ("any transcript source works") is true, but only if you can get the data onto disk first.

**The 98% human claim matters.** Reed explicitly labels the post "written 98% by a human." This matters in a world where AI-generated content is the default suspicion. It also quietly underscores the point: the knowledge graph was machine-extracted, but the *writing about* the knowledge graph was human. That boundary — machines can surface patterns, humans should narrate them — is worth preserving.

**Relationship to other knowledge architectures:** Reed's approach sits at the opposite end of the spectrum from [[jibrain Knowledge Architecture]]'s formal three-tier pipeline with seven-gate health audits. Where Joi builds containment, Harper builds a firehose. Both work; the difference is whether you have one person's meeting transcripts or a multi-agent system writing to a shared vault.

The post pairs naturally with [[Claude Code on the Go]] — Harper's other piece about remote Claude Code from a phone. Together they sketch a coherent vision: AI agents as always-available knowledge infrastructure, not just coding assistants.

---

*Sources: [[summary/imaculate-knowledge-graph]]*
*Last updated: 2026-05-15*
