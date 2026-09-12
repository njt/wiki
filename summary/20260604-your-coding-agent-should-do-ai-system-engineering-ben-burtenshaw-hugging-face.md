---
url: https://gist.github.com/njt/6f01cc4ac4835314dabf86b50c06947c
title: "Your Coding Agent Should Do AI System Engineering (Ben Burtenshaw, Hugging Face)"
author: njt (notes), Ben Burtenshaw (speaker, Hugging Face)
date_fetched: 2026-06-04
date_published: 2026-06-04
source: AI Engineer Conference talk, transcribed by njt
topics:
  - agent-coding-workflow
---

# Your Coding Agent Should Do AI System Engineering

Notes and transcript from Ben Burtenshaw's (Hugging Face) talk at the AI Engineer conference. The core thesis: coding agents can tackle the hardest AI engineering problems — CUDA kernel writing, zero-shot fine-tuning, and multi-agent auto-research labs — and standard repos on the Hub are the enabling infrastructure.

## Key Points (per njt's notes)

1. **Coding agents tackle hard AI engineering** — systems-level work like custom CUDA kernels, fine-tuning LLMs, and autonomous research loops. Framed as a way to stay contemporary and move "closer to the silicon."

2. **Standard repos on the Hub** — distributing kernels, training scripts, and experiment artifacts as versioned, discoverable repos enables agents to publish and makes workflows repeatable.

3. **Three "boss" levels of agent autonomy:**
   - *Interactive kernel writing* — agent-assisted CUDA kernel generation with benchmarking, yielding 94% speedup on Qwen2.5-0.8B for H100.
   - *Zero-shot fine-tuning* — single prompt triggers dataset selection, training, and model push to the Hub.
   - *Multi-agent auto-research lab* — team of agents (researcher, planner, workers, reporter) iterates on training scripts, runs experiments via HF Jobs, monitored via open dashboard.

4. **Open primitives beat abstracted APIs** — tools exposing their data layer let agents operate freely; hidden internals create a ceiling.

5. **Skills turn zero-shot into few-shot** — file-based context maintained alongside projects improves agent performance on specialized tasks like kernel writing.

## Quotes from the Talk

- "Your coding agent should do AI systems engineering."
- "Maybe the boring part is that in order to do this, we're gonna need standard repos…"
- "In most cases, memory is usually the bottleneck." (on GPU performance)
- "We keep the GPUs warm."
- "It takes a task from being zero shot to being few shot." (on skills)
- "Even though abstracted APIs are really useful… if we have a layer that we can't… get behind that is a ceiling."
- "If you think this was completely wrong, come and find me afterwards and bully me, that's fine."

## Tools and Practices Referenced

- **Hugging Face Kernels** — Hub repo type for distributing CUDA kernels with TOML-based compatibility specs
- **Skills (file-based context)** — markdown/script files with examples and workflows; maintained inside project repos
- **Upskill** — open-source library for generating skills, creating evals, comparing models on accuracy and token usage
- **HF CLI skills + Unsloth** — pre-built skills for fine-tuning with optimized model loading
- **Open Code** — agent configuration defining multi-agent setups with roles and templates
- **Tracheo** — open-source dashboard with parquet data layer for agent-driven metrics and custom visualizations
- **HF Jobs** — compute on the Hub for programmatic launch by agents
- **HF Papers** — CLI to search papers on the Hub; researcher agent uses it for literature scouting

## Multi-Agent Research Team Architecture

- **Researcher** — uses HF Papers CLI as literature scout, formulates hypotheses
- **Planner** — maintains experiment queue from hypotheses
- **Workers** — implement training script changes, launch via HF Jobs
- **Reporter** — monitors via Tracheo dashboard

Implemented in Open Code (also works with Codex and Claude Code). Uses git branches for experiments, a data structure on main for scores, and agent-to-agent communication via tables.

## Unanswered Questions (per njt)

- Reliability/safety of agent-generated kernels (no correctness verification beyond benchmarking)
- Real cost of auto-research lab (no cost controls or experiment pruning discussed)
- Human oversight requirements (no guidance on detecting rogue agents or failure modes)
- Skills maintenance and quality (no discussion of versioning or community contribution models)
- Risk of low-quality/malicious agent-generated artifacts on the Hub
- Portability beyond Hugging Face infrastructure
- Evaluation beyond speed metrics (accuracy, robustness not addressed)
- Agent coordination robustness when misinterpretation or queue logic fails

---

*Source: https://www.youtube.com/watch?v=JomVvNDjGb8 (original talk)*
*Transcript duration: 18m 25s*
