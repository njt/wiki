---
url: https://tinysuite.dev/blog/how-to-add-memory-to-ai-agent/
date_fetched: 2026-11-02
---

Engineering

# How to Add Long-Term Memory to an AI Agent (Step by Step)

**To add long-term memory to an AI agent, give it a persistent store plus two tools: one to save short facts and one to recall the facts that matter for the current task. Then tell the agent when to use them.** This guide does that with TinyContext, an open-source memory server that keeps everything in one local SQLite file. Setup takes about two minutes in Claude Code, Cursor or Codex, and about 60 lines of Python in your own agent.

You will finish with an agent that remembers your preferences, decisions and project facts across sessions, recalls only what is relevant, and fixes facts that change instead of piling up contradictions.

## The short version

- **Install a memory server.**Run- `claude mcp add --scope user tinycontext -- uvx --python 3.12 --from "tinysuite-context[server]" tinycontext`, or add the same command to your client’s MCP config.
- **Tell the agent when to use it.**Add five lines to- `CLAUDE.md`,- `AGENTS.md`or your system prompt: recall at the start of a task, save durable facts, update rather than duplicate, never save secrets.
- **Check it works.**Tell the agent a preference, start a new session, and ask a question that depends on it.

The rest of this page walks through each path in detail: MCP clients, your own Python agent, and an HTTP API for any language.

## What “memory” means for an AI agent

Language models are stateless. Everything a model knows about you in a conversation sits in its **context window**, which is short-term memory: it fills up, and it disappears when the session ends.

**Long-term memory** is a store outside the model that the agent writes to and reads from. Useful agent memory holds two kinds of facts:

| Kind | Examples | How it is recalled | 
|---|---|---|
| Profile | “Call the user Sam.” “Prefers TypeScript and short answers.” | Attached to every recall automatically | 
| Episodic | “The billing service moved to Postgres in October.” “Rejected Redis for caching: too much ops.” | Searched by relevance to the current task | 

The agent should not remember everything. Store conclusions, not transcripts: one sentence per fact, written so it still makes sense months later.

## Before you start

You need:

- uv installed (`uv --version`should print a version). It runs TinyContext without a manual Python setup.
- An agent client that supports MCP (Claude Code, Cursor, Codex, Claude Desktop, VS Code and others), **or**a Python 3.12+ project for the code path.
- A model that handles tool calling well. Memory is a tool, and a model that doesn’t call tools reliably won’t use it.

No account, API key, Docker or vector database is required. The first launch downloads a small local embedding model.

## Option 1: Add memory to Claude Code, Cursor or Codex

### Step 1: Register the memory server

**Claude Code.** Run this once. `--scope user` makes memory available in every project instead of just this one:

**Cursor, Claude Desktop and other JSON-configured clients.** Add this to the client’s MCP config (for Cursor, `~/.cursor/mcp.json` for all projects; for Claude Desktop, `claude_desktop_config.json`):

**Codex CLI.** Add this to `~/.codex/config.toml`:

Restart the client. The agent now has six memory tools: `save_memories`, `recall_memories`, `list_memories`, `get_memory`, `update_memory` and `delete_memory`. Configs for VS Code and other clients are on the agent setup page.

All of these clients launch the same server with the same default database, so **memory is shared between them**. A preference you save in Claude Code is there when you open Cursor.

### Step 2: Tell the agent when to remember

Tools alone aren’t enough. Most agents won’t save anything unless they are told what is worth keeping. Add this to `CLAUDE.md` (Claude Code), `AGENTS.md` (Codex and others), Cursor rules, or your system prompt:

### Step 3: Verify memory across sessions

In one session, say:

Remember that I deploy this site with Cloudflare Pages and that I use pnpm, not npm.


Check the tool activity: you should see a `save_memories` call. Close the session, open a new one, and ask:

How should I install dependencies and deploy?


The agent should call `recall_memories` and answer with pnpm and Cloudflare Pages without being told again. If it answers generically, see Troubleshooting.

To see what has been stored, ask the agent to “list my memories” (it calls `list_memories`), or run `uvx --python 3.12 --from "tinysuite-context[server]" tinycontext doctor` to print the database location.

## Option 2: Add memory to your own Python agent

If you’re building the agent yourself, use the TinyContext library directly. No MCP server or second process is needed.

### Step 1: Install

The example uses the OpenAI Python SDK because almost every model server speaks its API: OpenAI, OpenRouter, Ollama, LM Studio, vLLM and llama.cpp. Point it at your model with environment variables:

### Step 2: The memory pattern

Most reliable memory agents use the same split:

- **Recall automatically.**Before each turn, your code recalls memories relevant to the user’s message and puts them in the system prompt. The agent can’t forget to look.
- **Save through tools.**The model decides what is worth keeping and calls- `save_memory`or- `update_memory`. It knows which facts are durable better than a fixed rule does.

### Step 3: The complete agent

Save this as `memory_agent.py`:

### Step 4: Test it

Stop the script with Ctrl+C and start it again. The conversation history is gone, but memory isn’t:

The answer should mention SvelteKit and avoid an ORM, because both facts came back through `recalled()`. Open `agent-memory.db` with any SQLite browser to see exactly what was saved.

### Making it production-ready

- **One store per user.**- `session_id`narrows recall within one store, such as one project. It is not a security boundary. For several users, give each one their own- `memory_db_path`, or run TinyContext’s hosted multi-user mode, which keeps a separate database per authenticated user.
- **Tune the budget.**- `recall_max_tokens`caps how much memory goes into each prompt. 500 to 1,000 tokens covers most assistants.
- **Let users see and delete memories.**- `list_memories()`and- `delete_memory()`exist for this. Hidden memory that can’t be inspected is a trust problem.

## Option 3: Add memory over HTTP from any language

For a TypeScript, Go or Rust agent, run TinyContext’s HTTP API and call it like any web service:

Save a memory:

Recall memories for a task:

Keep it on `127.0.0.1` or behind an authenticated proxy. The FastAPI reference lists every endpoint, and a Docker image is available for a persistent Streamable HTTP MCP service.

## What an agent should and shouldn’t remember

| Save | Don’t save | 
|---|---|
| Stable preferences (“prefers pnpm”) | Whole conversations or transcripts | 
| Decisions and the reason behind them | Anything already in the code or docs | 
| Project facts that took effort to find | API keys, passwords, tokens | 
| Corrections (“the staging URL changed to …”) | One-off details that won’t matter next week | 

Two rules prevent most memory problems. Write each memory as **one fact that stands on its own**: “Uses Postgres 16 in production”, not “yes, that one”. And **update, don’t append**: when a fact changes, replace the old memory. TinyContext’s `update_memory` supersedes the old record, and near-duplicate saves are skipped automatically, so the store doesn’t fill up with contradictions.

## Why this beats the usual alternatives

Every memory approach is a trade-off between what gets remembered, how much context it costs, and how much infrastructure you run.

| Approach | How it works | Where it breaks down | 
|---|---|---|
| Keep the whole chat history | Resend every past message | Cost grows with every turn, and everything is lost when the session or window ends | 
| Running summary | The model rewrites a summary of the conversation | Details erode with each rewrite, and you can’t correct one fact without rewriting the blob | 
| A memory file(`CLAUDE.md`,`AGENTS.md`) | A Markdown file loaded into every session | Good for fixed rules, but every line costs tokens in every session, and nothing is searched | 
| DIY vector database | Embed chunks, run similarity search | Another service to run, and you still have to build deduplication, updates, keyword matching and token budgets | 
| Mem0, Zep | An LLM extracts facts into a vector or graph store | An extra model call per write, plus a second database or a hosted account (detailed comparison) | 
| TinyContext | Short facts in SQLite, hybrid keyword and embedding recall under a token budget | Built for thousands to tens of thousands of memories, not millions | 

What TinyContext does differently:

- **Recall costs a bounded number of tokens.**Recall returns only what fits the budget. On TinyContext’s benchmark, that was 96.7% fewer tokens than pasting all 300 stored memories.
- **Exact terms are found too.**Embedding-only search often misses names, version numbers and error codes. Recall combines BM25 keyword ranking with embeddings, so “Postgres 16” and “the database upgrade” both find the right memory.
- **Nothing extra to operate.**One SQLite file and a local ONNX embedding model. No vector database, graph database or hosted account, and no memory content sent to a third party.
- **Stale facts get replaced.**Updates supersede old memories with history kept, and near-duplicates are skipped on write.
- **Memory follows you between tools.**The same store works from Claude Code, Cursor, Codex, your own code and TinyWebUI.
- **It’s fast.**Recall p95 stayed under 13 ms from 100 to 20,000 memories on the published benchmark.

**When to choose something else:** if you need relationships between entities modelled as a graph, look at Zep. If you want a fully managed service with an SLA, look at Mem0’s hosted platform. If you are indexing millions of documents rather than remembering facts, use a vector database built for retrieval at that scale. TinyContext has no approximate-nearest-neighbour index, so latency will eventually rise far beyond the sizes tested.

## Troubleshooting

| Problem | Fix | 
|---|---|
| The memory tools don’t appear | Restart the client after editing its config, and check that `uvx --version`works in the same shell the client uses | 
| The first call is slow | The first launch downloads the embedding model. Later calls are fast | 
| The agent never saves anything | Add the memory instructions from Step 2. Without them, most models don’t save on their own | 
| The agent saves but doesn’t recall in a new session | Make the instructions say to recall at the start of every task, or recall automatically in code as in Option 2 | 
| Different clients see different memories | One of them sets `TINYCONTEXT_MEMORY_DB_PATH`or a config file. Point them all at the same database | 
| Recall returns too little | Raise `recall_max_tokens`or`recall_top_k`in the configuration | 

Run `tinycontext doctor` in the same environment as your client to check configuration and storage without downloading anything.

## Next steps

Memory tells an agent what you already know. To let it find what nobody has told it yet, add web search to your agent with TinySearch. Both use the same MCP setup. If you’d rather have a ready-made chat app with memory and web search already connected, run `npx tinywebui` (TinyWebUI).

The full tool reference, configuration options and hosted deployment guide are in the TinyContext docs. The source is on GitHub.

## Frequently asked questions

### How do I give an AI agent long-term memory?

Give the agent two tools, one that saves short facts to a persistent store and one that recalls the relevant facts for the current task, and tell it when to use them. With TinyContext you add one MCP server to Claude Code, Cursor or Codex, or call save_memories() and recall_memories() from your own Python agent. The memories live in a local SQLite file and survive restarts.

### What is the difference between short-term and long-term memory in an AI agent?

Short-term memory is the context window: the messages and tool results in the current conversation. It disappears when the session ends or the window fills. Long-term memory is an external store the agent writes to and reads from, so facts carry over between sessions, projects and even different agent tools.

### Do I need a vector database to add memory to an agent?

No. A vector database is one way to do semantic search, but for agent memory, which usually means thousands of short facts rather than millions of documents, SQLite with keyword and embedding search is enough. TinyContext stores text and embeddings in one SQLite file and recalled in 11.4 ms (median) at 20,000 memories on its published benchmark.

### Can Claude Code, Cursor and Codex share the same memory?

Yes. When each client launches TinyContext over stdio with the default settings, they all read and write the same per-user SQLite database, so a preference saved in Claude Code can be recalled in Cursor or Codex.

### How much does adding memory cost in tokens?

Recall returns only the memories that fit a token budget, 800 tokens by default in the Python example here, rather than the whole store. In TinyContext's benchmark of 300 memories, budgeted recall used 96.7% fewer tokens than pasting every memory into the prompt.

### Does agent memory work with local models like Ollama or LM Studio?

Yes. Memory is a tool the model calls, so any model with reliable tool calling works, whether it runs locally through Ollama, LM Studio or llama.cpp, or remotely through OpenAI or OpenRouter. TinyContext also computes its embeddings locally, so no memory content leaves your machine.
