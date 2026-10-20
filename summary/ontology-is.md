---
url: https://plato.stanford.edu/entries/ontology-is/
title: "Ontology and Information Systems"
author: Adam Pease
date_fetched: 2026-10-20
date_published: 2026-01-03
topics:
  - ideas-and-culture
  - ai-research-and-models
---

# Ontology and Information Systems

A Stanford Encyclopedia of Philosophy entry (first published January 2026) on ontology as practiced in computer and information science: the shared, formal cataloguing of what is assumed to exist in a domain for computational purposes. Author Adam Pease (SUMO) traces the lineage from the Tree of Porphyry through Linnaeus, Wilkins, expert systems, Cyc and the upper ontologies (SUMO, DOLCE, BFO), then distinguishes upper from domain ontologies, treats ontological commitment as a practical engineering trade-off (errors of commission vs. omission), surveys specification languages from glossaries to graphs, description logics and first-order logic, and closes with assessment criteria, applications, LLMs, and future directions.

---

The core claim is that ontology in information science is "the study of what there is" — not what ultimately exists, but what is *assumed* to exist in a domain for some computational purpose. Its two great motivations are preventing **concept drift** (the same symbol drifting in meaning across a codebase over time) and supporting **interoperability** between systems and people.

Three usage categories frame the field: (1) ontologies as communication among people (glossaries, natural-language definitions), (2) symbols as a standard between people and machines, and (3) fully computational meaning in an implemented logic. Expressiveness trades against decidability: description logics (OWL) buy decidable inference but can't express much; first-order and higher-order logics say more and enable consistency checking, at the cost of semi-decidability. "An implementation in a software program is not a formal semantics" — a point the entry returns to when judging knowledge graphs that ship with no specification at all.

On **ontological commitment**, the entry argues for a pragmatic middle path: avoid errors of *commission* (commitments that exclude reasonable models, like requiring everything to be located in spacetime, which excludes numbers) and errors of *omission* (saying so little the ontology is useless). The unicorn example — existential assertion vs. a conditional "if a unicorn exists it has a horn" — shows how formal form determines commitment strength. Against minimal-primitive purism, the entry argues from software libraries and dictionaries: a big vocabulary is an asset, not a liability.

The LLM section is brief but pointed: LLMs are "a model of language but not of the world"; neuro-symbolic approaches (training on symbolic knowledge, prompting with it, ontology matching with LLMs) are the proposed fixes. Most work so far uses knowledge graphs; expressive-logic ontologies with LLMs remains open.

---
*Source: [[raw/ontology-is]]*
*Last updated: 2026-10-20*
