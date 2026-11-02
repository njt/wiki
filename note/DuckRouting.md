# DuckRouting

`duckrouting` is a DuckDB community extension that puts a full pgRouting-shaped graph toolkit — shortest paths, flows, spanning trees, centrality, map matching — inside the database as SQL table functions, built on the Boost Graph Library. The interesting design move is that the graph is not a stored object: every function accepts *a query* that yields `id, source, target, cost, reverse_cost`, so any edge table is instantly a graph.

---

## The one-line idea

> "Any table with an id, two endpoints and a cost is a graph. No topology tables to build, no geometry required."

That is the whole pitch, and it is a good one. PostGIS/pgRouting made graph work feel like GIS: you build topology, you load geometry, you live in a spatial database. DuckRouting inverts that — the graph exists only for the duration of the function call, materialised from whatever SQL you hand it. A graph is a *view over your data*, not a second copy of it.

## What is actually in the box

The catalogue is startlingly complete for a community extension:

- **Pathfinding**: Dijkstra in seven flavours, A* (with x/y coordinates on the edges query), bidirectional Dijkstra and A*, Bellman-Ford and SPFA for negative weights, Yen's k-shortest-paths, Floyd–Warshall and Johnson all-pairs.
- **Structure**: connected/strong/biconnected components, articulation points, bridges, min cut (Stoer–Wagner), dominator trees, planarity (Boyer–Myrvold), colourings, bandwidth-reducing orderings (Cuthill-McKee, King, Sloan).
- **Trees and flow**: Kruskal and Prim with BFS/DFS/cost-limited walks; max-flow by four algorithms, min-cost max-flow, edge-disjoint paths, maximum-cardinality matching.
- **The routing extras that show this is a serious port, not a toy**: `trsp` for turn restrictions, `with_points` for routing to a house halfway down a street (a point partway along an edge), contraction hierarchies for preprocessing, `chinese_postman`, TSP from a cost matrix, `line_graph_full` that splits vertices into half-edges so turn costs attach, and HMM-based `map_match` for snapping GPS traces (Fast Map Matching, Yang & Gidofalvi 2018).

One convention deserves flagging because it bites everyone once: **a negative `cost` means the edge does not exist in that direction**. That is how one-way streets are encoded, inherited straight from pgRouting — it looks like a bug and is documentation.

## The verification story is the best part

> "Results are checked against pgRouting's own published output — the queries and expected results from its documentation and test suite — rather than against hand-written expectations."

This is differential testing against a twenty-year-old oracle, which is exactly the right call for an algorithm library where the bugs are subtle and the correct answers are already written down. Even better is the honesty clause: where the two implementations legitimately differ — a shortest-path tie, a non-unique spanning tree — the difference is documented and the tests assert the properties that *are* determined. Most ports fake determinism; this one refuses to.

## Opinionated take

DuckRouting is a quietly important data point in the "analytics database as application platform" trend. PostGIS made Postgres the default home for spatial work for fifteen years; a DuckDB extension that speaks pgRouting's exact semantics is a migration path, not a novelty — existing pgRouting queries port with minimal edits. And because the graph is just a query, graphs compose with everything else DuckDB is good at: you can `JOIN` a route against a Parquet file of telemetry without an ETL hop.

The trade-offs the page doesn't discuss: everything is computed in-process from scratch per call — no persistent graph index, no incremental updates — so contraction hierarchies exist but their preprocessed output lives only as query results. For interactive web routing engines this will never compete with Valhalla or OSRM; for *analytical* routing over data you already have ("which of these 10,000 depots is nearest, and how, over the data I loaded this morning"), it is exactly right. That analytical-versus-operational split is the same line every in-database algorithm extension walks.

**Tags**: #tool #databases #graphs #pattern

## How it relates

- Strengthens [[DuckDB ADBC Extension]]: both are evidence of DuckDB's community-extension ecosystem maturing beyond storage formats into domain engines — connectivity in one, algorithms in the other.
- Nuances [[Replace Athena with DuckDB (Lambda)]]: that source moved query workloads to DuckDB for cost; DuckRouting extends the same argument to workloads Athena never covered at all, like network analysis over raw edge tables.
- Complicates [[RAPIDS cuGraph]]: cuGraph is the GPU heavyweight for large-scale graph analytics with partitioning and a NetworkX API; DuckRouting is single-node CPU SQL with pgRouting semantics. They overlap on Dijkstra and components, but they answer different questions — scale versus analytical convenience — and it's worth being deliberate about which you need.
- Resonates with [[Procedural Graphs]] as a reminder that graph data in analytical systems keeps arriving by very different doors: generated, ingested, or merely implied by an edges table you already had.

---
*Sources: [[raw/duckrouting]], [[summary/duckrouting]]*
*Last updated: 2026-11-02*
