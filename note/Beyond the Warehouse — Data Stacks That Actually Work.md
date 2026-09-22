# Beyond the Warehouse — Data Stacks That Actually Work

Thomas in 't Veld (founder of Tasman Analytics, a consultancy that builds data stacks for fast-growing startups and scale-ups) gave this conference talk at an AI & Data Summit, preserved as a ytx YouTube transcription gist. He diagnoses why data projects fail — over-designed stacks, neglected data modeling, failed activation — and prescribes a lean stack organized around a "narrow waist" domain model, with the semantic layer as both the source of truth and, increasingly, the security boundary and interface for AI agents. It is one of the crispest practitioner statements of the semantic-layer-for-agents thesis from the data-engineering side, delivered with unusual bluntness and an unusually honest self-critique section.

---

## What it argues

The VentureBeat "85% of data projects fail" stat is his hook, not his argument. From ~60–70 client engagements over six years he names three failure modes: **over-designing the stack** (six months building the perfect stack before thinking about use cases), **neglecting data modeling**, and **failing at activation** (the marketing team downloads raw data into Excel anyway).

Three structural claims:

- **Analytics is not production.** "Your data stack is not a production engineering stack." Production can tolerate a duplicate pro-subscription event; analytics cannot, because duplicates double-count revenue. Event streams are not logs — they must be unique, complete, and objective, which is why tracking plans (Avo) and event governance exist. Treating event streams as logs "makes data modeling really, really, really shitty down the line."
- **Prioritize by business value, shift left.** Rank the insights stakeholders need and load sources in that order — in subscription startups, revenue reporting trumps everything, so RevenueCat goes in first. Fix data problems at source ("Don't treat the data source as canonical gospel. It will probably be wrong when you load it"), not in the model.
- **The narrow waist.** Query-driven development — bolting on a query per business question — is "the most surefire way to get your data stack and your data team into trouble." Instead, recognize the business is a relatively static target and model its entities once in the middle of the stack: Inmon-style entity modeling in the warehouse, fanning out into Kimball-style dimensional or one-big-table marts. The payoff is vendor-proofing: swap HubSpot for Salesforce and only extraction logic changes.

On top of that: infrastructure-as-code all the way down (dlt for ingestion-as-code, dbt for transformation, Prefect orchestration, CI/CD running the whole test suite on every change — "25, 30% actually building the pipelines and it's 70, 75% building the testing suite"), and a **semantic layer** carrying entities, dimensions, join rules, and canonical metric definitions, version-controlled and CI-enforced. Three build paths: homegrown text files, dbt's semantic layer, or Omni. His agent claim: the semantic layer is "the only possible way that you can get safe automation and... safe AI agentic applications."

Activation gets four pillars: presentational models linked to decisions, very short feedback loops, every insight treated as a data product with scope and acceptance criteria, and internal analytics on your own dashboards ("is my CEO actually looking at it every single day. And the answer is probably no").

## Key quotes

> "The world is littered with the corpses of companies that did everything right. They spent €200,000 on the best possible data stack. Still couldn't make a good decision."

His framing of why tool choice is necessary but nowhere near sufficient — the failure is upstream of the stack, in what the business is trying to decide.

> "Your data stack is not a production engineering stack."

The load-bearing principle behind event uniqueness and tracking plans. It's a sharper version of a distinction most data engineers feel but rarely state: analytics pipelines must be unique, complete, and objective, because the downstream consumer is a metric, not a human eyeballing a log.

> "If you end up with two different types of semantic layers, you've done it wrong."

Zero tolerance on metric inconsistency, from the Q&A. Consistency across platforms *is* the point of the semantic layer; two layers means you have two truths, which is the disease the layer exists to cure.

> "The main risk is querying raw data... one of the reasons that the data model exists is to remove raw data access as much as possible from downstream consumption patterns... you do not want your agentic implementation to access your raw data."

The most agent-relevant claim in the talk: PII exposure, prompt injection, and SQL injection all live at the raw layer, so the well-designed data model is the security control. Note what this is NOT: it is not a sandbox or a permission system — it is an architecture argument.

> "Burn it all, start from scratch... burn it with petrol and fire and just move on."

On legacy, inconsistent event data. Events are "written in stone" and can't be regenerated, and year-old event data "is just not that relevant as people think in current business decision making." His immediately following admission — that auditing and repairing in the data model is "a very expensive migration job, which actually we don't really like to do" — tells you which consultancy service line this advice protects.

> "I've never met a stakeholder who's able to describe in detail exactly the type of dashboard reporting that they need from scratch."

Why a dashboard is the start of a conversation, not an endpoint. Paired with "Keep receipts" — save the emails inviting stakeholders to definition sessions, so you can point to when they had their chance to define the logic. This is the talk's only genuinely political advice, and it's good.

## Key themes

- #concept — the **narrow waist**: model the business's entities once, in the middle, as a static target; let everything above and below churn.
- #concept — the **semantic layer** as source of truth, CI-enforced, and the precondition for AI in analytics.
- #tool — dlt, Rudderstack/Snowplow, Avo, RevenueCat, Databricks/Snowflake, dbt, Omni, Elementary; the talk doubles as a 2026 default-stack recommendation.
- #pattern — "buy tools, build the glue": never build what's off-the-shelf (text-to-SQL engines exist; use them), but own your CI/CD layer and your tests.
- #person — Thomas in 't Veld, Tasman Analytics; consultative, blunt, pragmatic ("Do not confuse intellectual purity... with what's really useful for you").

## Opinionated take

The strongest and most transferable idea here is the **narrow waist**, and it deserves to be read alongside specification-first material: the half-day, everyone-in-a-room domain-model paper exercise ("what is a user? does a user need an ID or just an email?") is a specification session, and the resulting model is a durable artifact that outlives any particular query, dashboard, or vendor. In 't Veld's own tension — "as long as your business doesn't change fundamentally" is doing enormous load-bearing work — is the honest cost of that spec: a static target is only static until a pivot, an acquisition, or a new product line moves it, and he has nothing to say about what happens then.

The **agent claims are the reason this talk belongs in this wiki, and they are under-argued.** "Semantic layers are critical to do anything with AI in analytics" and "never let agents query raw data" are asserted, not demonstrated: MCP is dismissed in half a sentence ("That's not just building an MCP server"), there is nothing on agent evaluation or hallucination risk, and the Power BI Copilot harmonization question gets answered with vendor-lock-in boilerplate. The claims still land, but only because independent evidence exists elsewhere — see the cross-links below. His answer is purely architectural: the model is the boundary. That's a real answer, but it's the *first* answer, not the whole one.

The gist's own "Unanswered Questions" section (generated by the transcription tool, and to its credit) catches what the talk evades: the 85% hook is never earned — no before/after outcomes from those 60–70 clients; enterprise is waved off as "very different beast altogether," and the data-mesh answer to multi-team consistency is deferred to a hallway conversation; the dbt criticism is teed up ("looking at alternatives... given how DBT as a tool is evolving") and never delivered; and the promised "one opinion" on the people problem never arrives — the talk stays almost entirely technical. Add the consulting frame: recruitment pitch included, and the burn-it-all advice conveniently steers clients away from the migration work his firm dislikes. Treat the failure statistics as marketing and the architecture advice as the real product — the architecture advice is good.

## Related pages

- [[Text-to-SQL in the Real World]] — strengthens the talk's central agent claim from the research side: Stonebraker & Chen show agents collapse to ~10% on real enterprise warehouses with schema rot, which is exactly why in 't Veld's "the main risk is querying raw data" is right — and why the modeled layer, not raw tables, is where agent access belongs.
- [[Lessons from Building Vercel v0 and the d0 Agent]] — convergent evolution: Ubl's YAML semantic layer as the durable artifact mirrors in 't Veld's homegrown "bunch of text files" semantic layer fed into an agentic engine; both treat semantics as the asset and SQL as disposable. It also complicates Ubl's 50-line-agent claim — in 't Veld is explicit that the semantic layer is where the sustained human maintenance burden lives.
- [[KTX Context Layer for Data Agents]] — the same design space one layer down: KTX's comparison of dbt semantic layer / MetricFlow / Looker import paths maps directly onto in 't Veld's three build paths, and KTX adds the wiki-context and review workflow he never discusses.
- [[Databricks Genie Spaces for SQL Analysts]] — nuances the Q&A pragmatism: Bhuva rebuilds the semantic layer as prompt context inside Databricks, and in 't Veld blesses Databricks' native metrics as pragmatically fine — but his "two different types of semantic layers, you've done it wrong" line is precisely the consistency risk a per-tool semantic layer creates.

The [[Databases and Data]] topic page collects more of the wiki's data-systems material; [[Bad Data in Production — Response Playbook]] covers what happens when the stack in 't Veld describes fails anyway.

---
*Sources: [[raw/beyond-the-warehouse-data-stacks-that-actually-work]], [[summary/beyond-the-warehouse-data-stacks-that-actually-work]]*
*Last updated: 2026-09-13*
