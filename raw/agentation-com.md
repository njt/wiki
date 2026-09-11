---
url: https://www.agentation.com/
date_fetched: 2026-09-11
---

# Visual feedback.

For agents.

Agentation turns UI annotations into structured context that AI coding agents can understand and act on. Click any element, add a note, and paste the output into Claude Code, Codex, or any AI tool.

## How you use it

- Click the icon in the bottom-right corner to activate
- **Hover**over elements to see their names highlighted
- **Click**any element to add an annotation
- Write your feedback and click **Add**
- Click to copy formatted markdown
- Paste into your agent

**Note:** With MCP, you can skip the copy-paste step entirely — your agent already sees what you're pointing at. Just say “address my feedback” or “fix annotation 3.”

## How agents use it

Agentation works best with AI tools that have access to your codebase (Claude Code, Cursor, etc.). When you paste the output, agents get:

- **CSS selectors**to grep your codebase
- **Source file paths**to jump directly to the right line
- **React component tree**to understand the hierarchy
- **Computed styles**to understand current appearance
- **Your feedback**with intent and priority

Without Agentation, you’d have to describe the element (“the blue button in the sidebar”) and hope the agent guesses right. With Agentation, you give it `.sidebar > button.primary` and it can grep for that directly.

## Agents talk back

With MCP integration and the Annotation Format Schema, agents don’t just read your annotations — they can respond to them:

- **“What annotations do I have?”**— List all feedback across pages
- **“Should this be 24px or 16px?”**— Agent asks for clarification
- **“Fixed the padding”**— Agent resolves with a summary
- **“Clear all annotations”**— Dismiss everything at once

Your feedback becomes a conversation, not a one-way ticket into the void.

## Best practices

- **Be specific**— “Button text unclear” is better than “fix this”
- **One issue per annotation**— easier for the agent to address individually
- **Include context**— mention what you expected vs. what you see
- **Use text selection**— for typos or content issues, select the exact text
- **Pause animations**— to annotate a specific animation frame

## Licensing

Agentation is free for individuals and companies for internal use. Use it to annotate your own projects, debug your own apps, or streamline feedback within your team. Contact us for a commercial license if you're redistributing Agentation as part of a product you sell.

Set up real-time sync with MCP →
