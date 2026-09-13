---
url: https://www.sqlservercentral.com/articles/databricks-genie-spaces-for-sql-analysts-natural-language-querying-without-leaving-your-data-platform
date_fetched: 2026-09-13
---

If you've spent years writing SQL for a living, you already know how this goes. Someone pings you on Teams asking, "How many active customers do we have in the Midwest this quarter?" There you are, back in the query editor, writing a JOIN you've written fifty times before.

Genie Spaces in Databricks is built to break that loop. It doesn't get rid of SQL. It puts a conversation in front of it, and lets the people who used to email you get their own answers.

## What a Genie Space Actually Is

A Genie Space is a chat interface that lives inside Databricks. A business user types a question in plain English (such as "what was revenue by region last quarter?") and gets back a real result: a table, a chart, a number. Genie writes the SQL, runs it, and hands back the answer. The person asking never opens a query editor and never waits on you.

The part worth understanding early is who does what. Analysts and data engineers build and configure the space. Business users consume it. You're not handing a colleague a query tool and wishing them luck. You're building a curated, governed layer on top of your data that already knows how your organization talks about its numbers.

Under the hood, the mechanics are simple: Genie takes a question, reads the metadata and rules you've given it, and translates the question into SQL. The theory is easy. The reality is that the answer is only ever as good as the context you feed it, the same garbage-in, garbage-out rule that's governed data work since the first spreadsheet.

## The Four Layers You're Actually Building

A Genie Space isn't a connection to a table. It's four layers stacked together, and they all have to be right. Get them all working and users get correct answers on the first try. Miss one and they start getting wrong answers, and once people stop trusting the tool, they don't come back to it.

The four layers are:

- **Data**— the tables or views Genie is allowed to look at.
- **Instructions**— the business context an analyst keeps in their head but never wrote down.
- **SQL Expressions**— the exact formulas for the metrics you can't afford to get wrong.
- **Example Queries**— worked examples that teach Genie how to handle your trickier questions.

Everything below walks through building each one, using a sales analytics space as the running example. I've tried to explain not just what to type but why it matters, because the settings are easy. Knowing what to put in them is the whole game.

## Step 1: Pick Your Tables — and Pre-Join Them

The instinct when you first set this up is to point Genie at everything and let it sort things out. Resist it. The more raw tables you expose, the more decisions Genie has to make about how they relate, and every decision is a chance to guess wrong.

A better approach is to hand Genie one clean, pre-joined view instead of a pile of raw tables. The view below isn't a single table. It's four of them (orders, customers, order_items, and products) already joined into one flat result. Genie now sees a single object where every relationship is already resolved and the test records are already filtered out. It never has to figure out how orders connect to customers, because you did that once, here.

-- Create a clean, pre-joined view in Unity Catalog. -- Four source tables collapsed into one flat view Genie can read. CREATE OR REPLACE VIEW catalog.sales_domain.vw_sales_summary AS SELECT o.order_id, o.order_date, o.region, o.status, c.customer_name, c.segment, p.product_name, p.category, oi.quantity, oi.unit_price, (oi.quantity * oi.unit_price) AS revenue FROM orders o JOIN customers c ON o.customer_id = c.customer_id JOIN order_items oi ON o.order_id = oi.order_id JOIN products p ON oi.product_id = p.product_id WHERE o.status != 'TEST'; -- keep test rows out of every answer, automatically

Point the Genie Space at this one view. That WHERE o.status != 'TEST' clause is doing quiet, important work: because the exclusion lives in the view, every question a user ever asks inherits it. You don't have to trust Genie to remember to filter test data. It's already gone.

## Step 2: Annotate Your Columns

This is the single highest-leverage thing you'll do, and it costs you nothing but a few sentences per column. Genie reads the COMMENT metadata on your columns as context before it writes any SQL. Without comments, a column called segment is just a word. Genie has no idea whether your segments are Enterprise/Mid-Market/SMB or something else entirely, so it guesses. With a good comment, it knows. When a user asks "how's the enterprise segment doing," Genie can map "enterprise" to the actual value in your data because you spelled it out.

So how do you structure a comment? A pattern that works well: say what the column is, list the values it can hold and what each one means, and flag any gotcha. That last part matters more than it looks.

-- Describe the column, enumerate its real values, and note any traps. COMMENT ON COLUMN catalog.sales_domain.vw_sales_summary.segment IS 'Customer tier: Enterprise (>$1M ARR), Mid-Market ($100K-$1M ARR), SMB (<$100K ARR).'; COMMENT ON COLUMN catalog.sales_domain.vw_sales_summary.status IS 'Order lifecycle: Pending, Processing, Shipped, Delivered, Cancelled. TEST rows are already excluded by the view.'; COMMENT ON COLUMN catalog.sales_domain.vw_sales_summary.region IS 'US sales region: Northeast, Southeast, Midwest, Southwest, West. State-to-region mapping lives in region_lookup.';

The region comment is a good example of why this pays off. When someone asks about "the Midwest," Genie now knows Midwest is one of five defined regions rather than a fuzzy geographic idea it has to reason about from scratch. You're not writing documentation for a human here. You're writing the vocabulary Genie translates against.

## Step 3: Write Your SQL Expressions (Your Certified Metrics)

Here's the term that trips people up the first time. A SQL Expression in Genie is a named, reusable metric definition that you register in the Genie Space UI. You give it a plain-English name, say "Active Customers," and paste in the SQL that calculates it. From then on, whenever a user asks a question that touches that metric, Genie doesn't invent a formula. It uses yours, exactly as written.

Ask ten analysts to define "active customer" and you'll get eleven answers. A SQL Expression ends the argument: you decide once what the number means, register it, and everyone downstream gets the same figure.

The name is not just a label for your own reference. It's what Genie matches user questions against. When a question comes in, Genie compares the wording to the names of your registered expressions, and if there's a match, it builds the answer around your SQL instead of writing the calculation itself. So the name you pick and the query you pair it with are two halves of one mechanism.

Take the first one. You register the name "Active Customers" with this SQL:

```
-- Register as: "Active Customers"
SELECT COUNT(DISTINCT customer_name)
FROM vw_sales_summary
WHERE status = 'Delivered'
  AND order_date >= DATE_TRUNC('quarter', CURRENT_DATE);
```
Now someone asks, "How many active customers do we have in the Midwest?" The phrase "active customers" matches the expression name, so Genie starts from your query: the distinct count, the Delivered filter, the current-quarter window. Then it adds the user's condition on top (AND region = 'Midwest'). What it doesn't do is decide for itself what "active" means. Both of your WHERE conditions travel with the metric wherever it goes. If you hadn't registered this, Genie would have to guess, and its guess might count anyone who placed any order this year, cancelled or not. That number would look perfectly reasonable right up until it didn't match the dashboard.

"YTD Revenue" works the same way:

```
-- Register as: "YTD Revenue"
SELECT SUM(revenue) AS ytd_revenue
FROM vw_sales_summary
WHERE order_date BETWEEN DATE_TRUNC('year', CURRENT_DATE) AND CURRENT_DATE
  AND status NOT IN ('Cancelled', 'Pending');
```
"YTD revenue," "revenue year to date," "how much have we sold this year" all hit the same name, and all of them resolve to this one query, including the part people forget: cancelled and pending orders are out. That exclusion used to live in one analyst's muscle memory. Now it lives in the metric.

This is also why I name expressions the way users talk, not the way the schema talks. "Active Customers" gets matched. cnt_dist_cust_qtd never will.

-- Register as: "Gross Margin %" (requires cost data in the view) SELECT ROUND(((SUM(revenue) - SUM(cost)) / NULLIF(SUM(revenue), 0)) * 100, 2) AS gross_margin_pct FROM vw_sales_summary;

Once "Gross Margin %" is registered, a question like "what's our gross margin this quarter?" resolves against that formula, NULLIF guard against divide-by-zero and all. Genie fills in the filters and grouping the user asked for, but the core calculation is locked. That's the difference between a number you can put in front of a VP and a number you have to caveat.

## Step 4: Add Business Instructions

Instructions are where you write down everything an analyst on your team already knows but has never actually documented. In the Genie Space UI you paste these in as plain English, and Genie reads them the way it would read a system prompt, before it does anything else.

How do you decide what belongs here? A simple test: if a new hire would get an answer subtly wrong because nobody told them a rule, that rule goes in the instructions. Here's the set from the sales space. For each one I've included the failure it exists to prevent, because an instruction only makes sense once you've seen what goes wrong without it. Each of the instructions is in bold below.

**"Fiscal year runs October 1 to September 30."** Without this, any question containing "this fiscal year" or "FY26" gets answered on a calendar year, because January 1 is the only default Genie has. Every FY number would be quietly off by a quarter, and it would look plausible enough that nobody catches it until finance does.

**"An 'active customer' is a customer with at least one Delivered order in the current quarter."** This backs up the Step 3 expression in plain English. The expression handles questions that match its name; this handles the ones that don't. "How many customers are actually buying from us right now?" contains no phrase resembling "active customers," but it's the same question, and this instruction gives Genie the definition to reach for however the concept is worded.

**"'Top performers' means ordered by revenue descending, limited to 10 rows, unless the user specifies otherwise."** Superlatives are ambiguous. "Top customers" could mean top by revenue, by order count, by growth, and "top" could mean 5, 10, or 25. Without a default, Genie picks one arbitrarily, and two people asking the same question on different days get differently shaped answers. This makes vague questions resolve the same way every time.

**"'Midwest' includes IL, IN, IA, KS, MI, MN, MO, NE, ND, OH, SD, WI."** Our view has a region column, but plenty of questions come in at the state level ("how's Ohio doing compared to the rest of the Midwest?"), and some of our supporting tables carry state codes rather than region names. This pins down exactly which states roll up into the term, so Genie doesn't fall back on its own general idea of the Midwest, which may or may not match your sales territory map. Ours doesn't include Kentucky. Genie's did.

**"Never include orders with status 'TEST' or customers whose names start with 'TST_'."** Half of this is redundant on purpose. The view from Step 1 already filters status = 'TEST', and I'd rather say it twice than trust one layer. The other half isn't redundant at all: test customers with the TST_ prefix slip through the status filter because their orders carry real statuses. Everyone on our data team knew to exclude TST_%. Nobody had ever written it down. This is exactly the kind of rule this box is for.

**"If the time period is unclear, ask the user to clarify before answering."** This is the one people forget, and it might be the most important. "How's revenue?" has no time period. Without this instruction, Genie assumes one, maybe all time, maybe this month, and answers confidently. With it, Genie replies "did you mean this quarter or this fiscal year?" A tool that occasionally asks a clarifying question earns far more trust than one that always answers instantly and is sometimes wrong.

Read the six back and they're all doing the same job: closing the gap between how the business talks and what the data literally says.

## Step 5: Seed Example Q&A Pairs

Some questions carry logic that Genie won't reliably infer on the first try: a specific set of filters, a particular grouping, an exclusion that isn't obvious from the schema. Example queries fix that.

First, what "registering" an example actually means, because the word hides the mechanism. In the Genie Space UI there's a section for example SQL queries. You add a pair: the question in plain English, and the SQL that answers it correctly. That's the whole act of registering. The pair is stored with the space, and when a new question arrives, Genie checks whether it resembles any of the stored ones. If it does, your SQL for the old question is sitting in front of Genie while it writes SQL for the new one, and it patterns the new query on yours.

No model gets retrained. Nothing is fine-tuned. It's closer to handing a new hire a worked example right before they attempt a similar problem. And the useful part is that an example reaches past its own exact question: its shape transfers to everything that resembles it.

Which questions deserve an example? Three kinds are worth the effort. The ones that come up every single week, because they're your highest-traffic path. The ones with non-obvious logic, where the right query has a filter or grouping a newcomer wouldn't guess. And the ones Genie has already gotten wrong at least once. Those last are your best candidates, because you've watched it fail and you know exactly what to correct.

Take a recurring one: "What is our revenue by product category this quarter?" It gets asked constantly, and the correct version quietly excludes cancelled orders, a detail a naive query would miss. So you register that question paired with this SQL:

```
SELECT
  category,
  SUM(revenue)             AS total_revenue,
  COUNT(DISTINCT order_id) AS order_count
FROM vw_sales_summary
WHERE order_date >= DATE_TRUNC('quarter', CURRENT_DATE)
  AND status NOT IN ('Cancelled', 'TEST')
GROUP BY category
ORDER BY total_revenue DESC;
```
A month later someone asks, "How did the product categories do in Q2?" Different words, different quarter, not the registered question. But it's shaped like the registered question, so Genie pulls up your example, sees the structure (group by category, sum revenue, exclude Cancelled), and rebuilds it with the date range swapped. The exclusion you encoded once now shows up in a query you never wrote, answering a question you never anticipated.

A few more that have earned their place in our sales space, and what each one is really pinning down:

- **"Which customers churned this quarter?"**Churn logic is never obvious. Is it no orders in 90 days? No Delivered orders this quarter after having some last quarter? Genie can't derive your churn window from the schema, so the registered SQL is where that definition gets settled.
- **"Show new customers acquired this month."**"New" depends entirely on your definition. Ours is "customers whose first-ever order date falls in the current month," which takes a MIN-over-customer pattern Genie won't produce on its own.
- **"Revenue this quarter versus the same quarter last year."**Easy to ask, fiddly to write: two date windows and a delta calculation. Get the date math right once and every year-over-year question that follows inherits it.

## What Your Stakeholders Get, and Which Layer Delivers It

Once the space is live, here's what the people who used to email you can do on their own. Pay attention to the third column: every one of these answers works because of a specific layer you built above, not because Genie is clever on its own.

| Natural language question | What Genie generates | The layer doing the work | 
|---|---|---|
| "Show me top 10 customers by revenue this quarter" | GROUP BY + ORDER BY revenue DESC + LIMIT 10 | The Step 4 instruction defining "top performers" (revenue descending, 10 rows), plus the precomputed revenue column from the Step 1 view | 
| "What's our YTD gross margin by region?" | Your Gross Margin % formula, grouped by region | The Step 3 "Gross Margin %" expression supplies the locked formula; the Step 2 comment on region tells Genie what the grouping values are | 
| "How did the Midwest perform vs last quarter?" | Quarter-over-quarter comparison filtered to Midwest | The Step 2 region comment resolves "Midwest" to a real column value; the Step 5 period-comparison example supplies the date math | 
| "Which product categories are declining?" | Month-over-month trend with a negative-growth filter | The Step 5 revenue-by-category example provides the grouping and exclusion structure; Genie extends it with a trend comparison | 
| "How many active customers do we have?" | Your certified Active Customers count | The Step 3 "Active Customers" expression, matched by name, used verbatim | 

That third column doubles as your debugging map. When a row comes back wrong, it tells you which layer to go fix.

## What I've Learned Running These in Production

The four layers above are the build. These are the things that actually keep a space trustworthy after it goes live, and every one of them ties back to a step you just did.

**Keep the scope tight.** The pre-joined view from Step 1 works precisely because it's narrow. A space aimed at sales analytics with one clean view will beat a space with sixty raw tables dumped into it every time. When you're tempted to add "just a few more tables," build a second space instead.

**When a metric can be a formula, make it one.** Step 3 exists for a reason: text instructions are guidance, but a SQL Expression is truth. If you catch yourself trying to describe a calculation in the instructions box in Step 4, stop. That belongs in an expression with a name on it.

**Always read the generated SQL.** After Genie answers, open the SQL panel and look at what it actually ran. When it misreads something, and early on it will, that's your cue to go back to Step 5 and register the corrected query. Every misread you fix is one that won't happen again.

**Mind your synonyms.** This is the most common gap I see. Your sales team says "client," your customer_name column says otherwise, and Genie has to guess they're the same thing. They're not, as far as it knows. Add the mapping to your Step 4 instructions and the guessing stops.

**Treat your examples as regression tests.** Once a question answers correctly, keep that pair around and re-run it after any change to the space: a new column comment, a tweaked expression, an added instruction. A change that fixes one question can quietly break another, and running your known-good examples is how you catch it before your users do.

## The SQL Analyst's New Role

The honest version: Genie doesn't replace SQL analysts. It changes what the job looks like. Instead of writing the same ad-hoc query for the fiftieth time, you become the person who decides what "active customer" means, who certifies the revenue formula, who curates the vocabulary the whole organization queries against.

You're building the knowledge layer, the certified metrics, the business rules, the shared language, so that everyone gets the same answer to the same question. That's a bigger job than writing JOINs, not a smaller one.

## About the Author

Mehul K. Bhuva is a Data & AI Platform Engineer with 22+ years of hands-on experience building enterprise-scale data and AI solutions on Databricks and Azure. Based in the Des Moines, Iowa area, he currently works at Corteva Agriscience — one of the world's leading agricultural science companies — where he designs and builds data engineering pipelines on Databricks, Genie Spaces, Dashboards, Databricks Apps, and AgentBricks applications.

Mehul is the developer of the Sprout-AI Framework (Seed Ops Savant), a production-grade RAG and Agentic AI solution built on Azure AI Foundry that has transformed how Corteva's teams interact with operational data. His work on metadata-driven ingestion pipelines and enterprise RAG systems has been recognized in peer-reviewed research, including a published paper on real-world RAG implementation using Microsoft AI Foundry: https://ijcotjournal.org/archive/ijcot-v15i2p302

A recognized voice in the Microsoft and Databricks community, Mehul is a Microsoft Azure Developer Influencer, a recurring guest on Microsoft Dev Radio, a regular speaker at Microsoft User Groups, and a frequent author at SQL Server Central. He holds a Master's degree in Computer Science from the Georgia Institute of Technology and writes regularly on data, AI, and cloud architecture at sharepointfix.com.
