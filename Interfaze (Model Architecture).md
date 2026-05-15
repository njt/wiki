# Interfaze (Model Architecture)

A hybrid model architecture that routes deterministic sub-tasks (OCR, STT, object detection, structured extraction) through specialized DNN/CNN encoders while using a shared transformer decoder for reasoning. The bet is that generalist LLMs are the wrong tool for tasks with one right answer — and that combining old-school DNN precision with transformer flexibility beats either approach alone.

---

## Key Quotes

> "You can activate parts of the model to run a specific task without using the full weights."

This is the architectural thesis in one sentence. Use a `<task>ocr</task>` tag in the system prompt and the model routes through only the relevant subnetwork. Compare with [[Smart Models Dumb Pipes]]: this isn't just a design philosophy, it's a runtime optimization. The model literally doesn't fire neurons irrelevant to the task.

> "Interfaze isn't designed to do well on MMMLU — we're built for deterministic tasks like OCR, object detection, and STT."

**yoeven** (Interfaze rep) responding to accusations of benchmark gaming on HN. The defensiveness is telling: they benchmarked on MMMLU because the market expects it, but it's measuring the wrong thing. This is the same dynamic [[Benchmark Exploitation]] documents — benchmarks become a compulsory ritual disconnected from the thing they're supposed to measure. The irony is that Interfaze's own MMMLU score (90.9%) probably hurts their pitch more than helps it, because it invites exactly this "you're playing the wrong game" criticism.

> "It's worse than whisper."

**icemaze** on HN, after testing STT. The bluntest negative signal in the thread. Interfaze's response ("use run task mode for one-to-one comparison") suggests the out-of-box experience doesn't match the benchmark claims. This is the gap between a model's best-case configuration and what a developer hits on first try — and it's where most "launch day" model evaluations fall apart.

> "Can models be chained like UNIX command-line programs?"

**euroderf** on HN, asking the most interesting unanswered question. The `<task>` tag routing mechanism implies composability — OCR output feeding into structured extraction feeding into translation — but the API is monolithic. UNIX pipes work because each program does one thing and the shell composes them. Interfaze's internal routing does the opposite: one model, many tasks, dispatched internally. The UNIX-pipe model would be multiple specialized models with clean interfaces between them.

> "20-25 seconds for 5 fields — unusable at scale."

**nickserv** on HN, after testing structured data extraction from images. This is the production reality check. Benchmarks don't measure latency, and latency kills adoption faster than accuracy gaps. A model that's 15% more accurate but 10x slower loses in production every time — unless the accuracy difference is existential (medical imaging, legal document review).

---

## Key Themes

#model-architecture #ocr #stt #deterministic-compute #structured-output #benchmark

- **DNN precision + transformer flexibility** — The hybrid architecture is the right bet for deterministic tasks. Pure transformers hallucinate bounding boxes; pure CNNs can't handle varied layouts. The combination means the DNN handles what it's trained for and the transformer cleans up the edges.
- **Partial model activation** — The `<task>` tag routing is genuinely novel. Most multi-modal models activate the whole model regardless of task. Interfaze's approach means OCR requests don't pay the compute cost of the STT subnetwork. This is efficiency through architecture, not quantization.
- **The specialist-generalist tension** — Interfaze explicitly positions as a complement to generalist LLMs, not a replacement. This is honest and strategically smart, but it also limits the addressable market. Nobody got rich selling "the other model you also need."
- **Launch-day skepticism pattern** — The HN thread follows a recognizable template: enthusiastic early testers, suspicious new accounts, latency complaints, benchmark gaming accusations. Every model launch hits these. What's unusual is the quality of the technical pushback — the STT and latency testers brought real data, not hot takes.

---

## Critical Analysis

**What's interesting:** The architecture is genuinely novel. Most "new model" announcements are scale plays (more parameters, more data, more GPUs). Interfaze is making a different bet: that the right architecture for deterministic tasks looks more like a router than a monolith. The partial-model-activation pattern is under-explored and probably correct — we don't need the whole transformer for every task.

**What's suspicious:** The benchmark table leads in 8/9 categories against Gemini-3-Flash, Claude-Sonnet-4.6, GPT-5.4-Mini, and Grok-4.3. That's either a genuinely breakthrough architecture or cherry-picked numbers. The HN thread doesn't resolve this — the one person who ran a rigorous test (nickserv) found the speed unusable, and the one person who got great OCR results (schanz) posted from a day-old account. The truth is probably in the middle: genuinely better at OCR, genuinely worse at latency, benchmarks that overstate the gap.

**The cost problem:** $1.50/M input + $3.50/M output puts it in Gemini-3-Flash pricing territory. But as [[Computer Use is 45x More Expensive Than Structured APIs]] documents, the real competition for deterministic extraction isn't other LLMs — it's structured APIs that cost orders of magnitude less. A dedicated OCR API will beat Interfaze on both speed and cost for pure OCR. The value proposition hinges on tasks that genuinely need both DNN precision AND transformer reasoning, and it's not clear how large that overlap is.

**The composability question matters:** euroderf's UNIX pipe question is the most important thing in the thread because it exposes the real design choice. You can either build one model that does everything (Interfaze's approach) or compose many specialized models (the UNIX approach). The UNIX approach is more flexible, easier to debug, and lets you swap components independently — but requires clean interfaces between models that don't currently exist. Interfaze's monolithic approach works today but locks you into their routing decisions.

**Compared to existing tools:** For OCR, Interfaze is competing with [[Dolphin]] (ByteDance's universal parser, open-source, production-deployed) and the extraction backends in [[Kreuzberg]] (97+ formats, MCP server). For STT, it's competing with Whisper (free, local, proven) and [[Doing]]/[[Handy]] (local-first, one-time cost). For structured extraction, it's competing with [[OpenAI Structured Outputs]] (protocol-level enforcement, SDK-native). Interfaze's pitch is "one model for all of these" — but the people who care most about accuracy usually want the best tool per task, not a Swiss Army knife.

**Bottom line:** Worth watching for OCR specifically — the HN tester's results were genuinely impressive, astroturfing suspicions aside. The architecture is interesting even if the product isn't there yet. But the launch-day gap between benchmarks and real-world latency is a red flag, and the "specialist that also wants to be compared on generalist benchmarks" positioning muddies the pitch.

---

## Cross-Links

- [[Dolphin]] — ByteDance's document parsing model, the open-source OCR competitor
- [[Kreuzberg]] — multi-backend document extraction; competes on format coverage vs. accuracy
- [[OpenAI Structured Outputs]] — the protocol-level alternative for structured extraction
- [[Smart Models Dumb Pipes]] — architectural parallel: route judgment to specialized components
- [[Benchmark Exploitation]] — why the MMLU criticism lands; benchmarks as compulsory ritual
- [[Computer Use is 45x More Expensive Than Structured APIs]] — the cost economics Interfaze is fighting against
- [[Doing]] / [[Handy]] — local STT alternatives that cost less and run on-device
- [[Building Production-Ready Voice Agents]] — STT in production context
- [[On a Year of Multi-Model Development]] — the multi-model workflow Interfaze would slot into
- [[Harness Engineering]] — "deterministic tasks" as the feedforward computational quadrant
- [[A Non-Anthropomorphized View of LLMs]] — models as mathematical functions; Interfaze makes this literal with DNN routing
- [[Optimise Anything]] — if quality is measurable, optimize it; Interfaze is optimizing for accuracy on measurable tasks
- [[Pocket TTS]] — the audio generation side of the same coin
- [[SAM Audio]] — another model using specialized architecture for a specific modality (audio separation)

---
*Sources: [[raw/interfaze]], https://interfaze.ai/blog/interfaze-a-new-model-architecture-built-for-high-accuracy-at-scale, https://news.ycombinator.com/item?id=48097078*
*Last updated: 2026-05-15*
