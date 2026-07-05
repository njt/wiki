---
url: https://github.com/prime-radiant-inc/engineering-notebook
date_fetched: 2026-07-05
backfilled: true
---

A CLI tool that ingests Claude Code and Codex session transcripts, generates LLM-powered daily summaries, and serves a web UI for browsing your engineering journal.

Think of it as an automatic engineering diary — it watches your AI coding sessions and distills them into a searchable, browsable narrative of what you built, what problems you hit, and what decisions you made.

- **Ingest**— Scans directories of Claude Code and Codex JSONL session files, parses out the human-readable conversation (stripping tool calls, thinking blocks, etc.), and stores them in SQLite.
- **Summarize**— Groups sessions by date and project, then uses Claude to write concise engineering journal entries with headlines, summaries, topics, and open questions.
- **Serve**— Runs a web server with a browsable UI: daily journal, project timelines, calendar/Gantt view, session transcripts, full-text search, and an iCal feed.

Requires Bun v1.1+.

```
git clone https://github.com/prime-radiant-inc/engineering-notebook.git
cd engineering-notebook
bun install
bun link  # makes `engineering-notebook` available globally
```
```
# 1. Ingest your sessions (defaults to ~/.claude/projects and ~/.codex/sessions)
engineering-notebook ingest
# 2. Generate journal summaries (requires ANTHROPIC_API_KEY)
engineering-notebook summarize --all
# 3. Browse your journal
engineering-notebook serve
# Open http://localhost:3000
```
The web interface has several views:

- **Journal**— Three-panel layout: date index, entries for the selected date, and entry detail with full conversation transcripts. Entries include headlines, summaries, topic tags, and open questions.
- **Projects**— Browse all projects sorted by recency. Each project shows a timeline of journal entries with on-demand summarization for new sessions.
- **Calendar**— Week and month views with Gantt-style bars showing project activity across dates. Days link back to the journal view.
- **Search**— Full-text search across journal entries and conversation transcripts with highlighted results.
- **Settings**— Configure sources, exclusions, remote sources, summary instructions, auto-sync interval, and day start hour. Includes an iCal feed URL for subscribing in your calendar app.

| Calendar/Gantt view | Full-text search | 

Each session transcript includes a resume command (`claude --resume <id>`) with a copy button for picking up where you left off.

Scan source directories and ingest session files into the database.

```
engineering-notebook ingest                    # scan default sources
engineering-notebook ingest --source ~/extra   # add an extra source directory
engineering-notebook ingest --force            # re-ingest already-processed sessions
```
Generate LLM summaries for ingested sessions.

```
engineering-notebook summarize --all                      # summarize everything unsummarized
engineering-notebook summarize --date 2026-02-22          # summarize a specific date
engineering-notebook summarize --project myapp            # summarize a specific project
engineering-notebook summarize --date 2026-02-22 --project myapp  # both filters
```
Start the web server.

```
engineering-notebook serve              # default port 3000
engineering-notebook serve --port 8080  # custom port
```
The server auto-syncs remote sources and re-ingests on a configurable interval (default: 60 seconds).

Config lives at `~/.config/engineering-notebook/config.json`:

```
{
  "sources": ["~/.claude/projects", "~/.codex/sessions"],
  "exclude": ["-private-tmp*", "*-skill-test-*"],
  "db_path": "~/.config/engineering-notebook/notebook.db",
  "port": 3000,
  "day_start_hour": 5,
  "summary_instructions": "",
  "remote_sources": [],
  "auto_sync_interval": 60
}
```
| Field | Description | Default | 
|---|---|---|
| `sources` | Directories to scan for session files | `["~/.claude/projects", "~/.codex/sessions"]` | 
| `exclude` | Glob patterns for directories to skip | `["-private-tmp*", "*-skill-test-*"]` | 
| `db_path` | SQLite database location | `~/.config/engineering-notebook/notebook.db` | 
| `port` | Web server port | `3000` | 
| `day_start_hour` | Hour (0-23) when a "day" starts (for grouping late-night sessions with the previous day) | `5` | 
| `summary_instructions` | Custom instructions appended to the LLM summarization prompt | `""` | 
| `remote_sources` | SSH remote sources to sync before ingesting | `[]` | 
| `auto_sync_interval` | Seconds between auto-syncs when serving | `60` | 

Sync session files from remote machines over SSH:

```
{
  "remote_sources": [
    {
      "name": "workstation",
      "host": "work.local",
      "path": "~/.claude/projects",
      "enabled": true
    }
  ]
}
```
When the server is running, subscribe to the iCal feed at:

```
webcal://localhost:3000/api/calendar.ics
```
This creates calendar events for each journal entry, viewable in Apple Calendar, Google Calendar, Outlook, etc.

```
bun install
bun test          # run tests
bun src/index.ts  # run from source
```
- Bun — runtime, bundler, test runner, SQLite
- Hono — web framework
- Anthropic SDK — LLM summarization (Claude Haiku)
- HTMX — interactive web UI without a JS framework

Apache 2.0 — see LICENSE.
