---
url: https://simonwillison.net/guides/agentic-engineering-patterns/agentic-manual-testing/
title: Agentic manual testing
author: Simon Willison
date_fetched: 2026-05-15
date_published: 2026-03-06
---

# Agentic manual testing

The defining trait of a coding agent is that it can execute the code it writes, making it far more useful than LLMs that just output unverified code.

## Opening Principles

- Never assume LLM-generated code works until it has been executed.
- Coding agents can confirm their output works or iterate until it does.
- Writing unit tests (especially TDD-style) is powerful but not sufficient.
- **Automated tests are no replacement for manual testing** — tests can pass while the code itself fails in obvious ways (crashing a server, missing UI elements, etc.).

## Mechanisms for Agentic Manual Testing

### Python libraries
Use `python -c "... code ..."` to execute snippets inline. Prompt: *"Try that new function on some edge cases using \`python -c\`"*

### Other languages
Write a demo file to `/tmp`, compile, and run, avoiding accidental commits. Prompt: *"Write code in \`/tmp\` to try edge cases of that function and then compile and run it"*

### Web apps with JSON APIs
Use `curl` against a dev server. Prompt: *"Run a dev server and explore that new JSON API using \`curl\`"*

### Fixing discovered issues
Use red/green TDD to ensure the fix gets permanent automated test coverage.

## Browser Automation for Web UIs

- **Playwright** (Microsoft, open source) — described as "the most powerful of these today." Supports multiple languages and browser engines.
- **agent-browser** (Vercel) — a dedicated CLI wrapper around Playwright designed for agent use.
- **Rodney** (Simon Willison's own project) — uses the Chrome DevTools Protocol to control Chrome directly.

### Example Prompt with Three Embedded Tricks
*"Start a dev server and then use \`uvx rodney --help\` to test the new homepage, look at screenshots to confirm the menu is in the right place"*

1. `uvx` auto-installs Rodney the first time via Astral's package manager.
2. `rodney --help` is deliberately designed to give agents everything they need to understand and use the tool.
3. "look at screenshots" cues the agent to use `rodney screenshot` and leverage its own vision capabilities.

## Managing Test Fragility

Many developers avoid automated browser tests due to flakiness — small HTML tweaks cause waves of failures. Willison notes that "having coding agents maintain those tests over time greatly reduces the friction involved."

## Have Them Take Notes with Showboat

Showboat creates documents capturing the agentic manual testing flow.

Prompt: *"Run \`uvx showboat --help\` and then create a \`notes/api-demo.md\` showboat document and use it to test and document that new API."*

### Three Key Showboat Commands
- `note` — Appends a Markdown note to the document
- `exec` — Records a command, runs it, and captures output
- `image` — Adds an image (e.g., browser screenshots from Rodney)

The `exec` command is most important because it captures both the command and its actual output, discouraging agents from writing what they *hoped* happened rather than what actually occurred.

## Chapter Context
This is Chapter 3.3 of the Agentic Engineering Patterns guide.
- Previous: First run the tests
- Next: Linear walkthroughs

## External Links
- Playwright: https://playwright.dev/
- agent-browser: https://github.com/vercel-labs/agent-browser
- Rodney: https://github.com/simonw/rodney
- Showboat: https://github.com/simonw/showboat
