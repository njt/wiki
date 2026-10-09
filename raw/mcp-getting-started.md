---
url: https://motherduck.com/docs/getting-started/mcp-getting-started/
date_fetched: 2026-10-09
---

# Talk to Your Data with AI

The MotherDuck **remote** MCP Server lets you analyze your data using natural language and generate interactive visualizations, all without writing SQL. Connect your favorite AI assistant (Claude, ChatGPT, Cursor, or others) and start asking questions about your databases, then turn insights into shareable Dives with a single prompt.

The remote MCP server is hosted at `https://api.motherduck.com/mcp`. Claude Desktop's connector uses this URL automatically; for clients that need manual configuration, see the setup guide.

This guide covers the **remote MCP server** (fully managed by MotherDuck). If you need to work with local DuckDB files or want full control over the server, see the local MCP server.

In this guide, you'll connect the MCP server in Claude Desktop, query your data, and create a Dive visualization, all in under 5 minutes.

## What you'll learn

- Connect the MotherDuck MCP Server to Claude Desktop
- List your databases
- Ask analytical questions about your data
- Create an interactive Dive visualization from your analysis

## Prerequisites

- A MotherDuck account (sign up free)
- Claude Desktop installed (download)

This guide uses Claude Desktop, but the remote MCP Server works with ChatGPT, Cursor, Claude Code, and other MCP-compatible clients. See the full setup guide for instructions for your preferred client.

## Step 1: Add the MCP server to Claude Desktop

Open Claude Desktop and add the MotherDuck connector:

- Open **Customize**→**Connectors**
- Click **+**to browse the directory and search for "MotherDuck"
- Open **MotherDuck**and click**Connect**
- A browser window opens for authentication with your MotherDuck account

## Step 2: Verify the connection and permissions

After adding the connector, confirm Claude has access to the MotherDuck tools:

- Open **Customize**→**Connectors**
- Select **MotherDuck**to review**Tool permissions**

You should see tools like `query`, `list_databases`, and `ask_docs_question` available. You can configure tool permissions to control how Claude uses each tool. See Configuring tool permissions for details.

## Step 3: List your databases

Test the connection by asking Claude to list your databases:

**Try this prompt:**

```
List all my databases on MotherDuck.
```
Claude will use the MCP tools to connect to MotherDuck and return your database list.

## Step 4: Analyze your data

To start your analysis, attach the sample Hacker News database:

**Attach the sample database:**

```
Attach this db 'md:_share/hacker_news/daa0cc99-d20c-4f5c-abd5-f7b22a1a1da9' give me some analytics.
```
The `hacker_news` database contains every Hacker News story and comment since 2006, refreshed daily. You'll see that even with a minimal prompt, you get great results for a first data exploration. For more tips on effective prompting and workflow patterns, check out the MCP Workflows Guide.

The `hacker_news` database is one of several sample datasets available. See Sample Data & Queries for more datasets to explore.

## Step 5: Create visualizations with Dives

Turn your insights into a persistent, interactive visualization. Dives are shareable visualizations that live in your MotherDuck workspace and stay up to date with your data.

**Try this prompt:**

```
Create a Dive based on these insights.
```
Claude renders the Dive inline in the conversation with the Dive Viewer MCP App, using the same components as the MotherDuck UI and running against live data. Iterate conversationally: *"add a filter for the last 30 days"*, *"switch to a bar chart"*. Each edit saves as a separate version.

```
Save it to MotherDuck.
```
The Dive is saved to your workspace. You can open it in the MotherDuck UI, share it with your team, and it will always query live data.

## Next steps

You're ready to analyze your data and create visualizations with AI. Here are some ways to go deeper:

- **MCP Workflows Guide**: Best practices and workflow patterns, including how it works under the hood
- **Creating Visualizations with Dives**: Go deeper into Dives by iterating on visualizations, sharing with your team, and managing version history
- **Connect to MCP Server**: Setup instructions for ChatGPT, Cursor, Claude Code, and other clients
- **MCP Server Reference**: Server capabilities, available tools, and regional availability
- **Building Analytics Agents**: Build custom AI agents that programmatically query your data
- **Work with agents through the CLI**: For coding agents with a terminal, when the MotherDuck CLI beats MCP on tokens and why
