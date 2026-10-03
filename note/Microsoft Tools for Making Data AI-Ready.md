# Microsoft Tools for Making Data AI-Ready

James Serra's survey of the Microsoft stack for AI-ready data — Fabric Dataflow Gen2, Power BI semantic models, Purview, Fabric IQ Ontology, Prep data for AI, Data Agent configuration, and Azure AI Search / Foundry IQ — organised by the problem each layer solves, with a working-backward-from-a-failure discipline that resists the "buy everything" reading.

---

This is part 2 of a three-part series. Serra's organising claim: making data AI-ready means reliable values, understandable business concepts, appropriate access, and enough context to answer real questions — and you should map tools to which of those problems you actually have. A `People` table carrying customers is the running example: storing data is not explaining it.

The layers, in order:

1. **Clean and curate** — Dataflow Gen2 (Power Query), notebooks, Data Factory pipelines, SQL views. Pipelines coordinate; dataflows, notebooks, and SQL transform. "Fix the actual data rather than instructing an agent to work around known problems every time someone asks."
2. **Reusable business meaning** — the Power BI semantic model exposes Customers while the table stays `People`, and carries shared measures so every consumer (human or AI) uses one Net Sales Amount instead of independently rewritten revenue logic. Purview supplies catalog, ownership, lineage — with the caveat that a glossary definition does not automatically enter an agent's prompt.
3. **Fabric IQ Ontology (preview)** — a business meaning layer bound to physical data: `Customer` as a concept over `Person_ID`/`Nm`/`Addr`, and relationships like Customer places Order, so "which customers bought Outdoor products" has an explicit business path. Reusable across consumers rather than re-explained per prompt.
4. **AI-specific context** — Power BI's Prep data for AI (AI data schemas, AI instructions, Verified Answers) plus Fabric Data Agent configuration (agent instructions, source descriptions, example queries). Crucially, instruction support differs by source type — the subject of part 3.
5. **Measuring readiness** — third-party BI Pixie scores a semantic model's AI readiness and benchmarks Copilot/Data Agent answers against expected results.
6. **Retrieval preparation** — Azure AI Search integrated vectorization, hybrid/semantic ranking, synonym maps, and Foundry IQ as a shared, managed knowledge base that multiple agents reuse instead of each agent wiring its own retrieval.

The closing recommendation: work backward from a specific question and a specific failure. Wrong values → cleaning; unclear calculations → modeling; ambiguous terms → definitions; missing document evidence → retrieval. You do not need every tool for one dataset.

---

> "This is where you fix the actual data rather than instructing an agent to work around known problems every time someone asks a question."

The sharpest sentence in the piece, and a quiet jab at the current pattern of bolting an agent onto dirty data and prompting around the defects. Preparation is upstream of configuration; an agent instruction is not a data fix.

> "I would not assume that writing a glossary definition automatically inserts it into every agent's prompt."

A governance-skeptic's caveat that most vendor demos elide: Purview metadata is only useful if the consuming architecture actually makes it available, and catalog permissions are distinct from data access. The plumbing between the governance layer and the agent is unglamorous and decisive.

> "The business meaning layer becomes reusable across consumers that connect to the ontology, rather than each consumer having to rediscover what the underlying data means."

This is the actual argument for the ontology layer — not prettier column names, but amortisation of explanation across every consumer. It also contains its own limit, which Serra states plainly elsewhere: an ontology does not discover business meaning because you gave a column a nicer label.

## Themes

- #concept — the **layered stack for AI-ready data**: physical curation → semantic model → ontology → AI-specific context → retrieval, each solving a distinct failure mode
- #pattern — **work backward from failure**: choose the layer by the symptom (wrong values, unclear calculations, ambiguous terms, missing evidence) instead of adopting the whole suite
- #tool — the Microsoft inventory: Fabric Dataflow Gen2, Data Factory, Power BI semantic models, Purview, Fabric IQ Ontology, Prep data for AI, Fabric Data Agent, Azure AI Search, Foundry IQ
- #concept — **readiness as a measurable, testable property** (BI Pixie scoring and benchmarking) rather than a configuration checkbox

## Analysis

The article is vendor-ecosystem documentation with an unusually honest spine. Serra repeatedly deflates his own subject matter: pipelines alone are worthless without transformation, glossaries don't auto-inject into prompts, ontologies don't discover meaning, and Prep-data settings don't replace correct relationships. That discipline — name the tool, then name what it cannot do — is what separates this from a product pitch, and it's why the piece survives contact with skeptical readers.

The strongest structural idea is the failure-driven mapping. Most "AI-ready data" writing is a forward march through product capabilities; this works backward from "what question failed and why," which turns the tool sprawl into a decision procedure. It also makes the vendor story falsifiable: if your problem is ambiguous terminology, no amount of retrieval infrastructure fixes it.

Two tensions are underexplored. First, the ontology-vs-semantic-model boundary is fuzzy in practice — a Power BI model with descriptions and measures already *is* a partial business meaning layer, and the article never says when the ontology is genuinely needed versus when it's the semantic model with a graph attached (his own criterion — multiple sources and teams needing shared concepts — is the best answer offered). Second, the piece is Microsoft-native by construction: the third-party BI Pixie aside, an AWS or Databricks shop maps the same failure taxonomy onto entirely different products, which suggests the durable content here is the taxonomy, not the tools. The preview-status churn across half the catalog (Fabric IQ Ontology, Prep data for AI, Data Agent source support, Foundry IQ features) is the practical warning: this stack is being built while people build on it.

## Related pages

- [[Beyond the Warehouse — Data Stacks That Actually Work]] — strengthens the shared thesis that a governed semantic layer is the precondition for safe AI on enterprise data; Thomas in 't Veld's CI-enforced semantic layer is the vendor-neutral, build-it-yourself counterpart to Serra's Power BI model + ontology, and his "two semantic layers is wrong" rule sharpens Serra's warning about agent-specific duplicate definitions.
- [[Databricks Genie Spaces for SQL Analysts]] — complicates the "AI-specific context" layer: Genie's curated examples, vocabulary, and metric definitions are the same artifacts as Prep data for AI's instructions and Verified Answers, hand-rolled on a different platform — evidence the pattern is convergent across vendors, not Microsoft-unique.
- [[Text-to-SQL in the Real World]] — explains *why* the preparation layers exist at all: Stonebraker and Chen's finding that schema rot and idiosyncratic enterprise data humble LLMs to ~10% is the failure mode Serra's cleaning, modeling, and definition layers are built to cure.
- [[Why AI Cannot Save an Enterprise That Doesn't Understand Its Data]] — the cultural argument behind Serra's engineering one: both claim AI amplifies rather than repairs an organisation's data incomprehension, with Serra supplying the tooling inventory and that essay the organisational diagnosis.

---
*Sources: [[raw/microsoft-tools-for-making-data-ai-ready]], [[summary/microsoft-tools-for-making-data-ai-ready]]*
*Last updated: 2026-10-03*
