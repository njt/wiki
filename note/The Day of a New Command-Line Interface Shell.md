# The Day of a New Command-Line Interface Shell

Björn Stahl's 2022 essay for the Arcan project dismantles the terminal stack piece by piece — tty devices, in-band escape sequences, terminal emulators — and rebuilds the CLI shell on top of a real display server, where jobs get their own bidirectional connections, stdin stays pure data, and the graphical and textual shells finally cooperate. The rare interface essay that shows working footage rather than mockups.

---

## The diagnosis: in-band signalling is the original sin

Stahl's central claim is that everything wrong with CLI UX traces to one design decision: control data and user data share the same channel. The terminal is not "a terminal" — it is *an emulator of an amalgamation of tens to hundreds of different hardware devices*, and every escape sequence it receives is an instruction executed against a fragile state machine:

> it is executing random instructions in a complex and varying instruction set.

That is why `cat /dev/random` permanently corrupts your scrollback: the shell "continues on unawares" while the device's character map, cursor, and flow control have been rewritten. Shells that hardcode resets "mask the danger" rather than fix the design. The whole edifice is, in his framing, an inefficiency and unreliability problem, not a nostalgia problem: "such a critical part of computing should not be limited to device restrictions set in place some 50-70 years ago."

## The shell's two roles, and how one ate the other

The sharpest analytical move is splitting the shell into two jobs that modern shells fused:

- a **"window manager" of sorts** — the prompt, job articulation, pipelines, foreground/background, which tmux and screen extend into tiling;
- a **scriptable programming environment** — "a secondary feature at best, and not at all necessary."

The multiplexers are his exhibit of compounding failure: to do window management they had to embed additional terminal emulators and other shells — "recursive premature composition" — because there is no proper handover or embedding mechanism. He also stresses that the terminal emulator is "a poor take on a display server": one shared triplet of files instead of a distinct bidirectional connection per client. Gdb's `tui enable`/`tui disable` dance and completion popups polluting scrollback are his demonstrations of the line-mode/screen-mode contradiction, inherited from the days when output was literally paper and "moving the cursor around arbitrarily back across previous lines is a privilege — not a right."

## The replacement: handover, not emulation

The Arcan architecture deletes the emulator and the tty layer. The shell requests resources from the graphical shell on behalf of each job and forwards connection primitives — **handover allocation** — retaining a chain of trust and custody. Jobs are *negotiated*: the graphical shell knows their purpose and origin, so they spawn as detachable windows that retain hierarchy. **Live migration** replaces multiplexers: clients redirect between servers at runtime, including over the network.

The gains list is long but the notable items are the ones today's stack cannot fake: ESC the key is distinguishable from ESC the byte; Ctrl+C is a symbolic key with a modifier, not a magic SIGINT broadcast; paste is separated from typing and undoable as a discrete whole; colours are always 24-bit or from a semantic palette; accessibility views propagate semantically tagged content so a screenreader can say "paste into job #1". Rendering moves to the display server with atomic commits and back-pressure control owned by the job, not emulator heuristics.

---

## Opinion

This is 2022 and mostly unbroadcast outside a niche, yet it reads today like a design document for exactly the interfaces agent tooling now stumbles over. When a coding agent needs to interleave a diff view, a test-runner TUI, a file picker, and streamed logs without corrupting each other, the tty stack is precisely the impedance mismatch everyone hits — and the current workarounds (structured output side-channels, OSC hacks, agent-specific rendering surfaces) are sidebands doing badly what shmif does properly. The "shell as window manager, scripting as secondary role" split also anticipates the current split between agent harnesses (orchestrating jobs) and shells (presenting them).

The honest criticism is adoption economics: Arcan asks you to replace the display server to fix the shell, and the four decades of invested tooling around VT semantics — including every agent harness built on terminal scraping — are a moat of sunk cost that pure architectural correctness cannot cross. His own list of "obvious" capabilities ("there is a Lovecraftian horror hiding behind each and every one") is the best one-paragraph justification for the attempt, even if the world answered by building worse copies of these ideas inside Electron apps instead.

## Connections

This essay strengthens [[Command Line Interface Guidelines]] by supplying the structural argument underneath its practical advice: those guidelines document how to work *within* the terminal's constraints; Arcan argues the constraints themselves are the bug. It nuances [[10 Principles for Agent-Native CLIs]] — that piece treats the CLI as an interface for agents and recommends structured, machine-readable output, which is precisely what a world without in-band signalling would make trivial. It complicates [[Tmux Resurrect]]: tmux's session persistence is a patch over the emulator-shell split, the exact "recursive premature composition" Stahl identifies as a symptom, not a feature. And it resonates with [[OpenDisplay]] as another attempt to replace legacy display/connection protocols with a purpose-designed, handover-capable alternative.

---
*Sources: [[raw/the-day-of-a-new-command-line-interface-shell]], [[summary/the-day-of-a-new-command-line-interface-shell]]*
*Last updated: 2026-10-10*
