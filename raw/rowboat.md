---
url: https://github.com/rowboatlabs/rowboat
date_fetched: 2026-07-05
backfilled: true
---

**Open-source AI coworker that turns work into a knowledge graph and acts on it**

Rowboat connects to your email and meeting notes, builds a long-lived knowledge graph, and uses that context to help you get work done - privately, on your machine.

You can do things like:

- `Build me a deck about our next quarter roadmap`→ generates a PDF using context from your knowledge graph
- `Prep me for my meeting with Alex`→ pulls past decisions, open questions, and relevant threads into a crisp brief (or a voice note)
- Track a person, company or topic through live notes
- Visualize, edit, and update your knowledge graph anytime (it’s just Markdown)
- Record voice memos that automatically capture and update key takeaways in the graph

Download latest for Mac/Windows/Linux: Download

⭐ If you find Rowboat useful, please star the repo. It helps more people find it.

**Download latest for Mac/Windows/Linux:** Download

**All release files:**   https://github.com/rowboatlabs/rowboat/releases/latest

To connect Google services (Gmail, Calendar, and Drive), follow Google setup.

To enable voice input and voice notes (optional), add a Deepgram API key in `~/.rowboat/config/deepgram.json`

To enable voice output (optional), add an ElevenLabs API key in `~/.rowboat/config/elevenlabs.json`

To use Exa research search (optional), add the Exa API key in `~/.rowboat/config/exa-search.json`

To enable external tools (optional), you can add any MCP server or use Composio tools by adding an API key in `~/.rowboat/config/composio.json`

All API key files use the same format:

```
{
  "apiKey": "<key>"
}
```
Rowboat is a **local-first AI coworker** that can:

- **Remember**the important context you don’t want to re-explain (people, projects, decisions, commitments)
- **Understand**what’s relevant right now (before a meeting, while replying to an email, when writing a doc)
- **Help you act**by drafting, summarizing, planning, and producing real artifacts (briefs, emails, docs, PDF slides)

Under the hood, Rowboat maintains an **Obsidian-compatible vault** of plain Markdown notes with backlinks — a transparent “working memory” you can inspect and edit.

Rowboat builds memory from the work you already do, including:

- **Gmail**(email)
- **Google Calendar**
- **Rowboat meeting notes**or- **Fireflies**

It also contains a library of product integrations through Composio.dev

Most AI tools reconstruct context on demand by searching transcripts or documents.

Rowboat maintains **long-lived knowledge** instead:

- context accumulates over time
- relationships are explicit and inspectable
- notes are editable by you, not hidden inside a model
- everything lives on your machine as plain Markdown

The result is memory that compounds, rather than retrieval that starts cold every time.

- **Meeting prep**from prior decisions, threads, and open questions
- **Email drafting**grounded in history and commitments
- **Docs & decks**generated from your ongoing context (including PDF slides)
- **Follow-ups**: capture decisions, action items, and owners so nothing gets dropped
- **On-your-machine help**: create files, summarize into notes, and run workflows using local tools (with explicit, reviewable actions)

Live notes are notes that stay updated automatically. You can create one by typing '@rowboat' on a note.

- Track a competitor or market topic across X, Reddit, and the news
- Monitor a person, project, or deal across web or your communications
- Keep a running summary of any subject you care about

Everything is written back into your local Markdown vault. You control what runs and when.

Rowboat works with the model setup you prefer:

- **Local models**via Ollama or LM Studio
- **Hosted models**(bring your own API key/provider)
- Swap models anytime — your data stays in your local Markdown vault

Rowboat can connect to external tools and services via **Model Context Protocol (MCP)**.
That means you can plug in (for example) search, databases, CRMs, support tools, and automations - or your own internal tools.

Examples: Exa (web search), Twitter/X, ElevenLabs (voice), Slack, Linear/Jira, GitHub, and more.

- All data is stored locally as plain Markdown
- No proprietary formats or hosted lock-in
- You can inspect, edit, back up, or delete everything at any time
