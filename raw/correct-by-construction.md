---
url: https://minimalmodeling.substack.com/p/my-take-on-data-quality
date_fetched: 2026-07-05
backfilled: true
---

# My take on data quality

### Whitelist approach, correct by construction

I keep hearing discussions on data quality, and here is my take on that topic. My view is strongly shaped by the Minimal Modeling approach. It helps to look through the layers at the underlying Small Data, which is easier to understand and to reason about.

This text describes an approach, it is not a concrete solution. It does not depend on any specific data stack, and it does not depend on a specific data storage topology. It doesn’t matter how your data is copied around, extracted, loaded and transformed, what sort of data warehouse you use and so on.

There are three tiers in this framework:

- **Tier 1**/ how to establish the input data quality for a specific query;
- **Tier 2**/ how to maintain that data for that specific query, so that it stays up-to-date;
- **Tier 3**/ how to maintain all data for all queries that need to be run, including both existing and new.

The main idea: we use **a whitelist approach, building the dataset that is correct by construction**.

## Tier 1. Establish input data quality for a specific query

Suppose that we have a query that we either need to write, or investigate because we’re not satisfied with the results. A query could be a specific SQL query, but it could also be more general: a script that queries and processes the data, a workflow, or even a piece of application code.

We treat queries as **functions that turn input data into output results**.  Remembering the GIGO principle (“*garbage in — garbage out*”) we first need to establish the input data quality.

According to Minimal Modeling, we think in terms of **anchors, attributes and links**.  Any input data needed for any query could be represented in such a way.

It’s tempting to start from attributes because they contain the actual data, but we need to start with anchors. When you see that you have an attribute, you need to first make sure that its anchor is also correct.

### Data quality: anchors

Anchors are things like users, orders, invoice lines, payments, etc.  It’s something that could be counted: for example, you could say “*there are 25120 orders in our database*”.

Anchors are slightly weird: they do not carry any data, they only carry IDs. Here is a checklist for anchor data quality. This checklist needs to be repeated for each anchor.

First you need to make sure that you can get **all the IDs for an anchor**.  For example, if you have 25120 orders in your database, you need to be able to run a query that returns 25120 unique IDs.  Again, a query could be a specific SQL query, but it could also be more general such as a script, or several queries.

An example for orders would be pretty trivial:

`SELECT id`

FROM orders

It’s not necessary to actually get the entire list, because that list may be quite long. But you need to clearly understand how to do that.

Second, you need to make sure that **the returned IDs are unique**. In the best case you just have a table with a primary key, so you can get the IDs directly.

In some cases you do not have a proper source of unique ID and you have to do a `DISTINCT` to get the IDs.  This is one of the cases where it actually makes sense to create an actual small one-column table that keeps all the IDs available to you, because it could greatly simplify your queries and will help with data quality.

Third: a very common scenario is **archived data**.  If your business deals with orders of any sort, you probably have a table with recent orders, and an archive table where old orders are moved to.  Dealing with archived data is a separate topic, just make sure that you understand what exactly is in that list of IDs.  Is it only a list of current orders?  Do you have a separate list of archived orders?  Or do you have a complete list that combines both.  You could even have all three, depending on the needs of your queries.

What we have so far is an SQL query that can return a compact dataset. We’ll discuss how to materialize this dataset below.

### Data quality: attributes

Attributes store the actual data: numbers, strings, dates and timestamps, monetary values, coordinates, yes/no and enums. Two important things:

- JSON and such is not an attribute (it may be seen as a form of table, so you need to drill down further); 
- ID is not an attribute (it’s part of the link, see below); so, if you have an Employee anchor then manager_id is not an attribute; 

Attributes always belong to some anchor. If you need an attribute, first you need to make sure that you defined its anchor, as explained above.

For orders, a good example of an attribute is “*when was this order placed?*”.  For employees, it’s “*what is the date of birth of this employee?*”  For an item, it’s “*what is the SKU of this item?*”

For attributes, we need to write down an SQL query that returns clean attribute data. Such queries would return just two columns: anchor ID and the attribute value. Examples:

`SELECT id, created_at`

FROM orders;

`SELECT emp_id, date_of_birth`

FROM employees

WHERE date_of_birth IS NOT NULL;

`SELECT item_id, sku`

FROM items

WHERE sku IS NOT NULL AND sku <> ‘‘;

Notice that we **filter out not only NULLs, but also all sorts of sentinel values**, such as empty strings.  You can also filter out strings such as “*UNKNOWN*”, or nonsensical date values such as “*0000-00-00*” which are often used in real databases.

Make sure that the ID is unique. This is only a problem when your data source is not normalized, but this needs to be considered.

Here you can also **canonicalize attribute values**, fixing common typos, spelling differences and other such things.  For that, you can use something like a CASE statement:

`SELECT id, CASE WHEN color = ‘crimson’ THEN ‘red’ ELSE color END AS color`

FROM items

This example canonicalized the value “*crimson*” to “*red*”.  If you have many such substitutions use a separate table and a LEFT JOIN.

It’s always possible that there are fewer rows in the attribute dataset than in the anchor dataset, due to filtering out some values.

Materialization of attribute queries is discussed below.

### Data quality: links

Links store the relationships between two anchors. It could be two different anchors: “*an Order was placed by the User*”, or involve the same anchor: “*an Employee is managed by another Employee*”.

Both anchors used in the link need to be defined as explained above.

For each link we need to write down an SQL query that returns exactly two columns: first ID and the second ID.  Only real IDs must be present: **we filter out NULLs and other sentinel values**, such as 0 or even -1.

Here are two examples:

`SELECT id AS order_id, user_id`

FROM orders

WHERE user_id IS NOT NULL

`SELECT manager_id, id AS employee_id`

FROM employees

WHERE manager_id IS NOT NULL AND manager_id <> 0

The first example assumes that orders could be made by the users but also in guest mode, without an account. That’s why we add the WHERE condition.

Make sure that **each pair of IDs is unique** in the dataset.  This is a strict requirement for any link.

In our definition of links there are only three possible cardinalities: M:N, 1:N, and 1:1. If an order is anonymous, there would be no row mentioning that order in the link dataset at all. This approach could be a bit counterintuitive if you’re used to thinking in terms of tables.

There could be multiple links between the same two anchors, for example:

- *an Expense was submitted by a User;*
- *an Expense was approved by a User;*
- *an Expense was processed by a User;*

Each one is a separate link, we never try to make it more abstract: no “`type`” column or anything of the sort.

Also, as opposed to the relational model, there are no 3-way links (this follows from the minimality considerations, more on that elsewhere).

### Deliverable summary so far

For each **anchor** you need to write down:

- how to get a one-column dataset: list of all unique IDs (SQL query, a piece of code, or a readable explanation); 
- the retention policy of those IDs: how far in the past could you expect this list to be available, and is there is a separate archive dataset; 

For each **attribute**:

- how to get a two-column dataset: anchor ID + attribute value (SQL query or something); 
- the anchor is well-defined; 
- no NULL values, no sentinel values; 
- attribute values cannot contain anchor IDs; 
- values are canonicalized (typos and spelling variations fixed); 
- only atomic types (no JSON); 
- anchor IDs are unique; 

For each **link**:

- how to get a two-column dataset: first anchor ID + second anchor ID; 
- both anchors are well-defined; 
- no NULL IDs, no sentinel values; 
- each pair of IDs is unique; 

NB: if you cannot satisfy those criteria that means that you’ve made a modeling error and probably need to rethink the corresponding data element. Most probably you need a different anchor.

### Materializing quality datasets: a peek into tier 2

So, you have a number of relatively small SQL queries that return compact datasets: one-column or two-column.  The most simple thing that you could do now is to *just* materialize those datasets: create compact tables, one per element.

I understand that this is not how things are commonly done, but I assume that you have a problem with data quality, and you want to try a different approach. So here is a different approach, let’s try it and see what happens when you do things that way. Maybe you’ll get better quality data, who knows?


Nothing, though, is ever “*just*”.  If we simply materialize those datasets, we’ll essentially have a current one-time snapshot of the data. We need to keep those datasets updated as the source data changes.  Basically, we need a pipeline.  That would be a tier 2, to be discussed in the follow-up post.

Another concern that you may have is how to query that dataset: people hate joining tables, and here we basically need the maximum possible number of joins. We’ll discuss more familiar table schemas when we finish talking about correctness. It’s easy to build a flat table from good data, if you need a flat table.

### Correctness by construction

For now we’ll be doing a bit of handwaving and pretend that we have the quality datasets magically up-to-date. Even with a snapshot of data, we can try using that data, and think about the properties of that data.

Notice that this approach to data quality is basically **a whitelist**: we attempt to build the dataset that is correct by construction. Rereading the requirements in “Deliverable summary so far”, we can see that each of them helps to avoid a common source of data problems:

- unclear identity of entities; 
- unclear cardinality; 
- duplicate rows; 
- NULLs; 
- sentinel values (more insidious than NULLs); 
- trivial inconsistencies in data, such as spelling and typos; 
- unrestricted containers such as JSON; 

Those are “static” properties: when your dataset is clean, your downstream queries would have a better chance to return correct data.

In the next post we’ll discuss the pipeline approach that is designed specifically for keeping such quality datasets up-to-date. We’ll cover:

- common mistakes in propagating changes; 
- dealing with breaking and non-cooperating upstream datasets; 
- dealing with pipeline outages; 
- data dependencies and latency; 

The further upstream we can detect the errors the easier it is to fix them.
