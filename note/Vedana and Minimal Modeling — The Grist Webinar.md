# Vedana and Minimal Modeling — The Grist Webinar

Olga Tataranova (co-founder of Epoch8) explains, in a Grist-hosted webinar, how her firm turned chatbots that "sound right" into agents that "are right" by grounding them in a hand-crafted domain model — anchors, attributes, and links — documented in Grist and queried against Memgraph. This page is a second take on the same webinar as [[Vedana — Domain Models as Agent Context]], with more attention to the Q&A: the Grist/Memgraph split, the Excel history, and the admissions about scale.

---

## The argument in one paragraph

Epoch8 claims that LLMs fail on client-domain questions not because they cannot reason but because they do not know the domain, and that the fix is not better chunking or bigger models but a structured, expert-authored data model the agent can explore and query directly. The falsifiable core: when the same legal question is asked of ChatGPT over chunked documents and of Vedana over the data model, the chunked version misses respondents and facts while Vedana crafts a Cypher query and returns the exact database value — so the competitive edge in enterprise AI comes from domain documentation, not model choice. If chunking-plus-a-bigger-model ever matches expert-modeled retrieval on their clients' workloads, the claim dies.

## Key quotes

> "ChatGPT is great in generating some good prose, some good texts. But ChatGPT doesn't know your domain, doesn't know how your business processes work, doesn't know anything about what you do."

The diagnosis in one sentence: the failure is knowledge, not intelligence — which is why the remedy is documentation rather than a better model.

> "So we stopped, stopped just sounding right and we started like being right."

The whole product thesis compressed into a slogan; note it is also an implicit admission that everything before this point was plausible-sounding fiction sold to clients.

> "Links are the most important part of the description because links actually describe how your domain work, how LLM, how the agent reasons across your domain."

The least obvious part of the method: the entities are easy; the sentences describing how anchors relate are what let the agent reason across the domain rather than retrieve isolated facts.

> "Don't try to extract data model automatically because it's your expertise, it's how your business works. No LLM can help you here, unfortunately."

The strongest and least defended claim in the talk — the modeling work is asserted to be irreducibly human, with no reported experiment in which an LLM proposes a model and experts correct it.

> "Expertise is the thing that is hard to scale."

Spoken twice in different forms; it quietly reframes the product: Vedana is not selling intelligence, it is selling a playbook for bottling the one or two experts every client has.

## Critical analysis

The non-obvious move here is the provenance of the notation. Minimal modeling was not invented for LLMs — Epoch8 adopted it, pre-LLM, to document data for *human* analysts, and only later discovered that "as human analysts understand this notation, LLMs also understand this notation." That is a better argument for the approach than any accuracy claim: a notation that had to work for humans before agents existed is less likely to be a prompt-engineering fad. It also explains the three-table discipline (anchor, attribute, link, each with a description, an example, and a sample query) — the sample queries are few-shot examples for the agent's query-writing, which is the actual mechanism making the Cypher crafting reliable.

The weak point is the evidence. The ChatGPT-versus-Vedana comparison is "some typical cases" — no error rates, no benchmark size, no methodology. For a company whose entire pitch is that wrong answers are expensive for their clients, they are surprisingly casual about quantifying their own improvement. The Q&A makes the architecture honest in ways the summary glosses: Grist is *not* the production backend. Experts edit dictionaries and expert data in Grist (which, by their own admission, buckles at hundreds of thousands of rows), an ETL pushes to Memgraph, and the agent queries Memgraph. So the real system is a graph database with a spreadsheet-shaped editorial front end — and the interesting engineering question, how the ETL stays in sync when experts press "refresh," is waved through.

What is left out: hallucinated queries. The talk never asks what happens when Vedana crafts a *wrong* Cypher query — the failure mode simply moves from guessing answers to guessing queries, and a confidently wrong query against a well-modeled database is still a confidently wrong answer. Security on the agent side (who can query what in Memgraph) goes unaddressed despite role-based access being a stated Grist requirement. And the "no AI can help you here" claim about modeling is asserted, not argued — it is exactly the kind of claim a 2026 experiment with LLM-proposed, expert-corrected models could test, and its absence is conspicuous. Finally, the claim that domains are "pretty stable" sits awkwardly next to their own legal focus, where regulations change; maintenance of the model over years is the unpriced cost of the whole approach.

## Related

- [[Vedana — Domain Models as Agent Context]] — covers the same webinar; this second take strengthens its account with the Q&A material that note treats lightly: the Grist-to-Memgraph ETL split, the Excel-to-Grist tool history, and the admitted performance ceiling.
- [[Text-to-SQL in the Real World]] — Stonebraker and Chen argue real enterprise text-to-SQL fails on schema rot and idiosyncratic institutional knowledge; this webinar is a practitioner's answer to exactly that diagnosis, encoding the institutional knowledge as an expert-built model rather than hoping the benchmark generalizes.
- [[Beyond the Warehouse — Data Stacks That Actually Work]] — Thomas in 't Veld's semantic-layer-as-agent-interface thesis from the data-engineering side; this source strengthens it from the consultancy side, with the same insistence that the curated model, not the raw documents, is what the agent should see.
- [[Building Apps in Grist with the MCP Connector (Webinar)]] — the other Grist-MCP webinar in this wiki; this source nuances it by showing Grist used not as the app platform but as the editorial backend for a domain model that gets ETL'd into a graph database for production queries.

---
*Sources: [[raw/cf5ec5b56a9c33b59446078fae57a971]], [[summary/cf5ec5b56a9c33b59446078fae57a971]]*
*Last updated: 2026-11-03*
