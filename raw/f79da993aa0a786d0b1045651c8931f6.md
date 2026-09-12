---
url: https://gist.github.com/f79da993aa0a786d0b1045651c8931f6
date_fetched: 2026-09-13
---

You signed in with another tab or window. Reload to refresh your session.You signed out in another tab or window. Reload to refresh your session.You switched accounts on another tab or window. Reload to refresh your session.Dismiss alert

LLMs fail on domain-specific questions because they lack business logic. Dumping documents into ChatGPT produces answers that sound right but are often wrong, especially when precision, traceability, and legal defensibility matter.

The fix is a structured domain model, not better chunking. Epoch8 uses “minimal modeling”—anchors (nouns), attributes, and links (relationships)—to describe how a domain works. This model is stable, expert-crafted, and serves as the reasoning backbone for AI agents.

Grist is the documentation and collaboration layer, not the production query engine. It was chosen because it combines a business-user-friendly UI, typed columns, open-source licensing, API access, Python friendliness, and role-based access—requirements that databases, spreadsheets, and CMSs each fail to meet individually.

The data model is fed to LLMs in two ways: (1) as a prompt to extract structured entities and links from documents, and (2) as a context file (the whole Grist SQLite database) so the agent can explore the model, craft Cypher/SQL queries, and fetch exact answers from the database instead of guessing from text chunks.

This shifts answers from “sounds right” to “is right, and here’s the source.” The agent follows a reasoning chain grounded in the domain model, returning precise values and traceable evidence—critical for legal, e‑commerce, manufacturing, and automotive use cases.

Expertise doesn’t scale, but the model can be bootstrapped across similar domains. The data model is maintained by domain experts (not engineers) and changes only when business processes change. Vedana collects playbooks of domain models (e.g., automotive) that stay largely consistent across businesses.

Production architecture separates expert editing from query serving. Data flows from Grist (where experts edit dictionaries and small reference data) to Memgraph (a graph database) via an ETL process. The AI assistant queries Memgraph, not Grist directly, to handle large or rapidly changing datasets.

Pithy and provocative quotes

“ChatGPT is great in generating some good prose, some good texts. But ChatGPT doesn’t know your domain, doesn’t know how your business processes work, doesn’t know anything about what you do.” — Olga frames the core failure of naive LLM document dumps.

“We stopped just sounding right and we started being right.” — On the shift from plausible-sounding answers to verifiable, exact answers.

“Links are the most important part of the description because links actually describe how your domain works, how the agent reasons across your domain.” — Emphasizing that relationships, not just entities, encode business logic.

“Expertise is extremely hard to scale. Very hard to scale. And no AI can help you here.” — A blunt admission that the domain modeling itself remains a human-intensive bottleneck.

“We just dumped this SQLite database to Claude or to any other AI agent and it is comfortable enough to explore the data model, to explore the tables.” — Revealing the surprisingly simple mechanism of handing the entire Grist file to the LLM as context.

“When MCP arrived it was like the game changer because we just fed the Grist MCP to Claude and we were like, ‘Dear Claude, we need to set up an AI assistant for our customer. Please go and check this data model. Please go check data for consistency and write some prompts.’” — On using Claude with the Grist model to bootstrap cheaper LLMs.

Tools, practices, and methodologies

Minimal modeling (Alexey Mahotkin) — A notation for domain description using anchors (nouns), attributes, and links (relationships). Used to document data for both human analysts and LLMs. The speaker says to start with 2–3 anchors and a handful of attributes, craft the model with domain experts, and never try to auto-generate it.

Grist — An open-source spreadsheet-database hybrid with typed columns, API, Python support, and role-based access. Used as the expert-facing documentation tool where the domain model lives and is maintained. The speaker emphasizes its UI for non-engineers and its ability to serve as a single source of truth for the model.

Grist as an LLM context file — The entire Grist document (an SQLite database) is dumped and given to a powerful LLM (e.g., Claude) so it can explore the data model, check consistency, and generate prompts or queries for cheaper models.

Memgraph — A graph database used as the production query backend. Data is replicated from Grist to Memgraph via an ETL process. The AI agent queries Memgraph using Cypher (a graph query language) to fetch exact answers. This separation avoids Grist performance issues with large datasets.

MCP (Model Context Protocol?) — Mentioned as the integration layer that allows feeding Grist’s data model directly to Claude. The speaker calls it a “game changer” for setting up AI assistants.

Cypher query examples in the data model — For each attribute and link, the model includes a sample query (Cypher or SQL) showing how to fetch that data. The AI agent uses these examples to construct its own queries at runtime, rather than hallucinating database operations.

Two-layer data strategy — Structured data (extracted from documents using the model) is the primary source for answers; raw text chunks are kept only as a fallback layer, with the understanding that answer quality drops significantly when the agent resorts to chunks.

Expert-maintained playbooks — Vedana collects and reuses domain models (e.g., for automotive) across clients. The model stays stable and is edited by domain experts who add attributes like “discontinued” to product tables without engineering help.

Unanswered questions and omissions

How exactly does the LLM “explore” the Grist SQLite file? The speaker mentions dumping the whole database to Claude and later references MCP, but the mechanics of how an LLM navigates a relational schema, formulates queries, and validates them are not explained. Is there a retrieval-augmented generation (RAG) step, or does the model read the raw SQLite bytes?

What is the extraction pipeline from PDFs to structured data? The talk says they “extract anchors, attributes and links from PDFs” using the data model as a prompt, but no details are given on accuracy, handling of ambiguous text, or how the extracted data is validated before entering Grist/Memgraph.

How are incorrect or unsafe AI-generated queries prevented? The agent crafts Cypher queries on the fly. There’s no mention of guardrails, query validation, or sandboxing to avoid destructive operations or performance issues on the production database.

The comparison with ChatGPT is a straw man. They chunked documents and dumped them into ChatGPT, but modern RAG systems use hybrid search, reranking, and chain-of-thought. The talk doesn’t address how Vedana compares against a well-tuned RAG pipeline with the same structured metadata.

Scalability of expert modeling across many domains or large organizations. The speaker admits expertise doesn’t scale, but doesn’t discuss how they onboard new domains, handle conflicting expert opinions, or maintain consistency when multiple experts edit the Grist model concurrently.

What happens when the domain model changes? The claim is that models are stable, but no process is described for versioning the model, migrating extracted data, or retraining/updating the AI assistant’s prompts when links or attributes are added or removed.

Why Memgraph instead of querying Grist directly? Performance with large data is cited, but for smaller datasets, could Grist’s API serve as the backend? The trade-offs (latency, query expressiveness, operational complexity) are not explored.

The role of the “playbook” and how Vedana shares models across clients. The speaker briefly shows an automotive model and says it’s similar across businesses, but doesn’t explain how a model is adapted, what customization is needed, or whether there are privacy concerns with reusing models.

Open-source and self-hosting details. They use open-source Grist, but what about the MCP integration, the ETL to Memgraph, and the Vedana agent itself? The talk doesn’t clarify which components are open-source and which are proprietary, or what a self-hosted deployment would require.

Evaluation and metrics. The only evidence given is a single anecdotal comparison where ChatGPT failed and Vedana succeeded. There’s no discussion of systematic evaluation, accuracy metrics, recall, or user studies across multiple domains or question types.

Speaker A: Welcome to today's webinar. My name is Anais, I'm CEO at Gris and I am joined by Olga Tataranova, the co founder of EPIC8. Hi, Olga. Hey. So why don't we just jump right in? What does Epoch 8 do and what do you do?

Speaker B: So Epoch 8 started in 2017, before machine learning was cool. And two lines of work have run through the whole time. So the first line is Computer Vision. So one of our best known projects is brickit. It's a consumer app that looks at the pile of LEGO bricks and shows you what you can build out of it. So it's Computer Vision Lab, Computer Vision app. And on the industrial side there is aci. So it's Computer Vision for cargo quality in a warehouse. It spots damage, measures dimensions, checks that every item made into its box, so nothing is missed. So it's the CV line and the second line is Chatbots. And this is the line that leads to our today's conversation. So we started with Chatbots many years ago, before LLMs existed. And when LLMs arrived, we moved to our own platform, to Vedana. And the clients who came to us shared one thing in common. For them, a wrong answer is expensive. So they need answers that are precise, that are traceable, and that means legal. So E commerce, manufacturing and automotive. So Vedana is the product that grew from the second line and it's the product where we use and this is where, this is what we're here to show.

Speaker A: Okay, so before you found grist, you had a problem that you needed to solve. So can you describe that problem that you needed to solve?

Speaker B: So the problem is that when LLMs arrived, all the business customers expect to have lots of documents, dump these documents into like some LLMs, into charge GPT into something, and they're like, let's go dump our documents. And we expect expect ChatGPT to give correct answers. And this never happens because ChatGPT is great in generating some good prose, some good texts. But ChatGPT doesn't know your domain, doesn't know how your business processes work, doesn't know anything about what you do. And so our task was to explain to LLMs the domain how it looks like and how to reason across links and across rules of your domain.

Speaker A: Okay, so when you were looking for a tool to help you with this problem, what were some of the requirements and what made Gris the right answer?

Speaker B: So we have a bunch of very unusual requirements actually. So first of all, We needed a tool that has good user interface. So business users can work with it without engineering support. So the data has to be typed, so we need typed columns, so the column is a number, a date or reference or like a fixed set of choices. The third thing, that tool has to be open sourced because our clients have a real privacy and data resiliency rules. It had to be API driven so so it could plug into our production system. And it had to be python friendly because that's where our team and our data jobs are written. And it needs to have role based access because each person needs to see only their part of work. And we looked all the usual shapes. So a plain database gives you types and API but no interface where domain experts can work. A spreadsheet like Google Sheet gives you the interface, but no real types and no real APIs. So we consider it CMS, but it's built for the content, it has no place for typed relational data. And somehow we find out that Grist meets all these requirements and we were quite happy to find it. And currently we use open sourced Grist as an open source backend for our open source tool.

Speaker A: Great. So why don't we make this real? If you would like to share your screen to kind of talk about how the data model documentation actually works.

Speaker B: Okay, so before we jump to the tables, I will show you, I will show you the corpus. So I have an example of legal corpus. So legal guys are the best users of Vedana currently because you can't guess when you answer legal questions. And legal questions have a lot of cross links, cross references, etc. So if we speak about some legal corpus, what we have here we have laws, some. Here's what typical document looks like. Some legal texts, some articles, some law numbers, some low names, some administrative administrating entities, etc. And some documents that are amended by this document, lots of them. We have some regulations, we have some court decisions. For example, if you drill inside some court decision, you can see some case numbers, some decision dates. Each court decision has judge, each court decision has parties. Sometimes court decisions have links to cases, etc. So all these documents are linked together and you can simply chunk them and expect LLMs to give correct answer. You have to model them. Here's the interesting part, how we approach to data modeling. And here's where we jump into Grist. So here's how our domain works. We have three main concepts that describe any domain. So these are anchors, they're nouns of the domain. For example, here we have load documents, we have articles, we have court cases, we have legal parties, we have Judges. So these are like anchors. Each anchor has attributes. For example, if we have a judge, a judge has a name, a judge has a title. If we speak about low document, low document has name, has number, has source file, has some enactment dates, et cetera. And the most important part are links between anchors. So we have sentences that describe how, for example, load document is linked to any other anchors. For example, load document can amend another load document or load document has article. Links are the most important part of the description because links actually describe how your domain work, how LLM, how the agent reasons across your domain. For each anchor, attribute and link, we write some description, we write some data example, and we write a query how an agent can fetch the example of this attribute for this, of this anchor, of this link from the database. Actually, this approach was developed by, this approach is called minimal modeling. It was developed by Alexey Mahotkin and he wrote a whole book about this which is available. Actually, we started working with Alexei long before LLMs arrived. So we started from, we jumped into minimal modeling because we have to optimize how our analytical department works, how we document data for, for human analysts. And then when LLMs arrived, it turned out that it's a great way to describe the domain. And as human analysts understand this notation, LLMs also understand this notation. So how it looks like actually there are three tables, anchor table, attribute table and link table. And it's pretty much it. We use this description for two purposes. First use case, we feed this description to LLM so it fetches attributes from the document, so we can feed so we can feed our document to Claude or some other LLM agent and we might prompt it to extract low number, low name, enactment year, etc. So we can prompt it just to look at the data model and extract the entities and the links that we need. And the second use case, we drop this data model to the AI agent as the context. We actually dropped the entire GRIS document and we say go explore the database, go explore the data model, craft some queries and fetch the data that you need to answer the user question. So let me show you the example. Here's the chart from the Vedana back office. So I asked the question about some case with some user ID. Actually we tried to ask the same question to ChatGPT. One second, we asked the same question to ChatGPT and ChatGPT failed because he was mistaken. He didn't get the answer right. He answered something in USD and the real answer is this. So how Vedana reasons Vedana. I will Ask this question to Vidana. So I expect Vedana to go to the data model, explore it and give the exact answer. Because yes, here it is. And what Vedana did, it crafted some cipher query which it got from the data model. So Vedana doesn't guess from chunks, it goes to the data model, explores it, fetches some cipher queries and gives exact value from the database, which is like absolutely correct. In this case we don't have to craft two cipher queries. So the agent knows the domain and can explore the domain freely and craft some more queries, craft some more vector searches because actually the logic of the domain is fully described. Very cool.

Speaker A: What did this unlock for your clients? How did this work before, if it worked at all? And how did this change things in the flow for them and for your team?

Speaker B: This approach unlocks actually it turns sounds right answers into this answer is right. And here's the source, here's the exact chunk for this answer. So we made a comparison between Vidana and ChatGPT and we see some typical cases where ChatGPT fails. What we did here, we chunked the documents and just dump them to ChatGPT. And where ChatGPT failed, it failed. When we need to answer, we need to give exact answer. Here ChatGPT missed some respondents from the case here ChatGPT missed some facts because he didn't fetch all the chunks. So we stopped, stopped just sounding right and we started like being right and we enabled LL Imagine to follow the reasoning chain and to give the exact answer like your experts would actually.

Speaker A: And you mentioned something interesting. You said that you give the whole Grist file as a file to the LLM, is that right?

Speaker B: Sometimes, sometimes we did like this because Grist is actually is SQLite database. So we just dumped this SQLite database to Claude or to any other AI agent and it is comfortable enough to explore the data model, to explore the tables. And when MCP arrived it was like the game changer because we just fed the GRISTMCP to Claude and we were like, dear Claude, we need to set up an AI assistant for our customer. Please go and check this data model. Please go check data for consistency and write some prompts. So the cheaper LLM model gives correct answers. Looking at the data model and looking at data.

Speaker A: Yeah. And then who maintains the domain, the documentation in Grist?

Speaker B: Mostly experts. So it's a two sided work. So what can't be generated is data model. It's very important thing. I will show how, how the visual representation of a data model looks like. So for our legal use case, actually the data model looks like this. So we have court cases, legal parties, charges, and law documents. And interesting that this simple notation can be generated. So Claude misses the point, misses the links, Mrs. Like domain logic. So first that you need to do is to sit with your domain experts and understand what your anchors look like, like what your attributes look like and what are the links between them. Good news here is that your data domain doesn't change often. So it changes only when you introduce some new business process or some dramatic changes arrive. Actually, it's pretty stable. And you do this data modeling work once and then you just give the whole GRIS document to your domain experts and they're happy enough to add some attributes to the data. For example, the typical case is when you dump your product database and then some products dump, become obsolete, some products become discontinued, but you still want to have this information in the database, but you want to answer that these products are discontinued and domain experts are happy enough to just add one more column to grist, add discontinued attribute, and that's it.

Speaker A: Great. Does this make it easier then to take the domain experts, who are not necessarily engineers or developers, to start data modeling in something that's more intuitive? I'm just curious if there's been any acceleration in kind of like building more

Speaker B: models,

Speaker A: getting expertise in new domains.

Speaker B: Expertise is the thing that is hard to scale.

Speaker A: Yeah, exactly.

Speaker B: So actually what we saw in every business that you have one or two super cool experts that know everything and everybody else are like, well, go ask these guys because they know everything and I know my tiny part. So expertise extremely hard to scale. Actually, Vidana collects some playbook of domains. For example, we have a pretty good data model for automotive. Let me show you. Let me quickly show you. So we have, for example, we did some automotive, some automotive projects. And actually the data model for automotive looks pretty much the same across all the businesses. So you have products, products have some technologies, some chemistry, they support some writing styles, they support some vehicle application. And here are some anchors, some attributes, and some links that really stay pretty much stable across businesses in different domains. So in some case, Vedana helps to bootstrap this expertise documentation. But still, expertise is very hard to scale. Very hard to scale. And no AI can help you here.

Speaker A: Yeah, great. And when you were building this kind of documentation layer, was Gris the first tool you used? I actually don't know the history. Did you try it with something else first?

Speaker B: We tried Excel. So actually when we started documenting data with Alexa, we started like with plain Excel and it worked fine until the data model started growing. So when you have a list of like 50 attributes, the Excel stops being enough because you need some visual representation of of how your data model looks like. And interesting that I've shared this template and interesting that this template swiped like three or four years of our analytical work and it stays pretty much unchanged because here you see all the anchors and you can filter attributes, attributes which match with the anchor and you can filter the sentences and actually you can grasp how for example, the court cases look like across the whole domain that you support.

Speaker A: Great. And we are sharing this template with anyone watching or anyone who's interested and it's nice to know that it's basically been tested for a few years now. So yeah, excellent. Do you have any parting advice for AI practitioners who might be watching who want to start documenting their own client data?

Speaker B: As for advices, first advice, don't try to document everything at once. The approach that work for us is to start anchor by anchor, to start with 2 or 3 5k anchors and with 3, 4 or 5k attributes. Don't try to extract data model automatically because it's your expertise, it's how your business works. No LLM can help you here, unfortunately. So our advice is to start with predefy with expert crafted data model which is not very big. Use this data model to extract structured data from the documents, put the data in structured tables and given an assistant some examples how to query the structured data, how to fetch exact answers. So you can give SQL examples of cipher examples as we do is like maybe some Python code examples which are totally fine. And then you can pass your data model as a context to your rug, to your AI assistant and you can, you will see that the AI assistant will use your SQL snippets, SQL examples to query the data and you will see that the quality of your answers will be dramatically more good than if you use just chat, GPT or something.

Speaker A: Great. So why don't we open it up for questions? If you have any questions, you can put it in the chat. Also seems like Alexa is here. Hello. And Nick shared Alexei's minimal modeling substack. Also there was mention of a book, Alexa. If you want to plug the book and chat that you're welcome to do so. So we have a question from does LLM reple to the grist directly as database back backend?

Speaker B: As database backend? Actually our full architecture, we replicate data from grist to MEM graph. So we use MEM graph as the hard backend for all our systems because sometimes we don't put all the data in Greece because we work with E Commerce. E Commerce might have like millions of products that change rapidly and we skip putting these products into Greece. We pass them to memograph and our usual setup is we have some fixed amount of data in Greece, for example documents, manuals, some dictionaries, etc. And sometimes we have a data flow that goes directly to MEM Grab. So no, Grist is not a production backend for the AI Assistant to craft answers, but Grist is a production backend for experts and for editorial team that works in Grist. And after they done editing Grist they press like refresh. Our ETL process fetches data from Grist, puts data in memograph and AI assistant also almost instantly fetches their changes. Great.

Speaker A: We have another question from Patrice. Is it ontology

Speaker B: in some definition of yes, you might call it an ontology. So it's opinionated ontology. So Alexei has a whole book when he explains why he chooses anchor attributes and links, how to extract them from data, how to extract them from databases, etc. So yeah, you can call it an ontology. Maybe links don't match the ontology name because links actually a description of how your business works.

Speaker A: Another question from Sylvan. Is it based on local LLM engine, GPU and local model?

Speaker B: We're LLM agnostic actually. So our current process looks like this. We use mighty powerful model in a powerful harness, for example CLAUDE or some similar tools to set up local LLM or some more or some cheaper and less powerful LLM. So we use CLAUDE basically to write all these prompts and all these playbook articles. How to how LLM should reason? So it's actually, it's agnostic. It can be run locally.

Speaker A: Okay, question from Isabel. Did you put some PDFs in the loop of one of workflows?

Speaker B: We extract data from PDF so we don't throw PDFs like as a full text into the LLM. So we might chunk them, but we use these chunks only as a backup data layer. Text layer. So we try to extract anchors, attributes and links from PDFs, put them into structured tables and then drop some chunks as a separate layer. But this text layer is a fallback for some reason. We can find data in structured part. We might look in PDFs, but then the quality drops dramatically. We just use chunks.

Speaker A: Okay, Any other questions? So Von says so Grifs is the preparation box for Engineer and the source of the work. So. So no Grist performance, Grist issue here, Grist performance.

Speaker B: Well, we stumbled upon grist performance issues when we have dramatically large data sets, like hundreds of thousands of rows. And it's the reason. And it's the reason for having all the data in mem graph, because sometimes we have extremely large data sets. But actually, we use grist mostly for experts. So the reason for our love to grist is that we let experts to manually edit some dictionaries, some expert data, and by definition, this can't be, like, millions of rows, because experts can support, like, small chunks of their data. Yeah.

Speaker A: Yeah. Okay.

Speaker B: Any other questions?

Speaker A: Yep. Nick is linking to a case study that we published about this exact same topic, which is very cool. Okay, well, thank you so much, Olga. This was great. I learned a lot.
