# Just Brute Force Your Embeddings

Doug Turnbull's empirically-backed case that most teams building embedding search don't need a vector database yet — a single line of NumPy on a laptop handles millions of documents at sub-15ms latency. The sweet spot is ~1M docs, low query traffic, and up-front embedding writes; past that, reach for FAISS before reaching for infrastructure.

---

## Key Quotes

> "My O(n) algorithm can run circles around your O(log n) algorithm" — Raymond Chen

Turnbull uses Chen's quip as the thesis statement and then proves it. On an M4 MacBook Pro, a single `np.dot()` line serves 79.7 QPS against 1M vectors at 12ms average latency. That's fast enough for most internal tools and many production use cases. The constant factors matter more than the asymptotic complexity until you're well past the million-document mark.

> "An exhaustive search may be all you need" — Jo Kristian Bergum

The Vespa creator's quiet observation, quoted as corroboration. Bergum builds approximate nearest neighbor search for a living and still tells people to try brute force first. When the domain expert and the vendor both say "you probably don't need my product," pay attention.

> The code: `scores = self.doc_vectors @ query_vector.astype(np.float32, copy=False)`

This is the entire implementation. One line of NumPy. No index builds, no configuration, no operational complexity. Turnbull's benchmarks show it scaling to 8.8M documents at 9.3 QPS single-threaded — slow for a user-facing app but perfectly fine for batch processing or internal tooling. The takeaway isn't "NumPy beats vector databases," it's "measure before you architect."

---

## Key Themes

- **Brute force as the right default** — Before approximate nearest neighbors, before vector databases, before FAISS, try the dumb thing. For many workloads, the dumb thing is fast enough and the smart thing adds complexity without benefit. #pattern
- **The constant-factor gap** — Asymptotic complexity (O(log n) vs. O(n)) dominates CS education but constant factors dominate real systems. A well-optimized linear scan on a modern CPU can beat a complex index structure until data sizes get genuinely large. #concept
- **Hardware is fast now** — An M4 MacBook Pro pushing 170 QPS against 1M 384-dim vectors with 10 threads is a reminder that the hardware has gotten absurdly capable. The bottleneck is rarely compute; it's usually architecture. #pattern
- **Measure before you build** — Turnbull's methodology is the argument: benchmark the simplest possible approach on your actual hardware with your actual data before reaching for infrastructure. Most teams skip this step and pay for it in operational complexity. #pattern

---

## Critical Analysis

Turnbull is making the "SQLite is all you need" argument for vector search, and he's mostly right. The numbers are convincing: 1M documents at 12ms is real, and that covers a surprising fraction of embedding workloads. The teams that need Pinecone or Weaviate or pgvector with HNSW indexes are the ones who already know they need them — they have tens of millions of documents or hundreds of QPS. Everyone else is over-provisioning.

The benchmark methodology has some honest gaps. Single-query latency and throughput are measured, but there's no measurement of what happens when the document matrix doesn't fit in L3 cache (8.8M × 384 × 4 bytes = 13.5 GB — way past any MacBook cache). The 10-thread benchmark at that scale shows zero latency improvement over 1 thread (0.106s in both cases), which screams memory bandwidth saturation. Turnbull doesn't dig into this, though he does gesture at batching multiple queries per matrix scan as the fix.

The FAISS recommendation as the next step is the right instinct. FAISS gives you the same in-memory brute force with optional IVF/PQ acceleration, flat indexing as the default, and GPU support — but it's still "load vectors into memory and compute dot products," not "stand up a database cluster." The progression of brute force → FAISS → vector database is a ladder where each rung solves a specific scaling problem, and Turnbull's contribution is showing that the first rung is higher than most people think.

The post's real contribution isn't the benchmarks — it's the reframe. The industry default has become "you have embeddings, therefore you need a vector database." Turnbull shows that the inference doesn't hold for the median use case. Like [[SQLite Is All You Need]], the argument is that simple tools on modern hardware cover more ground than conventional wisdom admits, and the operational cost of complexity is real even when the licensing cost is zero.

The companion piece to this would be "Just Brute Force Your Full-Text Search" — the same argument applied to BM25 vs. Elasticsearch. The pattern generalizes: measure the simple thing first, and be surprised how often it's enough.

---

## Related Pages

- [[Databases and Data]] — The hub page's "local vs. distributed" tension and the zvec entry ("billions of vectors on one machine") is the same argument applied to vector databases
- [[zvec]] — Alibaba's in-process vector DB: the "embedded library, not a service" philosophy applied to vectors at larger scale
- [[LatticeDB]] — an embedded property-graph database whose HNSW index claims 0.83 ms at 1M vectors with 100% recall: the "reach for a real ANN index" rung above brute force, bundled with graph traversal and BM25 in one file rather than sold as a standalone vector store
- [[SQLite Is All You Need]] — The same argument for relational databases: modern hardware makes the "toy" option viable for production. Turnbull is doing for vector search what DB Pro did for SQLite
- [[Playing with Vision Embeddings]] — Jensen's deep dive into what embeddings actually encode; Turnbull's post is about what you do with them once you have them
- [[Learning a Few Things About Running SQLite]] — Julia Evans' field report on the gap between "X is fine" and actually operating X; the same gap exists for brute-force embedding search in production
- [[Your Distributed System Is Slower Than a Laptop]] — The COST paper's core argument, applied to vector search: single-machine brute force often beats distributed approximate systems on both cost and latency
- [[Grimmory]] — A self-hosted digital library shipping the thesis in production: book recommendations from hand-rolled 128-dim hashed feature vectors and brute-force cosine over the whole catalog, stored as JSON in a MariaDB column — no vector database, no ANN index

---

*Sources: [[raw/just-brute-force-embeddings]]*
*Last updated: 2026-08-01*
