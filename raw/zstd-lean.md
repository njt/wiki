---
url: https://www.imperialviolet.org/2026/07/26/zstd-lean.html
title: We have proof automation now
author: Adam Langley (ImperialViolet)
date_fetched: 2026-07-29
date_published: 2026-07-26
---

# We have proof automation now

Adam Langley writes about building a Zstandard decompressor in Lean, using LLMs to automate the proof burden that has historically made dependently-typed languages impractical for everyday software engineering.

## On dependent types and proof burden

Langley expresses long-standing fondness for dependently-typed languages like Coq/Rocq and Lean, which permit encoding "arbitrarily subtle invariants" that in conventional languages end up as comments and get lost as teams grow. He recounts the Coq name change, and an anecdote about suggesting at a Princeton Coq conference that the name was an impediment in the English-speaking world — a suggestion the audience disagreed with at the time.

The core challenge: "with great type-system power comes great proof effort." He describes spending entire days proving simple things, and the particularly galling experience of spending hours proving something only to discover the goal was false. He cites the seL4 retrospective, which found engineers spent "about 10 times as much time proving as they did designing and implementing," ending up with more than 20 times as many lines of proof code as C code.

He discusses F* as an attempt to automate proofs via SMT solvers, comparing it to serving "a complex and fickle god" where users develop a sixth sense for what pleases the solver.

## The key insight: proof irrelevance + LLMs

Proof irrelevance means that once a statement is correct, the proof's contents don't matter — only its existence. The exceptions are "proof engineering" (structuring proofs for resilience to code changes) and the risk of type checker blowup on complicated proofs.

LLMs combined with proof irrelevance "promise to be an extremely capable form of proof automation," potentially making dependent-type systems dramatically more practical. To explore this, he built a Zstandard decompressor in Lean.

## On Zstandard

Zstandard seems to win the competition to replace gzip as the canonical compression utility. It's an LZ77-style compressor with better entropy coding and careful design for impressive decompression speeds. He notes that while bzip2 is more beautiful, "the shining elegance of the Burrows–Wheeler transform doesn't count for too much in the face of significant practical advantages." A chart shows compression ratios vs. decompression throughput, with gzip and Zstandard occupying their own speed class.

The Zstandard RFC is terse, requiring multiple re-readings; he points readers to Nigel Tao's better write-up. He focuses on explaining the entropy encoder.

### Entropy Coding: Huffman vs. FSE

Huffman trees build an optimal prefix tree by repeatedly combining the two least-probable symbols, but they can only use a whole number of bits per symbol. If -log2(p) = 2.3, Huffman forces rounding up or down.

Zstandard's higher-compression entropy encoder, FSE (finite state entropy), uses a state machine with more states than symbols. Each symbol gets a fraction of states mirroring its probability. Each state has three values: the symbol, a number of bits to read, and a baseline state number. While states also read whole numbers of bits, the trick is that if a symbol needs 1.5 bits on average, half its states read one bit and half read two, achieving the target on average. The state table is never transmitted; the RFC prescribes how to build it from symbol probabilities.

He illustrates with a 16-state table of four symbols (A, B, C, D), showing how B — with probability 5/16 and ideal bit cost of ~1.68 — has three states reading two bits and two reading one bit, averaging to nearly the right value. The central insight: giving multiple states to common symbols means the encoder doesn't just pick a symbol but also which of that symbol's states to land in, carrying fractional bits of information forward.

The wrinkle: you can't work forwards; FSE forces starting at the end of the sequence and working backwards, which is manageable since you usually need the whole sequence to compute probabilities anyway. The decompressor must seek to the end of a block and read bits backwards.

Basic entropy encoders ignore inter-symbol probabilities (like Q being followed by U in English). Zstandard handles those redundancies through a traditional Lempel–Ziv structure encoding literals or back references; FSE is primarily used for efficiently encoding back reference offsets and lengths.

## On Lean

Langley explains dependent types through examples: a function reading exactly n bytes from a stream that returns a byte array the type system knows is n bytes long; a more elaborate example returning two numbers and a byte array where the first number is prime, their sum is divisible by six, and the byte array is at least as long as the smaller number — "a type no one will ever need" but demonstrating how far one can take this.

He compares Lean to Haskell: Lean is strict (arguments evaluated before function calls) while Haskell is lazy. He acknowledges laziness's elegance but says it makes performance "hard to reason about." Lean has helpful syntactic sugar — its monadic `do` notation includes for loops, return statements, and break statements, making imperative-style programming reasonable. It also has an optimization for mutating objects in-place when their reference count is one, allowing efficient array mutation. However, Lean lacks linear type system features to guarantee single references, creating a "sharp edge" where minor code tweaks can crater performance.

He shows an example from his zstd decoder — an array index where blockBytes must not be empty. Lean avoids undefined behavior or runtime exceptions by letting you prove non-emptiness at compile time, using a short proof leveraging the fact that RLE blocks always have contentSize equal to one.

### Proving Universal Properties of FSE Table Construction

He describes proving four universal properties of his FSE table construction function against the RFC's algorithm:

1. The table has the correct size for the given accuracy
2. Each symbol has the correct number of states given its probability
3. For all states, reading nbBits and adding the baseline yields a valid state number
4. For every symbol with non-zero probability and every target state, exactly one state for that symbol can reach the target

These properties are the "subtle assumptions that an optimised decoding inner-loop requires" and things that are "only ever implicit or mere comments in weaker type systems." Several LLMs can now produce these proofs automatically in about 20 minutes for a fraction of a $20/month subscription, calling it likely "table-stakes next year." The LLMs needed to change his code — he had used too much imperative-style `Id.run`, which is harder for proof machinery — but the proofs type-check with no `sorry`s.

## Conclusions

He acknowledges that combining dependent types and LLMs isn't new, but applying it to quotidian software engineering needs more experience. Strong types can amplify the scope of changes as they propagate through derived types, and proof effort may scale poorly in larger systems. His toy decoder is 10× slower than the command-line zstd. Still: "proof automation is here now" and we "practically speaking, have a new type of programming language available to us."

He declines to publish his code, noting that for small, well-defined cases like this, the LLMs can probably do better than he did. His work was inspired by [lean-zip](https://github.com/kim-em/lean-zip), which does more, includes a compressor, and proves round-tripping.

## Aside: Verified Assembly

He mentions AWS's LNSym — a semantics and simulator for AArch64 — suggesting it could be used to prove equivalence between optimized assembly and Lean counterparts, then use the assembly at runtime. He wonders if this could make verified assembly "cheap." He experimented with the small Popcount32 example from the repo, which uses `bv_decide` (a certifying SAT solver), but found it required more memory than his system had. Tiny functions do work, and it's possible to get equivalence proofs to small Lean functions and use `extern` to call them at runtime, but he and several LLMs couldn't scale it further.
