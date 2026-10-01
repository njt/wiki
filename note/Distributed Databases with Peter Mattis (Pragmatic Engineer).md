# Distributed Databases with Peter Mattis (Pragmatic Engineer)

Peter Mattis — GIMP co-creator, Gmail's original storage engineer, Colossus founding team, Cockroach Labs CTO — spends 100 minutes connecting three decades of systems work to the agentic-coding present. The interview's spine: the same B-tree keeps reappearing (Gmail threading, the STL map replacement, Go's Swiss-table era, CockroachDB's range index), and the same person who once wrote 100k lines a year by hand now gets "database-quality" code out of agents — with firm testing discipline as the difference between magic and slop.

---

Mattis walked through building Gmail's backend on GFS (threading, B-trees, inverted index), seeded the build system that became Blaze/Bazel, then joined Colossus: erasure coding with Reed–Solomon at ~2x overhead versus 3x triplication, metadata in BigTable, and a bootstrap hack (a foundational BigTable not on Colossus) that survived for years — proof that "you can make hacks that go really long knowing that they're hacks." At CockroachDB he explains why append-only storage forces immutable files and LSM trees (LevelDB → RocksDB → Pebble), why consensus needs three replicas minimum, why strong consistency means writing to all replicas *fast* rather than avoiding it, and how range sharding over a contiguous key space is "a B-tree if you squint."

The AI half is where this source earns its place in an agent-centric wiki: a CTO who stopped coding in 2022, came back for Opus, and now runs 5–10 concurrent sessions with up to ~100 subagents, demands advanced testing from agents, and predicts humans will stop reviewing code the way we stopped reviewing assembly.

---

## Key quotes

> "You might want to say, like, I want to have eight replicas of this data... nine chunks of data, but any five of those chunks can be used to reconstruct it. And what this means is you can lose any four copies and you can still reconstruct your data."

Erasure coding explained in one breath — the Colossus payoff was *less* storage (2x vs 3x) with *higher* redundancy, standing on Reed and Solomon's 1970s math. The lesson he draws: proven algorithms are the cheap part; "you just take that and have to do all the engineering behind it to make it work in a storage system."

> "If you squint, everything's either a B-tree or it's a hash table."

The interview's running joke is actually its thesis. Gmail threading, the STL-map replacement at Google, and CockroachDB's shard index are all B-trees at different scales, and the "Ubiquitous B-tree" paper from the 1980s still holds. When a data structure keeps resurfacing across 30 years, that's the curriculum.

> "And you can't actually have consensus when you only have two replicas... if I'm on the secondary, how do I know if something was written to the primary?"

A clean, almost-painless exposition of why three is the magic number, with the detail practitioners often miss: reads normally come from one replica, writes go to all three, and the consensus read only happens during recovery.

> "This is a thing that I hope the model providers are listening to. But you need to give them a firm hand. They kind of get a little bit lazy on the testing side."

His quality thesis: defects scale with lines produced, so agentic output *will* carry more defects unless testing discipline is imposed. And the twist — "it's actually easier with agents" than with humans, because agents don't get tired of being told to use property-based, metamorphic, and deterministic simulation testing. This is the same shape as [[How SQLite Tests Software]] applied to agent harnesses rather than a codebase.

> "I suspect that... we're materially going to stop looking at the code in the same way that we don't look at assembly anymore."

But with a caveat he demonstrates live: he had Fable build a guardrail that checks decompiled output to verify a zero-overhead abstraction stayed zero-overhead. Review isn't deleted; it migrates from reading diffs to building machine-checkable constraints — the same move as [[The End of Code Review]] but from inside a database company that ships mission-critical systems.

> "If you're going to ask Fable or Astra, build me a distributed database like CockroachDB. You will get something out, but it'll kind of be ultimately hollow inside."

The wizard analogy: domain experts are "sorcerers" whose incantations produce magic while novices get "sparkle stuff." Amplification, not levelling — the same claim as [[An Honest Review of AI Programming]]'s pessimism but from the opposite baseline, and echoed by the Terence Tao ChatGPT-session anecdote.

## Key themes

- #concept — Erasure coding over triplication: Reed–Solomon at ~2x with better redundancy than 3x replicas
- #pattern — The bootstrap hack: Colossus metadata in BigTable, BigTable on Colossus, broken by a foundational instance — long-lived hacks as legitimate system architecture
- #pattern — Firm-hand agent management: agents are lazier about testing than humans but more responsive to discipline
- #concept — Consensus minimums and the recovery-time consensus read
- #person — Peter Mattis, as the archetype of the domain expert amplified

## Opinionated take

This is the most credible "AI output is high-quality" claim in the corpus precisely because Mattis names the failure modes and the mechanism. "My output is insane and it's not vibe-coded junk" is a claim everyone makes; "agents are lazy about testing, here are the four testing techniques I demand, and it's easier to discipline them than humans" is an operational claim you can copy. The catch he underplays: his expertise is the guardrail. A CTO who has implemented a dozen B-trees reviewing agent output is not the median engineer — the "hollow inside" failure happens when the sorcerer's incantations aren't there. His own CockroachDB example proves the model can't do distributed databases alone; the interview's AI enthusiasm rests on 30 years of accumulated taste.

The systems half is a gift for anyone who only knows these systems by name: the GFS→Colossus arc as a concrete scalability story (single master → distributed metadata), the storage-system/database distinction that explains why LSM trees exist at all, and the speed-of-light digressions (Starlink beats fibre; HFT firms bounce microwaves between Chicago and New York) that turn "latency" from an SLA number into physics.

## Related pages

This source strengthens [[Distributed Systems]] with a practitioner's first-hand account of Colossus, erasure coding, three-replica consensus, and range sharding — concrete history behind that topic's abstractions. It nuances [[How SQLite Tests Software]]: SQLite's famous discipline gets an agentic reading, where the discipline is imposed on models that are lazier than humans but more obedient about advanced testing. It complicates [[The End of Code Review]] with a date and a mechanism — assembly-level irrelevance plus machine-checked guardrails — rather than a prediction. And it supports [[The GUS Stack — Go, Unix, SQLite]] from the other side: Mattis contributed to the Go runtime itself (Swiss tables from a long flight to Bangalore), evidence that boring, deep infrastructure keeps repaying attention.

---
*Sources: [[raw/distributed-databases-with-peter-mattis]], [[summary/distributed-databases-with-peter-mattis]]*
*Last updated: 2026-10-02*
