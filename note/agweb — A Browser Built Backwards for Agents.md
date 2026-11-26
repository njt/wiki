# agweb — A Browser Built Backwards for Agents

agweb is a macOS WebKit browser whose entire architecture is inverted relative to every other browser: the *person* is the observer, and the *agent* is the primary user. It runs as one visible window that never activates or raises, driven by a `agweb` CLI speaking a one-line-JSON protocol over a Unix socket. Each agent gets a named workspace with its own tabs and profile; a ⌘D dashboard shows every agent's tabs live. It matters because it is the most thorough existing answer to a question most tooling dodges: what does a browser look like when the person is not the user?

---

## Architecture

Three Swift targets (~12,400 lines total), documented in the repo's own [CLAUDE.md](https://github.com/paul-nameless/agweb):

- **`Sources/AgwebCore`** — host-free, unit-tested library: the `Request`/`Response` socket protocol (`Protocol.swift`), snapshot diffing and pruning (`Snapshot.swift`), the injected page JavaScript (`PageScripts.swift`, 1600 lines), human motion synthesis (`Motion.swift`), uBlock filter lists compiled to WebKit content rules (`FilterList.swift`), WebAuthn encoding (`Passkey.swift`).
- **`Sources/AgwebApp`** — the GUI app. `Browser.swift` (2,214 lines) dispatches commands with one command at a time per workspace, workspaces in parallel; `Model.swift` holds workspaces/tabs/profiles; `Unload.swift` kills idle tab processes; `Visibility.swift` manages which tabs WebKit perceives as visible; `ClipboardRouting.swift` splits page copies between the person's and the workspace's clipboard.
- **`Sources/agweb`** — the CLI. It finds the app's bundle via its own symlink, and if the socket isn't serving, launches the app with `open -g -n` (the `-g` flag is essential: LaunchServices activation would itself steal focus).

The transport is deliberately primitive: plain AF_UNIX sockets, one JSON request line in, one JSON response line out, connection per command (`Protocol.swift`). A `flock` on `<socket>.lock` ensures one instance per `AGWEB_ROOT`; `AGWEB_ROOT=/tmp/x` runs a fully isolated second instance — the same trick the project's own selftests and any agent experimentation use.

## Key techniques

- **Never-steals-focus as an architectural invariant.** Native mouse/key events go directly to a `WKWebView` (`webView.mouseDown`) rather than through `NSWindow.sendEvent`, so they reach tabs stacked underneath and never activate the app. The `AgentWindow` subclass lies to WebKit about key status — reporting key *while the app is in the background* — because WebKit otherwise drops mouse-move events. The honest costs are documented: `document.hasFocus()` is false for pages, and WebKit leaves secure input (event-tap-blinding) enabled that a timer then releases.
- **Action replies carry diffs, not snapshots.** Every click/fill reply includes `### Changes` (added/removed snapshot tree lines) and `### Events` (dialogs, popups, downloads, fetches). Actions also wait for the fetches they caused. The context-saving claim rests on this: the agent almost never takes a second snapshot.
- **Hand-written byte scanners where Swift's Regex died.** `Snapshot.refLabel` is a manual UTF-8 scanner because a `Regex` per line took 9 s on a 50,000-line snapshot; `find` lowercases the tree once and collects enclosing context lines in a single pass (39 s → 1 s for 1,500 matches). The benchmark history in `docs/benchmarks.md` reads like a performance postmortem, with before/after tables per change.
- **Aggressive process killing for memory.** WebKit's own suspension keeps the page in memory (1,621 MB loaded → 1,135 MB suspended), so `Unload.swift` sends SIGKILL to idle tabs' web content process PIDs (`_killWebContentProcess` does nothing on macOS 26), keeping URL/history/cookies and transparently reloading on next use — including POST results, which it deliberately does *not* resend.
- **Human imperfection as a feature.** `Motion.swift` synthesises Fitts's-law pointer paths on bowed cubic Béziers with sub-pixel wobble, occasional ballistic misses with corrective second legs, and keystroke delays sampled log-normally from the Dhakal et al. 136M-keystroke dataset — so hover menus, drag-and-drop and bot checks see a plausible hand.
- **Secrets never in the transcript.** The CLI itself runs `--secret-cmd 'pass show site'` (with a 4 KB read cap and a grandchild-process pipe drain for gpg/pinentry) and passes the value as an option, never an argument, because the app's command log keeps arguments. Typed text is likewise excluded from the command log.

## Design decisions

- **WebKit-only, headed-only.** Accepting no Chrome DevTools Protocol means Playwright and extensions can't attach and Chrome-only sites may break — the price for a real, visible, low-overhead window. The README says outright: "For what it can't do, use Playwright."
- **CLI compatibility as adoption strategy.** Commands mirror `playwright-cli` where possible (even accepting no-op `--headed`/`--browser` flags out of habit), with `js` as an alias for `eval` because some agent sandboxes refuse the word.
- **Honest limits, documented at unusual depth.** The CLAUDE.md lists WebRTC proxy leaks, the passkey store being agweb-only pending Apple's browser entitlement, and the fact that popup tabs share their opener's process so must be co-unloaded. This level of written-down SPI folklore is itself a contribution.

## Comparison notes

- Unlike [[Lightpanda Browser]], which builds a *headless* browser from scratch in Zig for token efficiency, agweb keeps a real headed rendering engine and pays for it in macOS-specific SPI hacking — the opposite bet on the headless-vs-headed axis, both optimising for agent context size.
- [[Browser Use]] wraps existing browsers with anti-detection in Python; agweb instead makes detection-resistant behaviour a property of the input synthesis layer and owns the whole browser process.
- [[Chrome DevTools MCP — Debug Your Browser Session]] lets agents attach to a *person's* Chrome session; agweb is the mirror image — a browser the person shares *with* agents, with workspaces and clipboard routing as the boundary.
- Against [[10 Principles for Agent-Native CLIs]]' general claims, agweb is a worked example: reply-as-diff, refs that persist, output caps, and stderr-carried errors are exactly the principles it enumerates, implemented end to end.

---
*Sources: [[raw/agweb]], [[summary/agweb]]*
*Last updated: 2026-11-26*
