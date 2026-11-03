# Vedana Webinar — Minimal Modeling in Grist (Epoch8)

Olga Tataranova (Epoch8) walks through how Vedana grounds AI agents in an expert-authored domain model — anchors, attributes, and links stored in Grist — so agents query exact values from a database instead of guessing from text chunks. The competitive edge, they argue, is domain documentation, not model choice.

---

## What the source argues

LLMs fail on client-domain questions not because they can't reason but because they don't know the domain. Epoch8's clients (legal, e-commerce, manufacturing, automotive) share one trait: "a wrong answer is expensive," so they need precise, traceable answers — which chunk-dumping into ChatGPT cannot deliver.

The method is "minimal modeling" by Alexey Mahotkin: any domain is described with three concepts — **anchors** (the nouns: law documents, court cases, judges), **attributes** (name, title, enactment date), and **links** (how anchors relate). The model is stored in Grist as exactly three tables, each entry carrying a description, a data example, and a query an agent can use to fetch an example. The same notation works for human analysts and LLMs — they adopted it pre-LLM for human analysts, then repurposed it as agent context.

Two use cases: (1) feed the model to an LLM to extract structured entities from documents; (2) drop the whole Grist document (it's just SQLite) — or the Grist MCP, "the game changer" — into Claude as context and say "go explore the database, craft some queries and fetch the data that you need." In their head-to-head, ChatGPT on chunked documents missed respondents and facts; Vedana crafted a Cypher query and returned the exact value from the database.

Architecture nuance: Grist is not the production query backend. Experts edit in Grist, an ETL pushes data to Memgraph, and the agent queries Memgraph — necessary because Grist hits performance walls at hundreds of thousands of rows. Text chunks are kept only as a fallback layer, with quality that "drops dramatically" when used.

## Key quotes

> "ChatGPT is great in generating some good prose, some good texts. But ChatGPT doesn't know your domain, doesn't know how your business processes work."

The diagnosis in one sentence — the failure is knowledge of the domain, not reasoning.

> "So we stopped, stopped just sounding right and we started like being right."

The whole pitch in a phrase: sounding right vs being right, with sources attached.

> "Links are the most important part of the description because links actually describe how your domain work, how the agent reasons across your domain."

Interesting that the reasoning structure lives in the relationships, not the entities — the model is closer to a business-process description than a schema.

> "Don't try to extract data model automatically because it's your expertise, it's how your business works. No LLM can help you here, unfortunately."

The most debatable claim in the talk — asserted, never tested against LLM-proposed models that experts then correct.

## Key themes

#concept #pattern #tool

- **Minimal modeling**: anchors / attributes / links as an opinionated ontology with an accompanying book; deliberately small enough to hand to both humans and agents.
- **Documentation layer vs query layer**: Grist as the experts' editing surface, Memgraph as the production graph backend, ETL between them.
- **Playbooks compound**: the automotive data model "looks pretty much the same across all the businesses," so domain documentation becomes a reusable asset rather than a per-project cost.

## Critical analysis

This is a vendor webinar, and it shows. The ChatGPT-vs-Vedana comparison is anecdotal — no error rates, no benchmark size, no methodology. The claim that data models can't be auto-generated is the talk's most interesting assertion and its least examined: they never report trying LLM-drafts-then-expert-correction, which is the obvious experiment. Maintenance is hand-waved ("domains are pretty stable") with nothing on what happens when regulations change or the ETL drifts. Query-generation hallucination — what happens when the agent crafts a wrong Cypher query — goes unaddressed, as does agent-side access control on Memgraph.

Still, the core idea is strong and aligns with a wider shift: structured, expert-authored context beats raw chunk retrieval when exactness matters. The genuinely novel part is treating the data model itself as the artifact experts maintain — three tables, spreadsheet UI, done once — rather than as an engineering deliverable. The honest concession that expertise doesn't scale is worth more than most vendor decks offer.

## Related pages

This deepens [[Vedana — Domain Models as Agent Context]], the earlier ingest of a sibling gist on the same system: this transcript adds the Excel-history detail (the notation outgrew spreadsheets at ~50 attributes), the ETL-to-Memgraph split, and the head-to-head anecdote. It nuances [[Ontologies Come of Age (McGuinness 2003)]] by offering a deliberately stripped-down "opinionated ontology" — three tables, no formal semantics — that agents can actually use as context. It strengthens [[Making Your Data Ready for Agentic AI]]'s argument that the data layer, not the model, is where enterprise AI quality is won. And it complicates [[Grist]]'s picture of the tool: here Grist is a documentation and expert-editing layer with an admitted performance ceiling, not a system of record.

---
*Sources: [[raw/cf5ec5b56a9c33b59446078fae57a971]], [[summary/cf5ec5b56a9c33b59446078fae57a971]]*
*Last updated: 2026-11-03*
