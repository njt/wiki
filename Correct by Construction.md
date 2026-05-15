# Correct by Construction

A data quality methodology that treats clean data as a whitelist problem: instead of validating messy data after the fact, you build datasets that are correct by construction. Decompose everything into anchors (entity IDs), attributes (values), and links (relationships), then enforce completeness, uniqueness, and no-nulls at each level.

---

## Key Quotes

> "I understand that this is not how things are commonly done, but I assume that you have a problem with data quality, and you want to try a different approach."

## Key Themes

#data-quality #database

The three-tier framework is practical. Tier 1 (input quality for specific queries) is achievable today on any team. Tier 2 (data currency) requires ongoing process. Tier 3 (all data, all queries) is the aspirational target that most organizations never reach.

The anchor/attribute/link decomposition is essentially a normalized data model, but framed as a quality methodology rather than a schema design pattern. By forcing every dataset into two-column form (ID + value) and banning NULLs, sentinel values, and unrestricted JSON, it eliminates entire categories of downstream bugs. The insight that "sentinel values" like empty strings and "UNKNOWN" are just NULLs in disguise is especially sharp.

This connects to [[Write Snapshot Isolation]] in philosophy: both argue that correctness should be structural rather than detected after the fact. It also connects to [[Data Engineering for Large Models]] -- the LLM training pipeline needs exactly this kind of disciplined data curation.

## Critical Analysis

The framework is sound but deliberately radical. Banning NULLs and JSON containers outright will make many practitioners recoil -- real-world data is messy, and sometimes a NULL genuinely means "unknown." The author acknowledges this is unconventional, which is honest. The real question is whether the overhead of canonicalization and strict decomposition pays for itself in reduced downstream debugging. For analytics-heavy workloads where bad joins and wrong aggregations cost real money, the answer is clearly yes. For exploratory work and rapid prototyping, this level of discipline is probably overkill.

---
*Sources: [[raw/correct-by-construction]]*
*Last updated: 2026-05-14*
