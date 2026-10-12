---
url: https://tedhaley.ca/pensieve/
date_fetched: 2026-10-12
---

# Supercharges you

and your agents.

    Search your files, repos and agent sessions, locally, and give your agents the best context for the job. One search points them to the few passages that matter, instead of grepping the whole disk. Fewer tool calls, lower cost.

- Claude Code
- Codex
- Cursor
- Gemini CLI
- Windsurf
- Claude Desktop
- VS Code
- any MCP client

## One search. Every file, repo and session.

Press Control+Shift for a glass panel over your documents, folders, git repositories and conversations with coding agents. Search by meaning or by the exact words. Pick a result, tap Control, and choose what to do with it without touching the mouse.

**Exact**"retry"

**Meaning**payments

**Without**draft

**payments/retry.py**acme-app/payments · max_

*retry*_attempts = 5

**Incident review.md**~/Documents/Ops · card declines,

*retry*with backoff

**Receipts**Folder in ~/Downloads

**Refactor the auth flow**Codex · acme-app · Oct 2

`↩`

`⌘↩`

`⌥↩`

`⌘C`

### By meaning

Plain words find what you meant, even when the file never uses them.

### Exact words

Quotes match the phrase exactly. Mix both to rank exact hits by meaning.

### Leave things out

A minus sign excludes a word or a quoted phrase.

### Narrow it down

Filter to files, code or sessions, a file type, or a folder or repo with `in:`.

## The right context. Not the whole disk.

Agents are only as good as the context they're given. Connected over MCP, your coding agent asks Pensieve one question and gets back the few passages that matter, with file and line. It starts on the right code in fewer tool calls.

› Why do card retries stop after a decline? ● Grep "retry" in acme-app 212 matches ● Read payments/retry.py ● Read payments/gateway.py ● Read billing/invoices.py ● Read tests/test_payments.py … 36 more files Still looking for the backoff rule…

**1 grep · 40 reads and counting**

› Why do card retries stop after a decline? ● pensieve · search "card retry after decline" payments/retry.py:42 stop on hard_decline payments/gateway.py:118 decline codes Incident review.md why we changed it ● pensieve · read payments/retry.py:42 Hard declines skip the retry queue. Maya Rivera owns this; see the Mar 4 review.

**1 search · 1 read**

Illustrative example. In the benchmark below, agents with Pensieve averaged 2.0 tool calls per lookup against 3.4 with grep, and answered after a single call in 82 of 144 runs.

~ $ pensieve integrate recommended claude-mcp: on codex-mcp: on cursor-mcp: on claude: on # "search before you grep" in ~/.claude/CLAUDE.md pensieve-search: on claude-hook: on claude-repo-check: on Agents load MCP servers when they start: restart any that are open to give them Pensieve.

### Scoped to the job

An agent started in a repo sees that repo, its worktrees and its sessions. Name a scope to give it a few repos and a folder of specs.

### Exact or by meaning

Agents use the same search you do: by meaning, by the exact identifier, or both, across code, docs and past sessions.

### Works with your agent

Claude Code, Codex, Cursor or any MCP client, over stdio with `pensieve mcp` or over HTTP. `pensieve integrate` connects the agents on your Mac and adds a “search before you grep” block and skills, including optional ones that point you to teammates who know the code.

## Fewer searches. Right file first.

We gave Claude Code the same code-finding tasks with and without Pensieve: held-out tasks across real repos from 500 to 35,000 files, asked five ways, from plain descriptions to exact names and error messages. With Pensieve it finds more of them, with fewer tool calls, and costs less per lookup in every repo. Each search takes 0.1 to 0.2 seconds.

**131 / 144**runs that found the right file with Pensieve, against 123 / 144 with grep alone

**43% fewer**tool calls per lookup: 2.0 instead of 3.4

**18% cheaper**per lookup: $0.041 instead of $0.050, and cheaper in all four repos

**21% faster**per lookup: 8.6 seconds instead of 10.9

### Right file, first result

One search, before any agent: how often the answer is the top result.

### Agent cost per code lookup

Claude Sonnet in Claude Code, measured in four real repos. Pensieve costs less in every repo, and saves the most in the biggest one.

Method: Claude Sonnet run headless through Claude Code. Tasks are generated from the code and graded on the file found. “With Pensieve” is the default agent tool set plus the instructions block from `pensieve integrate`, measured with Pensieve 0.5.6. Both setups ran the same 72 held-out tasks across four repos (504, 1,783, 2,383 and 34,847 files), twice each, in one batch, because the cost depends on Claude’s prompt cache at the time. These tasks were never used to tune Pensieve. Costs are Claude Code’s reported API costs at list prices. Pensieve used 16% fewer tokens. The “right file, first result” chart uses the same held-out tasks: a single grep wins on exact error messages, and Pensieve wins on everything else. The benchmark scripts are open source in bench/; run them on your own repos.

## See who owns what.

Every repo as a map, with git blame for every line. Ask about any topic and get the people with the most domain knowledge, plus what each teammate has been working on lately. The fastest answer is often a conversation.

### Ownership from blame

Who wrote each area, weighted by the lines that are still there today.

### Domain knowledge

Ask about a topic, not a file, and get the people whose code is most about it.

### Team activity

What each teammate shipped in the last 90 days, and the areas next to your work you haven't explored.

## Know where the knowledge left.

When the people who wrote a part of the codebase are no longer active in it, that knowledge is at risk. Pensieve flags those areas, shows how few people still know them, and who knows them best today.

**Daniel Kim**Not active here since March

**Lena Schultz**Not active here since last year

**Priya Nair**Active this month · knows it best today

**Suggestion:**pair with Priya before changing tax rules, and document the rounding logic in

`vat.py`.### Bus factor at a glance

For every area of a repo: how much of the code was written by people still active there, and how many of them there are.

### Who still knows it

The active people with the most lines in that area, so you know who to pull in before you change it.

### Plan the handover

Spot gaps before an incident does. Point your agent at the area to draft the missing docs.

Example with fictional names and numbers.

## Edit a file. Search it seconds later.

Pensieve watches your folders, repos and agent sessions and re-indexes what changed. Uncommitted edits count, and every worktree is tracked by branch, so your agents never work from stale context.

**payments/retry.py**Uncommitted change in acme-app

**New worktree: fix-backoff**Files that differ from main, tagged with their branch

**Q3 planning.pdf**Saved to ~/Documents/Planning

**Price the annual plan correctly**New turns in a running Claude Code session

## Your work stays on your Mac.

Indexing and search run locally. You decide what gets indexed, and AI is entirely optional.

### Runs locally

An on-device embedding model and a local index in `~/.pensieve`. Its server answers only this Mac, not your network or other websites. No account, no cloud service.

### Switch any source off

Agents, folders, repos and apps each have a switch. Turning one off removes what it indexed.

### Secrets skipped

Password-manager exports, keys, `.env` files and credentials are never indexed.

### AI if you want it

Summaries and insights are optional. Pick who writes them, or leave it off:

## Get Pensieve.

Free and open source. Your coding agent does the setup.

### Ask your coding agent.

Paste this into Claude Code, Codex, Cursor, Gemini CLI, Qwen Code, Windsurf or VS Code.

- *1*- **Paste the prompt**Into whichever coding agent you use.
- *2*- **Your agent installs it**The app, then Pensieve for every agent on your Mac.
- *3*- **Choose what to index**Your agent asks. Nothing is indexed until you choose.
- *4*- **Press Control and Shift**Search while indexing runs. It uses a lot of energy, so plug in.

## Questions.

## What does Pensieve search?

Your files and folders (Documents, Desktop, Downloads and any folder you add), your git repositories including uncommitted changes and worktrees, and your Claude Code, Codex and Qwen Code sessions. Obsidian vaults too, and optionally Apple Notes. Everything is indexed and searched on your Mac.

## How does Pensieve help my agent?

Without it, a coding agent often greps the repo and reads many whole files to find what matters, and every file it reads costs tokens. With Pensieve connected over MCP, the agent asks one question and gets back the most relevant passages with their file and line, so it starts on the right code in fewer tool calls. In our benchmark that made each lookup 18% cheaper and 21% faster; see the measured results above.

## Does Pensieve make Claude Code cheaper?

In our benchmark, Claude Code with Pensieve found the right file in 131 of 144 runs, against 123 with grep alone, with 43% fewer tool calls. Each lookup cost 18% less overall, and 6–28% less in each of four repos from 504 to 34,847 files. The benchmark scripts are open source, so you can run them on your own code.

## How do I get my agent to use Pensieve instead of grep?

Agents grep by habit. The install prompt runs `pensieve integrate recommended`, which adds a short “search before you grep” section to the instructions files of Claude Code, Codex and Gemini CLI, plus a search skill and two hooks for Claude Code. You can also switch these on in Settings → Agents. With it, the agent used Pensieve in all 144 runs of our latest benchmark.

## Which agents work with Pensieve?

Any MCP client. `pensieve integrate recommended` connects Claude Code, Codex, Cursor, Gemini CLI, Windsurf, Claude Desktop and VS Code for you. Other agents can use `pensieve mcp` over stdio, or HTTP. Optional skills also suggest which teammates know an area and help plan new code around what already exists.

## How does it find knowledge gaps?

Pensieve reads git blame and recent commit history for each area of a repo. Areas where most lines were written by people with no recent commits there are flagged, along with how many active people know the area and who among them knows it best.

## Does anything leave my Mac?

Your files, code and sessions don't. Indexing and search run on your Mac. Pensieve only downloads its models and checks GitHub for updates, which you can turn off. The exception is if you choose Claude Code or Codex to write optional AI insights; those are labelled Cloud. The built-in model, a local server like LM Studio, or no AI at all keep everything local.

## Do I need an AI model?

No. Search, agent context, ownership and knowledge gaps all work without one. AI insights are optional: choose the built-in Qwen 3.5 9B, LM Studio, Claude Code, Codex, or leave it off.

## Is Pensieve free?

Yes. It's free and open source under the MIT license. The code is on GitHub.

## Why does macOS ask before opening it?

The app isn't notarized yet. If macOS blocks it, open System Settings → Privacy & Security and click Open Anyway. You only do this once.

## Does it work on Intel Macs or Linux?

The app needs Apple Silicon. On Linux and Intel Macs, the same prompt installs the backend and the browser visualizer. An Intel Mac also gets the search panel if the Xcode command-line tools are installed.

## How do I uninstall it?

First run `pensieve integrate all --remove` to disconnect your agents. Then quit Pensieve, drag it to the Trash, and delete `~/.pensieve`, `~/.local/share/uv/tools/pensieve` and `~/.local/bin/pensieve`. Its models are in `~/.cache/huggingface`.
