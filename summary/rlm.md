---
url: https://alexzhang13.github.io/blog/2025/rlm/
title: Recursive Language Models
author: Alex Zhang (MIT), with Omar Khattab
date: 2025-10
date_fetched: 2026-08-06
---

# Recursive Language Models — Summary

Alex Zhang introduces Recursive Language Models (RLMs), an inference strategy where language models recursively call themselves or other LLMs to process unbounded input context. Rather than cramming everything into a single context window, an RLM wraps an LM with a REPL environment (like a Jupyter notebook) pre-loaded with the context as a Python variable. The root LM can peek at, grep through, partition, and launch recursive sub-queries over the context — and only sees the query initially, not the full context.

The key results: **RLM(GPT-5-mini) outperforms GPT-5** on the hardest long-context benchmarks, doubling correct answers on OOLONG while being cheaper per query. On BrowseComp-Plus, RLM(GPT-5) is the only approach that maintains perfect performance at 1,000-document scale (10M+ tokens). The REPL environment lets the LM programmatically process context — writing regex, chunking, and spawning sub-calls — rather than relying on fixed retrieval pipelines or hoping a single forward pass handles everything.

RLMs are positioned as a third axis of inference-time scaling, after chain-of-thought reasoning and ReAct-style agents. Unlike agents (which decompose by *problem*), RLMs decompose by *context* and let the LM decide the strategy at test time. The paper is available at alphaxiv.org/abs/2512.24601, with code at github.com/alexzhang13/rlm.
