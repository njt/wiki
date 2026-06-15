# Indexing 669 GB of GoPro Videos with Local ML

Ilias Haddad built an open-source, local-first pipeline to make 628 GoPro cycling videos (669 GB, 15+ hours) semantically searchable — all running on an M1 Max with no cloud dependencies. The resulting project, **edit-mind** (1.6k GitHub stars), combines Whisper transcription, YOLO object detection, DeepFace facial recognition, OCR, and Qwen2.5-VL scene captioning into a five-stage pipeline backed by ChromaDB vector search and PostgreSQL. Total compute: 67 hours. The result: natural-language search over personal video that sends clips straight to DaVinci Resolve.

---

## Key Quotes

> "The Docker version couldn't access the M1 Max GPU to utilize its power."

The MPS (Metal Performance Shaders) backend fails inside containers, forcing CPU fallback. Haddad had to build a native macOS desktop app to get GPU acceleration. This is the local-ML tax: the convenience of containerization breaks at the GPU boundary on Apple Silicon.

> Running this project over an NVIDIA GPU like RTX 3060 … I was able to get faster results than running it over my M1 Max.

But the M1 Max has a countervailing advantage: unified memory lets it hold all four models (Whisper ~3GB, YOLOv8x ~250MB, DeepFace ~1GB, Qwen2.5-VL-7B ~14GB) simultaneously — something the 12GB RTX 3060 physically can't do. It's the fewer-GPUs-larger-memory tradeoff vs. dense-compute. The M1 Max wins on _capability bandwidth_ (what you can do in one pass); the RTX wins on _throughput_ (how fast each pass runs).

> Whisper transcription: 25h 13m (37.3% of total). Frame analysis: 24h 56m (36.8% of total).

The two most expensive stages consume nearly three-quarters of the pipeline. Everything else — embedding generation, scene structuring, text indexing — is rounding error by comparison. The lesson: if you're building something like this, throw your optimization effort at transcription and frame analysis. Embedding is cheap.

---

## Key Themes

- **#tool** — edit-mind: open-source local video indexing with multimodal AI analysis
- **#pattern** — Local-first ML pipeline: the five-stage extract→classify→caption→embed→search architecture generalizes beyond video
- **#concept** — Multimodal semantic search: text, visual, and audio embeddings as parallel retrieval paths over the same content
- **#tool** — Qwen2.5-VL-7B-Instruct: vision-language model used as a scene captioner, with multi-frame context (5 frames at 720p) for action understanding
- **#pattern** — GPU-in-container gap on Apple Silicon: MPS backend failure forces native desktop builds. The containerization promise breaks at the ML boundary on Macs
- **#concept** — Unified memory as ML superpower: the M1 Max's architectural advantage isn't raw FLOPS, it's the ability to keep all models resident simultaneously

---

## Critical Analysis

**The real insight isn't the tech — it's the category.** Haddad is solving a problem that millions of people have (mountains of unsearchable personal video) with tools that didn't exist five years ago (local Whisper, local VLMs, local vector DBs). The fact that a single developer on a laptop can build a functional video search engine is genuinely new. This was cloud-only territory until very recently.

**The pipeline is a commodity; the integration is the product.** None of the individual components (Whisper, YOLO, DeepFace, ChromaDB) are novel. The value is in wiring them together into a coherent local-first experience — and that's exactly what edit-mind does. This is the "dark factory" pattern applied to personal media: the pipeline DOT file is the valuable artifact, not any single model.

**The Docker-on-Mac GPU gap is a real constraint on local ML tools.** Haddad isn't the first to hit this and won't be the last. Until Apple Silicon GPU access from containers is solved (podman/runkit/vllm-metal is promising but immature), local-first ML tools face an ugly choice: ship a desktop app or accept CPU-only performance. The container-native path is simply not ready.

**67 hours of compute for 15 hours of video is simultaneously impressive and unusable.** It's impressive that it works at all on consumer hardware. But 0.22× realtime means you can't index as you shoot — this is a batch overnight/weekend job. For the hobbyist with an existing library, that's fine. For anyone hoping to make this part of a daily workflow, the economics don't close yet. Hardware will fix this (M4/M5, better GPU access), but we're not there.

**The unresolved question that matters most: incremental indexing.** Haddad's blog doesn't clarify whether adding a new video reuses existing embeddings or triggers a full re-index. For a tool positioned as a "video knowledge base," this is the difference between a one-off experiment and a living system. If every new GoPro ride requires re-processing the entire library, the tool breaks at scale no matter how fast the hardware gets.

**DaVinci Resolve integration is the killer feature nobody talks about.** The ability to search semantically and send clips directly to an NLE timeline transforms this from a curiosity into a tool. Most video indexing projects stop at "here are your search results." edit-mind closes the loop: find → edit. That's the difference between a demo and a workflow.

---

## Related Pages

- [[Local and Open Source Inference]] — Hub page for running models on your own hardware
- [[LocalAI]] — Open-source local AI stack with composable gRPC architecture
- [[Datacenter GPU in a Gaming PC]] — The other end of the local-ML hardware spectrum: £200 V100 in a gaming rig
- [[Playing with Vision Embeddings]] — What vision embeddings actually encode; relevant to understanding edit-mind's visual embedding layer
- [[QMD]] — Local CLI search with hybrid BM25+vector+LLM re-ranking; same "search your own stuff" philosophy, different domain
- [[Understand-Anything]] — Knowledge graphs from codebases; same "index local content with AI" pattern
- [[docmason]] — Local knowledge base from office documents; parallel approach for documents instead of video
- [[graphify]] — Codebase-to-knowledge-graph; the code-domain analog of video-to-knowledge-graph
- [[Process Flow]] — Choreography-as-a-service; relevant if edit-mind's pipeline were decomposed into discrete stages
- [[Smart Models Dumb Pipes]] — The pipeline is the dumb pipe; the models are the smart components

---

*Sources: [[raw/iliashaddad-gopro-video-indexing.md]]*
*Last updated: 2026-06-15*
