---
title: "AI Engineering for Developers"
author: "Luca Cavallin"
url: "https://www.lucavallin.com/blog/ai-engineering-for-developers"
date: 2026-06-02
ingested: 2026-06-05
tags: [ai-engineering, foundation-models, agents, rag, finetuning, inference, evaluation, prompting]
source_domain: lucavallin.com
topics:
  - misc
---

# AI Engineering for Developers

**Author:** Luca Cavallin
**Published:** 2 June 2026
**Tags:** ai-engineering, foundation-models, agents

## Core Thesis

Cavallin writes as a veteran backend engineer who found that "LLMs landed in production and a lot of the rules I trusted stopped applying." The system is "non-deterministic by default," input is natural language, and unit tests can't verify output quality. He aims to guide engineers who "already know how to ship software" through the AI engineering landscape.

**Key opening quote:** "The model is no longer the product. The product is the system around the model."

## The Three-Layer Stack

1. **Application layer** — prompts, RAG, agents, UI, business logic
2. **Model development layer** — finetuning, distillation, dataset engineering (optional for most teams)
3. **Infrastructure layer** — GPUs, inference servers, vector databases, gateways, observability

Most teams live in layer 1, occasionally dip into layer 2, and rent layer 3 from a cloud. "The art is knowing when you actually need to go down a layer."

## Three Knobs for Adapting an LLM (in cost order)

1. **Prompt engineering** — change the input, model unchanged
2. **RAG** — give new context at runtime via retrieval
3. **Finetuning** — change model weights

Default: "prompt first, then RAG, then finetune. Do not skip steps."

## Choosing an LLM (2026 Landscape)

Five buckets: Closed frontier (GPT-5.5, Claude Opus 4.7, Gemini 3.1 Pro), closed mid-tier (Claude Sonnet 4.6, Gemini 2.5 Pro), closed cheap (Claude Haiku 4.5, Gemini Flash-Lite), open weights (Llama 4, Gemma, Mistral, DeepSeek, Qwen), and specialized (Voyage embeddings, Cohere rerankers). Pick by task fit, cost, latency, context window, and output structure support. "Almost no one should be using just one."

## Planning AI Applications

AI features sit on a quality gradient. Cavallin's four checkpoints: use case evaluation (is the problem real and tolerant of probabilistic output?), setting expectations (at least 2x dev time, mostly eval and edge cases), milestone planning (barely works → eval pipeline → hardening), and maintenance (model drift, prompt rot, data changes).

The "barely works" milestone: "Ship it to real users, watch what breaks, then fix. Trying to perfect an AI feature in isolation... is how teams spend three months and ship nothing."

## Understanding Foundation Models

### Model Architecture
Almost everything is a decoder-only transformer, sometimes with mixture-of-experts. Size still matters but "is no longer destiny." Reasoning models shifted the axis from parameter count to test-time compute.

### Model Taxonomy
- **SLMs** (Gemma 3, Phi-4, Llama 3.1 8B) — run on single GPU
- **Multimodal** — text, images, video, audio as first-class inputs
- **Domain-specific** — worth it only after evaluation shows a general model fails consistently
- **Reasoning models** — emit internal chains of thought; better at math, code, planning; slower and pricier

Cavallin's production setup: an SLM for routing, a mid-tier model for most tasks, a frontier/reasoning model for hard cases. "Running everything through the most expensive model is like using a rack of H100s to serve a CRUD API."

### Post-Training: SFT and Preference Finetuning
- **SFT:** Trains on (prompt, ideal response) pairs so the model learns to follow instructions
- **Preference finetuning:** RLHF, DPO, ORPO, GRPO — trains on (prompt, chosen, rejected) triples

SFT failures: model ignores instructions or drifts back toward completion behavior. Preference finetuning failures: verbose, sycophantic, or subtly wrong outputs reflecting labeler biases.

### Sampling Strategies
- **Temperature:** Low (0–0.3) deterministic; high (0.8+) creative
- **Top-k, Top-p, Min-p** — min-p adapts to model confidence, often outperforms top-p
- For code generation: temperature 0.0–0.1; summarization/analysis: 0.2–0.5; creative: 0.7–1.0

### Structured Outputs
Rather than asking the model to "return JSON" via prompt, use the API's structured output mode. "They eliminate the entire class of 'the model added a comment before the JSON' bugs."

### The Probabilistic Nature
The model is "a probability distribution, not a function." Build for it with idempotency where state mutation is involved, eval datasets on every model change, and logging that captures inputs, outputs, model version, and seed.

## Prompting and Prompt Engineering

### Prompt Templates
Don't concatenate strings. Templates separate static instructions from dynamic input, enable versioning, and protect against injection. "Structure beats hope."

### In-Context Learning
Few-shot (3–8 examples) is "dramatically more reliable than zero-shot for anything with a non-obvious format." Choose examples covering edge cases and error modes.

### System vs User Prompts
"Put everything stable in the system prompt: persona, format rules, examples, tool definitions, any context that doesn't change per request." Claude gives strong weight to system instructions, GPT reliable, Gemini sometimes treats instructions as suggestions.

### Context Length
Frontier models now offer 1M-token windows, but bigger isn't always better. "Lost in the middle" is real. Use context efficiently — retrieve only what's needed, put critical instructions at start and end.

### Organizing Prompts
"Treat prompts like SQL: they live in your repo, in dedicated files, versioned in git, with a CI step that runs them against an eval set." Putting prompts in a database for hot updates is a "common antipattern that turns into a debugging nightmare."

### Defensive Prompt Engineering
"There is no purely-prompt-based defense against prompt injection. Architectural defenses... are the only real protection."

## Evaluation

### The Core Challenge
"Traditional ML evaluation has a ground truth. AI engineering often does not." Use a mix of programmatic metrics, LLM-as-a-judge, human review on samples, and A/B tests in production. "If you skip eval, you ship regressions."

### Minimum Viable Eval Pipeline
1. Dataset of 50–500 representative inputs
2. Ground truth or rubric for each
3. Function running the system end-to-end
4. Scorer (programmatic or LLM-as-judge)
5. CI integration

"The hard part is the dataset."

### Evaluation Criteria (Priority Order)
1. Domain capability
2. Generation (correct, fluent, well-formatted)
3. Instruction-following ("consistently underweighted in model selection")
4. Cost and latency

### Model Selection
"Benchmarks lie. Models are trained on benchmarks." Build vs buy: for foundation models, almost always buy; for eval pipelines, build; for finetuned variants, buy first.

## Retrieval-Augmented Generation (RAG)

### Core Pattern
"It is not magic. It is a search engine bolted onto a generator. Most RAG bugs are search bugs."

### Semantic vs Lexical Search
Best practice: hybrid search with rerankers. Their failure modes are complementary — lexical fails on different vocabulary, semantic fails on exact terminology.

### Core RAG Implementation (No Framework)
Three functions: `embed`, `retrieve`, `answer`. "Everything else is optimization on top of these three functions."

### Chatbot Memory Patterns
Buffer (last N messages), summary (older turns collapsed), hybrid (last N turns + summary of older context). Hybrid is what most production chatbots use.

## Advanced RAG

### Retrieval Optimization Path
Hybrid search first → reranker second (retrieve 50, rerank, pass top 5) → query expansion → metadata filtering.

### Splitting Strategies
Recursive character splitting at 512 tokens with 50–100 token overlap is the benchmark-validated default (69% accuracy vs 54% for semantic chunking). "'Context cliff' reduces quality beyond ~2.5k tokens."

### Embedding Strategies
Parent/child chunks (embed small, return large parent) is "usually the right default." Also: document summaries, hypothetical questions.

### Agentic RAG
Static RAG = single retrieve-then-answer. Agentic RAG = agent that decides when/what to retrieve and whether to retrieve again. "Use agentic RAG when your users ask multi-hop or open-ended questions."

## Finetuning

### Finetune vs RAG
"If the issue is 'the model doesn't know', RAG. If the issue is 'the model knows but won't say it the way I need', finetune. If the issue is both, both."

### Reasons To/Not To Finetune
**To:** stable narrow task with high volume, prompt engineering plateaued, can produce hundreds to thousands of labeled examples, have an eval pipeline.
**Not to:** task isn't stable, no data, no eval pipeline, haven't exhausted prompts and RAG. "The 'you don't have eval' condition is usually the most common blocking reason."

### Memory Bottlenecks
Full finetuning a 7B model requires 100–120 GB VRAM — "roughly $50,000 worth of H100 GPUs for a single training run."

### Parameter-Efficient Finetuning
QLoRA is the default for finetuning 7B–70B models on accessible hardware (fits a single 24GB GPU).

### Model Merging
Combine multiple finetuned models via TIES, DARE, SLERP — arithmetic on weight tensors, not training. "Model merging is underused partly because it sounds risky."

## Dataset Engineering

### Five Axes of Data Quality
Quality, coverage, quantity (hundreds for narrow style transfer to tens of thousands for serious specialization), acquisition, annotation. "A small high-quality dataset beats a large noisy one."

### Data Processing Steps
Inspect manually ("A 10-minute review of 100 random examples... will find quality problems... that no automated metric catches"), deduplicate (exact and near-duplicate with MinHash), clean (strip PII, normalize), filter (drop low-quality/short), format (apply chat template, verify with tokenizer). "Skipping any of these steps guarantees a worse model."

## Inference Optimization

### Two Phases
- **Prefill:** process input prompt — compute-bound, throughput dominates
- **Decode:** generate tokens one at a time — memory-bound, latency dominates

### Bottleneck Patterns
Waiting accelerator (GPU idle), memory wall (waiting on HBM), maxed-out but slow (increase batch size or switch model), more GPUs equals worse (communication overhead). "Benchmark before you scale horizontally, especially at low traffic."

### Model Optimization Techniques
Quantization (INT8 mostly free, INT4 small quality cost), distillation, pruning, speculative decoding (small model proposes, big model verifies — "2X-3X acceleration"), multi-token prediction. "For most teams: start with vLLM."

## AI Agents

### Agent Architecture
An agent is a loop: model decides → takes action → observes result → decides again. What makes agents different: "the loop with a variable exit condition." The model decides whether to keep going, not your code — both the source of flexibility and the source of most bugs.

### Workflows vs Agents
Workflow: predefined sequence, LLM is one node, predictable, debuggable, cheap. Agent: LLM decides the next step at runtime, flexible, opaque, expensive. **"Default to workflow."**

### Agent Failure Modes
Infinite loops (cap iterations), context overflow (truncate or summarize), tool call errors (validate inputs, return helpful errors), hallucinated tools (constrain via API), ignoring the user (keep user goal in state). "An agent that can loop forever will eventually loop forever, and it will do it on the worst possible user request at the worst possible time."

### Agent State
"A well-defined state schema with a clear mental model of which nodes own which fields is the difference between a graph you can debug and one you can't."

### Memory Types
Short-term (current conversation, in context window), long-term (user facts across sessions — needs a store and retrieval policy), semantic (searchable knowledge via embeddings — the RAG pattern applied to agent knowledge).

## Multi-Agent Systems and Agent Protocols

### The Monolithic Agent Bottleneck
A single agent with 30 tools, 3 personas, and 50k tokens of system prompt fails via conflicting instructions, tool selection paralysis, and token limits. "Decomposing into specialists each with focused instructions and 3 to 5 tools fixes most of this."

### Router-Based Architectures
A small cheap model classifies input and dispatches to specialists. "This is the cost-control pattern that everyone reaches for once their agent bill scares them."

### Supervisor Patterns
"The difference: A router dispatches and forgets. A supervisor maintains a shared understanding of the overall task."

### MCP and A2A
**MCP** (Anthropic, Nov 2024, now Linux Foundation): standardizes model-to-tool interaction via JSON-RPC 2.0. By Q2 2026, MCP servers exist for GitHub, Slack, PostgreSQL, Stripe, Figma, Docker, Kubernetes, and 200+ other tools.

**A2A** (Google, April 2025, now Linux Foundation): standardizes agent-to-agent collaboration via Agent Cards, task lifecycles, and context transfer.

"MCP lets your agent use a database. A2A lets your hiring agent delegate to a sourcing agent owned by a different team or vendor."

Cavallin predicts: "Your AI architecture in two years looks like a service mesh of agents."

## Key Quotes

- "The model is no longer the product. The product is the system around the model."
- "prompt first, then RAG, then finetune. Do not skip steps."
- "Structure beats hope."
- "There is no purely-prompt-based defense against prompt injection. Architectural defenses... are the only real protection."
- "If you skip eval, you ship regressions."
- "Benchmarks lie. Models are trained on benchmarks."
- "It is not magic. It is a search engine bolted onto a generator. Most RAG bugs are search bugs."
- "Default to workflow."
- "An agent that can loop forever will eventually loop forever, and it will do it on the worst possible user request at the worst possible time."
- "Your AI architecture in two years looks like a service mesh of agents."
