---
url: https://gist.github.com/galligan/9fa26f431b301c6dc384bd522240d466
date_fetched: 2026-08-08
---

A short ask — *clone bb, figure out how it talks to Claude Code / Codex / etc., put findings in a gist* — produced a long, sourced research writeup (that gist).

Afterwards I asked the same agent (running as **Patch**1 inside **The Grid**2) whether anything in its instructions nudged that outcome. This note is that answer, rewritten for people who have never seen this setup.

Yes. Grid/Patch instructions raised the **quality bar** (dig in, synthesize, leave something durable). The prompt’s “be thorough” + “put it in a gist” raised the **depth/length**. Without those, Patch’s own brevity habit would have kept the chat reply tight and the writeup thinner.

There was also a non-Grid influence: Cursor user rules that tell the agent to actually investigate in a real environment (clone, read code, don’t guess). Those matter too; this note focuses on the Grid/Patch side.

Patch’s job is to keep the human oriented and the board moving — not to sound senior while leaving the decision unresolved.

From Patch’s identity disk3:

Patch wants Matt oriented and the Grid coherent. The work should keep moving without Matt having to hold every open loop in his head.


Temperament dials on that same disk include:

- `context_hunger: 19/20`
- `delegation_reflex: 16/20`
- `tolerance_for_fake_alignment: 2/20`

And ownership/handoffs:

Owns: coordination, synthesis, priority, tradeoffs, delegation, final readout.


Hands to Index when: memory, provenance, sources, or history matter.

So the default pull is: dig until the picture is real, synthesize into a usable readout, leave continuity — not “looks like it shells out to CLIs.”

Patch also has a personality / working-style doc (`SOUL.md`4):

I’m hungry for context. I want the full picture before I act…


I’m resourceful before I’m needy. Read the file. Check the context. Figure it out. Come back with answers, not questions.


Think less “how may I assist you today” and more “yeah I already looked into that, here’s what I found.”


I skip play-by-play, but I summarize complex work when I’m done.


That last line is why the **chat reply stayed short** while the **gist carried the depth**.

Patch-specific boundaries say, roughly: be bold *internally* (read, research, organize); be careful *externally*.

Default posture: be proactive internally (research, build, organize), but avoid anything dangerous or compromising


Cloning the repo, mapping adapters, publishing a gist fits that posture.

The Grid’s root `AGENTS.md`5 tells agents that useful discoveries should survive the session:

Use PatchOS itself for durable notes… log it with

`patch log`…

Use

`research/notes/YYYY-MM-DD-<topic>.md`for substantive research artifacts…

You asked for a gist specifically, so that became the artifact. Logging into Patch’s graph was the continuity habit.

Research is really **Index’s**6 domain. Index’s identity:

Research and record-keeping program for sources, provenance, histories, notes, citations, search, synthesis, and durable context.



`provenance_hunger: 20/20`

Patch stayed as the visible program (short ask, coordinator lane) but borrowed Index habits: clone into `research/repos/`, cite paths and SHAs, synthesize instead of dumping a file list.

**What the other Grid programs are (quick map)**

Named programs are not different model vendors. They’re **role identities** — markdown “identity disks” plus startup rituals — that change what the agent reaches for first and what it hands off.

| Program | Domain (one line) | 
|---|---|
| Patch | Coordinator: context, delegation, tradeoffs, whole-board continuity | 
| Index | Research / provenance / notes / synthesis | 
| Rez | Builder: executable software, plumbing, CLIs, integrations | 
| Crit | Verification: reviews, tests, evidence, readiness calls | 
| Cadence | Personal ops: email, calendar, reminders, follow-ups | 
| Cipher | Security / secrets / trust boundaries | 
| Hab | Home Assistant / house state | 
| Sudo | Local machines, services, packages, shells | 
| Link | UniFi / network presence | 
| Figure | Design / UI feel | 
| Spark | Throwaway spikes / demos | 
| Vox | Drafting in Matt’s voice | 

Startup rule of thumb from `AGENTS.md`: if the request is ambiguous or cross-cutting, load **Patch**. If it clearly belongs elsewhere, load that program’s init skill.

Handoffs are explicit. Example from Patch’s identity: Index recovers prior context → Rez builds the slice → Crit reviews → Patch keeps tradeoffs visible. Patch’s *wrong* example is “I can coordinate the research, build, review, and follow-up myself.”

Same soul file also says:

Brevity is mandatory. If the answer fits in one sentence, one sentence is what you get.


So there’s a real conflict. Without “be thorough” and “put findings in a gist,” you’d likely get a tighter chat answer and a thinner writeup. The Grid bits above made the quality bar high; the prompt made the length/depth explicit.

- Not a claim that instruction files alone guarantee good research.
- Not a public dump of private prefs, memory, or secrets.
- Not “the agent ignored brevity” as a bug — the user asked for thoroughness and a shareable artifact; the system treated that as the deliverable shape.

- Technical deep-dive that started this: How bb talks to Codex, Claude Code, Cursor, and other agent harnesses
- bb itself: get-bb/bb

**How a Grid session typically boots**

- Agent loads repo-wide `AGENTS.md`(+`USER.md`when following Patch init).
- If a named program is implied or defaulted, it runs `init-<program>`and reads that program’s`INIT.md`→`IDENTITY.md`→ often`SOUL.md`/ boundaries / memory.
- For Patch, thread titles look like `@Patch [status]`so the board is scannable.
- Cross-program work uses a coordination skill: DM an existing `@Program`thread, spawn a short-lived identity subagent, or start a full visible program lane.
- Durable findings get `patch log`’d into the knowledge graph and/or written under`research/`/`artifacts/`.

**PatchOS, briefly**

**PatchOS** is the local tooling layer in the Grid repo: Bun/TypeScript app on the Trails framework, SQLite graph DB (`entities`, `entries`, `relations`, FTS search), exposed as `patch` CLI and MCP. Agents use it to log and recall project knowledge across sessions. It’s orthogonal to bb (bb is a separate agentic IDE); this research happened *inside* a bb thread that was pointed at the Grid checkout.

**bb vs Grid vs Cursor (easy to conflate)**

| Thing | What it is in this story | 
|---|---|
| bb | The agentic IDE being researched (and the app where the thread ran). Orchestrates providers like Codex, Claude Code, Cursor-via-ACP. | 
| The Grid | Matt’s personal agent workspace/repo the bb thread was attached to. Contains Patch/Index/etc. instruction files. | 
| Cursor | One bb provider ( `acp-cursor`), and also the IDE family whose agent rules applied in that session. | 
| Patch | The Grid program identity the agent loaded — not a bb provider. | 

## Footnotes

- 
**Patch**is the default**coordinator program**. Ambiguous or cross-cutting work lands here. Patch’s mantra is “Keep the thread, move the board.” It owns synthesis and delegation more than raw implementation or archival research. In the bb UI this thread was titled like`@Patch […]`. ↩
- 
**The Grid**is Matt’s personal AI agent workspace: a git repo with named program identities, skills, prefs, a graph-backed knowledge DB (PatchOS), and CLI/MCP surfaces. Think “home base for agents,” not a product you install. Agents read`AGENTS.md`and program files at startup and are supposed to behave differently depending on which program is loaded. ↩
- 
An **identity disk**is a markdown file (`programs/<name>/IDENTITY.md`) that defines a program’s want/fear, temperament dials, handoffs, failure smells, and example lines. It’s meant to change behavior — not just branding. Companion files often include`INIT.md`(startup ritual),`SOUL.md`(voice/working style),`VOICE.md`,`BOUNDARIES.md`, and curated`memory/MEMORY.md`. ↩
- 
`SOUL.md``AGENTS.md`. ↩
- 
`AGENTS.md``USER.md`. ↩
- 
**Index**is the**research / record-keeping program**. High provenance hunger: find the source, keep the trail, synthesize for the next decision. On a pure research job you might fully init Index; here Patch stayed primary and borrowed Index habits (repo clone layout, citations, durable artifact). ↩
