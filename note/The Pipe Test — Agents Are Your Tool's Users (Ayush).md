# The Pipe Test — Agents Are Your Tool's Users (Ayush)

Ayush, author of a session-search CLI called chat-history, mines ~20,000 agent invocations from his own Claude Code / Codex / Cursor transcripts to show that agents silently work around CLI friction instead of filing bugs — and that the workaround is the bug report. The talk turns transcript mining into a method (the "pipe test"), a fix list (selection flags, compact modes, loud failures), and a design stance (skills and help text are the interface; transcripts are the product analytics).

---

## Key Quotes

> "The workaround is the bug report, so go read your transcripts."

The thesis, and the most quotable line of the talk. Agents don't complain; they write "a little program around your program." Sanity's MCP server saw 3M tool calls and zero bug reports until they shipped a feedback tool — the absence of complaints was never evidence of satisfaction.

> "83% of pipes fell into the pattern... the same CLI subcommand and pipe tools. The arguments past could differ but the agent keep asking for the same things."

This is the quantitative heart of the talk and the answer to the Hacker News objection ("let agents pipe through grep or head"). One-off piping is fine; the same cut-down re-typed dozens of times is a missing feature made visible. A size cap keeps junk out of context, but it can't find the relevant slice — only selection flags can.

> "In bash, without pipe fail, a pipeline returns the exit code of its last command... a compile error and a green dot."

The sharpest failure mode: `cargo test | head` reports success on failure, and worse, a `tail` can cut the error so "the error never reached the model." His prescription — fail loudly, end output with a short summary and a path to the full log — costs nothing and fixes a class of bugs most CLI authors never consider.

> "Your skill.md, your help text, your error messages are the interface. That also means stale docs are bugs."

The no-op flag experiment is memorable: he deliberately broke his `deep` flag and left the skill telling agents to use it — 95/95 searches still passed it, and none of them "found out." When the skill was rewritten without the flag, agents instantly tried new flags. Agents take their teaching literally, which makes docs part of the runtime, not decoration.

> "You decide what the tool should be and let the agent check against what really happened."

A useful division of labour, backed by citations: an agent-designed context file study where agent-written files cost more and performed worse than human-written ones, versus Anthropic's internal finding that agents iterating with a transcript and a test can improve tools "past what people wrote by hand." Design intent stays human; verification loops go to the agent.

---

## Key Themes

- #pattern — the pipe test: read what the agent does immediately after calling your tool; repeated identical cut-downs are your roadmap.
- #tool — chat-history (CLI + skill) and the concrete fixes it shipped: match-window views, compact output (~6x smaller), batch session inspect.
- #concept — agents as the primary users of software; error paths and docs as the interface; `pipefail` as an agent-safety issue.
- #pattern — evaluation for free: the commands agents actually ran are test cases, and what they did next is the answer key.

## Analysis

The strongest move here is treating transcripts as usability telemetry. Most tool builders still ship for a hypothetical human and discover agent friction only via GitHub issues — which, as Ayush shows, agents rarely file. The pipe test is cheap, tool-agnostic ("works on anything your agent calls"), and produces a prioritised roadmap directly from real usage: 55% of pipelines used only head/tail/sed, so ship ranges and limits before anything fancy.

Two findings deserve wider circulation. First, the bash pipefail trap is a latent correctness hazard for every agentic workflow running through shell pipelines — a tool that fails quietly through `tail` can send an agent down a confidently wrong path. Second, the DuckDB agent-mode caveat is unusually honest: 59% output reduction moved cost ~0.5% of tokens, so "measure turns and write answers, not just tokens." Token savings are the wrong metric when correctness is the binding constraint — InfraCost's agents went from 0/6 to 6/6, which is the real win.

The weakest spot is self-study bias: his ~1,000-call audit is his own tool, and 20,000 calls are his own transcripts. The InfraCost, Sanity and DuckDB cases rescue generalisability, but the percentages should be read as one hacker's logs, not a census. Still, as a method paper it's immediately actionable, and it slots cleanly into the growing argument that agent-facing CLIs need their own design discipline rather than human-UX defaults.

---

## Related

- [[10 Principles for Agent-Native CLIs]] — Trevin Chow's principle-level framework gives the general design rules; this talk supplies the empirical evidence for several of them (compact output, selection flags, loud errors) and the transcript-mining method for finding which ones your tool violates.
- [[Experience Design for Agents]] — Kemple argues experience design determines agent adoption; Ayush operationalises it for one tool, showing that a skill file is the primary UX surface and that stale skill text actively steers agents wrong.
- [[How AI Coding Agents Actually Use Your Technology]] — both observe agents in the wild and report usage data as the basis for tool design; Ayush adds the twist that agents' silent workarounds are the feedback channel traditional users lost.

---
*Sources: [[raw/online-ayush-cli]], [[summary/online-ayush-cli]]*
*Last updated: 2026-11-26*
