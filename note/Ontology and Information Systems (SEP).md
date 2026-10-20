# Ontology and Information Systems (SEP)

The Stanford Encyclopedia of Philosophy's new entry on ontology in information systems, written by Adam Pease (of SUMO fame): what a computational ontology is, where it came from, how expressive its language should be, how to judge one, and what LLMs change. It is the rare piece that treats knowledge representation as an engineering discipline with philosophical roots rather than either pure metaphysics or pure plumbing.

---

## What it argues

Ontology in computer science is "the study of what there is" — but not what *ultimately* exists; what is *assumed* to exist in a domain for a computational purpose. The computer scientist is indifferent to the metaphysical status of categories and concerned only with their practical use. Two motivations recur: preventing concept drift (a symbol drifting in meaning across a codebase) and enabling interoperability.

The entry's spine is a ladder of expressiveness:

1. **Glossaries** — words with natural-language definitions, readable only by humans.
2. **Taxonomies and graphs** — the only machine-checkable error in a pure taxonomy is a cycle; knowledge graphs often ship with no formal specification at all.
3. **Description logics / OWL** — decidable inference, limited expressiveness; some OWL keywords' semantics must be defined in first-order logic anyway.
4. **First-order and higher-order logic** — more general axioms (transitivity stated once, not per-relation), consistency checking via theorem provers, at the cost of semi-decidability.

## Key quotes

> "A computer scientist is less concerned about the metaphysical status of categories – such as whether they actually exist or are just human constructs – than with their practical use in computation."

Commentary: this is the entry's thesis in one line, and it's what separates information-science ontology from philosophy proper. It's also quietly radical: correctness is defined relative to *useful models of the world*, not to reality.

> "An implementation in a software program is not a formal semantics. A semantics is a mathematical construct that exists outside any implementation."

Commentary: the sharpest line in the piece, and a direct rebuke to the "the graph is the spec" school. If your knowledge graph is only an implementation, nothing can check it for contradiction — a standard the entry applies when assessing ontologies.

> "One can avoid being wrong by saying nothing at all. The challenge is to say as many things about the world as possible while not saying things that are false."

Commentary: ontological commitment framed as an engineering trade-off between errors of commission (a false axiom that excludes uses) and errors of omission (a thin ontology that models nothing). The unicorn example is elegant: an existential assertion ∃x unicorn(x) is an unwise commitment, while the conditional ∀x(unicorn(x) → ∃y(horn(y) ∧ part(y,x))) commits only to what a unicorn *would be* if one existed.

> "LLMs are a model of language but not of the world."

Commentary: the entry's position on the current AI moment, citing the Stochastic Parrots paper. Neuro-symbolic combination — symbolic knowledge in prompts, LLMs for ontology matching, training on symbolically-expressed knowledge — is the proposed remedy, but the author concedes most work uses only knowledge graphs and it "remains to be seen" how expressive logics could be wired in.

## Key themes

#concept — ontology, ontological commitment, upper vs. domain ontology, differentia
#tool — OWL, description logics, theorem provers (Vampire, iProver), knowledge graphs
#pattern — the neat-vs-scruffy debate; expressiveness vs. decidability
#person — Adam Pease, John Wilkins, Cyc/Lenat

## Analysis

The entry is quietly polemical for its author's side of a decades-old fight. Pease co-authored SUMO, one of the few ontologies actually written in full first-order logic, and the assessment section is built to reward exactly that choice: more logical expressiveness, more formulas, more machine validation = better. The description-logic community would push back that decidability is not a footnote but the whole point — an ontology that can't guarantee termination is one you can't query at scale.

The historical thread is the best part. Running from Porphyry's tree through Wilkins' 1685 "Philosophical Language" (which anticipated functions and a constructed unambiguous language) to Cyc and the semantic web, it makes today's knowledge-graph revival look like the third or fourth lap of the same track — the entry even notes semantic networks were popular in the 1970s and "again in the 2020s." The Wilkins anecdote about 'glove' as a function denoting clothing-for-a-body-part is worth the price of admission.

The LLM section is the weakest — four paragraphs, no real argument — but its framing is useful: if an LLM is a model of language and an ontology is a formal catalog of assumed world-structure, then ontologies are exactly the kind of grounding that fixes "hallucination," and the open question is how to make an LLM conform to *expressive logic* rather than flat triples. That's a sharper statement of the neuro-symbolic problem than most industry writing manages.

## Relations

- [[Your Knowledge Graph Is Making Your Agent Dumber]] — this entry supplies the formal vocabulary for that critique: a graph without formal semantics is a glossary wearing a graph costume, and the SEP's "an implementation is not a formal semantics" is the precise version of that complaint.
- [[Immaculate Knowledge Graph]] — complicates it: that piece treats the graph as the curated artifact, while Pease would ask what logic backs it and what a theorem prover could actually check.
- [[An Introduction to Formal Logic (Peter Smith)]] — this entry is the applied sequel: it shows exactly where first-order logic's semi-decidability bites (refutation-based provers can't guarantee finding no contradiction) and why description logics exist as the decidable subset.
- [[Context Graphs]] — nuances it: agent-era "context graphs" are knowledge graphs with even less formality than RDF, so the SEP's three usage categories (human alignment, standard, computation) map neatly onto how much formal weight they can bear.

---
*Sources: [[raw/ontology-is]], [[summary/ontology-is]]*
*Last updated: 2026-10-20*
