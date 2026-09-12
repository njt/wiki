---
url: https://datafusion.apache.org/
title: "Apache DataFusion — Extensible Query Engine"
author: Apache Software Foundation
date_fetched: 2026-08-01
date_published: 2024
topics:
  - databases-and-data
---

Apache DataFusion is a query engine written in Rust, built on Apache Arrow's
in-memory columnar format. It targets developers building custom database and
analytic systems — not end users — and exposes both SQL and DataFrame APIs.

It ships with a full query planner and a columnar, streaming, vectorized
execution engine that supports partitioned data sources. File format support
covers CSV, Parquet, JSON, and Avro out of the box.

Customisation is a first-class design goal: nearly every layer can be extended,
from data sources and query languages to user-defined functions (scalar, window,
aggregate, and table) and custom query optimisers.

The project also maintains bindings for Python and Java, plus DataFusion Comet
(an Apache Spark accelerator) and the now-separate Ballista distributed
execution framework.

Hosted by the Apache Software Foundation, the project publishes Rust crates,
API docs, and maintains three guides: a user guide, a library guide covering
extension APIs, and a contributor guide.
