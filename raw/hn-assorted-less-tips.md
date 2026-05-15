---
url: https://news.ycombinator.com/item?id=46464120
title: "Assorted less(1) tips — HN Discussion"
author: Hacker News community
date_fetched: 2026-05-15
date_published: 2025-11-11
score: 238
comments: 55
---

# Assorted less(1) tips — HN Discussion

Full HN thread discussing Tim Chase's blog post on `less(1)` tips.

## Selected Comments

**teeray**: `less +F` starts less following stdin or file. `<C-c>` breaks following; `F` resumes. Better than tail in many circumstances.

**CBLT**: When following a pipe (`kubectl logs | less +F`), `<C-c>` is sent to all processes in the pipeline, killing the source. Less provides `<C-x>` to stop following, but most shells intercept it.

**mananaysiempre**: The shell isn't active while less is running. Terminal job control (^Z → SIGTSTP) is a kernel function, not shell-level input processing. Configurable via `stty susp`/`stty intr`/`stty quit`.

**fuzztester**: Requested tutorial-style book recommendations on Unix system programming beyond man pages and Kerrisk's TLPI.

**joombaga**: With `tail` you can press enter to add visual separators between log groups. The only reason they still use tail over less for following.

**xg15**: "Follow" is just "keep polling after EOF" — no sophisticated file descriptor trickery. Software can switch modes on the fly.

**layer8**: Wanted a mode that follows new output while letting you navigate around, with an autoscroll toggle.

**gerdesj / tstack**: Recommended `lnav` — always polls files for new data, auto-scrolls when at end, sticks to position otherwise, highlights search matches in new data.

**JayGuerette**: `-X` or `--no-init` prevents clearing the screen. Useful for copy/paste from content to command line.

**Izkata**: Combines with `-E` (quit immediately if output smaller than terminal). Their go-to: `less -SEXIER`. Specifying E twice doesn't do anything except make it easier to remember.

**jlokier**: Recommends `-FX` over `-EX`. Both quit if output smaller than screen, but `-FX` doesn't quit if output larger and you jump to end. git uses `LESS=FRX less` by default.

**kccqzy**: Hates `-E` — quitting immediately breaks muscle memory. The q key becomes input to the shell prompt. Values consistency over saving a keystroke.

**etra0**: Uses `&` to filter (show matching) and `&!` to filter-out (hide matching). Both support regex. A bit slow sometimes but saves on removing noise from logfiles.

**inejge**: `-L` skips preprocessing — prevents rotated log files (logfile.1, logfile.2) from being treated as man page source and piped through nroff. Also: Ctrl-R as first character of a search string does literal search, not regex.

**ilyagr**: Shows how to bind `^q` to quit without clearing screen (like `less -X`), while `q` clears screen. Uses `~/.config/lesskey`. The option is `--redraw-on-quit`, which is slightly better than `-X` in every way on newer less versions.

**jez**: Binds `s` to back-scroll in `~/.lesskey` so `d` and `s` are adjacent for one-handed page up/down. Notes macOS's default less doesn't support lesskey — install via Homebrew.

**Redisclever**: Uses `ma` (mark), navigation, then `|a` to pipe region to external command (e.g., `cat >somefile`). Great for saving snippets. Also `-j` setting for search context positioning.

**obezyian**: Uses less piping for interactive git-log: detects when git-log is running via bash debug trap + terminal title + keyd, sets up shortcuts to pipe the top-line commit hash to scripts for git-show or fixup.

**alkh**: Recommends lesspipe for syntax highlighting and file rendering (pdf, markdown) in less. Typically disabled in pipes so scripts behave as intended.

**somat**: OpenBSD man provides tags to the pager (`:t test` in man ksh). Neat but never used — the simple uniform interface of `/` search wins over sophisticated navigation, similar to how man pages beat info pages.

**GuB-42**: The `!` command is a privilege escalation vector — if a script running as root uses less, `!bash` gives a root shell.

**jmholla**: Set `LESSSECURE=1` to disable external commands. Most distros don't ship with restricted less by default.

**btdmaster / osmsucks**: Press `s` to save data from a pipe to a file — no need to copy/paste or tee ahead of time.

**fragmede**: Mentions `most` as alternative pager and `glow` for rendering markdown in terminal.

**linhns**: Recommends `ov` as a modern alternative: https://github.com/noborus/ov

**kseistrup**: Recommends `moor` (née moar): https://github.com/walles/moor

**_delirium**: Jokes about the blog post's claim of "more less tips than the Bible's got Psalms" — there are at least 150 Psalms.

**jemfinch**: Biggest problem with less: the regex engine is unreasonably slow on large files. Frequently pipes through grep/ripgrep with large -A/-B buffers first.

**nvader**: Wanted an emacs-lite alternative to less. Self-answered: `emacs -nw -e "(view-mode)"`.

**jsrcout**: Used less + PCRE for detailed code analysis for years. Bookmarks were a key microtool.

**anthk**: "If you want security, unset LESSOPEN."
