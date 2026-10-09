# MotherDuck Remote MCP Server

MotherDuck's getting-started guide for its hosted MCP server: a five-minute path from adding a connector in Claude Desktop to querying databases in natural language and saving shareable, live-data visualizations ("Dives") — a concrete example of a data platform shipping agent-facing access as a first-class product surface.

---

The pitch is that you "analyze your data using natural language and generate interactive visualizations, all without writing SQL." The remote server at `https://api.motherduck.com/mcp` is fully managed; a local server exists for local DuckDB files and full control. Setup is connector-based OAuth-style browser authentication in Claude Desktop, followed by a per-tool permission review: `query`, `list_databases`, `ask_docs_question`. Then the demo: attach a sample Hacker News share (`md:_share/hacker_news/...`) with a minimal prompt, get "great results for a first data exploration," and iterate a Dive conversationally — "add a filter for the last 30 days", "switch to a bar chart" — with each edit saved as a version and the saved artifact always querying live data.

The most interesting line is buried in next steps: *"Work with agents through the CLI: For coding agents with a terminal, when the MotherDuck CLI beats MCP on tokens and why."* A vendor admitting MCP is sometimes the wrong interface for its own product is worth more than the rest of the tutorial.

## Key quotes

> **"Tool permissions to control how Claude uses each tool."**

The permission surface is per-tool, not per-query — coarse-grained but present at the client. Compare the far harder problem of [[Securing MCP Servers — Database Access Control]], where the question is what the *tools* may touch, not whether the model calls them.

> **"Iterate conversationally... Each edit saves as a separate version."**

Dives are the interesting design decision: analysis output becomes a persistent, versioned artifact in the workspace rather than a transient chat rendering. That's the same move as [[Databricks Genie Spaces for SQL Analysts]] — keeping the human-facing deliverable inside the platform, with the agent as the interface rather than the destination.

> **"When the MotherDuck CLI beats MCP on tokens and why."**

MCP's tool descriptions and JSON envelopes cost tokens; a CLI over a terminal is cheaper for coding agents. This nuances both [[MCPs Aren't APIs — Stop Treating Them Like One]] and [[MCP Is Dead; Long Live MCP]] — the protocol is settling into a niche where it is one transport among several, chosen per client type.

## Themes

#tool #pattern #databases

## Analysis

As documentation this is fine and unremarkable; as a signal it's better than it looks. Three things stand out. First, agent access to a database is now *onboarding-level* — five minutes, one browser auth, no SQL — which is the culmination of the text-to-SQL arc described in [[Text-to-SQL in the Real World]]. Second, the tool list is deliberately tiny (`query`, `list_databases`, `ask_docs_question`), an example of the small-tool-surface pattern rather than exposing the whole SQL grammar as tools. Third, the share-URL attach (`md:_share/...`) shows the design pressure MCP puts on data platforms: the agent needs a handle to data it can be granted, not credentials to the warehouse.

The honest caveat is the vendor's own: for terminal-having coding agents, MCP loses to a CLI on tokens. This guide is really about the Claude Desktop/ChatGPT audience — chat-first users who will never open a terminal — and that bifurcation (chat clients get MCP, coding agents get CLIs) is becoming the standard architecture.

---
*Sources: [[raw/mcp-getting-started]], [[summary/mcp-getting-started]]*
*Last updated: 2026-10-09*
