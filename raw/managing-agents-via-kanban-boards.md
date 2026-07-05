---
url: https://x.com/geoffreylitt/status/2008941866515824748
date_fetched: 2026-07-05
backfilled: true
---

Some people asked how I built this demo of a coding agent asking follow-ups on a kanban board.
Here are some quick notes + reflections on MCP, vibe-coded CLIs, and malleable software:
level 1: MCP
I started by hooking up the Notion MCP in Claude Code (developers.notion.com/docs/get-start…).
Then made a task board with a couple useful properties
- a "blocked" checkbox property (conditional formatting on this property to get the red card)
- a "current status" text property so the user can see what the agent is up to right now
And then just told the agent to use the board the way I wanted, roughly like this:
Work on this task: <url>. Periodically update the current status property on the task so the user knows what you're up to. If you get blocked and need user input, set blocked: true and ask the user questions in the comments on the task. Once the user responds, set blocked: false and continue working. When you're done, move the task to the Done status.
That's it! Easy to get started and iterate via prompting.
level 2: use CLI tools against the API
I hit some walls with MCP, First I needed a couple Notion features which aren't yet supported in the MCP. But I also hit a more fundamental issue: for some things it's more efficient to express in code than to require the LLM to do it "manually" do it via MCP every time.
For those reasons, I used Claude Code to vibe code a Typescript CLI tool called `notion-cc` which calls Notion's public API to do various things. And then prompted Claude Code to use `notion-cc`!
One illustrative example: I made a command notion-cc wait-for-comment <task-url> which polls until a task has a new comment from the user, then returns that comment. This makes more sense to do in code than for an LLM to do "manually".
And then I just told Claude, "after you ask the user a question, run `notion-cc wait-for-comment` to block until they respond.
reflections on MCP vs code
I really enjoyed the pattern of "LLMs writing code to be called by future LLM sessions". This idea seems to be picking up steam these days, eg see Anthropic's programmatic tool calling. Makes a ton of sense that repeated deterministic logic can be encoded for future use and run more efficiently.
It's also wild how efficiently these CLI tools can be vibe-coded now. I don't even think it's worth me sharing the code for my `notion-cc` tool because you can just vibe code your own CLI tool that does exactly what you want.
reflections on malleable software
I didn't make this kanban workflow all at once. I gradually built it up step by step, as I worked on my project. No breaking flow! No huge yak shaving detours! Just used Claude to gradually evolve the process as I go, little by little.
The "red card" idea came to me while I was flipping between terminal tabs trying to figure out which agent needed my help. Took a minute or two to make it happen.
To me that's the dream of malleable software: evolving our tools as we use them, making them fit us like a leather shoe, rather than bending to the constraints of the software. What a fun time to be alive and building things with computers.

A workflow I'm enjoying for managing coding agents on a kanban board:
When an agent needs your input, it turns the task red to alert you that it's blocked!
And then you can respond right there on the card to unblock it 😎
