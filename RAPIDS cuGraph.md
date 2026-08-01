# cuGraph — GPU-Accelerated Graph Analytics

NVIDIA's library for running graph algorithms at GPU scale — PageRank, BFS, community detection, GNN sampling, and more — against graphs with billions of edges, in seconds rather than hours. Part of the RAPIDS data science ecosystem (cuDF, cuML), with a NetworkX-compatible Python API and a C++ core that compiles every algorithm four ways (32/64-bit × single/multi-GPU) for zero-overhead dispatch.

---

## Architecture

cuGraph is a four-layer stack, each layer adding a higher level of abstraction:

**1. C++/CUDA Core** (`cpp/` — ~33K lines source, 133 header files)

The engine. The graph is stored as CSR/CSC adjacency lists with a hybrid DCSR (Doubly Compressed Sparse Row) extension for scale-free graphs. Key types:

- **`graph_t<vertex_t, edge_t, store_transposed, multi_gpu>`** — Owning graph owning `rmm::device_uvector` arrays
- **`graph_view_t<...>`** — Non-owning view wrapping `raft::device_span`, what algorithms consume
- **`partition_t<vertex_t>`** — 2D edge partitioning mapping P GPUs arranged as `major_comm × minor_comm` to rectangular edge-matrix sub-blocks (based on Boman et al., 2013)

The `store_transposed` template parameter is baked in at compile time: `false` = CSR (source-major, push), `true` = CSC (destination-major, pull). Algorithms pick the model they need — PageRank uses pull, BFS uses direction-optimizing push/pull switching.

**2. C API** (`cpp/src/c_api/`)

Type-erased C wrappers around the C++ core for FFI. Uses an `abstract_functor` pattern: each algorithm defines a functor struct storing all parameters, then the C function constructs it, calls `operator()`, and extracts results. Type erasure is done via `reinterpret_cast` from opaque C handles — safe because the C API owns the objects.

**3. Cython Bindings** (`python/pylibcugraph/` — ~17K lines)

Thin Python wrappers. Each `.pyx` file wraps one algorithm: accept Python objects → convert to C types → call C API → check errors → return CuPy arrays. This is the low-level Python API for users who want direct control.

**4. Pure Python API** (`python/cugraph/cugraph/` — ~42K lines)

The NetworkX-compatible user-facing API. `cugraph.Graph()` wraps either `simpleGraphImpl` (single-GPU) or `simpleDistributedGraphImpl` (Dask multi-GPU). All vertices are internally renumbered to contiguous integers for GPU efficiency, with a `NumberMap` to translate back. The `dask/` subdirectory contains Dask-aware wrappers using `dask.delayed` and NCCL-based `Comms`.

Additional packages: **libcugraph** (C library with Python loader) and **nx-cugraph** (NetworkX backend plugin — transparently accelerates `networkx` code on GPU).

## Key Techniques

### CSR/CSC with degree-based segmentation

The adjacency is sorted by vertex degree into segments:
- **High degree** (>1024): many threads per vertex
- **Medium degree** (warp_size to 1024): warp-level parallelism
- **Low degree** (<warp_size): processed in aggregate
- **Hypersparse** (below threshold): stored in DCSR/DCSC — zero-degree vertices are excluded from the adjacency entirely, critical for scale-free graphs where most vertices have very few edges

The threshold constants live at `graph_view.hpp:242-253`. `use_dcs()` at `graph_view.hpp:551-557` checks whether the hypersparse segment exists.

### ~30 reusable GPU primitives

Algorithms aren't written as per-algorithm CUDA kernels. They compose from a library of ~30 graph-specific GPU primitives (`cpp/include/cugraph/prims/`), Thrust-inspired but graph-native:

- **`per_v_transform_reduce_incoming_outgoing_e`**: For each vertex, iterate incident edges, apply a quinary operator (src, dst, src_val, dst_val, edge_val), reduce. The workhorse — BFS, PageRank, SSSP all use it.
- **`vertex_frontier`**: Active vertex frontier with key-bucket abstraction for push/pull traversal and direction optimization.
- **`transform_reduce_e` / `transform_reduce_v`**: Edge and vertex reductions.
- **`count_if_e` / `count_if_v`**: Predicate counting.
- **`edge_bucket`**: Group edges by destination for aggregation.
- **`extract_transform_if_e`**: Filter-and-transform on edges.

The critical insight: **these primitives fuse vertex iteration, edge property lookup, computation, and reduction into single kernel launches**. A CPU graph framework makes O(E) random memory accesses; cuGraph makes O(V/block_size) coalesced reads.

### Four-way compile-time instantiation

Every algorithm exists in four explicit template specializations:

```
algorithm_sg_v32_e32.cu  — single-GPU, int32 vertex, int32 edge
algorithm_sg_v64_e64.cu  — single-GPU, int64 vertex, int64 edge
algorithm_mg_v32_e32.cu  — multi-GPU,  int32 vertex, int32 edge
algorithm_mg_v64_e64.cu  — multi-GPU,  int64 vertex, int64 edge
```

The `.cuh` header (`pagerank_impl.cuh`) holds the template. The `.cu` files instantiate concretely. Zero virtual dispatch, type-specific GPU codegen, at the cost of combinatorial build explosion.

### Direction-optimizing BFS

The BFS implementation (`bfs_impl.cuh`) maintains both top-down (push from frontier) and bottom-up (pull to unvisited) paths, switching based on frontier size. Uses a `visited_bitmap` for O(1) membership tests and approximate out-degree tracking to decide when to switch. This is the technique from Beamer et al. (2012) adapted to GPU with the primitives layer.

### MTMG — multi-threaded multi-GPU sharing

The `mtmg/` layer (`cpp/include/cugraph/mtmg/`) provides per-thread GPU access with shared immutable graph views. Multiple CPU threads can run algorithms concurrently on the same GPU-resident graph data, each with isolated CUDA streams from a pool. `graph_view_t` is copy-constructible between threads; results go to thread-local `vertex_pair_result_t` containers.

### PageRank implementation

Uses the power method in pull model (`store_transposed=true`). Each iteration calls `per_v_transform_reduce_incoming_e` with an edge operator computing `src_rank / out_degree`, then applies the damping factor and personalization vector, converging on L1 norm difference.

## Design Decisions

**Compile-time over runtime**: Template parameters for vertex/edge types, transposition, and multi-GPU are resolved at compile time via explicit instantiation. This eliminates virtual dispatch but creates a combinatorial explosion of `.cu` compilation units — worthwhile because GPU kernel performance is sensitive to integer width and memory access patterns.

**Primitive library over ad-hoc kernels**: Rather than per-algorithm CUDA kernels, cuGraph builds reusability into the primitives layer. This is the GraphBLAS insight (reusable graph building blocks) but with a Thrust-like functional API instead of linear algebra. The trade-off: easier to write correct algorithms, harder to squeeze out the last 10% of performance vs. a hand-tuned kernel.

**Renumbering by default**: The Python API always renumbers vertices to contiguous integers. This improves GPU memory locality and enables degree-based segmentation but requires maintaining a translation map. The cost is paid once at graph construction and amortized across all subsequent algorithm calls.

**2D partitioning for multi-GPU**: Uses 2D block-cyclic decomposition from the HPC literature rather than simpler 1D partitioning. More communication-efficient for algorithms needing both row and column access. Implementation in `partition_t` (graph_view.hpp) maps each GPU to `minor_comm_size` rectangular edge partitions.

**Owning/non-owning split**: `graph_t` owns memory, `graph_view_t` is a lightweight reference. This enables zero-copy algorithm composition and read-only sharing in MTMG without reference counting overhead.

**GPU memory optimization**: DCSC/DCSR for hypersparse vertices, key-value pair storage when unique edge sources/destinations are sparse (threshold: 10% fill ratio, `graph_view.hpp:242`), and `edge_mask` support for filtering without copying.

## Comparison Notes

- **vs. Gunrock**: Gunrock (UC Davis) pioneered the advance/filter GPU graph programming model. cuGraph has broader algorithm coverage, tighter RAPIDS ecosystem integration, and NVIDIA production support. Gunrock's research contributions (direction-optimizing traversal, load-balancing strategies) influenced cuGraph's primitives.

- **vs. GraphBLAS**: GraphBLAS expresses graph algorithms as sparse linear algebra over semirings. cuGraph's primitives serve the same role (reusable building blocks) but use a functional Thrust-like API rather than a linear algebra API. GraphBLAS is more general; cuGraph's narrower API enables tighter GPU optimization for common graph patterns.

- **vs. DGL / PyTorch Geometric**: DGL and PyG are GNN frameworks focused on deep learning over graphs. cuGraph provides the graph sampling and feature aggregation primitives that GNN frameworks need, and its heterogeneous/temporal sampling APIs are explicitly designed for GNN mini-batch training.

- **vs. NetworkX**: NetworkX is a pure-Python CPU library limited to graphs that fit in memory. cuGraph provides a NetworkX-compatible API and the `nx-cugraph` backend that transparently accelerates NetworkX code on GPU, handling graphs orders of magnitude larger at dramatically higher speed — but requires NVIDIA GPUs.

- **vs. Apache Spark GraphX**: GraphX runs on distributed CPU clusters; cuGraph runs on GPU(s) in a single node or multi-GPU node. The GPU approach achieves higher throughput per node but is limited by GPU memory. Both support the DataFrame/edge-list programming model.

---

*Sources: [[raw/rapidsai-cugraph]]*
*Last updated: 2026-08-01*
