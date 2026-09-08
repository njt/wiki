# Automatic Layout of Railroad Diagrams

The first formal treatment of railroad (syntax) diagram layout as a compilation problem, by Chiplunkar and Pit-Claudel at EPFL. A diagram language of conceptual components compiles through three passes — alignment, wrapping, justification — into a layout language of sized, positioned shapes. The wrapping step is framed as a principled optimization problem with practical heuristics that avoid the exponential blowup of naive enumeration.

---

## Key Quotes

> "Railroad diagrams occupy a '1.5-dimensional' layout space — between 1D (text wrapping, code pretty-printing) and 2D (graph layout)."

This is the paper's sharpest insight. Railroad diagrams aren't just text with lines — a row can fork into two that rejoin, or turn backwards. But they're not arbitrary graphs either; the reading order imposes structure. The 1.5D framing is what makes the formal treatment tractable: it captures enough complexity to be interesting without collapsing into the NP-hard swamp of general graph layout.

> "Wrapping as optimization with local heuristics avoids combinatorial explosion."

The wrapping space is 2^(k·(n−1)) possibilities for a sequence of n items with k lines. The paper's pragmatic dodge: encode preferences as an optimization function over *just* the wrapping step, rather than trying to optimize the entire 2D layout globally. Three lexicographic criteria — content width, wrapping shallowness, height — in that order. It's a textbook example of how to find the right level of formalism: ambitious enough to be principled, constrained enough to be implementable.

> "Having stacks be only binary is the simplest model that accounts exactly for all possible layouts."

The diagram language has just four constructors (terminal, nonterminal, sequence, binary stack). Stacks are binary because n-ary stacks would overgenerate layouts that don't correspond to any actual diagram. This is design-by-constraint: the formalism is minimal not by aesthetic preference but because each constructor carves reality at its joints.

> "Alignment depends on justification policy (not its realization), so the dependency is non-circular."

A subtle architectural insight hiding in the dependency graph. A sublayout cannot collapse if the justification policy *could* insert a rail between them — alignment needs to know the policy's *potential*, not its *actual choices*. This makes the three passes strictly ordered rather than mutually recursive, and it's the kind of dependency the authors probably discovered the hard way.

> "The diagram language is idiomatic but not canonical — nested positive stacks can be reassociated."

The translation from regex/BNF reveals a tension: the diagram language prioritizes expressive convenience over canonical representation. Equal regexes can map to different diagram representations. The authors are honest about this as a limitation — railroad diagrams simply lack the solid theoretical foundations of other syntax notations — rather than papering over it.

---

## Key Themes

- **#concept**: Railroad diagrams as a 1.5D layout problem — the sweet spot between text wrapping and graph layout that makes formal treatment tractable
- **#tool**: `librrd` — 1,065 SLOC Scala compiler + 260 SLOC SVG renderer, with a web UI. The formalism was built alongside the implementation, not retrofitted
- **#pattern**: Three-pass compilation (align → wrap → justify) à la Flexbox, with the alignment/justification dependency resolved by depending on policy rather than realization
- **#pattern**: Optimization confined to the wrapping step only — global full-layout optimization is unstable; local heuristics on the wrapping decision are sufficient
- **#concept**: Diagram language vs. layout language as a compilation target — the same separation of concerns as IR vs. machine code, applied to visual syntax

---

## Critical Analysis

**What works.** The paper is unusually honest about scope. It explicitly enumerates what the formalism *cannot* express (ill-nested diagrams, vertical ε lines, the never-seen AlternatingSequence) rather than pretending universality. The 1.5D characterization is genuinely novel — I've never seen railroad layout described this way before, and it immediately clarifies why previous approaches were either too rigid (text-like wrapping) or too ambitious (general graph layout). The SQLite case study is thorough: 71 diagrams audited, specific inconsistencies named. This is PL research doing what PL research should do: find the right abstraction level, formalize it, implement it, and check it against reality.

**What's missing.** The paper references performance data and related work in labeled sections that weren't included in the provided text. The 1,065 SLOC implementation is admirably small, but the paper never tells us whether it's fast enough for interactive use (beyond a reference to a missing LABEL:sec:performance). There's also an open question about the diagram *language* design: is four constructors really enough? The authors acknowledge that SQLite's pre-2020 manually-drawn diagrams used layout-specific constructors (optx) that the diagram language deliberately omits. This is the right call — the diagram language should be semantic, not layout-contaminated — but it means some existing practice is excluded by design rather than by limitation.

**The deeper bet.** This paper is betting on a particular vision of what syntax documentation should be: generated from a grammar specification, styled by policy, consistent by construction. It's the same bet that drove Knuth's TeX (separation of content and presentation) applied to a domain that's been stuck in hand-drawn ad-hockery since the 1970s. If widely adopted, this wouldn't just make railroad diagrams easier to produce — it would make them *queryable*, *skinnable*, and *differable*. A diff between two versions of a grammar rendered as railroad diagrams, with layout preserved across the diff, is genuinely useful infrastructure that doesn't exist today. [[TALA (Diagram Layout Engine)]] marks the far end of the same spectrum — general 2D orthogonal layout too hard to compile, so it approximates (multi-seed sampling) and scores rather than solves.

**The SQLite lesson.** The paper's most damning exhibit is Figure 22: two identical conceptual diagrams in SQLite's own documentation, laid out differently "for no apparent reason," one with an extra arrowhead. This isn't a SQLite-specific problem — it's what happens when every diagram is hand-drawn. The existence proof here is that a compiler can eliminate this class of inconsistency entirely. The question is whether anyone cares enough to switch.

---

*Sources: [[raw/automatic-layout-railroad-diagrams]]*
*Last updated: 2026-07-25*
