# cmux

A macOS-native terminal built on Ghostty's rendering engine, explicitly designed for developers who manage multiple AI coding agent sessions simultaneously. Native Swift + AppKit, no Electron. Free, by Manaflow.

---

## What Makes It Different

cmux is what happens when you design a terminal for the AI coding agent era instead of the Unix era. tmux was built for multiplexing shell sessions; cmux is built for managing multiple AI agent sessions with a visual attention system. The core insight: when you have four terminals each running a different Claude Code or Codex session, you need to know which one needs your attention *without* polling them manually.

The killer feature is **notification rings** — panes visually highlight when a process needs human input, with unread badges and macOS desktop notifications. This is triggered via OSC escape sequences that agents emit, or via the cmux CLI. It turns the terminal from a passive grid of text into an attention-aware workspace.

## Key Themes

- #tool — macOS terminal app, but the design language is agent management, not terminal emulation
- #concept — Notification rings as an attention layer over terminal sessions. This is a genuinely new interaction model
- #pattern — Terminal-as-control-room: vertical tabs show git branch, working directory, ports, and notification state per session
- #pattern — In-app browser split-paneable with terminal. Scriptable API. The terminal and browser aren't separate tools

## Architecture Notes

**Built on libghostty, not a fork.** This distinction matters. cmux uses Ghostty's terminal rendering library the way apps use WebKit — it gets GPU-accelerated, battle-tested terminal rendering without maintaining a fork. The relationship is supplier/consumer, not upstream/downstream. Ghostty config files at `~/.config/ghostty/config` drive terminal keybindings; cmux-specific shortcuts are separate.

**Native Swift + AppKit.** No Electron, no web views, no cross-platform compromise. This is a macOS-first tool and it shows in the design decisions: vertical sidebar tabs (a macOS idiom), native notifications, and macOS desktop integration that would be awkward to replicate cross-platform.

**CLI + socket API for automation.** The notification ring system is exposed via OSC escape sequences (9, 99, 777) and a cmux CLI. Agents can programmatically signal their state. This is the right instinct: make the terminal *scriptable by the processes running inside it*, not just scriptable from the outside.

## Critical Analysis

**The agent-aware terminal is a real category.** Most terminal innovation has been about rendering speed (Alacritty, Kitty, Ghostty) or multiplexing (tmux, screen). cmux is the first terminal I've seen that treats "I'm running multiple AI agents and need to know when they need me" as the primary design problem. The notification rings aren't a gimmick — they solve a real coordination cost that anyone running parallel agent sessions experiences.

**The libghostty bet is smart.** Building terminal rendering from scratch is a multi-year investment. Using Ghostty's rendering library gives cmux a production-grade terminal without the rendering R&D, letting the team focus on the UX layer where they're actually innovating. The "not a fork" framing is honest about what they're doing.

**macOS-only is the right call for now.** Cross-platform would spread a small team too thin. macOS is where the AI coding tool ecosystem is densest (Claude Code, Codex, Cursor, Zed all have strong Mac presence). Own one platform deeply before expanding.

**The risk is commoditization.** If notification rings and agent-aware tab management prove valuable, expect iTerm2, Warp, and even Ghostty itself to add similar features. cmux's window of differentiation is the quality of execution, not the uniqueness of the idea. The CLI/socket API could become a defensible protocol layer if it gains adoption — but that requires network effects.

**vs. tmux**: tmux is a terminal multiplexer that works inside any terminal. cmux is a native GUI app. tmux requires prefix keys and config; cmux is zero-config with visible tabs. The difference is philosophy: tmux layers multiplexing *inside* the terminal; cmux builds it *around* the terminal.

**vs. [[Ratty]]**: Ratty pushes the terminal into 3D graphics territory (terminal as canvas). cmux pushes it into agent-management territory (terminal as control room). Both are native tools rejecting the Electron paradigm, but they optimize for orthogonal things. Ratty is about expressiveness; cmux is about coordination.

**vs. [[Zed]]**: Both are native macOS apps betting that "no Electron" is a structural advantage that compounds. Both treat AI integration as a first-class design constraint rather than a feature. But Zed is an editor; cmux is a terminal. They're complementary — you could use both together in an agent-native workflow.

## Related Pages

- [[Ratty]] — GPU terminal emulator pushing boundaries in a different direction (3D graphics vs. agent coordination)
- [[Zed]] — Another native macOS developer tool betting on "no Electron" as compound engineering
- [[VTcode]] — Rust coding agent that uses Ghostty PTY snapshots; shares the Ghostty ecosystem
- [[10 Principles for Agent-Native CLIs]] — cmux is a terminal designed for agents; the principles apply to terminal design too
- [[surf-cli]] — Browser automation via CLI; cmux's in-app browser is a different take on the terminal/browser boundary
- [[Agent Coding Workflow]] — The daily loop cmux is designed to support: multiple parallel agent sessions needing human attention

---
*Source: [[summary/cmux]]*
*Last updated: 2026-06-21*
