---
url: https://github.com/weaviate/elysia
title: Elysia
author: Weaviate
date_fetched: 2026-05-14
date_published: 2025-11-04
topics:
  - agent-architecture
---

# Elysia: Decision Tree Agentic Framework

Elysia is an agentic platform from Weaviate that organizes tool usage through a decision tree architecture. A "decision agent" selects tools dynamically based on environmental context. It supports custom tools as well as pre-built tools for interacting with Weaviate clusters.

## Installation

Requires Python 3.12. Install via `pip install elysia-ai`. Also available from GitHub for development.

## Quickstart

**App:** `elysia start` then open `localhost:8000`. Configure API keys, Weaviate cluster details, and models in the settings page.

**Python:**
```python
from elysia import tool, Tree

tree = Tree()

@tool(tree=tree)
async def add(x: int, y: int) -> int:
    return x + y

tree("What is the sum of 9009 and 6006?")
```

With Weaviate:
```python
import elysia
tree = elysia.Tree()
response, objects = tree(
    "What are the 10 most expensive items in the Ecommerce collection?",
    collection_names = ["Ecommerce"]
)
```

## Architecture

Backend: FastAPI. Core logic: pure Python with DSPy handling LLM interactions. Each tree node is orchestrated by a decision agent with global awareness of environment, available actions, and past/future steps.

Supports OpenAI, OpenRouter, and local models via Ollama.

## Stats

- Stars: 1.9k, Forks: 264
- Latest release: v0.2.8 (Nov 4, 2025)
- License: BSD-3-Clause
- Python 95.8%, HTML 4.2%
- 572 commits, 14 open issues
