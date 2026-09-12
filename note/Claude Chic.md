# Claude Chic

Wes McKinney's alpha-stage alternative terminal UI for Claude Code, built on Python's Textual framework and the Claude Agent SDK. Its standout feature is a live roborev review sidebar that shows pass/fail verdicts inline during coding sessions, enabling a review-fix loop that never leaves the terminal.

---

## Key Quotes

> "A stylish terminal UI for Claude Code, built with Textual."

The word "stylish" is doing real work here. McKinney is explicitly competing on UX, not just capability. The Textual framework gives him Python-native widget composition, which means the sidebar, command palette, and multi-agent views are first-class UI elements rather than terminal hacks.

> The sidebar auto-updates -- the verdict flips to a green P.

The review-fix-review cycle completes without switching windows. This is the core value proposition -- not that roborev reviews exist (they already do), but that the feedback is ambient rather than requiring a context switch.

> "Built on the Claude Agent SDK."

Not wrapping Claude Code's CLI. This is a direct SDK integration, which means Claude Chic owns the agent loop rather than screen-scraping someone else's. This is architecturally significant -- it's the difference between a theme and a fork.

## Key Themes

#claude-code #terminal-ui #code-review #tool #agentic-development #git-worktrees

Claude Chic sits at an interesting intersection. It's both a Claude Code alternative (competing with the default terminal UI, [[vibes-cli]], [[Collaborator]], [[happy]]) and a roborev integration vehicle (competing with how you'd otherwise run code review in a separate terminal).

The sidebar is the conceptual centerpiece. It treats code review not as a gating event but as ambient information -- like a CI status indicator, not a pull request. This design decision matters: if review is ambient, you check it more often, fix issues sooner, and the loop tightens.

Git worktree support means you can switch branches and the sidebar filters to show only reviews relevant to that worktree. This connects directly to the [[Agent of Empires]] pattern of worktree-isolated parallel development, but with the review layer integrated rather than separate.

## Critical Analysis

**The good:** The SDK-native approach is the right call. Screen-scraping Claude Code's TUI would be fragile; owning the agent loop via the SDK means Claude Chic can add UI features (sidebar, command palette, multi-agent views) without fighting the underlying tool. The Textual choice is pragmatic -- Python is already the language of the Claude Agent SDK, so there's no impedance mismatch.

**The concern:** Alpha-status software from a solo developer pursuing many projects simultaneously (roborev, Kata, MSGVault, Money Flow, Spicy Takes). McKinney's output is remarkable but raises the obvious question of which projects get sustained attention. Claude Chic's value is tightly coupled to roborev's value -- if you're not using roborev for code review, the sidebar is dead space.

**The pattern:** This is part of a broader trend of alternative Claude Code interfaces. [[vibes-cli]] targets non-coders with single-file HTML apps. [[Collaborator]] offers an infinite canvas. [[happy]] provides mobile and web access with voice. Claude Chic targets the developer who wants code review integrated into their coding environment. Each alternative makes a different bet about what the default Claude Code interface gets wrong.

**The McKinney connection:** The author also wrote [[Radical Accountability]] and created [[Kata]]. Claude Chic is a downstream bet on his own ecosystem -- if you buy into roborev for review and Claude Code for development, Claude Chic is the integration layer that makes them feel like one tool. This is vertical integration by a solo developer, which is either visionary or fragile depending on your priors.

---
*Sources: [[summary/claude-chic]]*
*Last updated: 2026-05-15*
