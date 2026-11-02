---
url: https://duckdb.org/community_extensions/extensions/duckrouting
title: "duckrouting — Graph Routing for DuckDB"
author: DuckDB Community Extensions
date_fetched: 2026-11-02
date_published: unknown
topics:
  - databases-and-data
  - developer-tools
---

The DuckDB community extension catalogue page for `duckrouting`, an extension that brings graph routing — shortest paths, flow, spanning trees, traversal, centrality — into DuckDB as SQL table functions, following pgRouting's conventions and built on the Boost Graph Library.

The extension's design premise is stated in its example: "Any table with an id, two endpoints and a cost is a graph. No topology tables to build, no geometry required." Every function takes the graph as a string of SQL returning `id`, `source`, `target`, `cost` and optional `reverse_cost`, where a **negative cost means the edge does not exist in that direction** — pgRouting's convention for one-way streets, not an error condition.

The function catalogue is very large: Dijkstra (plus `_cost`, `_cost_matrix`, `_via`, `_near` variants), A* and bidirectional forms, Yen's k shortest paths, Floyd–Warshall and Johnson all-pairs, connected/strong/biconnected components, articulation points, bridges, Kruskal and Prim spanning trees with BFS/DFS traversals, breadth- and depth-first search, a full flow family (`max_flow`, `push_relabel`, `edmonds_karp`, `boykov_kolmogorov`, `max_flow_min_cost`, `edge_disjoint_paths`, `max_cardinality_match`), contraction hierarchies, and analysis functions (colouring, planarity, betweenness centrality, min cut, dominator tree). Beyond routing proper there are `tsp` and `tsp_euclidean`, `chinese_postman`, `line_graph` (including a full form that splits vertices into half-edges so turn costs can be attached), turn-restriction-aware `trsp` functions, `with_points` functions that route to points partway along an edge, and a hidden-Markov-model `map_match` for snapping GPS trajectories onto the network (Fast Map Matching, Yang & Gidofalvi 2018).

Two aspects stand out beyond the function list. First, verification: results are checked against pgRouting's own published documentation and test-suite output rather than hand-written expectations, and where the two legitimately differ — a shortest-path tie, a non-unique spanning tree — the difference is documented and the tests assert the properties that *are* determined. Second, the geometry helpers (`duckrouting_find_close_edges`, `duckrouting_separate_crossing`, `duckrouting_separate_touching`) are SQL macros over DuckDB's `spatial` extension, kept deliberately optional.

The page ends with the standard extension-catalogue machinery: a compatibility matrix across current DuckDB releases and platforms including WebAssembly, install/load instructions, the full added-functions table with examples, and no overloads, types, or settings.
