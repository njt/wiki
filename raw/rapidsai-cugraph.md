---
url: https://github.com/rapidsai/cugraph
title: cuGraph — GPU-Accelerated Graph Analytics
author: NVIDIA / RAPIDS
date_fetched: 2026-08-01
date_published: 2019
---

# cuGraph

RAPIDS cuGraph is a GPU-accelerated graph analytics library from NVIDIA. It provides a collection of graph algorithms (PageRank, BFS, SSSP, Louvain, Leiden, Triangle Counting, etc.) implemented in CUDA C++ with Python bindings, designed to interoperate with the RAPIDS ecosystem (cuDF, cuML) and NetworkX.

## Architecture

cuGraph is a four-layer stack:

1. **C++/CUDA core** (`cpp/`): The engine. 133 header files defining graph data structures, ~30 GPU primitives, and algorithm implementations. Located in `cpp/include/cugraph/` and `cpp/src/`.

2. **C API** (`cpp/src/c_api/`): A C wrapper (`cugraph_c`) around the C++ core, providing type-erased function interfaces for FFI. Files like `pagerank.cpp`, `bfs.cpp`, `louvain.cpp` implement the C-callable wrappers using an `abstract_functor` pattern with `reinterpret_cast` for type erasure.

3. **Cython bindings** (`python/pylibcugraph/`): ~17K lines of `.pyx`/`.pxd` files providing thin Python wrappers. Each algorithm gets a `.pyx` file (e.g., `pagerank.pyx`) that calls into the C API. This layer handles Python-to-C type conversion and memory management.

4. **Pure Python API** (`python/cugraph/cugraph/`): ~42K lines. The NetworkX-compatible user-facing API. Organized by algorithm category: `community/`, `centrality/`, `link_analysis/`, `sampling/`, `traversal/`, `link_prediction/`, `layout/`, `structure/`. Also includes Dask-based distributed wrappers in `cugraph/dask/`.

The repo also contains `libcugraph` — a C library package with minimal Python loading utilities, and `nx-cugraph` — a NetworkX backend plugin that transparently accelerates NetworkX code on GPU.

## Graph Representation

The core data structure is a CSR/CSC adjacency list with a hybrid DCSR (Doubly Compressed Sparse Row) extension for scale-free graphs with many low-degree vertices. The key types:

- **`graph_t<vertex_t, edge_t, store_transposed, multi_gpu>`** — Owning graph. Stores `rmm::device_uvector` for offsets and indices. `store_transposed` is a compile-time bool: false = CSR (source-major, push model), true = CSC (destination-major, pull model).

- **`graph_view_t<...>`** — Non-owning graph view. Wraps `raft::device_span` into the offset/index arrays. Created from `graph_t::view()`. This is what algorithms consume.

- **`partition_t<vertex_t>`** — 2D edge partitioning for multi-GPU. Given P GPUs arranged as major_comm_size × minor_comm_size, each GPU owns `minor_comm_size` rectangular partitions of the edge matrix. Based on Boman et al., "Scalable matrix computations on large scale-free graphs using 2D graph partitioning" (2013).

Both `graph_t` and `graph_view_t` are specialized via `std::enable_if` for single-GPU vs multi-GPU. The single-GPU versions are simpler — one offset array, one index array, the full vertex range.

### Graph construction

Graphs are constructed from edge lists. The pipeline:
1. Renumber vertices (optional, for performance — contiguous vertex IDs improve locality)
2. Sort by source (CSR) or destination (CSC)
3. Build offsets array (prefix sum over degree counts)
4. Optionally sort by degree and segment (high/mid/low/hypersparse) for load balancing
5. In multi-GPU: partition vertices across GPUs, shuffle edges to their owning GPU

### Segment-based degree sorting

Vertices are grouped into segments by degree:
- **High degree** (>1024): processed with many threads per vertex
- **Medium degree** (warp_size to 1024): processed with warp-level parallelism  
- **Low degree** (<warp_size): processed efficiently in aggregate
- **Hypersparse** (below threshold): stored in DCSR/DCSC format where zero-degree vertices are excluded from the adjacency

This is critical for GPU efficiency on scale-free graphs where degree distribution spans many orders of magnitude.

## Primitives Layer

The 30 primitives in `cpp/include/cugraph/prims/` are the building blocks for all algorithms. They're CUDA C++ template functions following Thrust-like conventions:

- **`per_v_transform_reduce_incoming_outgoing_e`**: For each vertex, iterate over its incident edges, apply a quinary operator (src, dst, src_val, dst_val, edge_val) per edge, and reduce. This is the workhorse — BFS, PageRank, SSSP all build on it.

- **`vertex_frontier`**: Manages the active vertex frontier with a key-bucket abstraction. Supports both push (iterate outgoing edges from frontier) and pull (iterate incoming edges to frontier) traversal. The BFS implementation uses this for direction optimization.

- **`transform_reduce_e` / `transform_reduce_v`**: Edge and vertex reductions.

- **`count_if_e` / `count_if_v`**: Counting operations with predicates.

- **`edge_bucket`**: Group edges by destination for aggregation.

- **`extract_transform_if_e`**: Filter and transform edges.

- **`update_edge_src_dst_property`**: Update edge source/destination properties (used for iteration state).

The critical design insight: **these primitives fuse vertex iteration, edge property lookups, computation, and reduction into single kernel launches**. A typical CPU graph framework would issue O(E) random memory accesses; cuGraph issues O(V/block_size) coalesced reads within each kernel.

## Algorithm Implementation Pattern

Every algorithm follows the same instantiation pattern with 4 explicit template specializations:

```
algorithm_sg_v32_e32.cu  — single-GPU, 32-bit vertex, 32-bit edge
algorithm_sg_v64_e64.cu  — single-GPU, 64-bit vertex, 64-bit edge
algorithm_mg_v32_e32.cu  — multi-GPU,  32-bit vertex, 32-bit edge
algorithm_mg_v64_e64.cu  — multi-GPU,  64-bit vertex, 64-bit edge
```

The `.cuh` header (e.g., `pagerank_impl.cuh`) contains the template implementation. The `.cu` files instantiate it with concrete types. This compile-time specialization avoids runtime dispatch overhead and enables GPU-specific optimizations for different integer widths.

### PageRank implementation (pagerank_impl.cuh)

Uses the power method with a pull model (`store_transposed=true` — reads incoming edges). Each iteration:
1. `per_v_transform_reduce_incoming_e` with a custom edge operator that computes `src_rank / out_degree`
2. Apply damping factor and personalization vector
3. Check convergence via L1 norm of difference

### BFS implementation (bfs_impl.cuh)

Direction-optimizing BFS: maintains both top-down (push from frontier) and bottom-up (pull to unvisited) paths, switching based on frontier size and edge count estimates. Uses `vertex_frontier.cuh` for frontier management and `visited_bitmap` for efficient membership testing.

## Multi-GPU Design

Multi-GPU communication uses NCCL via the RAPIDS RAFT library (`raft::comms`). Key patterns:

- **2D partitioning**: Vertices split 1D across GPUs. Edge matrix split 2D as major×minor — each GPU gets a rectangular submatrix per minor partition. This is more communication-efficient than 1D partitioning for algorithms that access both rows and columns.

- **Shuffle operations**: `shuffle_functions.hpp` redistributes vertex/edge data between GPUs. Uses `groupby_and_count` and `collect_comm` utilities.

- **Graph construction in multi-GPU**: Edge list is partitioned, vertices are renumbered locally, then global renumbering maps are exchanged.

## MTMG (Multi-Threaded Multi-GPU)

The newer `mtmg/` abstraction layer provides per-thread GPU access with shared memory semantics:

- `handle_t`: Per-thread GPU resource handle with isolated CUDA streams from a stream pool
- `graph_view_t`: Immutable shared graph view (copy-constructible between threads)
- `edge_property_view_t`: Read-only edge property access
- `vertex_pair_result_t`: Thread-local result containers

This allows multiple CPU threads to concurrently run algorithms on different subgraphs or different phases, sharing the same GPU-resident graph data.

## C API Design Pattern

The C API uses an `abstract_functor` base class with a virtual `operator()`:

```cpp
struct pagerank_functor : public cugraph::c_api::abstract_functor {
  // stores all parameters as members
  void operator()() override {
    // type-erased graph → reified graph_view_t via dispatch
    // call the C++ template implementation
    // write results to type-erased result struct
  }
};
```

The C function `cugraph_pagerank()` constructs the functor, calls it, and extracts results. Type erasure uses `reinterpret_cast` from opaque C structs back to typed C++ objects — safe because the C API constructs and owns them.

## Python Layers

### pylibcugraph (Cython)

Each `.pyx` file wraps one algorithm. Pattern:
1. Import C types from `pylibcugraph._cugraph_c.*`
2. Accept Python objects (ResourceHandle, _GPUGraph, arrays)
3. Convert to C types via helper functions
4. Call the C API function
5. Check errors, extract results back to Python (CuPy arrays)

### cugraph (Pure Python)

The `Graph` class wraps either `simpleGraphImpl` (single-GPU) or `simpleDistributedGraphImpl` (multi-GPU/Dask). Key design choices:
- **Renumbering**: Vertices are internally renumbered to contiguous integers for GPU efficiency, with a `NumberMap` that translates back to original IDs in results
- **Symmetrization**: Undirected graphs are symmetrized by adding reverse edges
- **Multi-column edges**: Supports edge types, edge IDs, and multiple weight columns
- **Dask integration**: Multi-GPU graphs use Dask DataFrames (`dask_cudf`) with the `simpleDistributedGraphImpl`

The `dask/` directory contains Dask-aware wrappers for each algorithm that handle distributed execution using `dask.delayed` and `Comms` (NCCL-based communication).

## Dependencies

- **RAPIDS RAFT** (`raft`): Core CUDA utilities — handles, communicators, device spans, random number generation
- **RAPIDS RMM** (`rmm`): GPU memory management — device_uvector, memory pools, streams
- **Thrust**: CUDA parallel algorithms library (included with CUDA toolkit)
- **cuDF**: GPU DataFrames for Python input/output
- **Dask**: Distributed computing for multi-GPU Python
- **CuPy**: GPU arrays for Python-level data exchange
- **NCCL**: GPU-to-GPU communication for multi-GPU

## Algorithms

The algorithm suite covers:
- **Centrality**: Betweenness, Eigenvector, Katz, Degree, PageRank (personalized)
- **Community**: Louvain, Leiden, ECG (Ensemble Clustering for Graphs), Triangle Count, K-Truss, Spectral Clustering
- **Traversal**: BFS, SSSP (Dijkstra), Multi-Source BFS, path extraction
- **Link Analysis**: PageRank, HITS
- **Link Prediction**: Jaccard, Sorenson, Cosine, Overlap (all-pairs and vertex-pair)
- **Sampling**: Uniform/biased neighbor sampling, node2vec random walks, temporal sampling, heterogeneous sampling (for GNN training)
- **Components**: Weakly Connected, Strongly Connected (experimental)
- **Layout**: ForceAtlas2
- **Cores**: K-Core, Core Number
- **Tree**: Minimum Spanning Tree
- **Similarity**: Various coefficient computations
- **Linear Assignment**: Hungarian algorithm (LAP)
- **Graph Generators**: R-MAT

## Code Size

- C++ headers: 17,138 lines (133 files)
- C++ source: 33,489 lines (~200+ files)
- Python cugraph: 41,612 lines
- Python pylibcugraph (Cython): 16,857 lines
- Build system (CMake, CI): extensive

Total: ~110K lines of active source code across C++, Cython, and Python.

## Key Design Decisions

1. **Compile-time over runtime**: Template parameters for vertex/edge types, transposition, and multi-GPU are resolved at compile time via explicit instantiation. This eliminates virtual dispatch overhead but creates a combinatorial explosion of compilation units.

2. **CSR/CSC with degree segmentation**: Unlike simpler GPU graph libraries that use COO format, cuGraph builds a sophisticated multi-segment CSR/CSC with DCSR for hypersparse regions. This is critical for scale-free graphs.

3. **Primitive library over ad-hoc kernels**: Rather than writing per-algorithm CUDA kernels, cuGraph builds a library of reusable primitives. This is the same architectural choice as the GraphBLAS approach but with a different API design (Thrust-inspired rather than linear algebra).

4. **Owning/non-owning split**: `graph_t` owns memory, `graph_view_t` is a lightweight reference. This enables zero-copy algorithm composition and read-only sharing in MTMG.

5. **2D partitioning for multi-GPU**: Uses the 2D block-cyclic decomposition from the HPC literature rather than simpler 1D partitioning. More communication-efficient for algorithms that need both row and column access.

6. **Renumbering by default**: Python API always renumbers vertices to contiguous integers. This improves GPU memory locality and enables degree-based segmentation, at the cost of maintaining a translation map.

## Comparisons

- **vs. Gunrock**: Gunrock was another GPU graph library (UC Davis) that pioneered the advance/filter programming model. cuGraph has broader algorithm coverage, tighter RAPIDS ecosystem integration, and production support from NVIDIA. Gunrock's research contributions (direction-optimizing traversal, load-balancing strategies) influenced cuGraph's primitives.

- **vs. GraphBLAS**: GraphBLAS expresses graph algorithms as sparse linear algebra. cuGraph's primitives are similar in spirit (reusable building blocks) but use a Thrust-like functional API rather than a linear algebra API. GraphBLAS is more general but harder to optimize for specific graph patterns.

- **vs. DGL/PyG**: Deep Graph Library and PyTorch Geometric are GNN frameworks. cuGraph provides the graph sampling and feature aggregation primitives that GNN frameworks need, and can serve as a backend for them. cuGraph's sampling APIs are explicitly designed for GNN mini-batch training.

- **vs. NetworkX**: NetworkX is a pure-Python CPU library. cuGraph provides a NetworkX-compatible API (including the `nx-cugraph` backend) that transparently accelerates NetworkX code on GPU. cuGraph targets much larger graphs and faster execution but requires NVIDIA GPUs.
