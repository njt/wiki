---
url: https://blog.thechases.com/posts/assorted-less-tips/
title: "Assorted less(1) tips"
author: Tim Chase
date_fetched: 2026-05-15
date_published: 2025-11-11
hn_url: https://news.ycombinator.com/item?id=46464120
hn_score: 238
hn_comments: 55
topics:
  - software-engineering-craft
---

# Assorted less(1) tips

Blog post by Tim Chase (2025-11-11), aggregated from a Reddit discussion.

## Tips

1. **Opening multiple files:** `less README.txt file.c *.md`
2. **Adding files after launch:** `:e file.h`
3. **Navigating between files:** `:n` (next), `:p` (prev), `:x` (rewind to first)
4. **Removing files:** `:d` deletes current file from argument list
5. **Line-number navigation:** `countG` (e.g., `3141G` jumps to line 3141)
6. **Percentage navigation:** `count%` (e.g., `75%` goes to 3/4 through file)
7. **Search modifiers:** `!` (invert match), `*` (cross-file), `@` (from first file), `@*` (both)
8. **Filtering:** `&pattern` shows matching lines; `&!pattern` hides matching lines
9. **Bookmarks:** `m` + letter to set, `'` + letter to jump back. All 52 letters available. Applies across files.
10. **Bracket matching:** Type `(`, `[`, or `{` on first screen line to jump to matching close
11. **Custom match pairs:** `alt+ctrl+f` or `alt+ctrl+b` followed by a pair
12. **Toggling options:** `-` + option letter (e.g., `-S` for line wrap, `-N` for line numbers)
13. **External commands:** `!command` runs shell command
14. **Default options:** `LESS="-RNe"` in .bashrc
15. **Tags:** ctags support (author has never used it)
16. **Editing:** `v` opens file in $VISUAL
17. **Logging:** `o` or `O` writes stdin to file

## HN Discussion Highlights

- `less +F` for follow mode, Ctrl-C to break, F to resume (teeray)
- `-X` flag to prevent screen clearing (JayGuerette)
- `&` filtering for log debugging; regex supported but slow (etra0)
- `-L` to skip nroff preprocessing on rotated logs (inejge)
- `~/.lesskey` for custom keybindings; macOS lacks this, needs Homebrew less (jez)
- `^q` to quit without clearing screen, `q` clears it (ilyagr)
- Pipe command with marks for saving text selections to file (Rediscover)
- `!bash` from less as root = privilege escalation vector (GuB-42)
- OpenBSD's man provides tags to less for section jumping (somat)
- `-F` auto-quits if file fits on one screen; alternative pagers like `most` (fragmede)
- `s` saves piped data to file (btdmaster)
- lesspipe enables syntax highlighting and file rendering (alkh)
- Vim muscle memory transfers to less; keybindings feel intuitive (eulgro)
- PCRE support useful for code analysis; bookmarks as "microtools" (jsrcout)
- Regex engine unreasonably slow on large files (jemfinch)
- "If you want security, unset LESSOPEN" (anthk)
- `LESSSECURE=1` disables dangerous features like `!` (jmholla)
