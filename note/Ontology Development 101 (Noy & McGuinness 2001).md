# Ontology Development 101 (Noy & McGuinness, 2001)

A citation record for the 2001 Stanford tech report that taught a generation to build ontologies — and, in its abstract, a snapshot of the moment the industry first believed machines would read the Web's structured knowledge.

---

The paper itself is the canonical tutorial for ontology engineering: what an ontology is (Gruber's "explicit formal specifications of the terms in the domain and relations among them"), how to scope one, decide on classes and hierarchies, and pick properties. The abstract captured here frames why that mattered: ontologies were moving from AI labs to domain experts' desktops, and the Web was growing machine-readable structure — RDF, DAML, standardized vocabularies in medicine and commerce — explicitly "aimed at facilitating agent interaction."

---

## Key quotes

> "explicit formal specifications of the terms in the domain and relations among them"

Gruber's definition, quoted approvingly, remains the best one-line answer to what a specification *is*: not prose describing intent, but named terms and named relations between them — the same move, twenty-five years early, that today's spec-driven development makes when it insists the durable artifact is a structured doc, not a chat transcript.

> "[RDF is] a language for encoding knowledge on Web pages to make it understandable to electronic agents searching for information."

Note the word *agents* in a 2001 abstract. The dream — software agents that traverse machine-readable knowledge — failed on the open Web, where nobody would annotate their pages, and is now being revived inside codebases and enterprises where the annotation is economically forced.

> "DARPA... is developing DARPA Agent Markup Language (DAML) by extending RDF with more expressive constructs aimed at facilitating agent interaction on the Web."

The 2001 toolchain (RDF → DAML → agents) maps almost one-to-one onto the modern one (structured outputs → tool protocols → agents). The expressiveness question — how much formal structure do agents actually need? — is still the live question in [[MCP and Tool Protocols]].

---

## Key themes

#concept #pattern #person

- **Ontologies as specs**: a formal vocabulary is a specification with an enforcement story — terms either resolve or they don't.
- **The agent problem is old**: "electronic agents" needing machine-readable knowledge was the founding motivation of the Semantic Web.
- **Standardized vocabularies win where there's a payer**: SNOMED and UNSPSC survived; the grand general-purpose Semantic Web largely did not.

## Analysis

The delicious irony of this document is that the Semantic Web project — explicit, centralized ontologies so agents can reason — is widely remembered as a flop, yet its premise is the premise of the modern agent stack. What changed is who does the annotating: in 2001 the burden fell on human webmasters, and they declined; today the burden falls on the agent's author, who writes CLAUDE.md files, schemas, and tool descriptions because an agent that misreads its world fails immediately and visibly. Economically forced structure beats voluntarily supplied structure.

It also sharpens the current debate about specs. The ontology camp believed shared formal vocabularies were the bottleneck; the modern camp believes shared context documents are. Both bet that making knowledge explicit converts coordination problems into lookup problems. The wiki's own spec-driven sources are, in a real sense, doing ontology engineering without the name.

The caveat the abstract can't show: Ontology Development 101's pedagogy was about *hand-crafting* shared vocabularies, and the hardest lesson of the era was that ontologies are cheap to write and brutal to keep current. That maintenance failure mode is exactly what [[Agent Memory]] systems are re-encountering — a stale memory store is a stale ontology with worse manners.

---

Related: this source strengthens [[The Coming Need for Formal Specification]] — it is the twentieth-century ancestor of the argument that formal artifacts beat informal ones when machines are in the loop, and its failure modes are the counterweight that page lacks. It nuances [[AI Agents Need Clear Specs]]: Gruber's definition is a stricter bar than most of today's "specs," being enumerable and checkable rather than prose. And it complicates [[Agent Memory]]: the Semantic Web was, at bottom, a memory architecture for the whole Web, and it died of maintenance costs, not wrongness.

---
*Sources: [[raw/citations]], [[summary/citations]]*
*Last updated: 2026-10-20*
