# Make Pages Interactive

A Claude Code skill by Paras Chopra that turns static HTML generation into a collaborative iteration loop: Claude builds HTML pages, you comment on them in the browser, and Claude watches those comments to update the output live. It's "Google Docs comments, but for local HTML artifacts" — and it's a concrete instance of showing Claude what you want instead of telling it.

---

## Key Quotes

> "My favorite way of interacting with Claude Code is to have it generate static HTML files as outputs (reports, explorations, code structure, mockups etc.)"

HTML as the universal output format for agent work. Charts, dashboards, reports, mockups — all rendered in the browser, all inspectable, all shareable. This is the quiet thesis behind a lot of agent workflows: `.html` beats `.md` when you need interactivity, layout control, or visual polish, and it uses fewer tokens than `.pptx` or `.docx`.

> "You're basically showing Claude instead of telling Claude. That gap is probably bigger than it seems once you're iterating at speed."

Nate Voss names the core insight. Text prompts are lossy compression of what you actually want. Pointing at a specific element on a rendered page and saying "make this bigger" or "this color is wrong" is higher-bandwidth and lower-ambiguity. When you're iterating fast, the gap between "describe what I want" and "show me and I'll point at what's wrong" compounds.

> "html files are the new whiteboard, except this one writes itself"

Victor's one-liner captures why this pattern keeps emerging independently. The whiteboard was the lowest-friction medium for thinking visually with other people. A self-writing HTML page is the same thing, but it renders at production quality and can be shared with a URL.

## Key Themes

- #pattern — **Show-don't-tell iteration**: In-browser visual feedback as a higher-bandwidth alternative to text-based prompting. The interaction model inverts: instead of describing changes in words, you point at what needs fixing on the rendered artifact itself.
- #tool — **Claude Code skills as workflow primitives**: This is a skill, not a standalone tool. It composes with Claude Code's existing capabilities (HTML generation, localhost serving, file watching). The skill is ~100 lines of glue, not a platform.
- #concept — **HTML as agent output format**: Multiple people in the replies report the same pattern — they've moved their entire workflow (presentations, proposals, dashboards, reports) to HTML because it's token-efficient, visually rich, and trivially shareable.
- #pattern — **Self-iteration loop**: The workflow is personal, not collaborative. You iterate with Claude, not with a team. Comments persist locally in the same folder as the HTML file, so they survive server restarts.

## Critical Analysis

**The right idea, already converging.** Paras built this, and within the same replies you learn that Codex has it natively, Vatsal points out it's now built into Claude Code's desktop preview mode, Nathan Baschez built Roughdraft for Markdown, and Bob Ulrich's team built something similar the same week. This is convergent evolution — the pattern of "comment on visual artifacts to steer agents" is so obviously useful that multiple independent teams arrived at it simultaneously. That's a strong signal. [[Agentation]] is a further instance of the same convergence, with one wrinkle worth stealing: its annotations carry structured coordinates (CSS selector, source file path, React component tree, computed styles) rather than prose, so the feedback survives context compaction as a stable address instead of a pointer into a conversation.

**The compaction problem is real and unaddressed.** Matt's warning is the most important critique in the thread: Claude Code auto-compacts mid-session and silently drops the context your in-browser comments referenced. This isn't a skill bug — it's a fundamental limitation of context-window-based agents. If you leave 50 comments across a complex page and Claude compacts away the first 40, you're iterating on a phantom. This isn't fixed by making the skill better; it's fixed by the underlying agent runtime handling long-lived feedback loops differently.

**"Show don't tell" is powerful but incomplete.** The pattern shines for visual/UI iteration — layout, colors, sizing, chart design. It's weak for structural or logical changes where the problem isn't visible on the page. "This query is wrong" is hard to point at on a rendered dashboard. The interaction model covers the visual surface but not the logic underneath.

**The platform-ification risk.** MagicPath's reply ("instead of having a bunch of local HTML scattered around, use our platform") is the classic platform pitch — and it's probably wrong for this use case. The whole point is zero-friction local iteration without accounts, platforms, or share links. Paras's response (silence) is the correct one. Not everything needs to be a SaaS.

**This should have been native from day one.** Dean Sacoransky's "this should be native to cc" is obviously right, and Vatsal confirms it now is. The interesting question is why it wasn't. Probably because the Claude Code team was focused on terminal-native interaction and didn't anticipate how much agent output would migrate to the browser. HTML as an output format snuck up on everyone, including the platform builders.

---
*Sources: [[summary/make-pages-interactive]]*
*Last updated: 2026-07-05*
