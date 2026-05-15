# Assorted less(1) Tips

Tim Chase compiled 17 `less(1)` techniques from a Reddit thread, and an HN discussion added another dozen from the community. The combined thread is a masterclass in a fading art: knowing your pager deeply. 238 upvotes, 55 comments.

---

## Key Quotes

> "I like to enable `-R` and `-e` and `-N` to show ANSI colors, show line-numbers, and exit automatically when you reach the end of the file." — Tim Chase

The `$LESS` environment variable is the simplest thing that pays off forever. Three flags, set once in `.bashrc`, and every pipe-to-less for the rest of your career gets colors, line numbers, and auto-exit. The product of a 30-second edit amortized across decades.

> "`less +F` starts less following stdin or whatever file argument you've provided. CTRL-C to break follow mode, and uppercase F to resume it." — teeray (HN)

The plus-command syntax is the secret handshake of pager power users. `+F` for follow, `+G` for jump-to-end — you can prepend any less command as a startup argument. This is the sort of thing you either learn in your first year of Unix or never learn at all.

> "The `&` filtering has been saving me when browsing log files — much like an internal `grep` command." — Tim Chase

This is the insight that makes `less` more than a viewer: it's an interactive exploration tool. Pipe a 50MB logfile in, filter to only ERROR lines, then unfilter when you need context. The workflow is iterative in a way that `grep | less` isn't. (Caveat: the regex engine is unreasonably slow on large files, per HN's jemfinch — so pipe through grep first for big logs.)

> "Less provides an alternative of `<C-x>` to stop following, but that is intercepted by most shells." — CBLT (HN)

The dark pattern of terminal control: `<C-c>` stops following but also kills the source process in a pipeline (`kubectl logs | less +F`). `<C-x>` is the correct escape but most shells eat it. A design flaw fossilized by decades of backward compatibility. mananaysiempre adds the system-programming deep cut: terminal job control is kernel-level (`stty susp`), not shell-level — the shell just handles the aftermath of SIGTSTP via `waitpid()`.

> "Press `s` to save data from a pipe to a file rather than manually copy pasting." — btdmaster (HN)

The workflow upgrade you didn't know you needed: pipe a long-running process into `less`, inspect the output, and only save it to disk if it's useful. No `tee`, no pre-commitment to disk. osmsucks confirms this is their daily driver for uncertain output.

> "If you want security, unset LESSOPEN." — anthk (HN)

Six words that encode a security worldview. `LESSOPEN` is a preprocessor that runs arbitrary commands on your input before displaying it — a feature most people don't know exists and didn't ask for. The comment is a reminder that Unix tools accumulate features for decades, and the safe subset is almost always smaller than the default one. See also: `LESSSECURE=1`.

> "The `!` lets you invoke an external command. Also useful for privilege escalation — if a script running as root uses less, just do `!bash` and you have a root shell." — GuB-42 (HN)

Matter-of-fact about what is arguably the scariest single sentence in the thread. `!` is a feature, not a bug, but in privileged contexts it's a backdoor. `LESSSECURE=1` disables it, but as jmholla notes, most distros don't compile with that support.

---

## The Full Thread's Additional Gems

What the HN crowd added beyond the blog post, in rough order of obscurity:

- **`<C-x>` vs `<C-c>` for follow mode**: In pipelines, `<C-c>` kills the source. `<C-x>` is correct but most shells intercept it. Gnome Console works.
- **`s` to save pipe data**: No pre-commitment to disk. Inspect first, save if useful.
- **`-X` / `--no-init` / `--redraw-on-quit`**: Don't clear the screen on exit. ilyagr's `lesskey` trick binds `^q` to quit-without-clear and `q` to quit-and-clear, giving you both behaviors.
- **`-L` to skip preprocessing**: Rotated log files named `logfile.1`, `logfile.2` get mistaken for man page source on some distros. `-L` skips nroff.
- **`Ctrl-R` as first character of search**: Literal string search, not regex. No escaping metacharacters.
- **Mark + pipe region**: `ma` to mark, navigate, `|a` to pipe the region to an external command. obezyian uses this for interactive git-log — detect the commit at the top line, pipe to a script.
- **`lesskey` for custom bindings**: jez binds `s` to back-scroll (adjacent to `d`). macOS's default less doesn't support it — install via Homebrew.
- **lima/lesspipe**: Syntax highlighting and file rendering (PDF, markdown) inside less. Use with `-R`.
- **`lnav`**: A purpose-built log navigator that polls files, auto-scrolls, and highlights search matches in new data. Better than less for structured logs.
- **Slow regex**: jemfinch's main complaint — pipe through grep/ripgrep first for large files.
- **Alternative pagers**: `ov`, `moor`/`moar`, `most` — all mentioned as modern successors.

---

## Key Themes

### #tool-craft

Deep tool knowledge is a force multiplier. The gap between "I pipe to less" and "I navigate, filter, bookmark, and save from within less" is enormous but invisible — nobody sees you not switching windows, not re-running grep, not losing context. The HN thread is proof that even experienced engineers who use less daily still pick up new tricks from their peers. This is the argument for reading man pages cover-to-cover at least once.

### #unix-philosophy

Less is 40 years old and still surprises people. That's not a bug — it's the result of composition-friendly design. Every feature (marks, filtering, following) is orthogonal and recombinable. You can follow a file, hit a filter, set a mark, and save the region to disk, all without leaving the pager. That's not feature bloat; it's an interactive programming environment for text.

### #security

The thread surfaces an uncomfortable truth: `less` is more powerful than most people realize. It can run shell commands (`!`), execute preprocessors (`LESSOPEN`), and write files (`s`, `o`). When a pager inherits root privileges, it inherits root capabilities. The community's advice is clear: `LESSSECURE=1` in privileged contexts, unset `LESSOPEN` if you don't need it, and understand what your pager can actually do.

### #terminal-ux

The vim-to-less keybinding overlap is a happy accident of shared heritage. `/` for search, `n`/`N` for next/prev, `G` for end, `gg` for start — these work in both tools. But less has its own idioms too: `-` to toggle options, `&` to filter, `m`/`'` to bookmark. The wall between "editor" and "pager" is thinner than it looks, and the people who treat less as a read-only editor get more out of it.

---

## Critical Analysis

This is not an article about AI. It's an article about the opposite of AI: the slow accumulation of tacit knowledge about a tool that hasn't fundamentally changed in decades. And that's exactly why it's worth reading.

The most striking thing about the HN thread is the tone. Nobody is arguing. Nobody is dismissing less as obsolete. Instead, people are trading tips like they're swapping recipes — earnest, generous, mildly competitive. It's the best version of Hacker News: practitioners sharing craft knowledge without positioning or self-promotion.

The meta-lesson is about learning curves. Less is a tool you can learn in five minutes (`space` for next page, `q` to quit) and still be discovering features in after twenty years. That's a property of great tools, and it's worth noticing which modern tools have it and which don't. Most Electron apps don't. Most CLIs don't either — `less` is unusual.

The security dimension is under-discussed in the original post but surfaces well in the comments. The fact that `less` can execute arbitrary commands through `!`, `|`, `s`, `LESSOPEN`, and `LESSCLOSE` means it has a larger attack surface than most people assume. The community's response — `LESSSECURE`, unsetting `LESSOPEN` — is pragmatic rather than paranoid, but the fact that most distros don't ship with `LESSSECURE` support compiled in is a quiet indictment of how we handle tool security.

What's missing from both the post and the thread: nobody talks about how to discover these features independently. `man less` is 1,700 lines. `less --help` is terse. The discoverability problem is real — you either learn from a colleague or you don't learn at all. The blog post and thread together function as the missing discoverability layer, which is both wonderful (community fills the gap) and depressing (the gap shouldn't exist).

---

## Cross-References

The craft of deep tool knowledge connects to broader themes in this wiki:

- [[Software Engineering Craft]] — Fundamentals that don't change. Tool mastery is the original compound interest.
- [[Elements of Code]] — "Wrong in correctable ways." Less gives you the affordances to inspect, filter, and navigate without breaking things.
- [[Designing Agentic Loops]] — Simon Willison's argument that shell commands beat MCP for many tasks. Less is the ur-example of a shell tool that rewards depth.
- [[Before Reading Code]] — Five git commands to diagnose a codebase before reading a single line. Less is what you're reading them in.

---

*Sources: [[raw/assorted-less-tips]], [[raw/hn-assorted-less-tips]]*
*HN thread: 238 points, 55 comments*
*Last updated: 2026-05-15*
