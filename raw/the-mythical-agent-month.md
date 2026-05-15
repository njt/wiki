---
title: "The Mythical Agent-Month"
url: https://wesmckinney.com/blog/mythical-agent-month/
date_fetched: 2026-05-14
section: "AI-enhanced Coding"
---

# The Mythical Agent-Month - Wes McKinney

(Fetch timed out; content reconstructed from annotation and existing wiki context.)

Wes McKinney's update to Brooks's "The Mythical Man-Month" for the age of AI coding agents.

## Core Argument

Agents are so good at attacking accidental complexity that they generate new accidental complexity that can get in the way of the essential structure you are trying to build. With new projects (roborev and msgvault), McKinney is already dealing with this problem as he reaches the 100 KLOC mark, watching agents begin to chase their own tails and contextually choke on the bloated codebases they have generated.

At some point beyond that (the next 100 KLOC, or 200 KLOC) things start to fall apart: every new change has to hack through the code jungle created by prior agents. Call it a "brownfield barrier."

At Posit they have seen agents struggle much more in 1 million-plus line codebases such as Positron, a VSCode fork. This seems to support Brooks's complexity scaling argument.

## Key Concept
The "brownfield barrier" -- the point at which agent-generated code becomes so thick and tangled that further agent work degrades rather than improves the system.
